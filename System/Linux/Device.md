# 介绍

在linux中，一切皆为文件，比如设备、网络、硬盘或其分区等等，都在表示	为文件。设备被表示为**设备文件（Device Files）**（也被称为**device node**），是一个与**设备驱动（device driver）** 交互的接口。在设备驱动被加载进内存后，如果有对应设备的插入或拔出，linux内核会发送事件**uevent**信息给用户空间中的守护进程（**daemon**）**udev**，udev收到事件后还会将它发送给其他进程。udev为了维护`/dev`目录，会动态创建或删除device node。udev的功能依赖于`sysfs`文件系统，使得udev能够在用户空间中获得更多关于设备的信息。

设备文件分为块设备和字符设备，他们的区别在于读写方式的不同。设备文件也可以对应虚拟设备，只要有对应的驱动程序。*貌似管道文件也是设备文件？*

# 基础

## /dev与/sys

设备文件都被存于`/dev`下，为了通过对设备文件操作直接作用到设备中，`/dev`会被挂载到专门的文件系统`devtmpfs`中（不要与`tmpfs`混淆）。

`ll /dev`会显示所有的设备文件，权限前的第一个字符表示文件类型，`b`表示块设备，`c`表示字符设备等等。日期前的两个数字分别表示**主和次设备号**，相似的设备通常拥有相同的主设备号，如sda3和sdb1。通过主次设备号用于标识设备。

`/dev`下的设备文件是用于操作设备的接口，但它不提供更多的信息，如设备的属性等。这些信息由`/sys`目录下对应的文件提供，如`/dev/vda`对应`/sys/devices/pci0000:00/0000:00:05.0/virtio2/block/vda`，这是一个**目录**，目录下的各种文件包含设备的详细信息，如dev文件含有主次设备号。

那么如何从`/dev`中找到`/sys`下对应的目录呢？

* 因为设备文件是通过udev创建的，因此可以通过它的管理工具`udevadm`来获得设备文件详细的信息。

  ```bash
  udevadm info --query=all --name=/dev/vda
  ```

* `/sys/block`目录下会含有系统中可用的块设备目录的符号链接。同样，`/sys/dev`含有主次设备号与对应目录的关系。

`/sys`目录下含有一些关于内核子系统、硬件设备等信息，这些信息由[虚拟文件][3]（即访问时临时计算出数据的文件）提供，需要挂载到专门的文件系统`sysfs`中。

[3]:https://en.wikipedia.org/wiki/Virtual_file

## 命名约定

系统加入设备后，udev给设备命名的规范：

* `SCSI driver`, also used by libATA (modern PATA/SATA driver), USB, IEEE 1394, etc.即使底层使用的设备和驱动不一样，但是这些设备最顶层的驱动还是scsi。然而真正的SCSI设备已经被淘汰了，但SCSI协议由于它的适用性被保留了下来。
  * `sd`: mass-storage driver
    * `sda`: first registered device
      * `sda4`: last partition on this disk (example)
      * `sda6`: second logical drive in the extended partition (example)
    * `sdb`, `sdc`, etc.: second, third, etc. registered devices
  * `ses`: Enclosure driver
  * `sg`: generic SCSI layer
  * `sr`: “ROM” driver (data-oriented optical disc drives; scd is just a secondary alias)
  * `st`: magnetic tape driver
* `hd`: (“classic”) **IDE driver** (previously used for ATA hard disk drive, ATAPI optical disc drives, etc.)老式PATA硬盘使用的文件名，已经很少见了，因此不展开讲解。
* `fd`: (platform) floppy disks（软盘）
* 终端`tty*`,`pts/*`：都代表终端（**terminals**），不过tty表示本地终端，或是真实硬件或是内核模拟出来的；pts表示伪终端，通过用户空间程序模拟出来的。
* 串行端口终端`ttyS*`：相当于windows上的串口。
* 并行端口`lp*`
* 等等

## terminal、console、shell

### 介绍

**terminal**（终端）和**console**（控制台）都是字符设备，功能上并没有什么区别，都是与你的显示器和键盘交互的。启动linux但没有启动xwindow的屏幕就是console，而进入xwindow后打开的terminal就是terminal，通过ssh连接打开的终端也是terminal。

[shell][4]不是一种设备，而是程序，一个与操作系统服务交互的用户接口。因此shell分两种：使用命令行接口（**command-line interface**）的Text(CLI) shell，和图形用户接口（**graphical user interface**）的Graphical shell。CLI shell提供命令解析的功能，通过命令与系统交互，gui shell提供图形化的操作与系统交互。

>不要以为gui程序只能在gui shell中运行，几乎只要有diplay server和它的客户端client就能运行，即使在CLI shell下。参考：[ssh 第四章][5]或尝试运行该命令：`xinit /usr/bin/firefox`

### 与设备文件的关系

console在很多linux发行版中被实现为7个virtual console，分别用设备文件`tty1`、`tty2`...`tty7`表示。一般桌面环境运行在第7个virtual console，用`tty7`表示。通过`ctrl+alt+F*`切换。

terminal则被表示为除了`tty1`...`tty7`之外的`tty*`或`pts/*`，`pts/*`是伪终端的设备文件，GUI shell打开的终端和ssh连接打开的终端都是伪终端。剩下的`tty*`是干什么用的呢？连接什么终端呢？我不知道。。

特殊的终端设备文件：

* `tty`：是当前程序的控制终端的引用，可以是`tty*`，`ttyS*`，`pts/*`等等。通过`ps`可以看到程序对应的具体终端。对于控制终端，参考[零碎知识3.8小节][6]
* `tty0`：是当前（或前台）virtual console的引用，可以是`tty1...`或`tty7`
* `console`：系统控制台（system console），默认指向`tty0`。这也是为什么开机后会输出很多系统信息的原因。

## 特殊设备文件

除了3.3.1中的特殊终端文件外，还有一些特殊的伪终端文件：

* `/dev/null` – accepts and discards all input; produces no output (always returns an end-of-file indication on a read)
* `/dev/zero` – accepts and discards all input; produces a continuous stream of NUL (zero value) bytes
* `/dev/full` – produces a continuous stream of NUL (zero value) bytes when read, and returns a "disk full" message when written to
* `/dev/random` and `/dev/urandom` – they produce a variable-length stream of pseudo-random numbers.

## 例子

命令`/etc/resolv.conf`的一个可能的数据流程如下：

>hard disk-->SATA driver-->SCSI driver-->`/dev/sda1`(分区设备文件)-->`cat /etc/resolv.conf`(命令或进程)-->`/dev/pts/1`(终端设备文件)-->终端设备驱动-->终端

----------

命令`cat /etc/resolv.conf > /etc/resolv.bak`的一个可能的数据流程如下：

>hard disk-->SATA driver-->SCSI driver-->`/dev/sda1`(分区设备文件)-->`cat /etc/resolv.conf`(命令或进程)-->`/dev/sda1`（分区设备文件）-->SCSI driver-->SATA driver-->hard disk

可以看出，重定向的原理就是重定向到了不同的设备文件。

-----------

打开virtual console1和virtual console2（ctrl+alt+F2），在console2上输入`echo "hello world" > /dev/tty1`，然后会发现console1出现了`hello world`。

# 相关命令

## dd

> 若仅仅只是创建一个文件, `fallocate`比`dd`更安全与快速

dd从文件或stdin读取数据，并写入数据到文件或stdout，这个过程中可能会做一些编码转换。

>* `if=file`：输入文件，默认stdin
>* `of=file`：输出文件，默认stdout
>* `bs=size`：一次读入和写入的字节数，默认512。可带有单位，如`b`(512),`K`(1024),`M`(1024^2^),`G`(1024^3^)
>* `ibs=size,obs=size`：分别是一次性读入或写入的字节数。如果相同，则使用`bs`
>* `count=num`：拷贝输入块的次数，默认一致拷贝直到文件尾。
>* `skip=num`：跳过输入的num个块，才开始读。
>* `conv=CONV`：按照CONV对数据进行转换

例子:

* 备份MBR

  ```bash
  dd if=/dev/sda of=/tmp/myMBR.bak bs=512 count=1
  ```

* 制作光盘镜像

  ```bash
  dd if=/dev/sda of=~/disk1.img conv=noerror,sync
  ```

  其中，noerror表示读取出错后继续；sync表示，如果读取字节不够，填充NULL（即数值0）。一般情况下sync无用，只有配合noerror在读取失败时才会填充。这样保证及时镜像文件部分损坏，但可用。

## mknod

当设备文件意外丢失或想要创建命名的管道文件时，可以使用mknod创建。但需要事先知道主次设备号。如：

```bash
 mknod /dev/sda1 b 8 2
```

b为块状设备，主次设备号分别为8，2。

## udevadm

udevadm用于管理守护进程udev，因此可以查出设备文件的全部信息：

```bash
udevadm info --query=all –-name=/dev/sda
```

## lsscsi

列出所有的scsi设备

## lsblk

列出所有的块设备（除了RAM disk）。实现：从sysfs文件系统中得到数据。

## lspci

列出所有pci设备的信息, 显卡就是该类设备.

* 查看集显

  ```bash
  lspci|grep -i vga
  ```

* 查看独显

  ```bash
  lspci| grep -i nvidia
  ```

  

## 实战

### 找出当前使用显卡

* 方法一: 使用`glxinfo`命令

  1. 一般不存在, 先安装该命令

     ```bash
     sudo apt install mesa-utils
     ```

  2. 输入

     ```bash
     $ glxinfo|egrep "OpenGL vendor|OpenGL renderer"
     OpenGL vendor string: Intel Open Source Technology Center
     OpenGL renderer string: Mesa DRI Intel(R) Sandybridge Mobile
     ```

     表明正在使用Intel集显

  3. 如果使用的是bumblebee方案, 你可强制它使用独显

     ```bash
     $ optirun glxinfo|egrep "OpenGL vendor|OpenGL renderer"
     OpenGL vendor string: NVIDIA Corporation
     OpenGL renderer string: GeForce GT 555M/PCIe/SSE2
     ```

* 方法二: 使用`lspci`命令, 活动的显卡会显示`[VGA controller]`信息

  ```bash
  $ lspci -v|grep '\[VGA controller\]'
  00:02.0 VGA compatible controller: Intel Corporation Device 3ea0 (rev 02) (prog-if 00 [VGA controller])
  ```

> 参考:[How to check which GPU is active in Linux?](https://unix.stackexchange.com/questions/16407/how-to-check-which-gpu-is-active-in-linux)

#  参考

* [Managing devices in Linux](https://opensource.com/article/16/11/managing-devices-linux)
* [Device file wiki](https://en.wikipedia.org/wiki/Device_file)
* [Udev wiki](https://en.wikipedia.org/wiki/Udev#Overview)
* [终端，Shell，“tty”和控制台（console）有什么区别？](https://www.zhihu.com/question/21711307/answer/124100057)
* [Why are there so many /dev/tty in Linux?](https://superuser.com/questions/449781/why-are-there-so-many-dev-tty-in-linux)
* [Linux系统dev/目录下的tty](https://blog.csdn.net/a746742897/article/details/52302394)
* [What is the difference between Terminal, Console, Shell, and Command Line?](https://askubuntu.com/questions/506510/what-is-the-difference-between-terminal-console-shell-and-command-line)
* [What is the exact difference between a 'terminal', a 'shell', a 'tty' and a 'console'?](https://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con)
* [shell wiki](https://en.wikipedia.org/wiki/Shell_(computing))
* [virtual console wiki](https://en.wikipedia.org/wiki/Virtual_console)

[4]:https://en.wikipedia.org/wiki/Shell_(computing)
[5]:https://blog.csdn.net/jdbdh/article/details/87383172#_GUIx11_137
[6]:https://blog.csdn.net/jdbdh/article/details/87189717#38_Linux__207

# 