# perfd-opt

**A high-performance, native systemless optimization module for Snapdragon devices with Adreno GPUs**

## Overview

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect

While the project [WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further for devices with HMP. However, with perfd-opt we seek a different alternative to EAS, which involves `QTI Boost Framework`  and extends the ability of override custom parameters. When launching APPs or scrolling the screen, apply more aggressive parameters and run at a higher energy efficiency OPP under heavy load to improve response at an acceptable power penalty. When there is no interaction, use conservative parameters, reduce the refresh rate to the minimum the SOC supports, and with that: we save as much energy as possible while the device is in standby, or even idle/suspended mode

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Main Features

- **Specific optimizations** - for Snapdragon SOCs that have EAS Scheduler and WALT Tracker
- **Automatic hardware detection** - Detects CPU architecture (4+4, 6+2, 4+3+1, 6+1+1, Big.LITTLE, DynamlQ), Governor used by System (Schedutil or Walt), EAS and WALT used (Generic or Full), GPU type, and UFS availability
- **Implementation of `Rice-to-idle` strategy** - for better performance by finding the most efficient frequency to solve the task without demanding maximum from the SOC, and then: ramping down quickly without residual consumption
- **Power modes**:
  - **`powersave`**: Designed for basic tasks like messaging and calls
  - **`balance`**: Ideal for most users, with lower power consumption than the stock config
  - **`performance`**: It modifies the scheduler to be more performance-oriented, seeking total frame stability
  - **`fast`**: Providing stable performance capacity considering the TDP limitation of device chassis
- **Structural Tunings in the EAS Scheduler** - Optimize how EAS selects the best cores, avoid over-boosting in the high-performance cluster, and allow EAS to efficiently package tasks onto efficiency cluster to save power whenever possible, making the EAS of generic devices as close as possible to that of well-tuned devices
  - **Improve the asymmetric behavior of the CPU Architecture** - Make the efficiency cluster support the high-performance cluster, instead of following a "sending everything to the high-performance cluster" behavior, make the high-performance and efficiency clusters work together, both respecting their limitations. This slightly increases the thermal headroom and allows the high-performance cluster to always be ready for UI tasks
  - **Improved single-thread and multi-thread performance** - By improving the scheduling method, we've made the scheduler's latency sensitivity better, allowing the CPU to deliver more stable and predictable work even at lower clock speeds by placing the task on the correct core at millisecond 0. Furthermore, we've improved cache locality, enhancing cache hits and reducing task ping-pong
- **Compatible with full and generic WALT** - For better tuning between different Snapdragon generations, allowing certain WALT parameters to be adapted according to the generation and needs of the SOC
- **Tuning the QTI Boost Framework** - For example, to improve the scheduler and governor's response to intermittent loads, allowing for boosted and bigger performance based on the needs and capabilities of each individual SoC
  - **Implementation of "Boost" Frequency** - Instead of simply reducing the maximum frequency for a more efficient OPP, allow the high-performance cluster to reach its maximum frequency when required, such as during app switching, opening, etc
  - **Improved Memory Management** - By tuning the Qualcomm framework, disabling unnecessary background services, and improving how cached processes are managed, the memory margin is slightly increased and the way LMK selects its kills is improved
- **Tune Camera Behavior** - By tuning the Qualcomm `powerhint.xml` , we can improve the camera's perflocks. This improves frame rate stability, efficiency, and raw camera performance with efficient scheduling and no overhead
- **Improvements to Thermal-Engine** - Tuning of the thermal engine to improve how the SoC manages thermal throttling, improving the headroom of these SoCs while reducing accumulated heat, which reduces power consumption under heavy load
- **Configured to use Schedtune or Uclamp** - To improve task placement and use of higher frequencies, such as in demanding games or tasks that require high CPU capacity
- **Improvements to the Display Refresh Rate** - Improvements to display behavior, including specific tuning of how the display manages refresh rates, making the decision to switch between low (60Hz) and higher (90Hz-144Hz) refresh rates more efficient and faster
- **Miscellaneous Tunings** - Improvements such as disabling background services to free up CPU and other miscellaneous improvements based on the needs of each SOC

## Supported SOCs at the moment

```plain
SDM730/SDM730G/SDM732
- powersave:    max 1.8+1.9g, boost 1.8+2.2g, min 0.3+0.6g
- balance:      max 1.8+1.9g, boost 1.8+2.2g, min 0.5+0.6g
- performance:  max 1.8+2.2g, min 0.5+0.6g
- fast:         max 1.8+2.2g, min 0.5+1.2g

SDM710/SDM712 
- powersave:    max 1.7+1.8g, boost 1.7+2.2g, min 0.3+0.6g
- balance:      max 1.7+2.0g, boost 1.7+2.2g, min 0.5+0.6g
- performance:  max 1.7+2.2g, min 0.5+0.6g
- fast:         max 1.7+2.2g, min 0.5+1.5g

SDM685
- powersave:    max 1.9+2.2g, boost 1.9+2.8g, min 0.3+1.0g
- balance:      max 1.9+2.4g, boost 1.9+2.8g, min 0.6+1.0g
- performance:  max 1.9+2.8g, min 0.6+1.0g
- fast:         max 1.9+2.8g, min 0.6+1.3g

SDM680
- powersave:    max 1.9+1.7g, boost 1.9+2.4g, min 0.3+1.0g
- balance:      max 1.9+2.2g, boost 1.9+2.4g, min 0.6+1.0g
- performance:  max 1.9+2.4g, min 0.6+1.0g
- fast:         max 1.9+2.4g, min 0.6+1.3g

SDM675
- powersave:    max 1.8+1.7g, boost 1.8+2.0g, min 0.3+0.3
- balance:      max 1.8+1.7g, boost 1.8+2.0g, min 0.5+0.6
- performance:  max 1.8+2.0g, min 0.5+0.6
- fast:         max 1.8+2.0g, min 0.5+1.2

SDM662
- No frequency change (if you want to help me by showing the frequencies of this SoC, please create an issue containing the name "SDM662 cpufreq")

SDM665
- No frequency change (if you want to help me by showing the frequencies of this SoC, please create an issue containing the name "SDM665 cpufreq")
```

## Requirements and Recommendations

- Have a device with a Snapdragon processor that includes Scheduler EAS and WALT Tracker
  - Not compatible with HMP or PELT; please check if your ROM/Kernel uses one of them to avoid any issues
- Use a thermal-engine from: qualcomm (MSM), the common thermal-engine, or mi_thermald; other thermals do not receive improvements
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

- No sources available other than citations

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