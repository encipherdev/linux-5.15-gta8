# Linux 5.15 GKI for Samsung Galaxy Tab A8 10.5 (SM-X200)

Linux 5.15.178 GKI bring-up tree for Samsung Galaxy Tab A8 10.5 (2021) with Unisoc Tiger T618 (`ums512_25c10` / `sharkl5pro`).

> **Status: Early Bring-Up / Not Booting (WIP)**
> - Currently in early testing phase; **does not boot to a usable system yet**.
> - Kernel reaches early Android init stage but fails during userspace handoff and display initialization.

## Work in Progress
- **Device Tree:** Initial `ums512-samsung-gta8.dts` with panel and power rail mappings.
- **Storage:** Unisoc SDHCI controller patches for `sprd,sdhc-r11p1` and clock fallbacks.
- **Root:** Experimental KernelSU-Next v3.4.2 hook dispatcher integration.
- **SMP:** Workaround for Cortex-A75 CPU7 stop IPI synchronization.
- **Subsystems:** Trusty TEE, SC2730 PMIC regulator, and persistent ramoops console logging.

## Build
```bash
export ARCH=arm64
export LLVM=1
make gta8_defconfig
make -j$(nproc) Image dtbs
```
