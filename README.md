# perfd-opt

**A high-performance, native systemless optimization module for Snapdragon devices with Adreno GPUs**

## Overview

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect

While the project [WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further for devices with HMP. However, with perfd-opt we seek a different alternative to EAS, which involves `QTI Boost Framework`  and extends the ability of override custom parameters. When launching APPs or scrolling the screen, apply more aggressive parameters and run at a higher energy efficiency OPP under heavy load to improve response at an acceptable power penalty. When there is no interaction, use conservative parameters, use small cluster as much as possible, reduce the refresh rate to the minimum the SOC supports, and with that: we save as much energy as possible while the device is in standby, or even idle/suspended mode

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Main Features

- **Specific optimizations** - for Snapdragon SOCs that have EAS Scheduler and WALT Tracker
- **Automatic hardware detection** - Detects CPU architecture (4+4, 6+2, 4+3+1, 6+1+1), Type of EAS and WALT (Generic or Full), GPU type, and UFS availability
- **Implementation of `Rice-to-idle` strategy** - for better performance by finding the most efficient frequency to solve the task without demanding maximum from the SOC, and then: ramping down quickly without residual consumption
- **Power modes**:
  - **`powersave`**: Designed for basic tasks like messaging and calls
  - **`balance`**: Ideal for most users, with lower power consumption than the stock config
  - **`performance`**: It modifies the scheduler to be more performance-oriented, seeking total frame stability
  - **`fast`**: Providing stable performance capacity considering the TDP limitation of device chassis
- **Structural Tunings in the EAS Scheduler** - Optimize the EAS to improve decisions about the best core for each task, while reducing the need for unnecessary boosts of high-performance cores, without requiring conservative migration margins
- **Compatible with full and generic WALT** - For better tuning between different Snapdragon generations, allowing certain WALT parameters to be adapted according to the generation and needs of the SOC
- **Tuning the QTI Boost Framework** - For example: Improving the scheduler's response to various performance demands. However, this optimization is selective, meaning that SOCs with the "- Boosted" prefix will have this feature
  - **Improved Memory Management** - By tuning the Qualcomm framework, disabling unnecessary background services, and improving how cached processes are managed, the memory margin is slightly increased and the way LMK selects its kills is improved
- **Tune Camera Behavior** - By further tuning the Qualcomm framework, we implemented tunings in the powerhint.xml configuration file, which contains perflocks for the camera. This allowed us to improve the raw camera performance and the sustained performance of the device, reducing stutters that occur when using the camera. To find out if your SOC has this tuning, just check if it has the prefix "- CamBoost"
- **Configured to use both Schedtune and Uclamp** - To improve task placement and use of higher frequencies, such as in demanding games or tasks that require high CPU capacity
- **Improvements to the Display Refresh Rate** - Improve display behavior and refresh rates (90Hz+) to make the device smarter and more efficient in handling on-screen content
- **Miscellaneous Tunings** - Improvements such as disabling background services to free up CPU and other miscellaneous improvements based on the needs of each SOC

## Supported SOCs at the moment

```plain
sdm865
sdm855/sdm855+
sdm845
sdm765/sdm765g
sdm730/sdm730g/sdm732 - Boosted
sdm710/sdm712 - Boosted
sdm685 - Boosted
sdm680 - Boosted, CamBoost
sdm675 - Boosted
sdm662 - Boosted
sdm665 - Boosted
sdm660
sdm652
sdm636
```

## Requirements and Recommendations

- Have a device with a Snapdragon processor that includes Scheduler EAS and WALT Tracker
  - Not compatible with HMP or PELT; please check if your ROM/Kernel uses one of them to avoid any issues
- Android 8.0 or higher
- Stock ROM or a Custom ROM that uses CAF/CodeLinaro components
- Use a stock kernel or a custom kernel with CAF/CodeLinaro components
- Your ROM should have the QTI Boost Framework (optional, only if your SOC is listed as supporting such optimizations and you want them)
- Magisk or another root manager
  - KernelSU is having problems with the module, especially in kernels with poor implementation of it, be careful
- Have busybox installed (Optional)

---

> [!IMPORTANT]
> **KernelSU & APatch Users:**  
> Ensure your environment has a magic mount helper module installed (e.g., **Hybrid Mount** or **Mountify**) to guarantee proper module filesystem overlay support.

---

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash in Magisk or another root manager
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists

## Documentation and Reference Guide

### Sources

- Studies on how Google modifies and uses EAS, from basic to advanced
- Studies on how Qualcomm's WALT Tracker works (both the full and generic versions)
- Several Qualcomm vendors containing the hint opcodes and some explanations of how the QTI Boost Framework works
- My own studies on the EAS scheduler and methods for selecting the best CPU core
- Some studies involving Intel on how they manage their thermal TDP and how we can adapt that for Snapdragon SoCs
- ... And a few other things that I can't mention here, otherwise it would be a spoiler

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