# perfd-opt

## Get the most out of Snapdragon's specialties, be fast and, most importantly, efficient and conscious

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler, furthermore, some SoCs have an inefficient frequency curve, as demonstrated by some in the 800 series, which are reported to overheat even during simple tasks. This shows that some Qualcomm SoCs are extremely inefficient, even with capabilities far superior to processors from even current generations. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect. In addition, more parameters are added in each generation, which causes everything more complex, but it opened up a lot of room for improvements in more specific areas. In return, the potential for error and reduced performance or even energy efficiency, creating a trade-off or scheduling deficiency. Furthermore, nowadays, most users don't look for optimization modules for a single purpose like playing games, etc., but rather optimizers that can be used for various purposes. Even users today would like to take better photos, although this is a minority. This means that finding a module with multiple performance modes and a focus on UX is proving difficult on the market, since creating a module that does this is challenging and complex, especially for beginner developers.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. Now, speaking about `perfd-opt`, he aims to achieve a fair **balance** between **power consumption** and **performance** in both modern (EAS) and older (HMP) generation Snapdragon devices. One of the optimization methods is through modifying the Boost Framework on the user's device, which must be disabled before applying optimization, is able to dynamically override parameters based on perf hint. The project utilizes the `Boost Framework` found on the device and extends the ability of override custom parameters. Currently, perfd-opt is only compatible with QTI Boost Framework, but don't worry: it's checked before being applied. In other words, Boost Framework configurations only occur if the device is compatible; otherwise, it focuses on sysfs optimizations. With boost framework + sysfs optimizations (or just sysfs): When there is user interaction, such as opening an app or touching/scrolling the screen, use aggressive parameters with an acceptable energy cost and within the expectations of each power mode. When the interaction ends, return to idle as quickly as possible. Follow this strategy called "Rice-to-idle". But instead of focusing on being as quick as possible between finishing a task and entering idle mode, it focuses on using the most efficient OPP possible, striking an optimal balance between performance and energy consumption for the current demand, whether light, medium, or heavy, avoiding unnecessary resource waste: For SOCs with inefficient frequency curves, micro-optimization is applied: reduction in maximum and minimum frequencies according to the selected profile, allowing a reduction in the inefficiency of these SOCs by selecting a more efficient minimum and maximum frequency ceiling for them, enabling an increase of up to +40% in energy efficiency. As a famous saying that the module follows: with more efficiency, less waste you will have for the same level of performance you traditionally have.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Profiles

- powersave: based on balance mode, but with more minimized energy consumption, users may report lower UX performance. The proposal aims to reduce energy consumption by half
- balance: smoother than the stock config with lower power consumption. Focuses on hitting the "sweet spot" between performance and power consumption. The proposal aims to reduce stock energy consumption by up to 35%
- performance: no frequency limiting and use of more aggressive boosts based on the frequency "sweetspot" of the most powerful clusters. It aims to deliver better performance with the device lasting AT MOST one hour less
- fast: providing stable performance capacity considering the TDP limitation of device chassis. Made purely for benchmarks or tasks that the user cannot perform without fully demanding the CPU or GPU. The battery may last longer or shorter, depending on the ambient temperature and the load exerted

```plain

Typos Meanings:
**EAS+Schedutil**: The processor is "modern" and traditionally uses EAS.
**HMP+Interactive**: The processor is "old" and traditionally uses HMP. There may be variations with EAS, but they will not count, as they are more specific to custom kernel/specific phones.
**Project WIPE**: Official support for Project Wipe has been implemented for the SOC if it is HMP+Interactive.
**Optimized DVFS Curve**: Its minimum and maximum frequencies are optimized for better efficiency within its architecture. Designed to reduce energy consumption by up to 40% or more. Only implemented in SOCs with overheating problems or a poor efficiency curve.
**Optimized Boost Framework**: Optimizations and improvements have been implemented in the SOC's boost framework, specifically in its QTI Boost Framework. The optimizations are designed to improve the user experience without increasing energy consumption costs.

List of compatible SOCs
sdm865 (EAS+Schedutil+Optimized DVFS curve)
sdm855/sdm855+ (EAS+Schedutil+Optimized DVFS curve)
sdm845 (EAS+Schedutil+Optimized DVFS curve)
sdm765/sdm765g (EAS+Schedutil)
sdm730/sdm730g (EAS+Schedutil)
sdm710/sdm712 (EAS+Schedutil)
sdm680 (EAS+Schedutil+Optimized Boost Framework)
sdm675 (EAS+Schedutil)
sdm835 (HMP+Interactive+Project WIPE)
sdm660 (HMP+Interactive+Project WIPE)
sdm652 (HMP+Interactive+Project WIPE)
sdm650 (HMP+Interactive+Project WIPE)
sdm636 (HMP+Interactive+Project WIPE)
sdm450 (HMP+Interactive+Project WIPE)
sdm430 (HMP+Interactive+Project WIPE)
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
2. Flash in your actual Root Manager
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists

## FAQ

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
```
