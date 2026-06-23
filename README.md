# perfd-opt

**A Magisk module for Snapdragon devices that use EAS.**

## Overview

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect

While the project [WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further for devices with HMP. However, with perfd-opt, we are looking for a different approach, which mainly involves: When launching APPs or scrolling the screen, applying more aggressive parameters and run at a higher energy efficiency OPP under heavy load to improve response at an acceptable power penalty. So when there's no interaction: use conservative parameters, if tasks still end up on the big/prime cores, disable all hysteresis to immediately return the task to the LITTLE cores when it finishes, reduce the refresh rate to the minimum the SOC supports, and with that: we save as much energy as possible while the device is in standby, or even idle/suspended mode

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- **Specific optimizations** - for Snapdragon SOCs that have EAS Scheduler and WALT Tracker
- **Automatic hardware detection** - Detects CPU architecture (4+4, 6+2, 4+3+1, 6+1+1), GPU type, and UFS availability
- **Implementation of `Rice-to-idle`** - for better performance by finding the most efficient frequency to solve the task without demanding maximum from the SOC, and then: ramping down quickly without residual consumption
- **Customizable profile configurations** - Edit profile settings via easy-to-understand `.txt` files
- **Persistent configuration storage**:
  - Profile configs: `/sdcard/Android/panel_powercfg.txt`
- **Power modes**:
  - **`powersave`**: Designed for basic use, such as using WhatsApp, etc
  - **`balance`**: Smoother than the stock config with lower power consumption
  - **`performance`**: It modifies the scheduler to be more performance-oriented, seeking total frame stability
  - **`fast`**: Providing stable performance capacity considering the TDP limitation of device chassis
- **Structural Improvements to the EAS Scheduler** - Optimizing core selection based on hot cache, keeping the load on smaller cores and reducing unnecessary boosting of high-performance cores without requiring extremely conservative migration margins
- **Compatible with full and generic WALT** - For better tuning between different Snapdragon generations
- **Tuning the SOC QTI Framework** - Such as: Optimizations in the SOC Boost Framework for scheduler fluidity and predictability, additions to `display_boot` in order to reduce the deficiencies of each SOC individually, and other improvements in other areas. To find out if your SOC has this feature, check if the word "- Boosted" is in the list of compatible SOCs
  - **Improve Adreno GPU behavior by mimicking certain gimmicks** - Through the QTI Boost Framework, we make the SOC capable of simulating Adreno input boost, Adreno Idler, Adreno Boost, and other mechanisms that efficiently improve the Adreno GPU without needing these kernel mechanisms, allowing stock ROMs/kernels to benefit from such mechanisms in user space
- **Configured to use both Schedtune and Uclamp** - To improve placement and boosting of tasks, such as gaming in performance profiles
- **Improvements to the Display Refresh Rate** - To reduce power consumption and improve SOC fluidity, allowing the user to utilize the SOC's higher refresh rate
- **Miscellaneous Tunings** - Such as disabling Perflock for SOCs that have the Schedtune or Uclamp camera-daemon and other optimizations that reduce SOC deficiencies

## Supported SOCs at the moment

```plain
sdm865
sdm855/sdm855+
sdm845
sdm765/sdm765g
sdm730/sdm730g - Boosted
sdm710/sdm712
sdm685
sdm680 - Boosted
sdm675 - Boosted
sdm662 - Boosted
sdm665
sdm660
sdm652
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
- My own studies on the EAS scheduler and methods for selecting the best CPU core

### Suggestions for Complementary Modules

- [AsoulOpt](https://github.com/nakixii/Magisk_AsoulOpt@) — A module that improves EAS decision-making regarding AAA game threads or games listed in the repository, allowing EAS to prioritize game threads on large cores, thus improving predictability and stability. It includes up to three migration modes that the user can choose, enabling EAS to better determine the performance and placement of game threads.

### Usage suggestions for users to get the most out of perfd ​​opt

1. Use the highest refresh rate your display has to allow for maximum fluidity that perfd ​​opt's rice-to-idle can deliver.
2. Switch to the performance profile only if you want a little more FPS in games; the balanced profile already handles most games nowadays that don't demand much from the CPU. Switch to the fast profile ONLY in extreme/demanding tasks, benchmarks, or games that your CPU can't handle running at 20 FPS; outside of these situations, using the fast profile is not recommended.

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

### The reason for the project's delayed release

- I'm having a lot of problems in my life, like breaking up with my boyfriend, starting to suffer from motivation issues, etc. Besides that, I'm absolutely obsessed with making this project as good as possible for everyone to use. Also, I'm trying to have at least 7 SOCs with the optimized Boost Framework, because I planned to add more SOCs (at least three) with the optimized Boost Framework as I updated the module, and finding the Boost Framework for certain SOCs is incredibly difficult. So, that's the reason for the delay in the release, I hope you understand.

## Credit

```plain
@Matt Yang
He is the real creator of the "perfd-opt" module; I am simply forking it

@JUANIMAN
Because it gave me inspiration to improve the perfd-opt README, basically our modules are "opposites" of each other
```
