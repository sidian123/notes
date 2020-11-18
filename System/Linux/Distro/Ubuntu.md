* 安装
  
  * BIOS中关闭Secure Boot, 防止无法安装引导程序
  
* 美化教程

  * 主题，图标

    https://blog.csdn.net/qq_42527676/article/details/91356154
  
* 手势

  https://blog.csdn.net/small_dong_0_o/article/details/83342228
  
* 配置

  * 修改Root密码, 允许Root登录

    ```shell
    sudo passwd root
    ```

  * SSH允许Root密码登录 (默认允许密钥登录)

    ```shell
    vim /etc/ssh/sshd_config
    PermitRootLogin yes
    ```

    重启SSHD服务

    ```shell
    systemctl restart sshd
    ```

  * 设置默认编辑器

    ```shell
    vim .bashrc
    export EDITOR=VIM
    ```

  * 合上盖子不休眠

    ```shell
    sudo vim /etc/systemd/logind.conf
    HandleLidSwitch=ignore
    ```

    重启服务

    ```shell
    systemctl restart systemd-logind
    ```

    