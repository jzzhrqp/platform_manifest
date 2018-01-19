# Build Android for Raspberry Pi3
本文引用了https://github.com/android-rpi 和 https://github.com/tab-pi ，
根据这两个项目的指引说明，结合国内资源下载，在这两个项目的基础上进行了适当改动。

# 树莓派3 android7.1-tv 源码编译
## 下载和编译环境配置
- 第一步，下载repo
```
git clone https://gerrit-google.tuna.tsinghua.edu.cn/git-repo
```

然后将git-repo目录下的repo加入到环境变量
```
vim ~/.bashrc
最后一行加入
export PATH="~/git-repo:$PATH"
保存，然后重载环境变量
. ~/.bashrc
```

- 第二步，下载源码
这里，源代码的来源，共来自三个地方，AOSP 部分，来自中国科学技术大
学（详细：https://lug.ustc.edu.cn/wiki/mirrors/help/aosp），
树莓派android内核和系统修改，共引用自 https://github.com/android-rpi 和  https://github.com/tab-pi，
下载源代码的前提是你能用ssh联接到github.
下载：

```
$ mkdir AOSP-work
$ cd AOSP-work 
$ repo init -u ssh://github.com/jzzhrqp/platform_manifest -b nougat
$ repo sync
```

- 第三步，配置编译环境
关于编译环境的配置，详细的介绍请看谷歌公司在中国的AOSP网站(https://source.android.google.cn/setup/initializing)

先，安装下面这三样：

-> Python 2.6 - 2.7从python.orghttps://www.python.org/downloads/()
-> GNU Make 3.81 - 3.82来自gnu.org(http://ftp.gnu.org/gnu/make/)
-> Git 1.7或更新从git-scm.com(https://git-scm.com/download)

然后：

1. 安装openjdk 8
'''
$ sudo apt-get update
$ sudo apt-get install openjdk-8-jdk
'''


2. 安装所需的软件包
```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g ++  -  multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa- dev libxml2-utils xsltproc unzip
```

3. 配置环境变量

高度并行的构建过程可能会超出此限制。
要增加上限，请将以下几行添加到您的~/.bashrc：
```
＃设置打开文件的数量为1024 
ulimit -S -n 1024
```

设置ccache
您可以选择告诉构建使用ccache编译工具，它是C和C ++的编译器缓存，可以帮助构建更快。这对构建服务器和其他大批量生产环境特别有用。Ccache充当一个编译器缓存，可以用来加速重建。如果make clean经常使用，或者经常在不同的构建产品之间进行切换，这种方法效果很好。
要使用ccache，请在源代码树的根目录中输入以下命令：
```
export USE_CCACHE=1
export CCACHE_DIR=/<path_of_your_choice>/.ccache
prebuilts/misc/linux-x86/ccache/ccache -M 50G
```
把下面这一行加入~/.bashrc：
```
export USE_CCACHE=1
```


-------------------------

当一切都准备好了，接下来就按照下面的说明开始编译吧。


## Build Kernel
 * Install gcc-arm-linux-gnueabihf (For Ubuntu: $ sudo apt install gcc-arm-linux-gnueabihf)
 * $ cd kernel/rpi
 * $ ARCH=arm scripts/kconfig/merge_config.sh arch/arm/configs/bcm2709_defconfig android/configs/android-base.cfg android/configs/android-recommended.cfg
 * $ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make zImage
 * $ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make dtbs

## Install python mako module
  sudo apt-get install python-mako

## Build Android source
 * $ source build/envsetup.sh
 * $ lunch rpi3-eng
 * $ make ramdisk systemimage
 
## Help for build failure :
   https://github.com/tab-pi/device_brcm_rpi3/wiki/Build-Errors

## Prepare SD card
 ### Partitions of the card should be set-up like following:
  1. p1 512MB for BOOT : Do fdisk : W95 FAT32(LBA) & Bootable, mkfs.vfat
  2. p2 1024MB for /system : Do fdisk, new primary partition, mkfs.ext4
  3. p3 512MB for /cache  : Do fdisk, mkfs.ext4
  4. p4 remainings for /data : Do fdisk, mkfs.ex4
  5. Set volume label for each partition - system, cache, userdata
  : use -L option of mkfs.ext4, e2label command, or -n option of mkfs.vfat
 
## Write system partition
  * $ cd out/target/product/rpi3
  * $ sudo dd if=system.img of=/dev/<p2> bs=1M
  
## Copy kernel & ramdisk to BOOT partition
  * device/brcm/rpi3/boot/* to p1:/
  * kernel/rpi/arch/arm/boot/zImage to p1:/
  * kernel/rpi/arch/arm/boot/dts/bcm2710-rpi-3-b.dtb to p1:/
  * kernel/rpi/arch/arm/boot/dts/overlays/vc4-kms-v3d.dtbo to p1:/overlays/vc4-kms-v3d.dtbo
  * out/target/product/rpi3/ramdisk.img to p1:/

## HDMI_MODE : If DVI monitor does not work, try followings for p1:/config.txt
hdmi_group=2
  
hdmi_mode=85

### How to set up Android-TV launcher :
  https://github.com/tab-pi/device_brcm_rpi3/wiki#how-to-apply-android-tv-leanback-launcher

Graphics HAL of this build : https://github.com/anholt/mesa/wiki/VC4
