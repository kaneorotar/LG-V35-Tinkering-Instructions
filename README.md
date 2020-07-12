# LG V35 ThinQ 解Bootloader锁、刷TWRP和去forceencrypt/解密方法
# LG V35 ThinQ - Bootloader Unlocking, TWRP Flashing and forceencrypt disabling

[English Version](https://github.com/kaneorotar/LG-V35-Tinkering-Instructions/blob/master/README_EN.MD)

### :warning: 警告：文中介绍的过程需要多次重置内置存储，请备份好所有数据之后再操作。

### :loudspeaker: 免责声明：对于因本文产生的任何数据或硬件的损坏或丢失，由操作者个人承担，本人概不负责。

:speech_balloon: 本文有可能存在错误，欢迎反馈指正。

----------

## 准备 Preparation

硬件：
 * 运行Windows系统的电脑 x1
 * LG V35 ThinQ x1 (运行官方Pie系统)
 * Micro SD卡 x1 (或者U盘 x1 + OTG 适配器/线 x1)
 
软件：
 * [安卓SDK平台工具(platform-tools)](https://developer.android.com/studio/releases/platform-tools.html?hl=zh-CN)（其实就是需要其中的adb.exe和fastboot.exe)
 * (如果你打算解剖boot镜像)boot.img编辑工具(例如[Android Image Kitchen](https://forum.xda-developers.com/showthread.php?t=2073775))， 

----------

## 步骤 Steps

### 解Bootloader锁 Bootloader Unlocking

参考自[这篇](https://forum.xda-developers.com/lg-v40/development/unlock-lg-v40-via-9008-root-t-mobile-t4042207)或[这篇](https://forum.xda-developers.com/lg-v35/development/bootloader-unlock-root-instruction-t4052145))。简述下来有这几步：

* 安装[QPST & QFIL](https://qpsttool.com/qpst-tool-v2-7-480), 下载LG的845通用的[Firehose](https://url.cn/5hRy6EO)
* 让V35进入Emergency Download (EDL，又称9008)模式
> * 手机连电脑
> * 按住电源键和Vol-重启
> * 熄屏的瞬间按住的键不要松并且开始狂按Vol+
> * 如果手机保持黑屏并且电脑上(设备管理器里)提示插入了新设备(Qualcomm HS-USB QDLoader 9008 (COM#))，则成功进入EDL模式。
> 
* 然后打开QFIL，稍作配置：
> * 点击 Select Port 选择 EDL/9008 模式中的手机
> * 选择 Flat Build
> * Programmer Path 一栏点击Browse， 然后选择刚才下载的firehose文件
> * 右下角 Storage Type 选择 ufs
* 准备就绪后点击顶栏菜单的Tools -> Partition Manager -> OK。如果一切顺利，大概5秒后会弹出一个窗口列出一大堆的分区。如果界面长时间(超过一分钟)卡住不动，建议重启手机重新进入EDL/9008模式。
* 分区列表载入完后可以对分区做清除 (Erase)，提取分区到文件 (Read Data...)，刷入镜像文件 (Load Image...)的操作。在对应分区上右键点击Manage Partiton Data即可。
* 在继续下一步之前，强烈建议先用"Read Data..."对下列分区做个备份，对应的文件会在`C:\Users\你的用户名\AppData\Roaming\Qualcomm\QFIL\COMPORT_##`内。
> * **abl_a** （此为必须，稍后需要还原）
> * abl_b
> * boot_a
> * boot_b
> * laf_a
> * laf_b
* 将[V35工程机的abl](https://url.cn/5Ni6nuO)刷入`abl_a`分区。然后关闭QFIL，否则一定几率下次连接无法读取分区。
* 重启手机（电源键和Vol-），应该会自动进入`fastboot`模式。在电脑端使用`fastboot oem unlock`解锁 (此操作会清除内置存储的数据)
* 再进入EDL/9008模式，从QFIL用刚才备份的文件还原到`abl_a`分区。
* (可选项) 清除`laf_a`分区(建议备份)，这样手机启动时如果试图进入Download模式(按住Vol+)会因为对应的代码被清空而退而进入`fastboot`模式。

### 安装TWRP TWRP Flashing

接下来，你得有V35能用的TWRP。~~虽然直接为V35编译的TWRP是不存在的，但是~~ 花了一天时间踩各种坑修改源码编译了一个[Pie能用的版本](https://github.com/kaneorotar/LG-V35-Tinkering-Instructions/releases)(基于[LG V40的TWRP](https://forum.xda-developers.com/lg-v40/development/twrp-lg-v40-judypn-t3970111)，暂时无法解密原生data)。 

安装方式有两(三)种：

**注意：所有TWRP的安装方式都会替换掉boot分区的ramdisk，意味着Magisk会被清除。只需要在安装好TWRP之后重启进入再刷入Magisk-vXX.X.zip即可。**

1. 如果你已经有Magisk (面具)，可以刷入`twrp-installer-vX.X.X-v35_*.zip` (`*`对应安装的slot，例如a则代表会安装到slot a)
2. 如果没有现成的Magisk，建议先备份boot_a分区内容，然后刷入`twrp-vX.X.X-v35.img`。此镜像的内核是从V350ULM20e提取的，在其他型号上的效果待测。
3. 利用boot.img编辑工具，你可以手动把原版boot提取的zImage和下载的`ramdisk-twrp.cpio`合成为新的含TWRP的boot镜像。具体操作和原理可以参考下面↓

**注意：如果你对slot A/B的概念不熟悉，可以阅读[Project Treble相关的文章](https://sspai.com/post/40890)。** 单来说就是手机内系统需要的分区统统准备了两份，系统更新的时候旧的系统能保留在另一个slot中应急。在一个时间点真正被使用的只有一个slot(通常是a)，所以一般你只需要安装到你实际在用的slot即可。使用`fastboot getvar current-slot`可以获取当前激活的slot的信息。

<details>
  <summary>爱搞机的你也可以选择自己魔改V40的TWRP，点击查看方法 Click to expand!</summary>

  由于硬件的相似度较高，我们可以对V40的TWRP进行魔改适配V35。

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
  > 请先使用QFIL备份原版的boot (boot_a)，然后下载[V40的TWRP](https://forum.xda-developers.com/lg-v40/development/twrp-lg-v40-judypn-t3970111)
  > 
  > 以下的文件需要从原版boot的initrd内提取覆盖到V40 TWRP的initrd内：
  > * etc/recovery.fstab
  > * ~~res/keys~~ (用来校验zip刷机包签名用的，并不需要)
  > * prop.default
  >        
  > 以下的文件需要重命名：
  > * fstab.judypn  ->  fstab.judyp
  > * init.recovery.judypn.rc  ->  init.recovery.judyp.rc
  > * ueventd.judypn.rc  ->  ueventd.judyp.rc
  > 
  > 以下的文件内容需要修改：
  > * 刚更名的initrd\init.recovery.judyp.rc内，搜索"judypn"并替换为"judyp"  
  
</details>

### 解密 Forceencrypt Disabling

本步骤需要使用到[Universal DM-Verity, ForceEncrypt, Disk Quota Disablers](https://forum.xda-developers.com/android/software/universal-dm-verity-forceencrypt-t3817389)

> 先把[zip](https://zackptg5.com/downloads/Disable_Dm-Verity_ForceEncrypt_03.04.2020.zip)下载好放在Micro SD卡/U盘上。内置存储由于还未解密所以TWRP暂时无法读取。(如果链接失效，请访问[作者官网](https://zackptg5.com/android.php#disverfe)找到“Dm-Verity & ForceEncrypt Disabler”进行下载)

然后想办法进TWRP。

> 最保险的方式是root之后用工具软件(例如Magisk Manager，“模块”界面菜单键->重启到 Recovery)重启到recovery。没有的话也可以开启USB调试后连电脑然后使用 `adb reboot recovery`指令(初次连接需要在手机端授权允许，请注意弹窗。如果没有弹，可以尝试切换到“仅充电 (Charging Only)”模式)
> 
> **注意：以下方法建议在`boot_a`和`boot_b`的recovery都替换成了TWRP之后再尝试，否则可能会因为切换slot进入正常的恢复出厂流程，将导致内置存储数据全部丢失。**
> 
> 另一种方式是先按住电源健和Vol-，等重启到LGV35画面亮起来之后松开电源键再按回去。手机会进入一个白屏一段文字两个按钮的恢复出厂的提示界面，选两次Yes后重启就会进入TWRP。

进入TWRP之后便可刷入上述的zip。
> 
> 刷zip的过程中你应该会看到几行红字提示无法卸载vendor分区之类的，只要没有提示zip包失败就没有关系。
> 
> 刷完后**不要直接点重启 (Reboot System)**。先返回到顶层菜单，然后点重启 (Reboot)，最后选Recovery。
> 
> 再次进入TWRP之后，选择清除 (Wipe)，然后点击右侧的格式化Data分区 (Format Data)，输入yes后确认。内置存储 (Internal Storage)会被格式化并且这次格式化之后data分区不会被加密。
> 
> 格式化结束后在TWRP内应该就能正常加载内置存储 (Internal Storage)了。重启回System然后重新配置设备就可以正常使用。

## 备注 Remarks

* 如果你只对slot A进行了上述的操作，在切换到slot B的时候依然会提示解密失败(Decryption Unsuccessful)，解决办法是对slot B重复以上步骤。

* 需要注意的是：在没有瞎折腾过的情况下slot A/B里存着的是LG原版系统的最近两个版本，意味着包括boot在内的分区内容都是不同的，很有可能不通用。
所以**请最好不要把为slot A准备的包含TWRP的boot.img直接刷入slot B的boot分区。**

* 有了TWRP之后，便可以比较方便地刷入GSI。
    <details>
    <summary>GSIs初步体验总结</summary>

    经过测试，GSI大多缺少以下功能：
    > * VoLTE
    > * 分辨率切换
    > * 双击屏幕唤醒
    > * DAC
    > * 屏幕投影
    > * AOD (可以通过使用overlay“修复”, 参考[这里](https://github.com/phhusson/vendor_hardware_overlay/tree/master/LG/G7))
    
    大部分Pie的GSI都能启动。部分存在振动失效问题，可以通过将device fingerprint修改为V35原版修复.

    可用的 Pie GSIs:
    > * Havoc OS
    > * ArrowOS
    > * AOSIP
    > * Lineage
    
    Q GSI 通病:
    > * 大多数不能引导 (可能可以通过selinux permissive补丁修复)
    > * 熄屏后无法再点亮
    > * WiFi无法正常工作 (能扫描，但无法连接)
    
    可进入系统的 Q GSIs:
    > * 部分ErfanGSIs 
    </details>

## 参考 Reference

* https://forum.xda-developers.com/lg-v40/development/unlock-lg-v40-via-9008-root-t-mobile-t4042207
* https://forum.xda-developers.com/lg-v40/development/twrp-lg-v40-judypn-t3970111
* https://bbs.lge.fun/thread-14.htm
* https://g7.lge.fun/guide/twrpboot.html
* https://www.cnblogs.com/stars-one/p/10723732.html
* https://forum.xda-developers.com/android/software/universal-dm-verity-forceencrypt-t3817389
