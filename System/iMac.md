# 快捷键

* 基本操作

  * Command-Z 撤销　
  * Command-X 剪切　　
  * Command-C 拷贝（Copy）　　
  * Command-V 粘贴　　
  * Command-A 全选（All）　　
  * Command-S 保存（Save)　　
  * Command-F 查找（Find）　

* 截图

  Command-Shift-Control-4 截取选中区域到剪贴板

* 隐藏

  * Command-H 隐藏当前程序
  * Command-Option-H 隐藏其他程序

* 关闭程序

  Command-Q 关闭当前程序

* 查单词

  Option-Z （需要存在有道词典）

*  打开搜索

  Command-Space

# 命令

* 打开Finder

  ```shell
  open <path>
  ```


# 密钥导入导出

https://www.digicert.com/kb/ssl-support/p12-import-export-mac-server.htm

# IOS开发

用的Cordova写的跨平台应用, 写IOS应用特别恶心!!!!! IOS真是垃圾中的战斗机!!!!

* 后端使用无效证书的解决方案

  编译后, 在Xcode中, 找到`Classess/AppDelegate.m`文件, 添加

  ```objective-c
  @implementation NSURLRequest(DataController)
  + (BOOL)allowsAnyHTTPSCertificateForHost:(NSString *)host
  {
      return YES;
  }
  @end
  ```

  > [Cordova/PhoneGap iOS HTTPS/SSL issues](https://stackoverflow.com/questions/12627774/cordova-phonegap-ios-https-ssl-issues)

* 不能打包, 按钮灰的

  设置选择*Generic iOS Device*