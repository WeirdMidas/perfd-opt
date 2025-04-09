# Perfd opt (Fork)

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect.  

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. `QTI Boost Framework`, which must be disabled before applying optimization, is able to dynamically override parameters based on perf hint. This project utilizes `QTI Boost Framework` and extends the ability of override custom parameters. When launching APPs or scrolling the screen, applying more aggressive parameters to improve response at an acceptable power penalty. When there is no interaction, use conservative parameters, use small core clusters as much as possible, and run at a higher energy efficiency OPP under heavy load.  

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## CPU Scaling Mode Support

1. FIXED
- Processors with *fixed* maximum and minimum frequencies, with no possibility of fine-tuning by profile.
- Technical limitations: no `schedtune`, `uclamp`, or full EAS.
- Automatic boosts are usually controlled by firmware (e.g. RPMh).
- Example: Snapdragon 680, Snapdragon 665, some older MediaTek.

2. PER-PROFILE
- Processors with technical support for per-profile adjustments: foreground, background, games, etc.
- Available features: `schedtune.boost`, `uclamp_min/max`, `task_groups`, full EAS.
- Allows granular control of performance vs battery.
- Example: Snapdragon 720G, 765G, 8xx, some recent Exynos.

Note: Devices classified as FIXED can still see significant gains from tweaking corectl, task priorities, and governors, even if they do not have fully controllable perf boost.

### Profiles & Supported Devices

- powersave: based on balance mode, but with lower max frequency
- balance: smoother than the stock config with lower power consumption
- performance: dynamic stune boost = 30 with no frequency limitation
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
sdm865
- powersave:    1.8+1.6+2.4g, boost 1.8+2.0+2.6g, min 0.3+0.7+1.1
- balance:      1.8+2.0+2.6g, boost 1.8+2.4+2.7g, min 0.7+0.7+1.1
- performance:  1.8+2.4+2.8g, boost 1.8+2.4+2.8g, min 0.7+0.7+1.1
- fast:         1.8+2.0+2.7g, boost 1.8+2.4+2.8g, min 0.7+1.2+1.2

sdm855/sdm855+
- powersave:    1.7+1.6+2.4g, boost 1.7+2.0+2.6g, min 0.3+0.7+0.8
- balance:      1.7+2.0+2.6g, boost 1.7+2.4+2.7g, min 0.5+0.7+0.8
- performance:  1.7+2.4+2.8g, boost 1.7+2.4+2.8/2.9g, min 0.5+0.7+0.8
- fast:         1.7+2.0+2.7g, boost 1.7+2.4+2.8/2.9g, min 0.5+1.2+1.2

sdm845
- powersave:    1.7+2.0g, boost 1.7+2.4g, min 0.3+0.3
- balance:      1.7+2.4g, boost 1.7+2.7g, min 0.5+0.8
- performance:  1.7+2.8g, boost 1.7+2.8g, min 0.5+0.8
- fast:         1.7+2.4g, boost 1.7+2.8g, min 0.5+1.6

sdm765/sdm765g
- powersave:    1.8+1.7+2.0g, boost 1.8+2.0+2.2g, min 0.3+0.6+0.8
- balance:      1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.6
- performance:  1.8+2.2+2.3g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.8
- fast:         1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+1.1+1.4

sdm730/sdm730g
- powersave:    1.7+1.5g, boost 1.7+1.9g, min 0.3+0.3
- balance:      1.7+1.9g, boost 1.7+2.1g, min 0.5+0.6
- performance:  1.8+2.2g, boost 1.8+2.2g, min 0.5+0.6
- fast:         1.8+1.9g, boost 1.8+2.2g, min 0.5+1.2

sdm680
- Sorry, this processor is in the "fixed" category.

sdm675
- powersave:    1.7+1.5g, boost 1.7+1.7g, min 0.3+0.3
- balance:      1.7+1.7g, boost 1.7+1.9g, min 0.5+0.6
- performance:  1.8+2.0g, boost 1.8+2.0g, min 0.5+0.6
- fast:         1.8+1.7g, boost 1.8+2.0g, min 0.5+1.2

sdm710/sdm712
- powersave:    1.7+1.8g, boost 1.7+2.0g, min 0.3+0.3
- balance:      1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+0.6
- performance:  1.7+2.2g, boost 1.7+2.2/2.3g, min 0.5+0.6
- fast:         1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+1.5
```

## Requirements

1. Android 11-14
2. Rooted
3. Magisk or KSU

## Installation

1. Download zip in Release Page
2. Flash in Magisk manager
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists
5. At every boot, some SOCs need to wait 15-30 minutes to stabilize completely (As is the case with my sdm680).

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
Matt Yang — For the Original Module and my inspiration for create this.
Tytydraco - Another inspiration for my attempt to create the old atlas.
yinwanxi - libcgroup and also for his discoveries, and also for the inspiration to create OPP PER-PROFILE as an addition.
NotZeeta - Brazilians like him and me never give up, and he is one of the good ones, too bad he abandoned YAKT, I hope it wasn't because of a headache.
```
