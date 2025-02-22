---
title: 制作macOS 安装 iso
date: 2021-01-18 18:21:00
tags: 
- mac
categories:
- 技术
- 其他
---

在虚拟机中安装 macOS, 通常需要一个 iso 文件引导(如 esxi), 但是我们往往下载到的是 app 格式的安装包, 如何才能将其打包为 iso 文件?
虽然网上有别人做好的 iso 文件, 但是无法得知是否有病毒木马之类的, 还是自己动手比较靠谱.
<!-- more -->

## 下载系统
如果需要下载到干净的系统, 那么你需要一台可以工作的 mac 电脑, 或是其他运行 macOS 的虚拟机/黑苹果等. 
可以参考这篇文档的 下载 macOS 部分, 里面有教你如何下载系统. 目前有[10.13(macOS High Sierra)](https://itunes.apple.com/cn/app/macos-high-sierra/id1246284741?ls=1&mt=12), [10.14(macOS Mojave)](https://itunes.apple.com/cn/app/macos-mojave/id1398502828?ls=1&mt=12), [10.15(macOS Catalina)](https://itunes.apple.com/cn/app/macos-catalina/id1466841314?ls=1&mt=12), [11.0(macOS Big Sur)](https://itunes.apple.com/cn/app/macos-big-sur/id1526878132?ls=1&mt=12), [10.11(OS X El Capitan, 这个链接不能直接打开, 右键复制链接地址后, 手动开启新的tab 页面打开)](http://updates-http.cdn-apple.com/2019/cert/061-41424-20191024-218af9ec-cf50-4516-9011-228c78eda3d2/InstallMacOSX.dmg)的系统提供下载.
如果是下载 10.13, 10.14, 10.15, 11.0, 基本就是打开连接后, 会提示是否在 App Store 中显示, 然后会进入系统设置的软件更新, 开始下载.
如果是10.11 则是直接下载一个dmg, 里面有一个InstallMacOSX.pkg 的安装包, 将它打开安装, 稍后就会多出现程序出现在你的/Application目录中.
下载完成后, 不用安装. 如果出现了安装界面, 直接退出即可.
## 制作 iso 镜像
大致流程: 
创建dmg, 将系统安装盘写入dmg, 将dmg 转换为iso.
首先需要打开终端执行命令.
### 创建dmg
``` bash
hdiutil create -o /tmp/dmgoutput -size 13g -layout SPUD -fs HFS+J
```
命令中的13g是指安装盘大小, 13gb, 并不一定需要是这么多, 你可以看看install app 有多大, 再决定这个尺寸.
### 挂载dmg
这里将上一步创建的dmg挂载起来, 以便后续写入
``` bash
hdiutil attach /tmp/dmgoutput.dmg -noverify -mountpoint /Volumes/install_build
```
### 写入安装盘
``` bash
sudo "/Application/Install xxx.app/Contents/Resources/createinstallmedia" --volume /Volumes/install_build
```
Install xxx.app 需要替换为实际的app 名称.
注意路径有空格, 需要添加引号, 否则需要自己添加转义符.

### 卸载安装盘
执行完后, 会自动挂载安装盘, 需要到finder 中将这个盘卸载掉.
如果提示正在使用, 确认上一步已经执行完毕的情况下, 可以强制卸载.
### 转换为 iso
``` bash
hdiutil convert /tmp/dmgoutput.dmg -format UDTO -o ~/Desktop/output
mv ~/Desktop/output.cdr ~/Desktop/output.iso
```
执行完命令后, 会在桌面生成一个iso 文件, 这个文件就可以用来安装了.
最后将 /tmp/dmgoutput.dmg, 以及 /Application/Installxxx .app 删掉即可.
## 参考资料

[Download and convert MacOS Mojave installer into ISO file](https://blog.petehouston.com/download-and-convert-macos-mojave-installer-into-iso-file)

