# RDP

* RDP协议 ( Remote Desktop Protocol )

  微软开发的远程桌面协议, 能够远程登陆主机和创建一个真实的桌面会话. 

* Xrdp

  是RDP协议的实现, 这里介绍Xrdp服务的安装

* 安装桌面环境

  需要安装X11协议相关工具和轻量级的桌面环境. 这里推荐XFce

  ```shell
  sudo apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils
  ```

* 安装Xrdp

  ```shell
  sudo apt install xrdp
  ```

  > 将自动安装为`xrdp`守护进程

* 配置

  ...

* 缺点

  当有用户以相同账号登录时, RDP将登录失败.

# VNC

* VNC协议 ( Virtual Network Computing )

* 原理: 传输图像



