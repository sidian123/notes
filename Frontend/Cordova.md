# 基本操作

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

# 踩坑

用的Cordova写的跨平台应用, 写IOS应用特别恶心!!!!! IOS真是垃圾中的战斗机!!!!

## 后端证书无效

* for IOS

    编译后, 在Xcode中, 找到`Classess/AppDelegate.m`文件, 添加

    ```objective-c
    @implementation NSURLRequest(DataController)
    + (BOOL)allowsAnyHTTPSCertificateForHost:(NSString *)host
    {
        return YES;
    }
    @end
    ```
    
* for Android

    找到并修改`project/platforms/android/CordovaLib/src/org/apache/cordova/engine/SystemWebViewClient.java`文件

    ```java
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
      final String packageName = this.cordova.getActivity().getPackageName();
      final PackageManager pm = this.cordova.getActivity().getPackageManager();
    
      ApplicationInfo appInfo;
      try {
        appInfo = pm.getApplicationInfo(packageName, PackageManager.GET_META_DATA);
        if ((appInfo.flags & ApplicationInfo.FLAG_DEBUGGABLE) != 0) {
          // debug = true
          handler.proceed();
          return;
        } else {
          // debug = false
          // THIS IS WHAT YOU NEED TO CHANGE:
          // 1. COMMENT THIS LINE
          // super.onReceivedSslError(view, handler, error);
          // 2. ADD THESE TWO LINES
          // ---->
          handler.proceed();
          return;
          // <----
        }
      } catch (NameNotFoundException e) {
        // When it doubt, lock it out!
        super.onReceivedSslError(view, handler, error);
      }
    }
    ```

> 参考:
>
> * [Ignoring invalid SSL certificates on Cordova for Android and iOS](http://ivancevich.me/articles/ignoring-invalid-ssl-certificates-on-cordova-android-ios/)
>
> * [Cordova/PhoneGap iOS HTTPS/SSL issues](https://stackoverflow.com/questions/12627774/cordova-phonegap-ios-https-ssl-issues)

## Xcode不能打包

不能打包, 按钮灰的

设置选择*Generic iOS Device*

## 无网络连

请求权限, 安装` cordova-plugin-whitelist`, 并在`config.xml`文件中添加

```xml
<allow-navigation href="*" />
<allow-intent href="http://*/*" />
<allow-intent href="https://*/*" />
```

接着, 还要为IOS单独添加`cordova-plugin-ios-plist`插件