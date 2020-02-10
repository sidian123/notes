# 引言

莫要把终端和Shell混淆了

* 终端是Shell的壳, 提供用户与Shell的交互功能; 
* Shell是操作系统内核API的壳, 提供用户操作系统的功能.

> 有时候, 大家说到终端时, 其实意思中也暗指其Shell.

# ConEmu

* 介绍

  一款好用的Windows终端, 特点: 小, 美观, 便携, 多Tab.

* 快捷键

  * `Win+W` 打开新Tab页, 载入默认Shell
  * `Win+Shift+W` 打开新Tab页前, 先弹出对话框, 供选择要载入的Shell
  * `LCtrl+Number` 跳转到对应数字的Tab页
  * `Ctrl+Tab`或`Ctrl+Shift+Tab` 下或上一个Tab页

* 配置

  * 默认Shell: 当初次运行时, 会有对话框提示. 若要默认使用WSL, 则输入`ubuntu1804.exe`或`wsl`即可

> 参考[ConEmu](https://conemu.github.io/)

# Cmder

## 介绍

基于ConEmu, 但也提供更多的功能, 如

1. 为cmd.exe提供了更好的Bash风格的代码补全和提示功能, 见[Clink](https://mridgers.github.io/clink/)

2. 可移植性高, 可直接装入U盘中使用.

3. 内置几乎所有常用Unix命令(包括git), 并且在`PATH`下可用, cmd下可用

   > 需要下载完整版的Cmder

## 我的理解

我认为, Cmder起到了组合和配置的作用, 将多个实用工具组合了起来. 

* 终端由ConEmu提供; 
* Clink提供第一个功能; 
* Cmder本就小, 因此有了第二个功能; 
* 第三个功能由Git Bash提供, Git Bash提供了大量exe格式的Unix命令.

总之, 已经帮我们配置到很好用的程度了!

## 目录结构

* `bin/`

  该目录下的文件, 在Cmder运行时会被注入到`PATH`下.

* `config/`

  Cmder所有配置文件

* `vendor/`

  用到的三方库, 如`Clink`, `ConEmu`, `Git for Windows`等

## 配置

在Cmder中, 打开一个tab, 就是打开一个任务.

现在需修改打开任务时的默认工作目录

![image-20191222231011254](.Terminals%20In%20Windows/image-20191222231011254.png)

> 通过`Startup dir...`设置好家目录后, 将文本框中的地址替换成`%userprofile%`, 防止配置特定于某一台电脑

设置默认启动任务

![image-20191222231150829](.Terminals%20In%20Windows/image-20191222231150829.png)

> 若没有自己Shell的任务, 请手动配置一个任务 ( 这操作...巨难 )

接下来配置自己想要的快捷键

![image-20191222231301398](.Terminals%20In%20Windows/image-20191222231301398.png)

设置自启, 见[我的回复](https://github.com/cmderdev/cmder/issues/532#issuecomment-584161122)

## 使用

关于复制, `Ctrl+V`粘贴, 格式可能会错误, 应该使用右键粘贴

## 参考

* [cmder](https://cmder.net/) 官网

* [Windows上的程序员神器——Cmder](https://zhuanlan.zhihu.com/p/28400466)

  # Windows Terminal

* 介绍

  微软出产的终端, 目前为预览版, 功能很简陋, 但实用, 简约, 大气, 美观等.

* 默认终端

  将`defaultProfile`设为某个`profile`的`guid`即可, 如

  ```json
      "defaultProfile": "{c6eaf9f4-32a7-5fdc-b5cf-066e8a4b1e40}",
      "profiles":[
  		...
          {
              "guid": "{c6eaf9f4-32a7-5fdc-b5cf-066e8a4b1e40}",
              "hidden": false,
              "name": "Ubuntu-18.04",
              "source": "Windows.Terminal.Wsl"
          },
  		...
      ],
  ```

* 粘贴复制

  是的, 这个也需要自己配置...

  ```json
      "keybindings": [
          { "command": "copy", "keys": ["ctrl+shift+c"] },
          { "command": "paste", "keys": ["ctrl+shift+v"] }
      ]
  ```
  
* 缺点

  所需Windows版本太高了... 我公司的电脑用不了.

> 参考[Editing Windows Terminal JSON Settings](https://github.com/microsoft/terminal/blob/master/doc/user-docs/UsingJsonSettings.md)
