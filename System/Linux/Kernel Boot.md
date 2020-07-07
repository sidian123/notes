# 介绍

linux启动过程涉及很多阶段，如固件初始化，引导程序执行，内核镜像加载和启动，各种守护进程和脚本的运行。每一步都有不同的不同的方法。

开机后，会执行主板上固件（firmware）里的程序，它会枚举硬件并正确的初始化，然后从配置的启动设备中加载引导程序。

>固件分为两种：BIOS和UEFI。BIOS会从MBR中读取程序（引导程序）并运行在实地址模式。
>而UEFI更为强大，几乎可以完成内核的工作。使用特定的EFI分区(或ESP)，避免了MBR512KB的限制。可以直接装入内核并运行，但一般EFi中储存引导程序。但UEFI也会兼容BIOS。UEFI与GPT分区表的组合很常见。

引导程序很多种，如LILO、GRUB2、SYSLINUX、Loadlin等等，但在linux中GRUB2最为常用。GRUB2能够识别一些文件系统，因此能够找到内核并加载。它会先加载自己的配置文件，然后列出启动菜单项供用户选择系统启动。之后加载内核。GRUB2功能很强大，可以链式加载（chain-loading）其他引导程序。

>GRUB2功能多，MBR并不能装下，GRUB2会想尽办法在其他地方存入GRUB2代码（如MBR与第一个分区之间），而MBR中只加载这些代码。非MBR的情况下，策略也不同。

>在启动菜单项中，我们可以进入GRUB2中修改内核启动参数，如果内核不能识别参数，就将参数传给第一个用户进程：init进程。

>注意, UEFI可提供选择引导程序的能力, 并且引导程序也存在于EFI/ESP分区中的, 同时引导程序也提供了选择其他引导程序的能力. 但在BIOS中, 只能从磁盘第一个扇区读取引导程序. 详细见[零碎知识之开机引导过程](https://blog.csdn.net/jdbdh/article/details/87189717#32__204)

内核被加载进来后先解压，然后内核会处理所有的操作系统任务，如内存管理、任务调度、I/O、进程通信和其他系统控制任务，之后挂载根目录。一旦内核全面运转，就会执行第一用户进程init。

>一般内核会挂载**内存**中的initrd，作为临时根目录，引导程序在加载内核的同时也会加载initrd。initrd中存有一些驱动模块，如访问文件系统的驱动，然后通过这些驱动挂载真正的根目录（只读）。通过将内核部分模块分离到initrd中，能够有效减小内核的大小。

init是所有进程的父进程，由内核执行，负责所有进程的启动和关闭。init在linux有很多种实现，如SysV、systemd、upstart。sysv是很老的方法，目前systemd已成为主流，同时也兼容sysv。尽管有了内核后，但没有用户进程，不能进行太多的操作，因此init进程需要启动大量的程序，让系统达到可能的地步。所有的linux发行版的区别就在于此。可以将系统启动到援救模式、单人默认、多用户模式、图形模式，因此出现了运行级别（sysv中）、target（systemd中）的概念，这些模式的区别在于运行着不同的程序。

# systemd

## 介绍

内核启动后，systemd主要用于启动各种服务、守护进程、挂载其他目录等任务，达到用户可用的状态。systemd对所有的任务提供了一个抽象的概念：**unit**。一共有12中类型的unit，unit之间有**依赖关系**，也即启动一个unit的同时会造成其他unit**同时**启动，这些通过依赖联系的unit位于同一个**事务**中。systemd会维护事务，通过去掉不必要的jobs（也即进程）解决它们的依赖冲突，然后启动事务中全部进程，事务结束。

所有的unit类型如下：

1. **Service units**, which start and control daemons and the processes they consist of.
2. **Socket units**, which encapsulate local IPC or network sockets in the system, useful for socket-based activation. 
3. **Target units** are useful to group units, or provide well-known **synchronization points**
4. **Device units** expose kernel devices in systemd and may be used to implement device-based activation.
5. **Mount units** control mount points in the file system
6. **Automount units** provide automount capabilities, for on-demand mounting of file systems as well as parallelized boot-up. 
7. **Snapshot units** can be used to temporarily save the state of the set of systemd units,which later may be restored by activating the saved snapshot unit. 
8. **Timer units** are useful for triggering activation of other units based on timers. 
9. **Swap units** are very similar to mount units and encapsulate memory swap partitions or files of the operating system. 
10. **Path units** may be used to activate other services when file system objects change or are
    modified.
11. **Slice units** may be used to group units which manage system processes (such as service and scope units) in a hierarchical tree for resource management purposes. 
12. **Scope units** are similar to service units, but manage foreign processes instead of
    starting them as well. 

>其中，我们经常配置或使用的是service unit和target unit。内核启动后，systemd会启动default.target，它一般是一个指向multi-user.target或graphical.targer的符号链接，即启动程序到出现命令行或图形界面的地步。

## 配置

unit一般的状态：active、inactive、中间态(activing,deactivating)，fail。unit通过配置文件来配置，配置文件放置的目录如下（同名文件，前面的优先级高）：

* `/etc/systemd/system`
* `/run/systemd/system`
* `/usr/lib/systemd/system`

配置文件语法很简单，所有的unit都通用的选项被配置在`[Unit]`或`[Install]` section中，其他的选项特定于对应类型unit的section中。还有一些选项适用于service、socket、mount、swap，请查看`systemd.exec`、`systemd.swap`的手册。

空行，以`#`和`;`开始的行会被忽略。

### [Unit]

`[Unit]` section中含有所有unit都通用的选项，下面列出常用的：

* `Description`：一段描述unit的字符串。

* `Requires`：配置该unit的依赖，多个依赖空格分隔。该unit被启动时，依赖也会被**同时**启动，加入到事务中；如果依赖结束或启动失败，该unit也会结束，事务失败。

* `Wants`：配置该unit的依赖，多个依赖空格分隔。是Requires的弱化版，如果依赖启动失败，则依赖不会被加入事务。

* `Conflicts`：配置与该unit冲突的依赖，空格分隔。假设运行着的A unit有对B unit的冲突依赖，则B被阻止运行。如果A与B在一个事务中同时运行，则systemd会考虑是否可以修复它。

* `Before`，`After`：配置该unit的顺序依赖。一般unit会与它的Requires依赖同时运行，与`Before`，`After`一起使用后，会造成该unit先或后于依赖执行。如果顺序依赖于该unit不处于同一事务，则无任何影响。

  >关机时，unit之间结束的顺序相反
  >After常被设置为：`After=network.target local-fs.target remote-fs.target`


### [Install]

[Install] section的选项并不会被systemd解析，而是被systemctl的enable或disable命令使用的：

* `WantedBy`，`RequiredBy`：空格分隔的unit列表。使用了systemctl enable命令后，会在列表依赖的`unit_name.wants/`或`unit_name.requireds/`目录下建立该unit的符号链接。使用disable命令则删除符号链接。

  >unit对应的wants、requires目录隐式称为该unit的Wants或Requires依赖。这样的好处是不用修改其他的配置文件，就能成为其他unit的依赖。

### [Service]

[Service] section的选项仅适用于service类型的unit配置文件，注意文件以`.service`结尾。默认隐含有对于basic.target的Requires和After依赖，和对shutdown.target的Conflicts和Before依赖。保证service在基本系统初始化后运行，系统关闭之前结束。

* `Type`：服务进程启动类型，可选值simple、forking、oneshot、dbus、notify、idle。

  * `simple`：无Type和BusName，仅有`ExecStart`情况下的默认类型。systemd认为`ExecStart`上运行的进程为主进程，一旦启动就会执行接下来的units。

    > 如果`ExecStart`运行的不是主进程, 则最好提供`ExecStop`, `ExecReload`, 这样也能正常关闭和重启进程.

  * `forking`：如果ExecStart配置的进程可以自己成为守护进程（即后台运行且脱离控制终端，则通过fork子进程实现，父进程结束），则适合forking。一旦父进程结束，systemd会启动其他的units。但是如果子进程多于1个，systemd并不知道哪个子进程是主进程，因此需要PIDFile选项指定进程自己提供pid文件（含有进程号）位置。

  * `oneshot`：与simple类似，但systemd期待在开启其他unit前结束，Type和ExceStart都不存在的默认类型。适合于RemainAfterExit一起使用。

  * ...：请看man手册

  >总之，进程在shell中运行不返回提示符的使用simple；进程在shell中运行，会自动后台运行且返回提示符的（能自己守护进程话），用forking；进程在shell中运行并返回提示符，此时进程也结束了，使用oneshot。

* `PIDFile`：指定守护进程的pid文件的绝对路径，适合与forking搭配使用。服务结束后，systemd会删除该文件。

* `ExecStart`：服务启动时执行的命令。

  >注意，命令不是通过shell执行的，而是systemd执行的，注意语法。下同

* `ExecStop`：停止服务所执行的命令。如果没有设置，systemd会发送对应的信号结束服务。

* `ExecReload`：服务需要重新加载时所执行的命令。如果没有设置，systemd会发送HUP信号重启服务。

* `User`，`Group`：设置进程用户或组，默认root。systemd.exec中的内容

### 其他

**target**类型的配置文件没有多余的选项，因为它仅仅只是作为一个同步点而存在的。比如假设systemd默认启动`graphical.target`，则最终系统会将所有图形界面环境运行起来，相关的依赖大多会在其他unit文件中的[Install]中配置。

```configuration
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

配置其他类型unit，如**socket**、**timer**，会在一定条件下，触发对应名字的service unit的执行。等等之类，请查看man手册。具体例子请看`man systemd.service`下最后一节内容。

### systemd参数

systemd也是有参数的，不过systemd是内核执行的，因此可以在GRUB2的开始菜单中修改内核参数达到设置systemd启动参数的目的，因此内核不认识的参数会传给init进程。部分参数如下：

* `systemd.unit=`：设置启动的unit，默认default.target。
* ...：没了？？。。是的，你没看错。

### Demo

```
[Unit]
Description=ngrok
After=network.target

[Service]
ExecStart=/myweb/ngrok/bin/ngrokd -tlsKey=/myweb/ngrok/server.key -tlsCrt=/myweb/ngrok/server.crt -domain="weixin.yangjiace.xyz" -httpAddr=":80" -httpsAddr=":443"

[Install]
WantedBy=multi-user.target
```

## systemctl

用于控制systemd系统和服务的管理器。命令如下：

* `list-units`：列出所有已知的units。

* `start`：启动unit

* `stop`：停止unit

* `restart`:重启unit。

* `status`：查看unit状态

* `enable`：根据`[Install]`创建对应的符号链接；如果手动创建符号链接，请使用`daemon-reload`重新加载配置文件

* `disable`:将符号链接删除。

* `isolate`：停止所有进程，启动指定的unit。如果unit名字没有后缀，默认target。类似于sysv的切换运行级别。

* `get-default`：获得自启时默认的default target，即返回default.target指向的unit名。

* `set-default`：设置自启时默认的default target，即修改符号链接default.target。

  > 一般设置的target:
  >
  > * multi-user.target 多用户命令行界面
  > * graphical.targer 图形界面

## 关机

关机命令很多，但实现原理一致，通过init进程关机，init进程阻止其他非root用户登陆，然后在systemd中会启动shutdown unit，它会关闭所有程序，最后通知内核结束自己。

>`shutdown`用于停止、关机、重启主机，还可定时、发送警告给所有用户。
>`shutdown [OPTIONS...] [TIME] [WALL...]`
>时间形式可以`hh:mm`，指定小时分钟；可为`+m`，指定m分钟后关机；`now`，立即关机，相当于+0；`无`，则使用+1。
>
>* `-H`:停止cpu。类似命令`halt`
>* `-P`:关闭主机。类似命令`poweroff`
>* `-r`:重启主机。类似命令`reboot`
>* `-c`:取消定时的关机。

# 参考

* man手册
* [Linux startup process wiki](https://en.wikipedia.org/wiki/Linux_startup_process)
* [In systemd, what's the difference between After= and Requires=?](https://serverfault.com/questions/812584/in-systemd-whats-the-difference-between-after-and-requires)
* [systemd forking vs simple?](https://superuser.com/questions/1274901/systemd-forking-vs-simple/1274913)

