---
title: "Installing LineageOS on an OnePlus {{< fa mobile-screen-button >}}"
description: "Moving to an ungoogled-ROM"
date: "2024-01-18"
image: lineage-logo.png
image-height: "100px"
categories: 
  - linux
  - DIY
format:
  html: 
    code-line-numbers: false
---

Following the previous attempt (successful) of installing the **LineageOS** [described here](../2023-11-16_lineagesos-s5/), I decided to really change my previous phone from 2018 which last security update was 2020-06-01. The hardware was still working, but the lack of updates was really annoying, stuck with Android 10 and all security holes.

## Candidate phone

My sister had a One Plus 6T in a drawer collecting dust. She wiped it and gave it to me, nice!


## LineageOS on the One Plus 6T

**LineageOS** support so many (and old) [devices](https://wiki.lineageos.org/devices/) it's amazing. The One Plus 6T corresponds to the [fajita](https://wiki.lineageos.org/devices/fajita/) release and builds are [here](https://download.lineageos.org/devices/fajita/builds).

### Installation

The 8-steps instructions are clearly described in the [wiki](https://wiki.lineageos.org/devices/fajita/install/).

I had to update from Android 10 to 11 as asked. 

After activating the USB debugging, I had to add my login to the `plugdev` group while 
it was not necessary for the Samsung S5.

``` bash
# usermod -aG plugdev login
```

Had to restart the laptop, check you are part of this group with `id`.

Of note, all commands are run as `root`.


### Unlock OEM

After rebooting in proprietary recovery: 

![](restart_bootloader.jpg){height=300}

The following commands:

``` bash
fastboot devices
fastboot oem unlock
```

Are super fast:

![](cli_unlock.jpg)

Result:

![](oem_unlock.jpg){height=300}

### Install lineage recovery

``` bash
sideload -d boot.img
adb -d sideload lineage-20.0-20240116-nightly-fajita-signed.zip 
```

![](cli_recovery.jpg)

![](recovery_adb.jpg){height=300}

Then sideload vbmeta.img and dtbo.img and [MindtheGapps](https://github.com/MindTheGapps/13.0.0-arm64/releases/tag/MindTheGapps-13.0.0-arm64-20231025_200931)

::: {.callout-important}
## Watch out

To use the right architecture and Android version (`13` here).
:::

I am not sure what the `dtbo` and `vbmeta` images actually are but I did follow all steps.

![](wipe_sideload.jpg){height=300}

``` bash
adb -d sideload dtbo.img 
adb -d sideload vbmeta.img 
adb -d sideload MindTheGapps-13.0.0-arm64-20231025_200931.zip
```

![](install_mind_the_gapps.jpg){height=300}

and reboot system.


## Aftermath

We started from Android 11, now we have 13.
Everything is working. The only annoying part is the Fingerprint reader which success reading rate is around 50%. Don't know what is wrong, the sensor or the software treating information.

The bank apps are working, Luxtrust and so on.

While installing message apps, it was interesting to see the different levels of 
security for restoring/transferring accounts.

- Signal
- Telegram
- Simplex Chat

Each boot is showing this warning, I gladly ignore.

## What's next?

Getting rid of Google Play. Or Google account altogether.
Next goal is to try [CalyxOS](https://calyxos.org/) and the **Aurora store**.
I have restored some security with **LineageOS** but no privacy since I kept Google Play.
