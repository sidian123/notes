安装cordova

```undefined
npm install -g cordova
```

安装指定版本的cordova

```css
npm install -g cordova@3.1.0-0.2.0
```

查看版本

```undefined
cordova -v
```

创建APP

```css
cordova create hello com.example.hello HelloWorld
```

进入项目

```bash
cd hello
```

查看本机安装的平台

```cpp
cordova platforms list
```

查看安装平台的先决条件

```undefined
cordova requirements
```

给项目添加平台支持【当然你也可以添加其他平台的支持，前提是你的本机有】

```csharp
cordova platform add android
```

对项目删除指定平台

```undefined
cordova platform rm android
```

保存平台

```undefined
cordova platform save
```

恢复平台

```undefined
cordova prepare
```

更新平台

```undefined
cordova platform update
```

添加cordova插件

```csharp
cordova plugin add cordova-hot-code-push-plugin【热更新插件】
```

或者使用repo url直接安装（不稳定）

```csharp
cordova plugin add [https://github.com/nordnet/cordova-hot-code-push.git](https://github.com/nordnet/cordova-hot-code-push.git)
```

或者安装本地插件

```csharp
cordova plugin add E:\project\plugins\cordova-hot-code-push-local-dev-addon
```

查看项目安装的插件

```cpp
cordova plugin list
```

唤起插件搜索列表页面

```undefined
cordova plugin search camera
```

安装指定版本号的插件

```csharp
cordova plugin add
```

删除插件

```csharp
cordova plugin remove
```

插件保存

```undefined
cordova plugin save
```

查看已经安装的插件列表以及环境版本情况

```undefined
cordova info
```

查看cordova全部命令

```bash
cordova help
```

编译项目

```undefined
cordova build android 【他会在 platforms/android/bin/ 下已经生成了 apk 文件】
```

启动 android 虚拟机

```bash
cordova emulate android
```

运行 app 项目（在虚拟机或者在真机）

```undefined
cordova run android
```

重新编译

```bash
cordova emulate
```