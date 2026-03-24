# perfd-opt

## Get the most out of Snapdragon — fast, efficient, and balanced

Perfd-opt is a specialized performance module designed to maximize user experience and energy efficiency on Snapdragon devices with multi-cluster architectures. By focusing exclusively on SoCs with a cluster hierarchy (Big.LITTLE and DynamIQ), the module targets the sophisticated task-migration and scheduling logic found in mid-to-high-end hardware. With the introduction of Energy Aware Scheduling (EAS) in devices released from 2018 onwards, we focused our implementation on the native integration offered by this generation of processors. 

This approach allows Perfd-opt to optimize task scheduling and placement based on energy efficiency. By improving the cost-benefit model of each SoC's EAS, Perfd-opt reduces hesitation and inaccuracies in decision-making. The result is faster task allocation, minimizing cache locality loss (L1/L2) and ensuring extremely efficient process migrations

Unlike projects that rely solely on governor simulations, perfd-opt applies direct, architecture-aware adjustments to Qualcomm’s QTI Boost Framework and core sysfs parameters. When launching APPs or scrolling the screen, applying more aggressive parameters to improve response at an acceptable power penalty. When there is no interaction, use conservative parameters, use small core clusters as much as possible, and run at a higher energy efficiency OPP under heavy load

The goal is simple: deliver the best balance between performance, responsiveness, and power consumption, with configurable profiles that adapt to everything from light usage to heavy workloads

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features
- **Specific optimizations** for Snapdragon SOC and the EAS environment. However, instead of following an "all-for-one" methodology, perfd-opt has a compatibility list where each listed processor is optimized individually, attempting to extract the "maximum" balance between performance and energy efficiency as it can receive optimizations
- **Automatic Hardware Detection**: Detects the CPU architecture (4+4, 6+2, 4x3x1 & 6+1+1), whether it has full EAS or generic EAS, the GPU model, whether it is capable of receiving secondary optimizations (such as improvements in the SOC boost framework and/or frame pacing), and the availability of UFS storage. Even without including I/O and memory optimizations, it still shows a complete overview of the user's hardware, allowing them to understand their device and know which optimizations are applied to it
- **Predefined Power Profiles**:
  - **powersave**: Maximum energy efficiency, designed for basic use such as using messaging apps, etc
  - **balance**: Balance between performance and efficiency, ideal for daily use
  - **performance**: Full performance without restrictions, aims to make EAS core selection more "performance-oriented"
  - **fast**: Stable performance and throughout depending on the device's TDP and chassis
- **Compatibility between full Snapdragon WALT (which comes with a directory) and generic WALT (which comes without a directory)**: We have also implemented optimizations that extract the maximum from these two Qualcomm device scenarios, allowing each device to achieve its efficiency and performance according to the availability of parameters to optimize, enabling all SOCs to be adapted to ALMOST ANY SCENARIO
- **Compatible with both Uclamp and Schedtune**: Optimizations to the boost mechanism are applicable to both, allowing users of newer and older kernels to benefit from task boosting with their respective mechanisms
- **Maintains the use of Governor Walt if it exists**: If the Snapdragon comes with Governor Walt, it is maintained to have a governor capable of meeting the needs of user UX fluidity, where it comes with optimizations that filter unnecessary peaks, reducing the energy consumption of this somewhat "power-hungry" governor in a slight way
- **Tuning the QTI Boost Framework (When Possible)**: Adjust the perfboost if the SOC is compatible. All SOCs marked as "boosted" mean they will come with a tuned QTI Boost Framework, which will provide several improvements in scheduling predictability and performance consistency without consuming as much extra energy
- **Recommended to use the device's maximum screen refresh rate**: The way the device switches between refresh rates has been optimized, allowing you to take advantage of the device's maximum refresh rate without incurring as many energy efficiency penalties, since the idle transition is much more efficient and faster
- **Miscellaneous and Secondary Optimizations**: Such as improvements to the SOC's thermal behavior, to maintain the SOC temperature between 38-48 degrees even in hot places, and other optimizations such as the addition of frame pacing, Triple Buffer for weak Adreno GPUs, and other optimizations that help make the SOC more predictable and efficient

## Profiles

- **powersave**: ideal for basic use such as using messaging apps
- **balance**: smoother than the stock config with lower power consumption and temperature
- **performance**: without imposing limits on the capabilities of the SOC
- **fast**: providing stable performance capacity considering the TDP limitation of device chassis

##### Supported SOCs at the moment
```plain
sdm865
sdm855/sdm855+
sdm845
sdm765/sdm765g
sdm730/sdm730g - sm6150
sdm710/sdm712
sdm685
sdm680 (Boosted)
sdm675 - sm6150
sdm662
sdm665
```

## Requirements

1. Android 8 or higher
2. Magisk, KSU or Apatch, the most up-to-date version possible if you can
3. It is exclusive to Snapdragon processors! It will not be compatible with other SOCs (until further notice)
4. Currently, it is only compatible with EAS. Other custom schedulers like CASS or old HMP, etc., may not be as compatible
5. We prefer the QTI Boost Framework as the primary framework we will optimize; therefore, there may not be optimizations for other Boost frameworks, such as libperfmgr and others
6. Prefers stock kernels; extremely modified kernels that even modify the frequency table may not work very well with the module
7. It might not work as efficiently if your kernel uses PELT Tracker instead of WALT. Many consider WALT a "villain," even though it can be drastically optimized for efficiency; you just need to be smart

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
@屁屁痒
provide /vendor/etc & sched tunables on Snapdragon 845

@林北蓋唱秋
provide /vendor/etc on Snapdragon 675

@酪安小煸
provide /vendor/etc on Snapdragon 710

@沉迷学习日渐膨胀的小学僧
help testing on Snapdragon 855

@NeonXeon
provide information about dynamic stune

@rfigo
provide information about dynamic stune

@Matt Yang
I am the original creator of the "perfd-opt" module; I am simply forking it

@JUANIMAN
Because it gave me inspiration to improve the perfd-opt README, basically our modules are "opposites" of each other
```
