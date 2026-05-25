# perfd-opt

**A Magisk module for Snapdragon devices that use EAS.**

## Overview

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect

While the project [WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further for devices with HMP. However, with perfd-opt, we are looking for a different approach, which mainly involves: When launching APPs or scrolling the screen, applying more aggressive parameters and run at a higher energy efficiency OPP under heavy load to improve response at an acceptable power penalty. So when there's no interaction: use conservative parameters, use the LITTLE cores as much as possible, reduce the refresh rate to the minimum the SOC supports, and with that: we save as much energy as possible while the device is in standby, or even idle/suspended mode

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- **Specific optimizations** for Snapdragon SOCs that have EAS Scheduler
- **Automatic hardware detection** Detects CPU architecture (4+4, 6+2, 4+3+1, 6+1+1), GPU type, and UFS availability
- **Implementation of "Rice-to-idle"** for better performance by finding the most efficient frequency to solve the task without demanding maximum from the SOC, and then: ramping down quickly without residual consumption
- **Customizable profile configurations**: Edit profile settings via easy-to-understand `.txt` files
- **Persistent configuration storage**:
  - Profile configs: `/sdcard/Android/panel_powercfg.txt`
- **Power modes**:
  - **`powersave`**: Designed for basic use, such as using WhatsApp, etc
  - **`balance`**: Ideal for general use, balancing performance with efficiency
  - **`performance`**: It modifies the scheduler to be more performance-oriented, seeking total frame stability
  - **`fast`**: Stable performance and throughout depending on the device's TDP and chassis
- **Compatible with full and generic WALT** for better tuning between different Snapdragon generations
- **Configured to use both Schedtune and Uclamp** for better task placement and boosting
- **Improvements to the Display Refresh Rate** such as reduced power consumption and improved fluidity
- **Miscellaneous Tunings** such as disabling Perflock for SOCs that have the Schedtune or Uclamp camera-daemon, in addition to other optimizations

## Supported SOCs at the moment

```plain
sdm865
sdm855/sdm855+
sdm845
sdm765/sdm765g
sdm730/sdm730g 
sdm710/sdm712
sdm685
sdm680
sdm675 
sdm662
sdm665
sdm660
sdm652
```

## Requirements

- Devices with Snapdragon processors that use EAS
- Android 8.0 or higher
- Magisk, MAYBE other Root Managers are compatible, but it's not guaranteed

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash in Magisk manager
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists

## Documentation and Reference Guide

### Sources

- Studies on how Google modifies and uses EAS, from basic to advanced
- My own studies on the EAS scheduler and methods for selecting the best CPU core

### Suggestions for Complementary Modules

- [AsoulOpt](https://github.com/nakixii/Magisk_AsoulOpt@) — A module that improves EAS decision-making regarding AAA game threads or games listed in the repository, allowing EAS to prioritize game threads on large cores, thus improving predictability and stability. It includes up to three migration modes that the user can choose, enabling EAS to better determine the performance and placement of game threads.

### How to suggest SOCs to add to the compatibility list

Submit an issue with these questions answered:

1. Name of your SOC (e.g., sdm680, sdm730)

2. Name of your SOC platform (can be viewed via: `getprop ro.board.platform`)

3. Does your SOC use Schedtune, Uclamp, or both?

4. List the available frequencies of your SOC, for all its clusters if possible.

5. Does your SOC use EAS? (If the answer is not "yes", your issue will be automatically ignored)

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
