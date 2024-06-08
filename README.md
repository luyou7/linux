树莓派/RaspberryPi 内核编译
1.获取所需源码
1)下载地址：

官方网址：https://github.com/raspberrypi

上面列出了树莓派所有的开源软件：

linux:内核源码
tools:编译内核和其他源码所需的工具——交叉编译器等
我们只需要以上两个文件即可，下面的工程可以了解一下

firmware:树莓派的交叉编译好的二进制内核、模块、库、bootloader
documentation:树莓派离线帮助文档，教你如何使用、部署树莓派（树莓派官方使用教程）
userland：arm端用户空间的一些应用库的源码——vc视频硬浮点、EGL、mmal、openVG等
hats：Hardware Attached on Top，树莓派 B+型板子的扩展板资料
maynard：一个gtk写成的桌面环境
scratch：一个简易、可视化编程环境
noobs:一个树莓派镜像管理工具，他可以让你在一个树莓派上部署多个镜像
weston：一个应用程序
target_fs：树莓派最小文件系统，使用busybox制作
quake3：雷神之锤3有线开发源码firmwareb
2)下载方法：

1
2
git clone https://github.com/raspberrypi/tools
git clone --depth=1 https://github.com/raspberrypi/linux
 具体见：http://www.cnblogs.com/qiengo/p/5888559.html

 

2.配置交叉编译环境
　　root@ubuntu:......./tools/arm-bcm2708# ls

1
2
3
4
5
arm-bcm2708hardfp-linux-gnueabi
arm-bcm2708-linux-gnueabi
arm-rpi-4.9.3-linux-gnueabihf
gcc-linaro-arm-linux-gnueabihf-raspbian
gcc-linaro-arm-linux-gnueabihf-raspbian-x64
 　　32系统将如下路径加入环境变量：

/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian

/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin

64系统将如下路径加入环境变量：

/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin

修改/etc/profile文件即可，完成后source /etc/profile,然后终端输入arm-linux后双击Tab有指令提示说明添加成功

3.编译、提取内核及其模块
1)配置内核，配置makefile的ARCH类型和编译器路径

可以直接修改内核根目录的Makefile文件，修改下面的这两行

　　ARCH　　？=$(SUBARCH)

　　CROSS_COMPILE ?=$(CONFIG_CROSS_COMPILE:"%"=%)

但是从这两行上面的注释可以看出，可以直接使用make指令设置这两个参数，make ARCH =arm CROSS_COMPILE=arm-linux- ......

　　执行find ./ -name "*bcm*defconfig*" 查找下对应的默认配置，只保留arm下的结果：

1
2
3
4
./arch/arm/configs/bcm2835_defconfig
./arch/arm/configs/bcmrpi_defconfig
./arch/arm/configs/bcm2709_defconfig
./arch/arm/configs/bcm_defconfig
 　　Pi 1 使用bcmrpi_defconfig

　　 Pi 2/3 使用bcm2709_defconfig

执行指令

$ cd ....../linux-rpi-4.4.y
$ KERNEL=kernel7
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig
 配置内核模块，执行

　　make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

　　如下图所示，顶部显示arm，说明参数设置成功

 

 

2）编译内核镜像

$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs

如果是多处理系统可以添加选项-jn ,n为数字，表示多处理器的数量*1.5。可以加快编译速度
$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 zImage modules dtbs

4.升级内核及文件系统
将树莓派的SD卡插在Linux系统电脑上，最好直接使用读卡器，使用lsblk指令对比插入前后的变化，可以sd中的两个分区如下:

1
lsblk
 

mmcblk0p1是FAT（boot）分区

mmcblk0p2是ext4文件系统（root）分区

挂载SD卡分区：

1
2
3
4
mkdir mnt/fat32
mkdir mnt/ext4
sudo mount /dev/mmcblk0p1 mnt/fat32
sudo mount /dev/mmcblk0p2 mnt/ext4
 安装modules：

1
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=mnt/ext4 modules_install
如果把INSTALL_MOD_PATH设为本地目录可以提取出对应的modules，最后，把kernel and Device Tree blobs复制到SD卡：

1
2
3
4
5
6
7
sudo cp mnt/fat32/$KERNEL.img mnt/fat32/$KERNEL-backup.img       //备份原先的img文件
sudo scripts/mkknlimg arch/arm/boot/zImage mnt/fat32/$KERNEL.img //将zImage格式转成树莓派需要的img格式，并复制到SD卡
sudo cp arch/arm/boot/dts/*.dtb mnt/fat32/
sudo cp arch/arm/boot/dts/overlays/*.dtb* mnt/fat32/overlays/
sudo cp arch/arm/boot/dts/overlays/README mnt/fat32/overlays/
sudo umount mnt/fat32
sudo umount mnt/ext4
升级内核的另一个办法是将img文件复制到相同目录下，使用不同的文件名，如 kernel-myconfig.img，然后修改boot目录下的config.txt文件，加入：

1
kernel=kernel-myconfig.img
 最后，将SD卡插入树莓派启动。


https://www.raspberrypi.com/documentation/computers/linux_kernel.html

树莓派/RaspberryPi 内核源码下载
树莓派的源码有两种下载方式：压缩包下载和git clone指令下载。

1.压缩包下载

　　选择对应分支，点击Github界面的 下载按钮即可，如下图：

　　

　　测试发现，同样的分支，用压缩包方式下载后编译会出错，而用git clone 方式下载编译正常，因此推荐使用git clone方式

2.git clone下载

 1)下载master分支

1
git clone --depth=1 https://github.com/raspberrypi/linux
　　git clone默认下载master分支，所以上述操作只会下载master分支，如果要下载其他分支，见下文。

 2)下载指定分支

1
git clone https://github.com/raspberrypi/linux.git
 　　该操作会把整个Git 项目仓库克隆到本地，并默认处于master分支下，下载完成后使用ls -al指令查看，可以看到两个文件：

　　　　.git   git项目仓库

　　　　linux 当前项目分支，默认为master分支

 cd linux进入linux目录，查看该目录下的Makefile文件中顶端的内核信息：

1
2
3
4
5
VERSION = 4
PATCHLEVEL = 4
SUBLEVEL = 21
EXTRAVERSION =
NAME = Blurry Fish Butt
 内核版本4.4.21，可知是master分支，在该目录下执行git branch -a查看所有分支

1
git branch -a
 结果如下:



其中，remotes下的为远程分支，其余是本地分支，*开始的为当前分支

git branch用法如下

1
2
3
4
5
6
7
git branch     列出本地已经存在的分支，并且在当前分支的前面加“*”号标记
git branch -r  列出远程分支
git branch -a  列出本地和远程分支
git branch name 创建新的本地分支，但不进行分支切换
git branch -m | -M oldbranch newbranch 重命名分支，如果newbranch名字分支已经存在，则需要使用-M强制重命名，否则，使用-m进行重命名
git branch -d | -D branchname 删除branchname分支
git branch -d -r branchname 删除远程branchname分支
  如果要下载rpi-4.1.y分支，执行

1
git checkout -b rpi-4.1.y origin/rpi-4.1.y
 checkout远程的rpi-4.1.y分支，在本地起名为rpi-4.1.y分支，并切换到本地的rpi-4.1.y分支,该操作是从.git目录中提取，而不是通过网络远程下载。

如果要在本地的不同分支见切换，使用

1
git checkout rpi-4.1.y
 如果当前分支有修改，可以使用git reset重置，或者使用git stash保存修改。

1
git reset --hard

Linux kernel
============

There are several guides for kernel developers and users. These guides can
be rendered in a number of formats, like HTML and PDF. Please read
Documentation/admin-guide/README.rst first.

In order to build the documentation, use ``make htmldocs`` or
``make pdfdocs``.  The formatted documentation can also be read online at:

    https://www.kernel.org/doc/html/latest/

There are various text files in the Documentation/ subdirectory,
several of them using the Restructured Text markup notation.

Please read the Documentation/process/changes.rst file, as it contains the
requirements for building and running the kernel, and information about
the problems which may result by upgrading your kernel.

Build status for rpi-5.15.y:
[![Pi kernel build tests](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml/badge.svg?branch=rpi-5.15.y)](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml)
[![dtoverlaycheck](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml/badge.svg?branch=rpi-5.15.y)](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml)

Build status for rpi-6.1.y:
[![Pi kernel build tests](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml/badge.svg?branch=rpi-6.1.y)](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml)
[![dtoverlaycheck](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml/badge.svg?branch=rpi-6.1.y)](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml)

Build status for rpi-6.6.y:
[![Pi kernel build tests](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml/badge.svg?branch=rpi-6.6.y)](https://github.com/raspberrypi/linux/actions/workflows/kernel-build.yml)
[![dtoverlaycheck](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml/badge.svg?branch=rpi-6.6.y)](https://github.com/raspberrypi/linux/actions/workflows/dtoverlaycheck.yml)
