# Linux 5.15 GKI for Samsung Galaxy Tab A8 10.5 (SM-X200)

Linux 5.15.178 GKI bring-up tree for Samsung Galaxy Tab A8 10.5 (2021) with Unisoc Tiger T618 (`ums512_25c10` / `sharkl5pro`).

## Features & Fixes
- **Device Tree:** Custom `ums512-samsung-gta8.dts` with native panel mappings and power rails.
- **Storage:** Unisoc SDHCI controller patches supporting `sprd,sdhc-r11p1` and clock fallbacks.
- **Root:** Integrated KernelSU-Next v3.4.2 hook dispatcher.
- **SMP:** Cortex-A75 CPU7 stop IPI workaround for stable multi-core execution.
- **Subsystems:** Trusty TEE, SC2730 PMIC regulator, and persistent ramoops console logging.

## Build
```bash
export ARCH=arm64
export LLVM=1
make gta8_defconfig
make -j$(nproc) Image dtbs
```
