# XTU Add-on
![1000003154](https://github.com/user-attachments/assets/a344c48b-1f93-4760-a958-263a3e71f7f0)

"Each new generation of computers demoralizes the previous ones and their creators."
## Modern and efficient for the end user experience

Older projects like `Uperf`, `Project Wipe`, and Matt Yang's `Perfd Opt` were very good for their time. `Uperf` in particular demonstrated a high degree of evolvability. However, after Matt Yang's disappearance and the disappearance of the `Uperf` code, everything became even more obscure, making `Uperf` a solution that some considered "abandoned." Depending on the device using `Uperf`, it can perform incredibly well or incredibly poorly. And because support has been abandoned, it's difficult to find solutions on the market today that could replace `Uperf's` genius. Even more so, optimizations have become more complicated now, due to the integration of the EAS scheduler, and then trackers like WALT/PELT, which were subsequently integrated years later. This complicated the optimization field and made everything even more complex!

Based on this, the XTU Add-on is a `scheduler`, `CPU`, and `GPU` optimization tool that aims to achieve two things: unify the scheduler and tracker so they work together, and aims to achieve a fair **balance** between **power consumption** and **performance**. However, rather than adjusting purely based on intuition and guesswork, it's done through knowledge of how the basic EAS scheduler (found on poorly optimized devices), moderately optimized, and highly optimized devices works. This approach allows us to take into account the performance and efficiency needs of each SOC, allowing us to simulate EAS behavior on each SOC, allowing us to improve each one's behavior while respecting its immediate performance needs. For devices with HMP, optimizations are based on Project WIPE, which simulates EAS behavior by taking into account power consumption and rendering performance needs to avoid frame drops. It integrates strategies such as "`rice-to-idle`," optimal use of `small cores`, and "`guidecap`" to allow the scheduler to better guide tasks, as well as additions such as `GameSpace` `Performance Mode`, `Thermal Mode`, and "`Battery Saver Mode`" and other functions In addition to the classic power profiles, it also adds optimizations that depend on various subsystems, such as refresh rate and others. This allows the device to extract its maximum potential, regardless of the situation, meaning the module will always strive to run on the most efficient OPP possible.

Remember: The focus is to run the task with the lowest OPP and as quickly as possible.

Details see [the lead project](https://github.com/WeirdMidas/XTUAddon/commits/master/) & [XTU Add-on commits](https://github.com/WeirdMidas/XTUAddon/commits/master/)    

## Features
- Includes specific improvements and optimizations for the device's scheduling, CPU, and GPU. Respecting the capabilities, limitations, and specialties of each SOC and its architecture, this in turn allows for the best possible performance from the SOC itself, both in terms of energy efficiency and raw performance.
- Introduce the "Rice-to-idle" strategy, a form of tracker optimization that allows the CPU to immediately jump to a frequency that finishes the task quickly and then rest immediately, saving as much power as possible while using a linear frequency curve.
- Use an efficient form of hotplug, based on the performance needs of the clusters, allow the cores to turn off and on in an efficient curve, allowing the CPU to save maximum energy in this aspect.
- Understand Android's dynamic workload and pin certain tasks that small cores can handle easily. Such as playing videos/media like YouTube, in addition to the SensorService. Allow essentially lightweight tasks, which small cores are designed to handle. However, as the SoC's power is modest, don't pin too many tasks, allowing the Scheduler to have more freedom in this regard.
- Use additional settings that allow the device to use modern power management technologies, buffers, etc. Allowing the CPU to work and with lower costs per watt.
- SELinux can still be enabled

## Compatibility

- powersave: based on balance mode, but with lower max frequency. Ideal for saving as much energy as possible with low impact on fluidity
- balance: smoother than the stock config with lower power consumption. There is no maximum frequency limitation, allowing the device to scale freely between performance demands
- performance: based on balance, but with added boosts and more aggressiveness in rice to allow better performance per watt (cost~pwr)
- fast: purely maximum performance, ideal for benchmarks and other extremely demanding tasks. This is achieved by maximizing performance based on the device's base TDP

```plain
New Generation
sdm865
sdm855/sdm855+
sdm845
sdm765/sdm765g
sdm730/sdm730g
sdm710/sdm712
sdm680 (+ Includes Boost Framework tweaks)
sdm675

Old Generation
sdm835
sdm660
sdm636
Kirin 658 (Huawei)
Kirin 659 (Huawei)
Kirin 970 (Huawei)
```

## Requirements

1. Android 8-15
2. Magisk, KSU or Apatch
3. It may not be compatible with heavily modified/optimized kernels and ROMs. Please be aware of this
4. Busybox is required to ensure that some optimizations occur more deeply
 
## Installation

1. Download zip in [Release Page](https://github.com/WeirdMidas/SkyPERFAddon/releases)
2. Flash in your root manager
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists

## FAQ

### Sources

- Loosely based on the King Tweaks and LKT idea, but now more specialized and less error-prone. Because I have a lot of knowledge about EAS and trackers in general due to many projects that I abandoned and did due to limitations.
- Knowledge of the EAS scheduler and PELT/WALT trackers that I acquired through months of optimization with the old perfd ​​opt and Uperf projects. These were never officially released due to limitations in both and lack of access to the source code.

### 使用解答

No questions

### 技术解答

No response

## About existing features

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

## How to use add-ons

### GameSpace

[still writing...]

### Performance Mode (GameSpace Add-on)

[still writing...]

### Battery Saver Mode

[still writing...]

### Thermal Mode

[still writing...]

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

@yc9559
the creator of perfd ​​opt and uperf, the GOAT of the 2018-2023 modules

@cover image artist
I couldn't find the artist, so I'll owe you this one.
```

