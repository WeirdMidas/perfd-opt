# perfd-opt

## Get the most out of Snapdragon — fast, efficient, and balanced

perfd-opt is a module designed to improve the user experience and energy efficiency of Snapdragon devices, both on modern SoCs using EAS + schedutil and older ones using HMP + interactive.

Unlike projects such as WIPE, which rely on simulation of the interactive governor, perfd-opt applies direct adjustments to Qualcomm’s QTI Boost Framework (when available) and complementary sysfs optimizations. This enables more aggressive behavior during user interactions (touch, scrolling, app launches) and a fast return to efficient OPPs (Operating Performance Points) once activity ends—an efficient race-to-idle strategy that prioritizes the most economical operating point for the current workload.

For Snapdragon models with inefficient power–frequency curves (common in certain 8xx series SoCs), the module also introduces optimized DVFS limits, refining minimum and maximum frequencies to reduce waste and improve efficiency by up to ~40% in specific workloads without sacrificing responsiveness.

The goal is simple: deliver the best balance between performance, responsiveness, and power consumption, with configurable profiles that adapt to everything from light usage to heavy workloads.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Profiles

- powersave: based on balance mode, but with more restrictions on the capabilities of the SOC
- balance: smoother than the stock config with lower power consumption and temperature
- performance: without imposing limits on the capabilities of the SOC
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
Terms Meaning:
**OctaCore & QuadCore**: These are older processors that don't have large cores, only small ones. They have optimizations to improve fluidity and slightly extend battery life.
**big.LITTLE**: This means that the SOC has a big.LITTLE structure and, in turn, receives optimizations that fit this aspect. This improves cache locality and EAS/HMP decision-making regarding tasks, prioritizing maximizing performance per watt for each task individually.
**DynamlQ**: These are processors with a modern structure and good L3 cache, featuring optimizations that improve migration and the scheduler's ability to decide the best core for a specific task.
**EAS+Schedutil**: The processor is "modern" and traditionally uses EAS. Then it receives optimizations that balance performance and energy efficiency as much as possible.
**HMP+Interactive**: The processor is "old" and traditionally uses HMP. There may be variations with EAS, but they will not count, as they are more specific to custom kernel/specific phones. They then receive optimizations that mitigate the shortcomings of HMP+Interactive in being inefficient at decision-making, offering longer battery life.
**Old Project WIPE**: Official support for Project Wipe has been implemented for the SOC if it is HMP+Interactive.
**Modern Project WIPE**: For HMP SOCs that could be modernized to current standards, an evolution of the older Project WIPE which was more focused on its time period.
**Optimized DVFS Curve**: Its minimum and maximum frequencies are optimized for better efficiency within its architecture. Designed to reduce energy consumption by up to 40% or more. Only implemented in SOCs with overheating problems or a poor efficiency curve.
**Optimized Boost Framework**: Optimizations and improvements have been implemented in the SOC's boost framework, specifically in its QTI Boost Framework. The optimizations are designed to improve the user experience without increasing energy consumption costs.
**Optimized Thermals**: It provides thermal optimizations for the SOC, where it is optimized so that the SOC has a thermal capacity that keeps it within 38-48 degrees, never exceeding 48 degrees where the user feels discomfort to the touch.
**Curved Input Boost**: Basically, the input boost doesn't change between profiles because it finds an "ideal" boost across all frequencies. This is only relevant for SoCs that require efficiency and where the voltage curve isn't too high.

Supported SoCs:
sdm865 (DynamlQ+EAS+Schedutil+Optimized DVFS curve)
sdm855/sdm855+ (DynamlQ+EAS+Schedutil+Optimized DVFS curve)
sdm845 (big.LTTLE+EAS+Schedutil+Optimized DVFS curve)
sdm765/sdm765g (DynamlQ+EAS+Schedutil)
sdm730/sdm730g (big.LTTLE+EAS+Schedutil)
sdm710/sdm712 (big.LTTLE+EAS+Schedutil)
sdm680 (big.LTTLE+EAS+Schedutil+Optimized Boost Framework+Optimized Thermals+Curved Input Boost)
sdm675 (big.LTTLE+EAS+Schedutil)
sdm835 (big.LTTLE+HMP+Interactive+Modern Project WIPE)
sdm660 (big.LTTLE+HMP+Interactive+Modern Project WIPE+Curved Input Boost)
sdm652/sdm650 (big.LTTLE+HMP+Interactive+Old Project WIPE)
sdm636 (big.LTTLE+HMP+Interactive+Old Project WIPE)
sdm450 (OctaCore+HMP+Interactive+Old Project WIPE)
sdm430 (QuadCore+HMP+Interactive+Old Project WIPE)
```

## Requirements

1. Android 8 or higher
2. Magisk, KSU or Apatch, the most up-to-date version possible if you can
3. It is exclusive to Snapdragon processors! It will not be compatible with other SOCs (until further notice)
4. Currently, it is only compatible with EAS & HMP. Other custom schedulers like CASS, etc., may not be as compatible
5. We prefer the QTI Boost Framework as the primary framework we will optimize; therefore, there may not be optimizations for other Boost frameworks, such as libperfmgr and others
6. Prefers stock kernels; extremely modified kernels that even modify the frequency table may not work very well with the module

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
 