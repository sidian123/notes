# ConEmu

* 介绍: 一款好用的Windows终端, 特点: 小, 美观, 便携, 多Tab.

  > 个人感觉比Cmder好用

* 快捷键

  * `Win+W` 打开新Tab页, 载入默认Shell
  * `Win+Shift+W` 打开新Tab页前, 先弹出对话框, 供选择要载入的Shell
  * `LCtrl+Number` 跳转到对应数字的Tab页
  * `Ctrl+Tab`或`Ctrl+Shift+Tab` 下或上一个Tab页

* 配置

  * 默认Shell: 当初次运行时, 会有对话框提示. 若要默认使用WSL, 则输入`ubuntu1804.exe`或`wsl`即可

> 参考[ConEmu](https://conemu.github.io/)

  # Windows Terminal

微软出产的终端, 目前为预览版, 功能很简陋. 

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

> 参考[Editing Windows Terminal JSON Settings](https://github.com/microsoft/terminal/blob/master/doc/user-docs/UsingJsonSettings.md)

