# LG V35 ThinQ - Bootloader Unlocking, TWRP Flashing and forceencrypt disabling

[中文版本](https://github.com/kaneorotar/LG-V35-Tinkering-Instructions/blob/master/README.md)

### :warning: Warning: The procedure below requires formatting internal storage. Please backup all your data.

### :loudspeaker: Disclaimer: I will not be responsible for any possible damage to your device/data. Proceed at your own risk.

:speech_balloon: Please feel free to correct me if you spot any errors in this guide.

----------

## Preparation

Hardware：
 * Computer running Windows x1
 * LG V35 ThinQ x1 (running Official Pie)
 * Micro SD Card x1 (or USB Drive x1 + OTG Adapter/Cable x1)

Software：
 * [Android SDK Platform Tools](https://developer.android.com/studio/releases/platform-tools.html)（We need adb.exe and fastboot.exe). 
 * (Optional) [Android Image Kitchen](https://forum.xda-developers.com/showthread.php?t=2073775) (Only needed if you want to manually edit the boot.img). 

----------

## Steps

### Bootloader Unlocking

Originally mentioned [here](https://forum.xda-developers.com/lg-v40/development/unlock-lg-v40-via-9008-root-t-mobile-t4042207) and [here](https://forum.xda-developers.com/lg-v35/development/bootloader-unlock-root-instruction-t4052145). To summarize：

1. Install [QPST & QFIL](https://qpsttool.com/qpst-tool-v2-7-480), Download [Firehose for LG SDM845 Devices](https://url.cn/5hRy6EO)

2. Put V35 in Emergency DownLoad Mode (EDL, also known as 9008 mode)
> * Connect the phone to your computer. 
> * Press and hold the Power and Vol- keys. 
> * As soon as the screen goes out, start tapping the Vol+ key like your life depends on it. 
> * If the phone screen stays off and a new device (Qualcomm HS-USB QDLoader 9008 (COM#)) shows up on the computer (can be seen in Device Manager), you've successfully entered EDL. 
> 
3. Run QFIL and change a few configurations：
> * Click 'Select Port' and select the phone in EDL/9008 mode
> * Choose 'Flat Build'
> * Click the 'Browse' button next to 'Programmer Path', then select the firehose you downloaded
> * (At the bottom right) Click 'Storage Type' and choose 'ufs'

4. Navigate to the top menu: Tools -> Partition Manager -> OK. If all goes well, you will be greeted with a dialog listing all partitions in ~5s. If this is not the case even after a full minute, try re-entering EDL/9008 mode.

5. Once the partiton list is loaded, you can perform:
> * 'Erase' *(Clear the partiton)*
> * 'Read Data...' *(Dump the partition)* 
> * 'Load Image...' *(Flash the partiton)* 
> 
> on partitons by right clicking a partiton and then clicking 'Manage Partiton Data'.
> 
6. **Before proceeding to the next step, it is highly recommended to backup your partitions with 'Read Data...' before modifying them. The dumped partition can be found in `C:\Users\%USERNAME%\AppData\Roaming\Qualcomm\QFIL\COMPORT_##`.** 
> Partitions to backup:
> * **abl_a** (required for restoring)
> * abl_b
> * boot_a
> * boot_b
> * laf_a
> * laf_b
* Flash [V35 Engineering abl](https://url.cn/5Ni6nuO) into the `abl_a` partition. Then close QFIL.
* Reboot the phone (by pressing and holding Power and Vol- keys). It should now boot into `fastboot` mode. Then unlock the bootloader by running `fastboot oem unlock` from command line (**The phone's internal storage will be cleared**)
* Re-enter EDL/9008 mode. Run QFIL again and restore the `abl_a` partiton with the 'Load Image...' feature.
* (Optional) Clear the `laf_a` partition (backup recommended). You will then be able to enter `fastboot` mode by press and holding Vol+ when booting. (`laf_a` holds the image for the LG Download mode and falls back to `fastboot` when failing to enter Download mode)

### TWRP Flashing

To do this, you need TWRP for V35. ~~(which does not exists yet)~~ I spent a day and compiled [one that works on Pie](https://github.com/kaneorotar/LG-V35-Tinkering-Instructions/releases) (based on the [LG V40 TWRP](https://forum.xda-developers.com/lg-v40/development/twrp-lg-v40-judypn-t3970111). Cannot decrypt encrypted data partition yet.) 

You can install it in 2 ways:

**Notice: All TWRP installation methods replaces the ramdisk of the `boot` partition. This means existing Magisk will be gone. But don't panic, you can simply flash Magisk-vXX.X.zip from TWRP afterward.**

1. If you already have Magisk installed, you can flash `twrp-installer-vX.X.X-v35_*.zip` from Magisk Manager (`*` can be `a` or `b` depending on which slot you want it installed).
   
2. If you do not have Magisk installed, backup your `boot_a/b` partition with QFIL and then flash `twrp-vX.X.X-v35.img` with 'Load Image...' or `fastboot flash boot_a/b twrp-vX.X.X-v35.img` from command line. (The kernel of this image was extracted from V350ULM20e and *seems* to work fine for Pie in general)

**Notice: You can read more about slot A/B [here](https://www.xda-developers.com/how-a-b-partitions-and-seamless-updates-affect-custom-development-on-xda/)。** To put it simply, your phone holds 2 sets (`a` and `b`) of OS partitions for seamless. OS upgrades are being flashed to slot `a` or `b` in turns which allows the phone to fallback to the previous version if anything goes wrong with the new one. Only one of the slots is being used (active) at any point in time (usually slot `a`) and that's where you want to install TWRP. The active slot can be found using `fastboot getvar current-slot`.

### Forceencrypt Disabling

We will be using [Universal DM-Verity, ForceEncrypt, Disk Quota Disablers](https://forum.xda-developers.com/android/software/universal-dm-verity-forceencrypt-t3817389) for this step.

Download the [zip](https://zackptg5.com/downloads/Disable_Dm-Verity_ForceEncrypt_03.04.2020.zip) and place it in the Micro SD card or USB Drive. (In case the link becomes invalid, visit [the author's website](https://zackptg5.com/android.php#disverfe) and download the new zip under 'Dm-Verity & ForceEncrypt Disabler')

Now enter TWRP/Recovery.

> The safest method is to root your phone first (with Magisk). Then use utility apps (e.g. Magisk Manager, '☰' -> 'Modules' -> '⋮' -> 'Reboot to Recovery') to reboot to recovery. You can also do it without the help of any app: enable USB debugging, connect phone to computer and run `adb reboot recovery` (A dialog should pop up for you to authorize this on your phone the first time. If that did not happen, try switching to 'Charging Only' mode)
> 
> **Notice: It is recommended to have TWRP installed on both `boot_a` and `boot_b` before trying the method below. Otherwise you risk losing all userdata if the phone switches to vanilla recovery and starts the factory reset process.**
> 
> Alternatively, you can first press and hold Power and Vol- keys. Once the device reaches the 'LG V35' screen, briefly release the Power key and then press it (multiple times if you want). The phone should enter a white screen with two buttons. Choose 'Yes' twice and it will reboot into TWRP.

After entering TWRP, you can flash the zip downloaded above.
> During flashing, you may see error messages about 'unable to unmount vendor'. This is fine as long as the zip flashing doesn't fail halfway.
> 
> **Do not click 'Reboot System' after the flashing is finished**. Navigate back to the top menu first, then 'Reboot' -> 'Recovery'
> 
> Once the device re-enters TWRP, click 'Wipe'. Then 'Format Data', enter `yes` and proceed. **The Internal Storage will be formatted and won't be encrypted by the OS.**
> 
> You should now be able to access Internal Storage from TWRP. You can reboot back to System, setup the device and start using it normally.

## Remarks

* If you only did the steps above to one of the slots (e.g. `a`), you will be greeted by a 'Decryption Unsuccessful' message when switching to the other slot (e.g. `b` in this case). This can be fixed by going throught the steps for the other slot as well.

* Try not to mix the boot.img between slot `a/b` as they may not be fully compatible across different OS versions. 

* With a working TWRP, you can now flash GSIs.
    <details>
    <summary>Comments on GSIs</summary>

    Based on my limited experiences, GSIs in general lack the following features:
    > * VoLTE
    > * Resolution switching
    > * Double tap to wake
    > * DAC Support
    > * Screen Casting/Mirroring
    > * Always on Display (Can be 'fixed' by using an overlay, refer to [this](https://github.com/phhusson/vendor_hardware_overlay/tree/master/LG/G7))
    

    Most Pie GSIs boot. Some have vibration issues that can be fixed by patching the device fingerprint with the original V35 one.

    Working Pie GSIs:
    > * Havoc OS
    > * ArrowOS
    > * AOSIP
    > * Lineage
    

    Common problems with Q GSI:
    > * Most won't boot (can possibly be fixed by applying the selinux permissive patch)
    > * Screen won't light up after device goes to sleep
    > * WiFi not working properly (Scans but does not connect)
    

    Q GSIs that boot:
    > * Some of the ErfanGSIs 
    </details>


## Reference

* https://forum.xda-developers.com/lg-v40/development/unlock-lg-v40-via-9008-root-t-mobile-t4042207
* https://forum.xda-developers.com/lg-v40/development/twrp-lg-v40-judypn-t3970111
* https://bbs.lge.fun/thread-14.htm
* https://g7.lge.fun/guide/twrpboot.html
* https://www.cnblogs.com/stars-one/p/10723732.html
* https://forum.xda-developers.com/android/software/universal-dm-verity-forceencrypt-t3817389
