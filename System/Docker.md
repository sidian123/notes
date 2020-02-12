* Container

  * Windows Container

    * Windows Server Container

      原生的Windows容器, 于Linux类似, 基于进程隔离 , 运行WIndows的软件.

    * Hyper-V Container

      通过Hyper-V启动**很小的**Linux虚拟机, 然后运行的Linux容器

  * Linux Container

    Linux上的容器

  > 参考
  >
  > * [Windows Container 和 Docker：你需要知道的5件事](https://www.cnblogs.com/ups216/p/6385663.html)
  >
  > * [Windows系统下的Windows Container和Linux Contaner](https://blog.csdn.net/littleworm0/article/details/102626516)

# 删除

我工作用的电脑运行Docker报错, 无法解决, 于是想删除Docker, 却一直卡死, 只能另图它径.

新建脚本, 如`remote-docker.ps1`, 内容:

```powershell
$ErrorActionPreference = "SilentlyContinue"

kill -force -processname 'Docker for Windows', com.docker.db, vpnkit, com.docker.proxy, com.docker.9pdb, moby-diag-dl, dockerd

try {
./MobyLinux.ps1 -Destroy
} Catch {}

$service = Get-WmiObject -Class Win32_Service -Filter "Name='com.docker.service'"
if ($service) { $service.StopService() }
if ($service) { $service.Delete() }
Start-Sleep -s 5
Remove-Item -Recurse -Force "~/AppData/Local/Docker"
Remove-Item -Recurse -Force "~/AppData/Roaming/Docker"
if (Test-Path "C:\ProgramData\Docker") { takeown.exe /F "C:\ProgramData\Docker" /R /A /D Y }
if (Test-Path "C:\ProgramData\Docker") { icacls "C:\ProgramData\Docker\" /T /C /grant Administrators:F }
Remove-Item -Recurse -Force "C:\ProgramData\Docker"
Remove-Item -Recurse -Force "C:\Program Files\Docker"
Remove-Item -Recurse -Force "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Docker"
Remove-Item -Force "C:\Users\Public\Desktop\Docker for Windows.lnk"
Get-ChildItem HKLM:\software\microsoft\windows\currentversion\uninstall | % {Get-ItemProperty $_.PSPath}  | ? { $_.DisplayName -eq "Docker" } | Remove-Item -Recurse -Force
Get-ChildItem HKLM:\software\classes\installer\products | % {Get-ItemProperty $_.pspath} | ? { $_.ProductName -eq "Docker" } | Remove-Item -Recurse -Force
Get-Item 'HKLM:\software\Docker Inc.' | Remove-Item -Recurse -Force
Get-ItemProperty HKCU:\software\microsoft\windows\currentversion\Run -name "Docker for Windows" | Remove-Item -Recurse -Force
#Get-ItemProperty HKCU:\software\microsoft\windows\currentversion\UFH\SHC | ForEach-Object {Get-ItemProperty $_.PSPath} | Where-Object { $_.ToString().Contains("Docker for Windows.exe") } | Remove-Item -Recurse -Force $_.PSPath
#Get-ItemProperty HKCU:\software\microsoft\windows\currentversion\UFH\SHC | Where-Object { $(Get-ItemPropertyValue $_) -Contains "Docker" }
```

以管理员权限打开Powershell, 在里面运行该脚本即可.

> 参考: [How to completely remove Docker in Windows 10](https://success.docker.com/article/how-to-completely-remove-docker-in-windows-10)

# 参考

* [Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
* [Docker 微服务教程](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)
* [【 全干货 】5 分钟带你看懂 Docker ！](https://zhuanlan.zhihu.com/p/30713987)
* [Docker Documentation](https://docs.docker.com/) 官方文档
* [Docker Quick Start](https://hub.docker.com/?overlay=onboarding) 官方入门
* [Docker最全教程——从理论到实战(一)](https://www.cnblogs.com/codelove/p/10030439.html) 

