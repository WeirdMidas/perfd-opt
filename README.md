# perfd-opt

**A Magisk module for Snapdragon devices that use EAS.**

## Overview

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect

While the project [WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further for devices with HMP. However, with perfd-opt we seek a different alternative to EAS, which involves: When launching APPs or scrolling the screen, apply more aggressive parameters and run at a higher energy efficiency OPP under heavy load to improve response at an acceptable power penalty. So when there's no interaction: use conservative parameters, disable all hysteresis that keeps the load on the high-performance cores and quickly return the load to the LITTLE cores, reduce the refresh rate to the minimum the SOC supports, and with that: we save as much energy as possible while the device is in standby, or even idle/suspended mode

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- **Specific optimizations** - for Snapdragon SOCs that have EAS Scheduler and WALT Tracker
- **Automatic hardware detection** - Detects CPU architecture (4+4, 6+2, 4+3+1, 6+1+1), GPU type, and UFS availability
- **Implementation of `Rice-to-idle` strategy** - for better performance by finding the most efficient frequency to solve the task without demanding maximum from the SOC, and then: ramping down quickly without residual consumption
- **Customizable profile configurations** - Edit profile settings via easy-to-understand `.txt` files
- **Persistent configuration storage**:
  - Profile configs: `/sdcard/Android/panel_powercfg.txt`
- **Power modes**:
  - **`powersave`**: Designed for basic tasks like messaging and calls
  - **`balance`**: Ideal for most workloads, with lower power consumption than the stock config
  - **`performance`**: It modifies the scheduler to be more performance-oriented, seeking total frame stability
  - **`fast`**: Providing stable performance capacity considering the TDP limitation of device chassis
- **Structural Tunings in the EAS Scheduler** - Optimize the EAS to improve decisions about the best core for each task, while reducing the need for unnecessary boosts of high-performance cores, without requiring conservative migration margins
- **Compatible with full and generic WALT** - For better tuning between different Snapdragon generations, allowing certain WALT parameters to be adapted according to the generation and needs of the SOC
- **Tuning the QTI Boost Framework** - For example: Improving the scheduler's response to various performance demands. However, this optimization is selective, meaning that SOCs with the "-Boosted" prefix will have this feature
- **Configured to use both Schedtune and Uclamp** - To improve task placement and use of higher frequencies, such as in demanding games or tasks that require high CPU capacity
- **Improvements to the Display Refresh Rate** - Improve display behavior and refresh rates (90Hz+) to make the device smarter and more efficient in handling on-screen content
- **Miscellaneous Tunings** - For example: disabling camera perflock for SOCs that have the Uclamp or Schedtune camera-daemon directory, allowing the EAS + WALT Tracker to efficiently manage the camera's processing needs

## Supported SOCs at the moment

```plain
sdm865
sdm855/sdm855+
sdm845
sdm765/sdm765g
sdm730/sdm730g - Boosted
sdm710/sdm712 - Boosted
sdm685 - Boosted
sdm680 - Boosted
sdm675 - Boosted
sdm662 - Boosted
sdm665 - Boosted
sdm660
sdm652
sdm636
```

## Requirements

- Devices with Snapdragon SOC that have the EAS Scheduler
- Snapdragon device that is using WALT as a tracker instead of PELT
- Android 8.0 or higher
- A non-unstable ROM or custom kernel
- Magisk, MAYBE other Root Managers are compatible, but it's not guaranteed
- Have busybox installed (Optional)

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash in Magisk manager
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists

## Documentation and Reference Guide

### Sources

- Studies on how Google modifies and uses EAS, from basic to advanced
- Several Qualcomm vendors contain the hint opcodes and some explanations of how the QTI Boost Framework works
- My own studies on the EAS scheduler and methods for selecting the best CPU core

### Suggestions for Complementary Modules

- [AsoulOpt](https://github.com/nakixii/Magisk_AsoulOpt@) — A module that improves EAS decision-making regarding AAA game threads or games listed in the repository, allowing EAS to prioritize game threads on large cores, thus improving predictability and stability. It includes up to three migration modes that the user can choose, enabling EAS to better determine the performance and placement of game threads.

## Switch modes

### Switching on boot

1. Open `/sdcard/Android/panel_powercfg.txt`
2. Edit line `default_mode=balance`, where `balance` is the default mode applied at boot
3. Reboot

### Switching after boot

Option 1:  
Exec `sh /data/powercfg.sh balance`, where `balance` is the mode you want to switch

Option 2:  
Install [vtools](https://www.coolapk.com/apk/com.omarea.vtools) and bind APPs to power mode

## Credit

```plain
@Matt Yang
He is the real creator of the "perfd-opt" module; I am simply forking it

@JUANIMAN
Because it gave me inspiration to improve the perfd-opt README, basically our modules are "opposites" of each other
```
