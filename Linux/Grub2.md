# 介绍

Linux中常用的引导程序, 目前最新Grub2版本.

Grub2的配置文件位于`/boot/grub/grub.cfg`, 该文件由`update-grub`命令构建, 它会自动搜索内核和引导程序信息, 并结合`/etc/default/grub`和`/etc/grub.d/*`中的配置文件构建出`/boot/grub/grub.cfg`.

# 从Grub中引导

引导程序主要用于加载内核和initrd. initrd会被内核挂载为临时根目录, 提供部分驱动程序, 如访问文件系统的驱动, 然后通过这些驱动挂载真正的根目录.

> 实际上, Grub2越来越强大, 自身就能识别ext2文件系统, ext系列也存在一定的向下兼容.

因此, 因为某种缘故, 如引导项被删除了, 只能手动引导.

1. 首先需要找到根分区

   ```bash
   grub> ls
   (hd0),(hd1),(hd1,gpt3),(hd1,gpt2),(hd1,gpt1)
   grub > ls (hd1/gpt2)/   #每个分区都这么尝试
   ```

   > 因为grub2识别ext2分区, 如果你的根分区也是ext系统的, 则会打印该分区的子目录, 从而找到根分区.

2. 加载内核和initrd

    ```grub
    grub> set root=(hd0,1)
    grub> linux /boot/vmlinuz-3.13.0-29-generic root=/dev/sda1
    grub> initrd /boot/initrd.img-3.13.0-29-generic
    grub> boot
    ```
    
    > `set root`设置为找到的根目录, 之后的目录都是缺省从该分区找的.
    >
    > linux命令中的`root`选项用于告诉内核根目录在哪个分区.
    >
    > 加载好后使用`boot`启动内核.
    

# 参考

* [How to Rescue a Non-booting GRUB 2 on Linux](https://www.linux.com/learn/how-rescue-non-booting-grub-2-linux)
* [开机进入GRUB不要慌，命令行也可启动Linux](https://www.cnblogs.com/zztian/p/10289083.html)