# 快捷键

* 打开终端 ```Ctrl+` ```
* 打开命令面板: `F1`
* 代码格式重构
  * On Windows `Shift + Alt + F`
  * On Mac `Shift + Option + F`
  * On Ubuntu `Ctrl + Shift + I`
  * 列选择`Ctrl+Alt+下移光标`

# 搜索

* 正则中反向引用: `$num`

* 匹配标题前标号: `(#+ )(\d+\.)+\d+ `

  > 不包括一级标题
  >
  > 注意最后有个空格, 可根据自己的需求改.

# Java开发

## 环境搭建

1. 首先安装了JDK, 并配置了环境

2. 安装以下常用插件
   1. [Language Support for Java(TM) by Red Hat](https://marketplace.visualstudio.com/items?itemName=redhat.java)
   2. [Debugger for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug)
   3. [Java Test Runner](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-test)
   4. [Maven for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-maven)
   5. [Java Dependency Viewer](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-dependency)
   6. [Visual Studio IntelliCode](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode)
3. 以下是可选的
   1. [Spring Boot Tools](https://marketplace.visualstudio.com/items?itemName=Pivotal.vscode-spring-boot)
   2. [Spring Initializr Java Support](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-spring-initializr)
   3. [Spring Boot Dashboard](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-spring-boot-dashboard)
   4. [Tomcat](https://marketplace.visualstudio.com/items?itemName=adashen.vscode-tomcat)
   5. [Jetty](https://marketplace.visualstudio.com/items?itemName=SummerSun.vscode-jetty)
   6. [CheckStyle](https://marketplace.visualstudio.com/items?itemName=shengchen.vscode-checkstyle)

	> 除此之外可以自行扩充

------------

或者使用扩展包的方式

* [Java Extension Pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) 包含了Java开发的常用扩展
* [Spring Boot Extension Pack](https://marketplace.visualstudio.com/items?itemName=Pivotal.vscode-boot-dev-pack) 包含Spring Boot开发的常用扩展

## 配置

`.vscode`中有两个配置文件

* `launch.json`: 运行所需的配置项
* `settings.json`: vscode相关的局部配置项

> 参考: [Java in Visual Studio Code](https://code.visualstudio.com/docs/languages/java)

# 插件

* `Remote-WSL`: 使code能够编辑WSL中的文件.

  > 在WSL中, Windows是不能够编辑WSL文件的, 否则会造成WSL系统的损坏.
  >
  > 该插件本质是通过在WSL开启一个远程server, 而Win上的VSCode作为client, 来实现的.
  
  > 然而, 在WSL中打开Windows软件, 也会被该插件接管, 效率不太行, 故删除.

# 远程开发

## 介绍

在远程环境下开发时又获取到了与本地开发同等的体验, 如智能代码补全, 代码导航和调试等.

工作原理图:

![Architecture](.VSCode/architecture.png)

## 准备

先下载插件包[Remote Development extension pack](https://aka.ms/vscode-remote/download/extension), 该包包含

* [Remote - SSH](https://code.visualstudio.com/docs/remote/ssh) 用于打开任意含有SSH Server的远程主机上的目录
* [Remote - Containers](https://code.visualstudio.com/docs/remote/containers) 用于连接容器
* [Remote - WSL](https://code.visualstudio.com/docs/remote/wsl) 用于连接WSL
* ...

对于SSH, `PATH`下需存在`ssh`命令, 如OpenSSH或Git附带的`ssh`命令

## 使用

* 连接远程主机

## 放弃

这是个玩具啊... 实在不太好用

# 其他

* 换行符: 搜索`end of line`

* 全局配置文件`settings.json`: 基本上, 配置与默认不一致时, 则会被记录在这

* 隐藏的文件: `Commonly Used->Files:Exclude`

  > 疑问? 怎么插件隐藏的文件呢?

## 疑问

* 怎么主动打开Java Overview