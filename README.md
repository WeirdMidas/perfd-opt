# perfd-opt

## Get the most out of Snapdragon — fast, efficient, and balanced

Perfd-opt is a specialized performance module designed to maximize user experience and energy efficiency on Snapdragon devices with multi-cluster architectures. By focusing exclusively on SoCs with a cluster hierarchy (Big.LITTLE and DynamIQ), the module targets the sophisticated task-migration and scheduling logic found in mid-to-high-end hardware. With the introduction of Energy Aware Scheduling (EAS) in devices released from 2018 onwards, we focused our implementation on the native integration offered by this generation of processors. This approach allows Perfd-opt to optimize task scheduling and placement based on energy efficiency. By improving the cost-benefit model of each SoC's EAS, Perfd-opt reduces hesitation and inaccuracies in decision-making. The result is faster task allocation, minimizing cache locality loss (L1/L2) and ensuring extremely efficient process migrations.

Unlike projects that rely solely on governor simulations, perfd-opt applies direct, architecture-aware adjustments to Qualcomm’s QTI Boost Framework and core sysfs parameters. It is engineered to distinguish between efficiency and performance clusters, ensuring that high-demand tasks are instantly routed to "Big" cores while background activity remains on "Little" cores to save power.

For Snapdragon models with inefficient power–frequency curves (common in certain 8xx series SoCs), the module also introduces optimized DVFS limits, refining minimum and maximum frequencies to reduce waste and improve efficiency by up to ~40% in specific workloads without sacrificing responsiveness.

The goal is simple: deliver the best balance between performance, responsiveness, and power consumption, with configurable profiles that adapt to everything from light usage to heavy workloads.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features
- **Specific optimizations** for Snapdragon SOC and the EAS environment. However, instead of following an "all-for-one" methodology, perfd-opt has a compatibility list where each listed processor is optimized individually, attempting to extract the "maximum" balance between performance and energy efficiency as it can receive optimizations.
- **Automatic Hardware Detection**: Detects the CPU architecture (4+4, 6+2, 4x3x1 & 6+1+1), whether it has full EAS or generic EAS, the GPU model, whether it is capable of receiving secondary optimizations (such as improvements in the SOC boost framework and/or frame pacing), and the availability of UFS storage. Even without including I/O and memory optimizations, it still shows a complete overview of the user's hardware, allowing them to understand their device and know which optimizations are applied to it.
- **Predefined Power Profiles**:
  - **powersave**: Maximum battery power, with penalties to UX performance, possibly depending on whether the device can handle it.
  - **balance**: Balance between performance and efficiency, ideal for daily use.
  - **performance**: Full performance without restrictions, aims to make EAS core selection more "performance-oriented".
  - **fast**: Stable performance and throughout depending on the device's TDP and chassis.
- **Miscellaneous and Secondary Optimizations**: Such as improvements to the SOC's thermal behavior, to maintain the SOC temperature between 38-48 degrees even in hot places, and other optimizations such as the addition of frame pacing, Triple Buffer for weak Adreno GPUs, and other optimizations that help make the SOC more predictable and efficient.
- **Compatibility between full Snapdragon WALT (which comes with a directory) and generic WALT (which comes without a directory)**: We have also implemented optimizations that extract the maximum from these two Qualcomm device scenarios, allowing each device to achieve its efficiency and performance according to the availability of parameters to optimize, enabling all SOCs to be adapted to ALMOST ANY SCENARIO.
- **Maintains the use of Governor Walt if it exists**: If the Snapdragon comes with Governor Walt, it is maintained to have a governor capable of meeting the needs of user UX fluidity, where it comes with optimizations that filter unnecessary peaks, reducing the energy consumption of this somewhat "power-hungry" governor in a slight way.

## Profiles

- **powersave**: based on balance mode, but the maximum capacity is capped
- **balance**: smoother than the stock config with lower power consumption and temperature
- **performance**: without imposing limits on the capabilities of the SOC
- **fast**: providing stable performance capacity considering the TDP limitation of device chassis

```plain
Terms Meaning:

Architectures and Topologies:
**big.LITTLE**: This means that the SOC has a big.LITTLE structure and, in turn, receives optimizations that fit this aspect. This improves cache locality and EAS decision-making regarding tasks, prioritizing maximizing performance per watt for each task individually.
  - Best Core by Profile: In efficiency profiles, the core that saves the most energy is preferred over the core that delivers the most performance. In performance profiles, the core that delivers the most performance is preferred over the core that saves the most energy.
**DynamlQ**: These are processors with a modern structure and good L3 cache, featuring optimizations that improve migration and the scheduler's ability to decide the best core for a specific task.
  - Best overall core: In DynamiQ SOCs, the preference for the best, most energy-efficient core is global, favoring SOT time compared to the overall performance of these devices.
  
Schedulers and Templates:
**EAS&Schedutil**: The processor is "modern" and traditionally uses EAS. Then it receives optimizations that balance performance and energy efficiency as much as possible. In addition to delivering on-time performance, it reduces hesitation and scheduling errors, allowing the EAS to extract the maximum possible potential from its energy model.
  - **Alignment with Modern Standards**: Implement energy efficiency optimizations such as big cluster min clocks being between 600-800. These implementations are intended to make the EAS of the aforementioned SOCs closer to the EAS of recent SOCs designed for efficiency.
  - **Input Boost Disabling**: If the SOC proves to be optimized enough for the user to trust their optimized EAS scheduler and qti boost, input boost is disabled. This allows schedutil to precisely define the necessary frequencies for each interaction and uses qti boost to boost during moments of hesitation. Each SOC with this optimization demonstrates that the user can trust their EAS scheduler and qti boost to handle all cases with maximum efficiency and without voltage spikes.
  - **Aligned Migration Thresholds**: For SOCs that have only two clusters and need to make ideal decisions about what to keep on the little cores and send to the big cores, filtering tasks that can be resolved on the small cores while maintaining margin for processing peaks, avoiding underutilizing the little cores and making them spend more energy when the big cores could resolve them more efficiently and at an average frequency.
  - **No migration cost**: If the SOC has an efficient architecture, we can afford to have no migration cost and allow the EAS to make decisions about which tasks to assign to the cores based solely on energy costs.
  
Miscellaneous and Secondary Optimizations:
**Optimized DVFS Curve**: Its minimum and maximum frequencies are optimized for better efficiency within its architecture. Designed to reduce energy consumption by up to 40% or more. Only implemented in SOCs with overheating problems or a poor efficiency curve.
**Optimized Boost Framework**: Optimizations and improvements have been implemented in the SOC's boost framework, specifically in its QTI Boost Framework. The optimizations are designed to improve the user experience without increasing energy consumption costs.
**Optimized Thermals**: It provides thermal optimizations for the SOC, where it is optimized so that the SOC has a thermal capacity that keeps it within 38-48 degrees, never exceeding 48 degrees where the user feels discomfort to the touch.
**Triple Buffer**: For SOCs with very weak GPUs, such as the sdm680, a triple buffer is added to trade a portion of RAM for better rendering speed.
**Frame Pacing**: A vendor extension used to improve stability and GPU usage in frame rendering by better synchronizing with screen timers.

Supported SoCs:
sdm865 (DynamlQ+EAS&Schedutil+Alignment with Modern Standards+No migration cost+Optimized DVFS curve)
sdm855/sdm855+ (DynamlQ+EAS&Schedutil+Alignment with Modern Standards+No migration cost+Optimized DVFS curve)
sdm845 (DynamlQ+EAS&Schedutil+Alignment with Modern Standards+Aligned Migration Thresholds+Optimized DVFS curve)
sdm765/sdm765g (DynamlQ+EAS&Schedutil+Alignment with Modern Standards+No migration cost+Optimized DVFS curve)
sdm730/sdm730g (DynamlQ+EAS&Schedutil+Alignment with Modern Standards+Aligned Migration Thresholds)
sdm710/sdm712 (DynamlQ+EAS&Schedutil+Alignment with Modern Standards+Aligned Migration Thresholds)
sm6225 (sdm662/sdm680) (big.LTTLE+EAS&Schedutil+Aligned Migration Thresholds)
- We do not support sdm685 because we could not find its SOCID; we only have the SOCIDs for sdm662 and sdm680
sdm675 (DynamlQ+EAS&Schedutil+Alignment with Modern Standards+Aligned Migration Thresholds)
sdm665 (big.LTTLE+EAS&Schedutil+Aligned Migration Thresholds)
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
Exec `sh /data/powercfg.sh balance`, where `balance` is the mode you want to switch.  

Option 2:  
Install [vtools](https://www.coolapk.com/apk/com.omarea.vtools) and bind APPs to power mode.  

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
Based on the original module and also ideas from Uperf

@ROM Devs
Thanks to certain props that improved the efficiency and performance of the UI

@Pedrozzz (King Tweaks)
For some tweaks found on King Tweaks that depend on the manufacturer, I will give credit for them

@JUANIMAN
Because it gave me inspiration to improve the perfd-opt README, basically our modules are "opposites" of each other
```
