# 一 介绍

尽管我们可以直接在设备文件上对整个磁盘或分区进行读写操作，但是极其不方便，只有在特定范围内使用该方式，如备份MBR、分区等。而文件系统抽象出了文件的概念，以文件夹分层构建文件的树状命名空间，可以访问的读写数据。文件系统定义了文件和目录的存储结构，致力于高的读写性能、可靠的数据完整性和安全性等。因此会出现很多的文件系统，文件系统一般属于内核实现，但是也存在用户空间内的文件系统。linux使用[VFS][41]（Virtual File System）接口层来同一不同文件系统的调用，屏蔽文件系统的细节，开发者只需要调用VFS的系统调用即可操作数据。磁盘数据访问的模式如下：
![在这里插入图片描述](.File%20System/20190308125912620.png)

# 二 Disk结构

disk有很多中，计算机中光盘、磁带已被弃用，常用硬盘（hard disk）和固态硬盘（SSD）作为存储设备。这里给出硬盘的通用结构图：
![在这里插入图片描述](.File%20System/20190308133338367.png)
可以看出，硬盘拥有多个盘片（platter），每个盘片有两个面，对应两个读写头（head）。读写头固定在arm assembly上，可以伸长或缩短，当盘片围绕轴心（spindle）旋转时即可扫描全部数据。一个head扫面过的圆称为磁道（track），磁道可划分为多个扇区（sector），**扇区是磁盘驱动读取数据的最小单位**（以前一个扇区512KB，现在4K）。所有head
同时扫描过的track组成一个柱面（cylinder）。

磁盘驱动通过[CHS][42]寻址模式，即通过Cylinder-Head-Sector三元组来确定具体的一个扇区位置。但是在操作系统中并不使用CHS寻址模式，而是[LBA][43]（Logical block addressing）寻址模式。LBA简单的认为所有的存储设备都是线性化的，很好的屏蔽了具体存储设备寻址的细节，使得其他结构的存储设备更好的被操作系统使用（如SSD）。在硬盘中，[LBA与CHS的转化][44]由磁盘驱动完成。

>**块（block）是操作系统读取disk数据的最小单位**。为了分离对底层的依赖，操作系统不使用扇区作为读写单位，而是抽象了**块**的概念。一般块大小是扇区的整数倍，可以自己设置，如格式化时可以设置。参考：[电脑中常用的“扇区”、“簇”、“块”、“页”等概念](https://blog.csdn.net/jdbdh/article/details/88350421)

# 三 文件系统实现

每个文件系统的实现都不同，这里介绍linux中常用的实现。

## 文件系统布局

在LBA寻址模式中，disk被视作线性空间，一个**可能的**文件系统布局如下：
![在这里插入图片描述](.File%20System/2019030815472333.png)
MBR占512KB，储存开机引导程序（446字节），目前基本淘汰，使用[GPT][45]。MBR后66个字节（**分区表**）最多只能存4个**主分区**，想要更多的分区，只能新建一个**扩展分区**，扩展分区中可以建立多个**逻辑分区**。分区仅仅只记录分区在disk中的边界，其中一个分区会被标记为**活动分区**，引导程序会执行活动分区中的Boot block。

MBR之后是分区，或者没有建立分区的空余储存空间。每个基本分区前会有一个**Boot block**，活动分区中的Boot block里有能够加载操作系统的程序，而其他分区可能也有（双系统的情况下）。**Superblock**保存了关于文件系统的关键信息，如标识分区文件系统类型的[模数][46]（分区表也有）、块总数等等。**Free space mgmt**记录空闲块位置信息，常用位图（bitmap）或指针来记录。**I-nodes**，文件和目录都被表示为i-node，这里记录所有的i-node。**Root dir**，所有的文件i-node都会被记录在对应目录i-node指向的数据块中，几乎所有的目录也会被父目录记录，除了根目录，因此这里记录根目录的i-node number。*我猜，文件系统挂载后该文件系统的根目录也会有父目录*。最后的**Files and directories**也就是储存i-nodes表中指向的文件或目录数据了，真正数据存放的地方。

[41]:https://en.wikipedia.org/wiki/Virtual_file_system
[42]:https://en.wikipedia.org/wiki/Cylinder-head-sector
[43]:https://en.wikipedia.org/wiki/Logical_block_addressing
[44]:https://en.wikipedia.org/wiki/Logical_block_addressing#CHS_conversion
[45]:https://blog.csdn.net/jdbdh/article/details/87189717#39_MBRGPT_238
[46]:https://en.wikipedia.org/wiki/Partition_type

## 文件实现

文件是操作系统中一个抽象的概念，就像进程、地址空间一样。文件的实现就是记录哪个块地址属于哪个文件。一般有三种实现方式：Contiguous Allocation、Linked List Allocation和I-nodes，linux中常用的是I-nodes，因此这里介绍I-nodes。一个i-node例子如下：
![在这里插入图片描述](.File%20System/20190308165217703.png)
over。。。

>注意
>
>* i-node存在于i-nodes表中，i-node通过指针指向Files and directories中的块数据。
>* 文件的属性存在于i-node的头部

## 目录实现

目录本身也是被表示为I-node，i-node指向的数据块，仅仅存储着目录下文件或子目录的名字和i-node number，被称为entry。一般目录都会存在指向自己或父目录的entry，如`.`和`..`，除了根目录外（只有`.`）。因为文件名是可变的，因此有两种存储文件名的方法：固定最大长度或可变长度：
![在这里插入图片描述](.File%20System/20190308172244425.png)

>注意，我认为上图不太对，因为entry不会存储文件属性的，文件属性只会存在于它自己的i-node上，避免增加冗余。其他的上同。

# 四 文件系统类型

列举常用文件系统：

* `ext*`：linux本土文件系统，目前使用ext4。ext2改自unix的传统文件系统，ext3增加了日记，ext4支持更大的文件、更多的目录。
* `iso9660`：CD-ROM使用的文件系统，只可读。
* `FAT filesystems`（msdos,vfat,umsdos）:属于windows的文件系统。vfat更好的被linux支持。
* `HFS+`(hfsplus)：属于苹果Macintosh使用的系统
* `NTFS`：windows目前使用的文件系统

然后，还有一些特殊的文件系统，它并不表示实际的物理存储设备，而是用作系统接口，获取系统信息的：

* `proc`： Mounted on `/proc`. The name proc is actually an abbreviation for process. Each numbered directory inside /proc is actually the process ID of a current process on the system; the files in those directories represent various aspects of the processes. The file /proc/self represents the current process. The Linux proc filesystem includes a great deal of additional kernel and hardware information in files like /proc/cpuinfo. (There has been a push to move information unrelated to processes out of /proc and into /sys.)就是和进程相关的数据
* `sysfs`：Mounted on `/sys`. 和设备相关的数据，见3.1小节
* `tmpfs`： Mounted on `/run` and other locations. With tmpfs, you can use your physical memory and swap space as temporary storage. For example, you can mount tmpfs where you like, using the size and nr_blocks long options to control the maximum size. However, be careful not to constantly pour things into a tmpfs because your system will eventually run out of memory and programs will start to crash. (For years, Sun Microsystems used a version of tmpfs for /tmp that caused problems on long-running systems.)就是将内存（包括虚拟内存）当做文件系统。

# 五 常见操作

## 操作分区

有四种常用的分区操作工具：

* **parted** ：A text-based tool that supports both MBR and GPT.
* **gparted** ：A graphical version of parted.
* **fdisk** ：The traditional text-based Linux disk partitioning tool. fdisk does not support GPT.
* **gdisk** ：A version of fdisk that supports GPT but not MBR.

因为parted同时支持MBR和GPT格式的分区表创建，因此建议使用parted。鉴于`man parted`中介绍很详细并且我基本使用不到该命令，因此不深究。。

需要稍微知道的一些知识点：

* `parted -l`：打印所有块设备的分区表
* MBR格式的分区只能建立四个主分区（包括扩张分区），扩张分区上可以建立多个逻辑分区。GPT无限制。

## 创建文件系统

分区创建后，还需在分区上创建文件系统，类似于windows上的格式化：

>方式一
>mkfs [-t  type] device
>在设备文件device上创建type类型的文件系统，如果type没有给出则创建ext2

>方式二
>mkfs.type device 
>解释同上

通过`ll /sbin/mkfs.*`可以列出所有可用文件系统。

## 挂载文件系统

文件系统只有被挂载后才能使用。将文件系统挂载到linux目录下的一个文件夹后，可以通过该文件夹访问被挂载的文件系统。

>mount [-t type] device dir
>将设备文件（含文件系统）挂载到目录dir上

取消挂载：

>umount {dir|device}

mount无参数时列出所有挂载的文件系统：

>mount
>sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
>/dev/vda1 on / type ext4 (rw,relatime,data=ordered)
>....
>第一项为特殊文件系统名或设备文件（含文件系统），on后为挂载点，type后文件系统类型，最后为挂载选项。

### 使用UUID

使用设备名挂载有一定缺陷，因为设备命名取决于内核找到设备的顺序，因此设备名可能会改变。但是创建文件系统时，mkfs会同时为文件系统创建一个唯一的**UUID**，可以通过uuid来替代设备文件挂载。

使用`blkid`获得块设备（含文件系统）的UUID，然后挂载：

>blkid
>/dev/sdf2: UUID="a9011c2b-1c03-4288-b3fe-8ba961ab0898" TYPE="ext4"
>...

> mount UUID=a9011c2b-1c03-4288-b3fe-8ba961ab0898 /home/extra

### 自动挂载

在自启期间内，linux会读取`/etc/fstab`的配置，然后自动挂载（貌似是执行了`mount -a`）。只要按照它的格式写就配置自动挂载自己的文件系统。`/etc/fstab`中的内容如下：

```unknow
proc /proc proc nodev,noexec,nosuid 0 0
UUID=70ccd6e7-6ae6-44f6-812c-51aab8036d29 / ext4 errors=remount-ro 0 1
UUID=592dcfd1-58da-4769-9ea8-5f412a896980 none swap sw 0 0
/dev/sr0 /cdrom iso9660 ro,user,nosuid,noauto 0 0
```

第一项为设备文件（含文件系统）或其UUID或特殊文件名；第二项为挂载点（对于swap填none）；第三项为文件系统类型；第四项为逗号分隔的挂载选项；第五项和备份信息有关，填0禁止就对了；第六项为是否自启时用fsck检查文件系统完整性，一般自己挂载的都填0。

使用`mount -a`几乎可以挂载`/etc/fstab`中所有的文件系统；或者使用`mount /dir`，会挂载`fstab`文件中对应的挂载点的文件系统。

### mount选项

无论手动挂载和自动挂载，都会用到`mount`命令的选项。mount选项（也即参数）分为两类，短选项，如`-r`；长选项，放在`-o`后面，逗号分隔，如`-o ro,conv=auto`。而长选项又分为独立与文件系统的选项，和特定于某一具体文件系统的选项。自动挂载只能使用长选线吧。

这里只介绍文件系统独立的选项，其中一些只适用于自动挂载：

* `ro`：Mount the filesystem read-only，同`-r`
* `rw`：Mount the filesystem read-write，同`-w`
* `exec`,`noexec`：Enables or disables execution of programs on the filesystem
* `suid`,`nosuid`：Enables or disables setuid programs
* `dev`,`nodev`：是否解析设备文件
* `sync`,`async`：对文件系统的读写为同步还是异步的，一般为异步的，效率会高些
* `user`,`users`,`nouser`：user允许普通用户挂载，同时只有该用户可以卸载；users允许任意挂载和卸载；nouser只允许普通用户挂载。`user`,`users`都隐含着`noexec`,`nosuid`和`nodev`，可以被后面的选项覆盖（仅适用自动挂载）
* `noauto`：只能显示挂载，即`mount -a`不起作用。（仅适用自动挂载）
* `defualts`：默认使用`rw`, `suid`,`dev`, `exec`, `auto`, `nouser`, 和 `async`，其他特定于文件系统的选项默认值取决于该文件系统。最常用！（仅适用自动挂载）

## swap space

并不是所有的分区都含有一个文件系统，swap space（置换空间，又称虚拟内存）不需要。置换空间将磁盘空间当做RAM，用于将不活动的程序或活动程序的部分内存放入置换空间内，从而允许它运行更多的活动程序。通过`free`可以查看物理和置换空间的使用情况。

### 分区作为swap space

1. 存在一个分区，最好空的
2. 将置换标志写入到分区：`mkswap device`
3. 手动注册到内核中：`swapon device`

> 上述重启配置会消失

4. 持久化配置

   自启时自动注册，在`/etc/fstab`中添加一行：

   ```
   /dev/sda5 none swap sw 0 0
   ```

   > 第一项是分区文件名

### 文件作为swap space

与上面同理：

1. 创建文件

   ```shell
   dd if=/dev/zero of=swap_file bs=1024k count=num_mb
   ```

   或

   ```shell
   fallocate -l 8G swap_file
   ```

2. 写入置换标志

   ```shell
   mkswap swap_file
   ```

3. 注册

   ```shell
   swapon swap_file
   ```

4. 查看已注册的虚拟内存

   ```shell
   swapon --show
   ```

> 上述配置重启后会小时

5. 持久化配置

   自启时自动注册，在`/etc/fstab`中添加一行：

   ```
   path/to/swapfile none swap sw 0 0
   ```

   > 第一项是swap文件名

为了禁止使用置换空间，使用`swapoff`，具体方法请查询man手册。

> 新增虚拟内存, 即多挂载一个文件
>
> ```shell
>fallocate -l 16G swapfile
> mkswap swapfile
> swapon swapfile
> ```

## 其他命令

### 查看磁盘, 分区, 文件系统

https://unix.stackexchange.com/questions/157154/how-to-list-disks-partitions-and-filesystems-in-linux

### sync

对disk的读写是异步的，写文件时不会立刻刷新到disk中，但可以使用sync手动刷新缓存。

### df

默认输出已挂载文件系统的磁盘使用情况。其中文件系统已使用量不包含大约5%的保留空间。

### du

* 介绍

  默认打印当前目录及其所有子目录总大小, 单位KB

* 使用

  ```shell
  du [FILE]...
  ```

  FILE默认指向当前目录，如果指定文件，则仅打印该文件大小

  * `-a`:同时也打印目录内的文件的大小
  * `-s`:仅打印FILE的总大小

* 例子

  ```shell
  # 将打印当前目录中所有文件和文件夹大小（不打印文件夹中的内容）
  du -sh *
  ```

### fsck

检查文件系统是否有不正确的地方，并给出解决方案。不怎么会用。

### free

打印物理和置换内存的使用情况。

### ls -i 和 stat

`ls -i`同时打印inode number；stat打印文件的相关属性。

### fallocate

创建一个指定大小的文件

```shell
fallocate -l 8G swapfile
```

# 参考

* [File Systems](https://download.csdn.net/download/jdbdh/11006201)
* [电脑中常用的“扇区”、“簇”、“块”、“页”等概念](https://blog.csdn.net/jdbdh/article/details/88350421)