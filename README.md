# perfd-opt

## Get the most out of Snapdragon's specialties, be fast and, most importantly, efficient and conscious

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect. In addition, other parameters were added, such as schedtune, uclamp, and even some assistance. This number of parameters made everything more complex, but it opened up a lot of room for improvements in more specific areas of the UI or CPU/GPU. In return, the potential for error and reduced performance or even energy efficiency, creating a trade-off or scheduling deficiency. Furthermore, nowadays, most users don't look for optimization modules for a single purpose like playing games, etc., but rather optimizers that can be used for various purposes. Even users today would like to take better photos, although this is a minority. This means that finding a module with multiple performance modes and a focus on UX is proving difficult on the market, since creating a module that does this is challenging and complex, especially for beginner developers.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. Now, speaking about `Perfd-opt`, we will be modifying the Boost Framework on the user's device, which must be disabled before applying optimization, is able to dynamically override parameters based on perf hint. The project utilizes the `Boost Framework` found on the device and extends the ability of override custom parameters. Currently, perfd-opt is only compatible with QTI Boost Framework, but don't worry: it's checked before being applied. In other words, Boost Framework configurations only occur if the device is compatible; otherwise, it focuses on sysfs optimizations. With boost framework + sysfs optimizations (or just sysfs):When there is user interaction, such as opening an app or touching/scrolling the screen, use aggressive parameters with an acceptable energy cost and within the expectations of each power mode. When the interaction ends, return to idle as quickly as possible. Follow this strategy called "Rice-to-idle". But instead of focusing on being as quick as possible between finishing a task and entering idle mode, it focuses on using the most efficient OPP possible, striking an optimal balance between performance and energy consumption for the current demand, whether light, medium, or heavy, avoiding unnecessary resource waste. As a famous saying that the module follows: with more efficiency, less waste you will have for the same level of performance you traditionally have.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Profiles

- powersave: based on balance mode, but with more minimized energy consumption, users may report lower UX performance. It aims to increase the device's base SOT by 2.5 hours
- balance: smoother than the stock config with lower power consumption. Focuses on hitting the "sweet spot" between performance and power consumption. It aims to increase the device's base SOT by 1.5 hours
- performance: no frequency limiting and use of more aggressive boosts based on the frequency "sweetspot" of the most powerful clusters. It aims to deliver better performance with the device lasting AT MOST one hour less
- fast: providing stable performance capacity considering the TDP limitation of device chassis. Made purely for benchmarks or tasks that the user cannot perform without fully demanding the CPU or GPU. The battery may last longer or shorter, depending on the ambient temperature and the load exerted

```plain
sdm865 (EAS+Schedutil)
- powersave:    1.8+1.6+2.4g, min 0.3+0.7+1.1
- balance:      1.8+2.0+2.6g, min 0.7+0.7+1.1
- performance:  1.8+2.4+2.8g, min 0.7+0.7+1.1
- fast:         1.8+2.0+2.7g, min 0.7+1.2+1.2

sdm855/sdm855+ (EAS+Schedutil)
- powersave:    1.7+1.6+2.4g, min 0.3+0.7+0.8
- balance:      1.7+2.0+2.6g, min 0.5+0.7+0.8
- performance:  1.7+2.4+2.8g, min 0.5+0.7+0.8
- fast:         1.7+2.0+2.7g, min 0.5+1.2+1.2

sdm845 (EAS+Schedutil)
- powersave:    max 1.7+2.0g, min 0.5+0.8
- balance:      max 1.7+2.4g, min 0.5+0.8
- performance:  max 1.7+2.8g, min 0.5+0.8
- fast:         max 1.7+2.4g, min 0.5+1.6

sdm765/sdm765g (EAS+Schedutil)
- powersave:    1.8+1.7+2.0g, min 0.3+0.6+0.8
- balance:      1.8+2.0+2.2g, min 0.5+0.6+0.6
- performance:  1.8+2.2+2.3g, min 0.5+0.6+0.8
- fast:         1.8+2.0+2.2g, min 0.5+1.1+1.4

sdm730/sdm730g (EAS+Schedutil)
- powersave:    max 1.8+1.5g, min 0.5+0.6
- balance:      max 1.8+1.9g, min 0.5+0.6
- performance:  max 1.8+2.0g, min 0.5+0.6
- fast:         max 1.8+1.9g, min 0.5+1.2

sdm710/sdm712 (EAS+Schedutil)
- powersave:    max 1.7+1.8g, min 0.5+0.6
- balance:      max 1.7+2.0g, min 0.5+0.6
- performance:  max 1.7+2.2g  min 0.5+0.6
- fast:         max 1.7+2.0g, min 0.5+1.5

sdm680 (EAS+Schedutil+CAF CPU Boost Framework)
- powersave:    max 1.9+2.2g, min 0.6+0.8
- balance:      max 1.9+2.2g, min 0.6+0.8
- performance:  max 1.9+2.4g, min 0.6+0.8
- fast:         max 1.9+2.2g, min 0.6+1.3

sdm675 (EAS+Schedutil)
- powersave:    max 1.8+1.5g, min 0.5+0.6
- balance:      max 1.8+1.7g, min 0.5+0.6
- performance:  max 1.8+2.0g, min 0.5+0.6
- fast:         max 1.8+1.7g, min 0.5+1.2

sdm835 (HMP+Interactive+Project WIPE)

sdm660 (HMP+Interactive+Project WIPE)
sdm652 (HMP+Interactive+Project WIPE)
sdm650 (HMP+Interactive+Project WIPE)
sdm636 (HMP+Interactive+Project WIPE)

sdm450 (HMP+Interactive+Project WIPE)
sdm430 (HMP+Interactive+Project WIPE)
```

- **EAS+Schedutil**: The device traditionally has EAS, therefore its optimizations focus primarily on energy efficiency. Exclusive features of EAS+Schedutil:
  - Minimum and maximum frequency control according to power modes. Specifically controls the frequency of the big/prime cores, leaving the LITTLE cores untouched.
  - Use the big/prime cluster only for sustained and/or heavy tasks. Avoid using the big/prime cluster for demands below that level; use the LITTLE cluster to handle those tasks efficiently.
  - Reduce erroneous decision-making, make the EAS more energy-conscious and faster at making decisions, allowing for even greater energy consumption reduction without impacting the user experience.
- **HMP+Interactive+Project WIPE**: The device has HMP, therefore its optimizations focus on aligning the HMP's behavior with the EAS. Allowing the HMP to be "energy conscious" similar to the EAS. Exclusive features of HMP+Interactive+ProjectWipe:
  - Less energy waste at intermediate frequencies, delivering the necessary power for each specific task.
  - Avoid using frequencies higher than necessary; be mindful and allow each demand to be met according to its need and "hunger."
- **CAF CPU Boost Framework**: The module also uses Qualcomm's boost framework for each compatible SOC IF I have the time and patience. The optimizations and improvements resulting from optimizing the boost framework are: reduction of micro-jitter and latency variation in "common" UX demands, such as scrolling for example. And to improve the ability of the LITTLE Cluster to handle demands efficiently and within the demand limit they can handle, reduce the energy consumption of the big/prime cluster, allowing the LITTLE cluster to handle light to medium demands as much as possible, where the Big cluster should only be used for heavy loads and to receive tasks that the LITTLE cluster cannot handle.
  - Modify the hints to something that the EAS understands better. Provide only the MINIMUM so that the EAS can decide to work with maximum precision, reducing deficiencies and improving CPU, I/O, and GPU scheduling in various areas of the system, allowing the QTI Boost Framework to follow the EAS ideal of "costing the least possible energy for a fluid UX".
  - Let the LITTLE cluster handle all light and medium demands, within their expected consumption. Do this as efficiently as possible and without negatively impacting the UX, allowing the LITTLE cluster to be useful in these situations and not disrupt the overall system schedule. This allows the EAS to become more aware.

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
```
