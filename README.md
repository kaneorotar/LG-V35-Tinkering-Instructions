# LG-V35-Tinkering-Instructions
Unleash your LG V35 ThinQ

# LG V35 ThinQ 刷TWRP和去forceencrypt/解密方法

### 警告：解密过程需要重置内置存储，请备份好所有数据之后再操作。
### Warning: The process below requires formating internal storage. Please backup all your data.

### 免责声明：对于因本文产生的任何数据或硬件的损坏或丢失，由操作者个人承担，本人概不负责。
### Disclaimer: I will not be responsible for any possible damage to your device/data. Proceed at your own risk.

本文有可能存在错误，欢迎指正。Please feel free to correct me if you spot any errors in this guide.

所需的材料：LG-V35, Micro SD卡一张

所需的工具：boot.img编辑工具(例如[Android Image Kitchen](https://forum.xda-developers.com/showthread.php?t=2073775))

----------

首先，你需要解开Bootloader锁 (参考[这篇](https://forum.xda-developers.com/lg-v40/development/unlock-lg-v40-via-9008-root-t-mobile-t4042207)或[这篇](https://forum.xda-developers.com/lg-v35/development/bootloader-unlock-root-instruction-t4052145))。简述下来有这几步：

* 安装QPST & QFIL
* 让V35进入Emergency Download (EDL，又称9008)模式
> 手机连电脑，按住电源键和Vol-重启，熄屏的瞬间按住的键不要松并且开始狂按Vol+。如果手机保持黑屏并且电脑上提示插入了新设备，则成功进入模式，
* 备份原版的boot分区
* 刷入V35工程机的boot并重启手机
* 手机会自动进入fastboot模式。在电脑端使用`fastboot oem unlock`解锁 (会清除内置存储的数据)
* 再进入EDL/9008模式，还原boot分区
* (可选项) 清除laf_a分区(建议备份)，这样手机启动时如果试图进入Download模式(按住Vol+)会因为对应的代码被清空而退而进入fastboot模式。

建议在对任何分区做修改之前先用"Read Data"做个备份，对应的文件会在`C:\Users\你的用户名\AppData\Roaming\Qualcomm\QFIL\COMPORT_##`内。

接下来，你得有V35能用的TWRP。虽然直接为V35编译的TWRP是不存在的，但是由于硬件的相似度较高，我们可以对V40的TWRP进行魔改达成目的。

> ### 背景知识 Background
> 
> boot.img分三个部分
> * zImage/kernel (内核)
> * bootimg.cfg (设置参数)
> * 和initrd (内存盘/ramdisk)。
> 
> Recovery，例如TWRP，一般是存在于initrd内的。

> ### 魔改 Modification
> 
> 混血的boot.img将由以下三个部分组成：
> * 原版boot的zImage
> * 原版boot的bootimg.cfg
> * 替换了部分V35文件的V40 TWRP的initrd
>
> 请先使用QFIL提取原版的boot (boot_a)，然后下载[V40的TWRP](https://forum.xda-developers.com/lg-v40/development/twrp-lg-v40-judypn-t3970111)
> 
> 以下的文件需要从原版boot的initrd内提取覆盖到V40 TWRP的initrd内：
> * etc/recovery.fstab
> * res/keys
> * prop.default
>        
> 以下的文件需要重命名：
> * fstab.judypn  ->  fstab.judyp
> * init.recovery.judypn.rc  ->  init.recovery.judyp.rc
> * ueventd.judypn.rc  ->  ueventd.judyp.rc
> 
> 以下的文件内容需要修改：
> * 刚更名的initrd\init.recovery.judyp.rc内，搜索"judypn"并替换为"judyp"

然后想办法进Recovery。

> 最保险的方式是root之后用工具软件重启到recovery，没有的话可以开启USB调试后连电脑然后 `adb reboot recovery`
> 
> **注意：以下方法建议在A/B slot的boot的recovery都替换成了TWRP之后再尝试，否则可能会因为切换slot进入正常的恢复出厂流程，将导致内置存储数据全部丢失。**
> 
> 另一种方式是先按住电源健和Vol-，等重启到LGV35画面亮起来之后松开电源键再按回去。手机会进入一个白屏一段文字两个按钮的恢复出厂的提示界面，选两次Yes后重启就会进入recovery。

刷[Universal DM-Verity, ForceEncrypt, Disk Quota Disablers](https://forum.xda-developers.com/android/software/universal-dm-verity-forceencrypt-t3817389)

> 请提前把[zip](https://zackptg5.com/downloads/Disable_Dm-Verity_ForceEncrypt_01.19.2020.zip)下载好放在microSD卡上。内置存储由于还未解密所以TWRP暂时无法读取。(如果链接失效，请访问[作者官网](https://zackptg5.com/android.php#disverfe)找到“Dm-Verity & ForceEncrypt Disabler”进行下载)
> 
> 刷zip的过程中你应该会看到几行红字提示无法卸载vendor分区之类的，没有关系。
> 
> 刷完后**不要直接点重启 (Reboot System)**。先返回到顶层菜单，然后点重启 (Reboot)，然后选Recovery。重启后应该会回到TWRP。这次点重启然后选Recovery。
> 
> 再次进入TWRP之后，选择清除 (Wipe)，然后点击右侧的格式化Data分区 (Format Data)，输入yes后确认。内置存储 (Internal Storage)会被格式化并且这次格式化之后data分区不会被加密。
> 
> 格式化结束后在TWRP内应该就能正常加载内置存储 (Internal Storage)了。重启回System然后重新配置设备就可以正常使用。

完事儿了。

## 备注 Remarks

* 如果你只对slot A进行了上述的操作，在切换到slot B的时候依然会提示解密失败(Decryption Unsuccessful)，解决办法是对slot B重复以上步骤。

* 需要注意的是：在没有瞎折腾过的情况下slot A/B里存着的是LG原版系统的最近两个版本，意味着包括boot在内的分区内容都是不同的，很有可能不通用。
所以**请不要把为slot A准备的包含TWRP的boot.img直接刷入slot B的boot分区。**

* 一切顺利的话，你的V35就基本解放了。可以通过TWRP自由的切换slot A/B，备份/恢复数据，刷Magisk。

* 按照这个方法生成的TWRP还存在以下的不足：
  1. 屏幕顶端有留空(因为V40有挖孔/notch)
  2. MTP模式不能正常使用
  3. 挂载USB存储模式不能正常使用
  4. 备份的时候会有"system_root"无法卸载的错误信息(但是好像不影响使用)

## 参考 Reference

* https://forum.xda-developers.com/lg-v40/development/unlock-lg-v40-via-9008-root-t-mobile-t4042207
* https://forum.xda-developers.com/lg-v40/development/twrp-lg-v40-judypn-t3970111
* https://bbs.lge.fun/thread-14.htm
* https://g7.lge.fun/guide/twrpboot.html
* https://www.cnblogs.com/stars-one/p/10723732.html
* https://forum.xda-developers.com/android/software/universal-dm-verity-forceencrypt-t3817389
