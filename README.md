# Advanced Charging Controller (ACC)



---
## LEGAL

Copyright (C) 2017-2019, VR25 @ xda-developers

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.



---
## DISCLAIMER

Always read/reread this reference prior to installing/upgrading this software.

While no cats have been harmed, the author assumes no responsibility for anything that might break due to the use/misuse of it.

To prevent fraud, do NOT mirror any link associated with this project; do NOT share builds (zips)! Share official links instead.



---
## WARNING

ACC manipulates Android low level ([kernel](https://duckduckgo.com/?q=kernel+android)) parameters which control the charging circuitry.
The author assumes no responsibility under anything that might break due to the use/misuse of this software.
By choosing to use/misuse ACC, you agree to do so at your own risk!



---
## DESCRIPTION

ACC is primarily intended for [extending battery service life](https://batteryuniversity.com/learn/article/how_to_prolong_lithium_based_batteries). On the flip side, the name says it all.



---
## PREREQUISITES

- Any root solution
- Terminal emulator
- Text editor (optional)



---
## BUILDING FROM SOURCE


Dependencies

- curl (optional)
- git
- zip


Steps

1. `git clone https://github.com/VR-25/acc.git`
2. `cd acc`
3. `sh build.sh` (or double-click `build.bat` on Windows, if you have Windows subsystem for Linux installed)


Notes

- The output file is _builds/acc-$versionCode.zip.

- By default, `build.sh` auto-updates the [update-binary](https://raw.githubusercontent.com/topjohnwu/Magisk/master/scripts/module_installer.sh). To skip this, run `sh build.sh f` (or `buildf.bat` on Windows).

- To update the local repo, run `git pull -f`.

- To install/upgrade straight from source, refer to the next section.



---
## SETUP


### Magisk 18.2+

Install/upgrade: flash live (e.g., from Magisk Manager) or from custom recovery (e.g., TWRP).

Uninstall: use Magisk Manager (app) or [Magisk Manager for Recovery Mode (utility)](https://github.com/VR-25/mm/).


### Any Root Solution (Advanced)

Install/upgrade: extract `acc-*.zip`, run `su`, then execute `sh /path/to/extracted/install-current.sh`.

Uninstall: for Magisk install, use Magisk Manager (app); else, run `su -c rm -rf /data/adb/acc/`.


### Notes

ACC supports live upgrades - meaning, rebooting after installing/upgrading is unnecessary.

The demon is automatically started ~30 seconds after installation.

For non-Magisk install, `/data/adb/acc/acc-init.sh` must be executed on boot to initialize acc. Without this, acc commands won't work. Additionally, if your system lacks executables such as `awk` and `grep`, [busybox](https://duckduckgo.com/?q=busybox+android) or similar binary must be installed prior to installing acc.



---
## DEFAULT CONFIGURATION
```
# This is used to determine whether config should be patched. Do NOT modify!
versionCode=XXXXXXXXX

# shutdown,coolDown,resume-pause
capacity=0,60,70-80

# Change this only if your system reports incorrect battery capacity ("acc -i" (BMS) vs "dumpsys battery" (system)).
capacityOffset=+0

# This is an alternative to capacityOffset. It tells acc whether the battery capacity reported by Android should be updated every few seconds to reflect the actual value from the battery management system.
capacitySync=false

# <coolDown-pauseCharging_waitSeconds> - <waitSeconds> allow battery temperature to drop below <pauseCharging>. Temperature values are interpreted in Celsius degrees. To disable temperature control entirely, set absurdly high temperature values (e.g., temperature=90-95_90).
temperature=40-45_90

# Charge/pause ratio in seconds (e.g., coolDownRatio=50/10) - reduces battery stress induced by prolonged high temperature and high charging voltage by periodically pausing charging. If charging is too slow, turn this off (null value) or change the ratio. When set to null, <coolDown capacity> and <coolDown temperature> values are nullified.
# Generally, you don't need this if you're limiting the maximum charging voltage.
coolDownRatio=

# Reset battery stats after <pauseCapacity> is reached.
resetBsOnPause=true

# Reset battery stats every time charger is unplugged, as opposed to only when <pauseCapacity> is reached.
resetBsOnUnplug=false

# Seconds between loop iterations - this is essentially a sensitivity "slider". Do not touch it unless you know exactly what you're doing! A value lower than 5 may cause unexpected behavior (e.g., increased battery drain). Anything above 30 will lead to major delays in charging control, but it will dramatically improve acc energy efficiency.
loopDelay=10

# Custom charging switch parameters (<path> <onValue> <offValue>), e.g., chargingSwitch=/sys/class/power_supply/battery/charging_enabled 1 0, pro tip: </sys/class/power_supply/> can be omitted (e.g., chargingSwitch=battery/charging_enabled 1 0).
chargingSwitch=

# Settings to apply on boot - e.g., applyOnBoot=usb/device/razer_charge_limit_enable:1 usb/device/razer_charge_limit_max:80 usb/device/razer_charge_limit_dropdown:70 /sys/kernel/fast_charge/force_fast_charge:1 --exit
# --exit stops accd.
applyOnBoot=

# Settings applied every time an external power supply is connected - e.g., applyOnPlug=wireless/voltage_max:9000000 usb/current_max:2000000
# Tip: applyOnPlug=wireless/voltage_max:9000000 forces fast wireless charging.
applyOnPlug=

# Charging voltage limit (file:millivolts, e.g., chargingVoltageLimit=?attery/voltage_max:4200)
# Voltage range (millivolts): 3920-4349
chargingVoltageLimit=

# Reboot after <pauseCapacity> is reached and <seconds> (e.g., rebootOnPause=60) have passed (disabled if null). If this doesn't make sense to you, you probably don't need it.
rebootOnPause=

# Reboot after charger is unplugged and <seconds> (e.g., rebootOnUnplug=60) have passed (disabled if null). This is a workaround for re-enabling charging.
rebootOnUnplug=

# Minimum charging on/off toggling interval (seconds)
chargingOnOffDelay=1

# English (en) and Portuguese (pt) are the main languages supported. Refer to the localization section below for all available languages and translation information.
language=en

# Wakelocks to unlock after pausing charging (e.g., wakeUnlock=chg_wake_lock qcom_step_chg)
# Use this only if you known what you're doing. It may cause unexpected behavior.
wakeUnlock=
```


---
## USAGE


ACC is designed to run out of the box, without user intervention. You can simply install it and forget. However, as it's been observed, most people will want to tweak settings - and obviously everyone will want to know whether the thing is actually working.

If you feel uncomfortable with the command line, skip this section and use the [ACC app](https://github.com/MatteCarra/AccA/releases/) to manage ACC.

Alternatively, you can use a `text editor` to modify `/sdcard/acc/acc.conf`. Changes to this file take effect almost instantly, and without a [daemon](https://en.wikipedia.org/wiki/Daemon_(computing)) restart.


### Terminal Commands
```
acc <option(s)> <arg(s)>

-c|--config <editor [opts]>   Edit config w/ <editor [opts]> (default: vim|vi)
  e.g., acc -c

-d|--disable <#%, #s, #m or #h (optional)>   Disable charging (with or without <condition>)
  e.g.,
    acc -d 70% (do not recharge until capacity drops to 70%)
    acc -d 1h (do not recharge until 1 hour has passed)

-D|--daemon   Show current acc daemon (accd) state
  e.g., acc -D (alias: "accd,")

-D|--daemon <start|stop|restart>   Manage accd state
  e.g.,
    acc -D restart
    accd -D stop (alias: "accd.")

-e|--enable <#%, #s, #m or #h (optional)>   Enable charging or enable charging with <condition>
  e.g., acc -e 30m (recharge for 30 minutes)

-f|--force|--full <capacity>   Charge to a given capacity (fallback: 100) once and uninterrupted
  e.g., acc -f 95

-i|--info   Show power supply info
  e.g., acc -i

-l|--log -e|--export   Export all logs to /sdcard/acc-logs-<device>.tar.bz2
  e.g., acc -l -e

-l|--log <editor [opts]>   Open <acc-daemon-deviceName.log> w/ <editor [opts]> (default: vim|vi)
  e.g., acc -l grep ': ' (show explicit errors only)

-L|--logwatch   Monitor log
  e.g., acc -L

-r|--readme   Open <README.md> w/ <editor [opts]> (default: vim|vi)
  e.g., acc -r

-R|--resetbs   Reset battery stats
  e.g., acc -R

-s|--set   Show current config
  e.g., acc -s

s|--set <r|reset>   Restore default config
  e.g., acc -s r

-s|--set <var> <value>   Set config parameters
  e.g., acc -s capacity 5,60,80-85 (5: shutdown, 60: cool down, 80: resume, 85: pause)

-s|--set <resume-stop preset>   Can be 4041|endurance+, 5960|endurance, 7080|default, 8090|lite 9095|travel
  e.g.,
    acc -s endurance+ (a.k.a, "the li-ion sweet spot"; best for GPS navigation and other long operations)
    acc -s travel (for when you need extra juice)
    acc -s 7080 (restore default capacity settings (5,60,70-80))

-s|--set <s|chargingSwitch>   Set a different charging switch from the database
  e.g., acc -s s

-s|--set <s:|chargingSwitch:>   List available charging switches
  e.g., acc -s s:

-s|--set <s-|chargingSwitch->   Unset charging switch
  e.g., acc -s s-

-t|--test   Test currently set charging ctrl file
  Exit codes: 0 (works), 1 (doesn't work) or 2 (battery must be charging)
  e.g., acc -t

-t|--test <file on off>   Test custom charging ctrl file
  Exit codes: 0 (works), 1 (doesn't work) or 2 (battery must be charging)
  e.g., acc -t battery/charging_enabled 1 0

-t|--test -- <file (fallback: $modPath/switches.txt)>   Test charging switches from a file
  This will also report whether "battery idle" mode is supported
  Exit codes: 0 (works), 1 (doesn't work) or 2 (battery must be charging)
  e.g., acc -t -- /sdcard/experimental_switches.txt

-v|--voltage   Show current charging voltage
  e.g., acc -v

-v|--voltage :   List available/default charging voltage ctrl files
  e.g., acc -v :

-v|--voltage -   Restore default charging voltage limit
  e.g., acc -v -

-v|--voltage <millivolts>   Set charging voltage limit (default/set ctrl file)
  e.g., acc -v 4100

-v|--voltage <file:millivolts>   Set charging voltage limit (custom ctrl file)
  e.g., acc -v battery/voltage_max:4100

-x|--xtrace <other option(s)>   Run under set -x (debugging)
  acc -x -i

Tips

  Commands can be chained for extended functionality. Note that accd must be stopped first.
    e.g., acc -D stop && acc -e 30m && acc -d 6h && acc -e 85 && accd (recharge for 30 minutes, halt charging for 6 hours, recharge to 85% capacity and restart daemon)

  Pause and resume capacities can also be set with acc <pause%> <resume%>.
    e.g., acc 85 80

  That last command can be used for programming charging before bed. In this case, the daemon must be running.
    e.g., acc 45 44 && acc --set applyOnPlug usb/current_max:500000 && sleep $((60*60*7)) && acc 80 70 && acc --set applyOnPlug usb/current_max:2000000
    - "Keep battery capacity at ~45% and limit charging current to 500mA for 7 hours. Restore regular charging settings afterwards."
    - For convenience, this can be written to a file and ran as "sh <file>".
    - If your device supports custom charging voltage, it's better to use it instead: "acc -v 3920 && sleep $((60*60*7)) && acc -v -".

Run acc --readme to see the full documentation.
```


---
## NOTES/TIPS FOR FRONT-END DEVELOPERS


It's best to use full commands over short equivalents - e.g., `--set chargingSwitch` instead of `-s s`.

Use provided config descriptions for ACC settings in your app(s). Include additional information (trusted) where appropriate.


### Online ACC Install

- The installer must run as root (obviously).
- Log: /sbin/.acc/install-stderr.log
```
1) Check whether ACC is installed (exit code 0)
which acc > /dev/null

2) Download the installer (https://raw.githubusercontent.com/VR-25/acc/master/install-latest.sh)
- e.g., curl -#L [URL] > [output file] (progress is shown)

3) Run "sh [installer]" (progress is shown)
```

### Offline ACC Install

Refer to [SETUP > Any Root Solution (Advanced)](https://github.com/VR-25/acc/tree/master#any-root-solution-advanced) and [SETUP > Notes ](https://github.com/VR-25/acc/tree/master#notes).



---
## TROUBLESHOOTING


### Charging Switch

By default, ACC cycles through all available [charging control files](https://github.com/VR-25/acc/blob/master/acc/switches.txt) until it finds one that works.

Charging switches that support battery idle mode take precedence - allowing the device to draw power directly from the external power supply when charging is paused.

However, things don't always go well.
Some switches may be unreliable under certain conditions (e.g., screen off).
Others may hold a [wakelock](https://duckduckgo.com/?q=wakelock) - causing faster battery drain - while in plugged in, not charging state.

Run `acc --set chargingSwitch` (or `acc -s s` for short) to enforce a particular switch.

Test default/set switch(es) with `acc --test`.

Evaluate custom switches with `acc --test <file onValue offValue>`.


### Charging Voltage Limit

Unfortunately, not all devices/kernels support custom charging voltage limit.
Those that do are rare.
Most OEMs don't care about that.

The existence of a potential voltage control file doesn't necessarily mean it works.


### Restore Default Config

`acc --set reset` (or `acc -s r`)


### Slow Charging

Check whether charging current in being limited by `applyOnPlug` or `applyOnBoot`.

Nullify coolDownRatio (`acc --set coolDownRatio`) or change its value. By default, coolDownRatio is null.


### Logs

Logs are stored at `/sbin/.acc/`. You can export all to `/sdcard/acc-logs-$device.tar.bz2` with `acc --log --export`. In addition to acc logs, the archive includes `charging-ctrl-files.txt`, `charging-voltage-ctrl-files.txt`, `acc.conf` and `magisk.log`.



---
## POWER SUPPLY LOG


Please upload `/sbin/.acc/acc-power_supply-*.log` to [this dropbox](https://www.dropbox.com/request/WYVDyCc0GkKQ8U5mLNlH/).
This file contains invaluable power supply information, such as battery details and available charging control files.
A public database is being built for mutual benefit.
Your cooperation is greatly appreciated.


Privacy Notes

- When asked for a name, give your `XDA username` or any random name.
- For the email, you can type something like `noway@areyoucrazy.com`.

Example
- Name: `user .`
- Email: `myemail@iscool.com`


See current submissions [here](https://www.dropbox.com/sh/rolzxvqxtdkfvfa/AABceZM3BBUHUykBqOW-0DYIa?dl=0).



---
## LOCALIZATION

Currently Supported Languages
- English (en): complete
- Portuguese (pt): partial

Translation Notes
- Translators should start with copies of [acc/strings.sh](https://github.com/VR-25/acc/blob/master/acc/strings.sh) and [README.md](https://github.com/VR-25/acc/blob/master/README.md) - and append the appropriate language suffix to the base names - e.g., `strings_it`, `README_it`.
- Anyone is free and encouraged to open translation [pull requests](https://duckduckgo.com/?q=pull+request).
- Alternatively, `strings_*.sh` and `README_*.md` files can be send to the developer.



---
## TIPS


### Generic

Control the max USB input current: `applyOnPlug=usb/current_max:MICRO_AMPS` (e.g., 1000000, that's 1A)

Force fast charge: `applyOnBoot=/sys/kernel/fast_charge/force_fast_charge:1`


### Google Pixel Family

Force fast wireless charging with third party wireless chargers that are supposed to charge the battery faster: `applyOnPlug=wireless/voltage_max:9000000`.


### Razer Phone

Alternate charging control configuration:
```
capacity=5,60,0,101
applyOnBoot=razer_charge_limit_enable:1 usb/device/razer_charge_limit_max:80 usb/device/razer_charge_limit_dropdown:70
```

### Samsung

The following files could be used to control charging current and voltage (with `applyOnBoot`):
```
battery/batt_tune_fast_charge_current (default: 2100)

battery/batt_tune_input_charge_current (default: 1800)

battery/batt_tune_float_voltage (max: 4350)
```


---
## Frequently Asked Questions (FAQ)


- How do I report issues?

A: Open issues on GitHub or contact the developer on Telegram/XDA (linked below). Always provide as much information as possible, and attach the output file of `acc --log --export`.


- What's "battery idle" mode?

A: That's a device's ability to draw power directly from an external power supply when charging is disabled or the battery is pulled out. The Motorola Moto G4 Play and many other smartphones can do that. You can run `acc --t --` to test yours.


- What's "cool down" capacity for?

A: It's meant for reducing stress induced by prolonged high charging voltage (e.g., 4.20 Volts). It's a fair alternative to custom charging voltage limit.


- Why won't you support my device? I've been waiting for ages!

A: First, never lose hope! Second, several systems don't have intuitive charging control files; I have to dig deeper and improvise; this takes extra time and effort. Lastly, some systems don't support custom charging control at all;  in such cases, you have to keep trying different kernels and uploading the respective [power supply logs](https://github.com/VR-25/acc#power-supply-log) - so I can check these for potential charging control files.



---
## LINKS

- [ACC app](https://github.com/MatteCarra/AccA/releases/)
- [Battery University](http://batteryuniversity.com/learn/article/how_to_prolong_lithium_based_batteries/)
- [Donate](https://paypal.me/vr25xda/)
- [Facebook page](https://facebook.com/VR25-at-xda-developers-258150974794782/)
- [Git repository](https://github.com/VR-25/acc/)
- [Telegram channel](https://t.me/vr25_xda/)
- [Telegram group](https://t.me/acc_magisk/)
- [Telegram profile](https://t.me/vr25xda/)
- [XDA thread](https://forum.xda-developers.com/apps/magisk/module-magic-charging-switch-cs-v2017-9-t3668427/)



---
## LATEST CHANGES

**2019.6.25 (201906250)**
- Faster `acc -D|--daemon stop` (alias `accd.`)
- Fixed config patching error
- Fixed install-latest.sh unset parameter issue
- General fixes & optimizations
> Note: AccApp 1.0.10 is up; it brings a bunch of bug fixes.

**2019.6.23 (201906230)**
- "acc -D|--daemon" alias: "accd,"
- "acc -D|--daemon stop" alias: "accd."
-  acc -t -- <file (fallback: $modPath/switches.txt)>: test charging switches from a file; this will also report whether "battery idle" mode is supported
- General fixes & optimizations
- Increased power efficiency
- Striped down (for easier patching) and renamed config.txt --> acc.conf; comprehensive config information is in the README.md
- Updated documentation: FAQ section and more
- wakeUnlock is null by default

**2019.6.20 (201906200)**
- Additional charging control files
- Enhanced daemon reliability
- `install-current.sh` no longer requires absolute path
- Updated documentation
- wakeUnlock - auto-unlock select wakelocks after charging is disabled
