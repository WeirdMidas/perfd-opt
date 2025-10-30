# perfd-opt

## Get the most out of Snapdragon's specialties, be fast and, most importantly, efficient

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect. In addition, other parameters were added, such as schedtune, uclamp, and even some assistance. This number of parameters made everything more complex, but it opened up a lot of room for improvements in more specific areas of the UI or CPU/GPU. In return, the potential for error and reduced performance or even energy efficiency, creating a trade-off or scheduling deficiency.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. Now, speaking about `Perfd-opt`, we will be modifying the Boost Framework on the user's device, which must be disabled before applying optimization, is able to dynamically override parameters based on perf hint. The project utilizes the `Boost Framework` found on the device and extends the ability of override custom parameters. When there is user interaction, such as opening an app or touching/scrolling the screen, use aggressive parameters with an acceptable energy cost and within the expectations of each power mode. When the interaction ends, return to idle as quickly as possible. Follow this strategy called "Rice-to-idle". Also, include preliminary optimizations such as: use bw_hwmon for LLC and DDR, and unlock the maximum available GPU and Devfreq frequency, where the GPU's power will be limited according to the power mode that selects a maximum power level that is ideal for the specific mode, Improvements to the WALT tracker for the purpose of improving EAS scheduling, allowing SOCs that have the full WALT solution to greatly benefit from this, with improvements in latency, sustainability and energy efficiency, Using the boost mechanisms of the HMP and EAS schedulers respectively, depending on the selected profile, a greater or lesser boost is imposed on the tasks the user is currently interacting with. This also involves the GPU, which scales aggressively according to the selected power mode, potentially allowing marginal gains of +fps in games as well as better stability in high-performance profiles, automatic selection of the best TCP algorithm based on compatibility. However, even with this set of adjustments, we will try run at a higher energy efficiency OPP under heavy load.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Profiles

- powersave: based on balance mode, but with more minimized energy consumption, users may report lower UX performance. It aims to increase the device's base SOT by 2.5 hours
- balance: smoother than the stock config with lower power consumption. Focuses on hitting the "sweet spot" between performance and power consumption. It aims to increase the device's base SOT by 1.5 hours
- performance: no frequency limiting and use of more aggressive boosts based on the frequency "sweetspot" of the most powerful clusters. It aims to deliver better performance with the device lasting AT MOST one hour less
- fast: providing stable performance capacity considering the TDP limitation of device chassis. Made purely for benchmarks or tasks that the user cannot perform without fully demanding the CPU or GPU. The battery may last longer or shorter, depending on the ambient temperature and the load exerted

```plain
sdm865
sdm855/sdm855+
sdm845
sdm835

sdm765/sdm765g
sdm730/sdm730g
sdm710/sdm712

sdm680
sdm675
sdm660
sdm652
sdm650
sdm636

sdm450
sdm430
```

## Requirements

1. Android 8+
2. Magisk, KSU or Apatch, the most up-to-date version possible if you can
3. It is exclusive to Snapdragon processors! It will not be compatible with other SOCs (until further notice)

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
