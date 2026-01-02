# perfd-opt

## Get the most out of Snapdragon — fast, efficient, and balanced

perfd-opt is a specialized performance module designed to maximize user experience and energy efficiency on Snapdragon devices with multi-cluster architectures. By focusing exclusively on SoCs with a cluster hierarchy (Big.LITTLE and DynamIQ), the module targets the sophisticated task-migration and scheduling logic found in mid-to-high-end hardware.

Unlike projects that rely solely on governor simulations, perfd-opt applies direct, architecture-aware adjustments to Qualcomm’s QTI Boost Framework and core sysfs parameters. It is engineered to distinguish between efficiency and performance clusters, ensuring that high-demand tasks are instantly routed to "Big" cores while background activity remains on "Little" cores to save power.

For Snapdragon models with inefficient power–frequency curves (common in certain 8xx series SoCs), the module also introduces optimized DVFS limits, refining minimum and maximum frequencies to reduce waste and improve efficiency by up to ~40% in specific workloads without sacrificing responsiveness.

The goal is simple: deliver the best balance between performance, responsiveness, and power consumption, with configurable profiles that adapt to everything from light usage to heavy workloads.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Profiles

- powersave: based on balance mode, but with more restrictions on the capabilities of the SOC, may lag in some scenarios where the load fluctuates dramatically
- balance: smoother than the stock config with lower power consumption and temperature
- performance: without imposing limits on the capabilities of the SOC
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
Terms Meaning:
**OctaCore & QuadCore**: These are older processors that don't have big or prime cores, only small ones. They have optimizations to improve fluidity and slightly extend battery life.
**big.LITTLE**: This means that the SOC has a big.LITTLE structure and, in turn, receives optimizations that fit this aspect. This improves cache locality and EAS/HMP decision-making regarding tasks, prioritizing maximizing performance per watt for each task individually.
**DynamlQ**: These are processors with a modern structure and good L3 cache, featuring optimizations that improve migration and the scheduler's ability to decide the best core for a specific task.
**EAS&Schedutil**: The processor is "modern" and traditionally uses EAS. Then it receives optimizations that balance performance and energy efficiency as much as possible. In addition to delivering on-time performance, it reduces hesitation and scheduling errors, allowing the EAS to extract the maximum possible potential from its energy model.
  - **EAS Alignment**: Implement energy efficiency optimizations such as big cluster clocks being between 600-800. These implementations are intended to make the EAS of the aforementioned SOCs closer to the EAS of recent SOCs designed for efficiency.
**HMP&Interactive**: The processor is "old" and traditionally uses HMP. There may be variations with EAS, but they will not count, as they are more specific to custom kernel/specific phones. They then receive optimizations that mitigate the shortcomings of HMP+Interactive in being inefficient at decision-making, offering longer battery life.
**Project WIPE**: Official support for Project Wipe has been implemented for the SOC if it is HMP+Interactive.
  - **Project Wipe: Days of Tomorrow**: An evolution of the traditional project wipe, focused on making the HMP scheduler and the interactive governor "energy conscious," this means that HMP and Interactive become closer to the EAS scheduler and Schedutil governor of modern devices. This allows HMP and Interactive to be usable today for both efficiency and performance.
**Optimized DVFS Curve**: Its minimum and maximum frequencies are optimized for better efficiency within its architecture. Designed to reduce energy consumption by up to 40% or more. Only implemented in SOCs with overheating problems or a poor efficiency curve.
**Optimized Boost Framework**: Optimizations and improvements have been implemented in the SOC's boost framework, specifically in its QTI Boost Framework. The optimizations are designed to improve the user experience without increasing energy consumption costs.
**Optimized Thermals**: It provides thermal optimizations for the SOC, where it is optimized so that the SOC has a thermal capacity that keeps it within 38-48 degrees, never exceeding 48 degrees where the user feels discomfort to the touch.
**Curved Input Boost**: Basically, the input boost doesn't change between profiles because it finds an "ideal" boost across all frequencies. This is only relevant for SoCs that require efficiency and where the voltage curve isn't too high.
  - **Input Boost Disabling**: If the SOC proves to be optimized enough for the user to trust their optimized EAS scheduler and qti boost, input boost is disabled. This allows schedutil to precisely define the necessary frequencies for each interaction and uses qti boost to boost during moments of hesitation. Each SOC with this optimization demonstrates that the user can trust their EAS scheduler and qti boost to handle all cases with maximum efficiency and without voltage spikes.
**Triple Buffer**: For SOCs with very weak GPUs, such as the sdm680, a triple buffer is added to trade a portion of RAM for better rendering speed.
**Frame Pacing**: A vendor extension used to improve stability and GPU usage in frame rendering by better synchronizing with screen timers.

Supported SoCs:
sdm865 (DynamlQ+EAS&Schedutil+Optimized DVFS curve+EAS Alignment+Frame Pacing)
sdm855/sdm855+ (DynamlQ+EAS&Schedutil+Optimized DVFS curve+EAS Alignment+Frame Pacing)
sdm845 (DynamlQ+EAS&Schedutil+Optimized DVFS curve+EAS Alignment+Frame Pacing)
sdm765/sdm765g (DynamlQ+EAS&Schedutil+EAS Alignment+Frame Pacing)
sdm730/sdm730g (DynamlQ+EAS&Schedutil+EAS Alignment+Frame Pacing)
sdm710/sdm712 (DynamlQ+EAS&Schedutil+EAS Alignment+Triple Buffer)
sdm680 (big.LTTLE+EAS&Schedutil+EAS Alignment+Optimized Boost Framework+Optimized Thermals+Curved Input Boost+Triple Buffer)
sdm675 (DynamlQ+EAS&Schedutil+EAS Alignment+Triple Buffer)
sdm835 (big.LTTLE+HMP&Interactive+Project Wipe: Days of Tomorrow+Curved Input Boost)
sdm820/821 (big.LTTLE+HMP&Interactive+Project Wipe: Days of Tomorrow+Optimized DVFS Curve+Curved Input Boost)
sdm810 (big.LTTLE+HMP&Interactive+Project Wipe: Days of Tomorrow+Optimized DVFS Curve+Curved Input Boost)
sdm808 (big.LTTLE+HMP&Interactive+Project Wipe: Days of Tomorrow+Optimized DVFS Curve+Curved Input Boost)
sdm660 (big.LTTLE+HMP&Interactive+Project Wipe: Days of Tomorrow+Curved Input Boost+Triple Buffer)
sdm652/sdm650 (big.LTTLE+HMP&Interactive+Project Wipe: Days of Tomorrow+Curved Input Boost+Triple Buffer)
sdm636 (big.LTTLE+HMP&Interactive+Project Wipe: Days of Tomorrow+Curved Input Boost+Triple Buffer)
```

## Requirements

1. Android 8 or higher
2. Magisk, KSU or Apatch, the most up-to-date version possible if you can
3. It is exclusive to Snapdragon processors! It will not be compatible with other SOCs (until further notice)
4. Currently, it is only compatible with EAS & HMP. Other custom schedulers like CASS, etc., may not be as compatible
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

### Recommendations for a better experience

- Use the highest maximum refresh rate of your device. Due to optimizations we implemented in the refresh rate driver and SurfaceFlinger, we have reduced the power penalties of high refresh rates by up to 7%. However, if you want even greater savings, we recommend using a lower "peak" refresh rate. A tip: if you don't like using the highest refresh rate on your screen (for example, you have 120Hz but want to use 90Hz), lower the refresh rate and install perfd-opt. You will receive the optimizations for that refresh rate, as long as you maintain it. This means that perfd-opt has not yet been optimized enough to adapt to changes in the maximum refresh rate after installing the module.

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
```
 