# Linux 5.15 GKI for Samsung Galaxy Tab A8 10.5 (SM-X200)

Linux 5.15.178 GKI bring-up tree for Samsung Galaxy Tab A8 10.5 (2021) with Unisoc Tiger T618 (`ums512_25c10` / `sharkl5pro`).

> **Status: Early Bring-Up / Not Booting to Userspace Yet (WIP)**
> The kernel initializes core subsystems and executes `/init`, but halts during userspace handoff.

---

## 🐛 Documented Bugs & Issues to Fix

### 1. Cortex-A75 CPU7 / Stop Machine Soft Lockup
- **Symptom:** At late boot (~50s), watchdog reports soft lockup across all online cores:
  `watchdog: BUG: soft lockup - CPU#X stuck for 48s! [migration/X] <- multi_cpu_stop <- stop_machine_cpuslocked`.
- **Root Cause:** When Android init or CPU hotplug invokes `stop_machine_cpuslocked` (or CPU takedown), all online CPUs wait in `multi_cpu_stop` for an atomic acknowledgement (`thread_ack`). CPU 7 (big core) does not process the cross-core stop IPI, causing CPUs 0–6 to spin indefinitely.
- **Current Workaround:** Disabled CPU7 from `cpu_possible_mask` in `arch/arm64/kernel/smp.c` (`set_cpu_possible(7, false)`). Needs proper PSCI / GICv3 interrupt affinity investigation.

### 2. Display Freeze / SimpleFB & Panel Handoff
- **Symptom:** Screen stays frozen on the Samsung bootloader splash logo; does not transition to Android boot animation.
- **Root Cause:** The bootloader leaves the framebuffer at physical address `0x9e000000` (1200x1920, 32-bit). `simplefb` and the Unisoc DPU/DSI DRM driver (`sprd_disp`) have not completed panel power-up and timing handoff for the Novatek `lcd_nt36523b_qc_inx_mipi_hdp` panel.
- **Next Step:** Port panel power sequencing and DSI initialization sequences from Samsung 4.14 kernel DTS.

### 3. Userspace Init Crash & Reboot to Download Mode (Odin)
- **Symptom:** Android init runs for ~36 seconds, then executes `reboot: Restarting system with command 'shell'`, which triggers Samsung Download (Odin) Mode.
- **Root Cause:** 
  1. GSI HALs fail to communicate with vendor services (Trusty TEE / `skeymaster4dev` reports `error: -49` due to missing `tsupplicant`).
  2. Successive boot failures trigger Samsung's hardware watchdog and write `DLOW` (PARAM_BOOT_DOWNLOAD_FAIL) to the `param` partition (offset `0x900000`).

### 4. Boot Image DTB Format Requirements
- **Symptom:** Immediate drop to Odin mode with black screen upon flashing kernel.
- **Root Cause:** Unisoc LK bootloader requires the DTB section in `boot.img` (header v2) to be an Android DT Table (`0xd7b7ab1e` magic). A raw FDT blob (`0xd00dfeef`) triggers an assertion in LK (`fdt_chosen_bootargs_append` / overflow).
- **Solution:** Repack using `magiskboot repack` to preserve header checksums, offsets, and the DT table structure.

### 5. eMMC SDHCI Compatibility
- **Status:** Patched in `drivers/mmc/host/sdhci-sprd.c` to match DT compatible `sprd,sdhc-r11p1`, add clock name fallbacks (`sdio_clk`), and make `state_uhs` pinctrl non-fatal. Needs validation under sustained I/O.

---

## 🛠️ Build
```bash
export ARCH=arm64
export LLVM=1
make gta8_defconfig
make -j$(nproc) Image dtbs
```
