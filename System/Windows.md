# PowerShell

Win10中有两种Shell：PowerShell和Cmd。

* Cmd功能有限
* PowerShell功能强大, 且有很多命令与Linux Bash命令相似，推荐使用.

## 常用命令

* `ls`：显示目录或文件

* `pwd`：显示当前目录

* `explorer .`：于当前目录打开文件浏览器

* `cd`：改变工作目录

  > `cd ~`：回到用户目录

* 获取帮助

  * `man`、`help`命令或`-h`、`--help`、`/?`选项
  * Powershell保证`-?`必定获取到*cmdlets*的帮助

  > 其实没啥子用

* `cat`：输出文件内容

* `notepad filename`:用记事本打开文本

* 支持重定向，可用于ssh中传输文件，暂不熟

* `clear`：清空屏幕

* `echo $Env:PATH`：打印环境变量, 如`PATH`

  > 与cmd不同，cmd中通过`%PATH%`可以获取值

  > 关于更多环境变量和变量的知识, 见1.2小节-Script

* `where.exe <Command>` 查找名字所在目录

* ...

>其余命令可参考：
>[Table of Basic PowerShell Commands](https://blogs.technet.microsoft.com/heyscriptingguy/2015/06/11/table-of-basic-powershell-commands/)

## 相关概念

* 大部分Shell命令的处理对象是字符串, 而PowerShell命令的处理对象是一个包含结构化信息的对象
* PowerShell中的原生命令被称为*cmdlets*, 使用者也可提供自己的*cmdlets*
*  PowerShell基于.Net平台, 支持C#部分语法的使用
* 命令和变量大小写不敏感

## 语法

### 命令名

PowerShell的命令拥有自己的一套命令规则, 同时也提供了匿名, 帮助习惯Bash Shell的人使用.

略, 以后补充

### 变量

* 命名: 变量名包含`_`和任意数字和字符, 使用时必须以前缀`$`标识

* 创建或赋值

  ```powershell
  PS> $loc # 创建变量
  PS> $loc = Get-Location # 创建变量的同时赋值
  PS> $loc # 打印变量
  
  Path
  ----
  C:\temp
  ```

  > 可以看出, 声明变量时, 如果变量不存在则创建; 如果变量存在, 则打印.

### 使用环境变量

* 打印所有环境变量

  ```powershell
  Get-ChildItem env:
  ```

* 打印某个环境变量

  ```powershell
  PS> $env:SystemRoot
  C:\WINDOWS
  ```

* 修改环境变量

  ```powershell
  $env:LIB_PATH='/usr/local/lib'
  ```

### 注释

以`#`为前缀的字符串

### 管道

以`|`连接多个命令, 每个命令的输出将作为下一个命令的输入. 

> 与其他Shell的管道不同, PowerShell的管道传输的是对象

## 特殊变量

下面列出部分特殊变量

* `$HOME` 用户家目录
* ...

> 详细参考[Powershell - Special Variables](https://www.tutorialspoint.com/powershell/powershell_special_variables.htm)

## 脚本

脚本文件以`.ps1`为后缀

## 参考

* [PowerShell Scripting](https://docs.microsoft.com/en-us/powershell/scripting/how-to-use-docs?view=powershell-6) 官方文档
* [PowerShell tutorialspoint.com](https://www.tutorialspoint.com/powershell/index.htm)

# 配置

## 个性化

* 去除系统桌面图标(垃圾箱): `个性化->主题->相关的设置->桌面图标设置`

## 虚拟内存

* 查看虚拟内存大小: CLI中输入`systeminfo`
* 修改大小: `控制面板\系统和安全\系统\高级系统设置`, 然后自己找

## 手动更新

1. 按`Win`, 输入`update`, 找到OS版本, 及设备信息

   ![windows10 update manually build version](.Windows/windows10-update-version-build.jpg)

   ![manually download windows10 updates 64-bit](.Windows/manually-download-windows-10-updates-64-bit-1574644714319.jpg)

2. 在[Windows 10 Update History](https://support.microsoft.com/en-us/help/4099479) 中找到对应更新, 获取更新号, 如KB4525241

   > 里面的1709,1803,1909都是Windows的不同版本, 类似Linux不同的发行版, 只能更新到版本最新, 不能从1709更新到1803, 除非使用其他方式.

3. 在 [Microsoft Update Catalog](http://www.catalog.update.microsoft.com/Home.aspx) 输入更新号, 找到对应版本下载

4. 下载后, 点击安装

> 参考[How To Download Windows 10 Updates Manually (And Install)](https://www.thetechmentor.com/posts/how-download-windows-10-updates-manually-and-install/)

## 安装Store

> 最新更新:
>
> 推荐使用[LTSB-Add-MicrosoftStore](https://github.com/kkkgo/LTSB-Add-MicrosoftStore)的方案, 下面的方案不太方便.

环境:

* 无本地安装包 *Microsoft.WindowsStore* , 即`Get-AppxPackage`命令找不到包

解决方案:

安装VirtualBox虚拟机, 下载Window10镜像, 创建虚拟机.

> 或使用[Vmware](https://www.7down.com/soft/310739.html)

1. 将虚拟机中` C:\Program Files\WindowsApps `中的以下文件拷贝到物理机对应目录中

   - Microsoft.VCLibs.140.00_14.0.22810.0_x64__8wekyb3d8bbwe
   - *Microsoft.VCLibs.140.00_14.0.22810.0_x86__8wekyb3d8bbwe*
   - Microsoft.NET.Native.Runtime.1.0_1.0.22929.0_x64__8wekyb3d8bbwe
   - *Microsoft.NET.Native.Runtime.1.0_1.0.22929.0_x86__8wekyb3d8bbwe*
   - Microsoft.WindowsStore_2015.7.1.0_x64__8wekyb3d8bbwe
   - *Microsoft.WindowsStore_2015.701.14.0_neutral_~_8wekyb3d8bbwe*

2. 打开Powershell, 重新注册该安装包, 如

   ```powershell
   Add-AppxPackage -DisableDevelopmentMode -Register "C:\Program Files\WindowsApps\Microsoft.WindowsStore_2015.7.1.0_x64__8wekyb3d8bbwe\AppxManifest.xml
   ```

3. 安装完毕, 按`Win`可搜索到商店

> 参考[Restore Microsoft Store application in Windows 10](https://superuser.com/questions/949112/restore-microsoft-store-application-in-windows-10)

## 设置混合睡眠模式

关于待机,休眠和休眠的区别, 见[零碎知识]

首先开启睡眠

![image-20191125165129171](.Windows/image-20191125165129171.png)

![image-20191125165203635](.Windows/image-20191125165203635.png)

然后开启混合睡眠模式

![image-20191125165356753](.Windows/image-20191125165356753.png)

![image-20191125165446198](.Windows/image-20191125165446198.png)

