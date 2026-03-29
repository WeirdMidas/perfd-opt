# perfd-opt

**A Magisk module for Snapdragon devices that use EAS.**

## Overview

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect.  

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further for devices with HMP. However, with perfd-opt, we are looking for a different approach, which mainly involves: When launching APPs or scrolling the screen, applying more aggressive parameters to improve response at an acceptable power penalty. When there is no interaction, use conservative parameters, use small core clusters as much as possible, and run at a higher energy efficiency OPP under heavy load.  

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features
- **Specific optimizations**: for Snapdragon SOCs and those using the EAS Scheduler
- **Automatic Hardware Detection**: Detects CPU architecture (4+4, 4+3+1, 6+2, 6+1+1), GPU type, and UFS availability
- **Power modes:**
  - **`powersave`**: Designed for basic use such as using WhatsApp or messaging apps
  - **`balance`**: Ideal for everyday use, offering a near-perfect balance between performance and efficiency
  - **`performance`**: Maximum performance without restrictions, aims to make EAS core selection more "performance-oriented"
  - **`fast`**: Stable performance and throughout depending on the device's TDP and chassis
- **Compatibility between full Snapdragon WALT (which comes with a directory) and generic WALT (which comes without a directory)**: for internal improvements to the WALT tracker and EAS scheduler
- **Compatible with both Uclamp and Schedtune**: for different forms of boosting and task placement
- **Maintains the use of Governor Walt if it exists**: to maintain responsiveness and efficiency on demand
- **Recommended to use the device's maximum screen refresh rate**: due to internal optimizations in the way apps automatically change the refresh rate to save energy
- **Miscellaneous and Secondary Optimizations**: such as disabling PerfLock, enabling triple buffering for weak GPUs, and other additional improvements

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
```

## Requirements

- Devices with Snapdragon processors that use EAS
- Android 8.0 or higher
- Magisk, MAYBE other Root Managers are compatible, but it's not guaranteed

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash it using your root solution (Magisk, KSU, or Apatch)
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists

## References

### Sources

- Studies on how Google modifies and uses EAS, from basic to advanced.
- Matt Yang's tests on the cost-benefit of migration. Using the binder as a test (seen in Uperf's old project before V3).
- Explanations from engineers about certain aspects of EAS, mainly its ideology and the ways it decides to be more energy-conscious.

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
I am the original creator of the "perfd-opt" module; I am simply forking it

@JUANIMAN
Because it gave me inspiration to improve the perfd-opt README, basically our modules are "opposites" of each other
```
