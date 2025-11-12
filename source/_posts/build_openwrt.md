---
title: openwrt 编译小记
date: 2025-06-29 15:39:00
tags: 
- openwrt
- 软路由
categories:
- 技术
- 其他
---
## 本文记录了编译openwrt 的过程, 仅供参考
不过虽说是openwrt, 但是实际编译的时候, 选择的是[immortalwrt](https://github.com/immortalwrt/immortalwrt)
相对更适合国内用户.
<!-- more -->

## 关于openwrt(原版), immortalwrt, lede 的区别
这里说的 lede 特指 [coolsnowwolf/lede](https://github.com/coolsnowwolf/lede)
这里有详细的对比说明 [https://github.com/immortalwrt/immortalwrt/discussions/1109](https://github.com/immortalwrt/immortalwrt/discussions/1109)
这里还有一篇[https://www.cnblogs.com/Magiclala/p/18418732](https://www.cnblogs.com/Magiclala/p/18418732)
我这里简单说说我的理解, 欢迎各位指证.
### 1. openwrt
我愿称之为一切的起源, 后两者都是在openwrt 上进行优化, 添加更多功能, 支持更多设备. 
#### 1.1. 优点
1. 更新速度快
2. 原版代码, 功能简约, 如果要学习代码, 这个必然是最好的.
3. 纯粹的开源项目, 你的设备完全由自己掌控
#### 1.2. 缺点
1. 对国内用户不太友好, 主要体现在两方面
   1. 软件源基本在国外, 导致安装缓慢甚至失败
   2. 对国内设备优化较少, 又些特殊设备可能无法驱动.
2. 简约得优点简陋了, 大部分软件都需要额外添加
### 2. immortalwrt
我理解为国内优化版 openwrt. 尽量保持原汁原味的openwrt, 但是改进了国内使用的体验.
#### 2.1. 优点
1. 体验接近原版openwrt
2. 添加了一些原版没有的软件包支持
3. 更适合国内用户的软件源等优化
#### 2.2. 缺点
1. 缺少一些闭源的驱动支持, 部分设备可能兼容比较差.
### 3. lede
改动更大的版本, 提供了一些闭源驱动, 对设备兼容性更好.同样适合国内环境. 
#### 3.1. 优点
1. 支持的设备更多
2. 对内核做了很多优化, 对性能提升一定帮助
3. 砍掉了不必要的模块, 启动速度以及内存占用, 包大小都更好, 对性能有限的机器有很大的帮助.
4. 提供了详尽的编译指南, 只要照着流程就可以编译通过.
#### 4.1 缺点
1. 由于改动更大, 所以对openwrt 主线代码的功能无法保持那么快的同步, 新功能没办法第一时间用到
2. 同样由于改动的原因, 有些软件可能没办法直接使用.

## 编译前的准备
首先你需要一台 amd64 架构的机器(普通的64位intel/amd cpu 就可以), 至少 4gb 内存, ~~25g硬盘空间.~~ 实验证明, 25g 是不够的, 准备100g 的磁盘空间. 我实际花费了 42g.
操作系统需要ubuntu 或者 debian.
当然, 网络环境也是需要的, 安装过程中需要下载各种东西, 所以需要保证能链接github 等网站.

## 安装编译需要的软件
如果是第一次编译, 需要安装必要的软件包
可以使用这个命令一键安装
``` bash
sudo bash -c 'bash <(curl -s https://build-scripts.immortalwrt.org/init_build_environment.sh)'
```
## 下载代码
### 第一次编译
如果是第一次编译, 那么需要下载代码
```
git clone -b master --single-branch --filter=blob:none https://github.com/immortalwrt/immortalwrt
cd immortalwrt
```
### 更新代码
如果已经编译过了, 那只需要更新就行了
```
cd immortalwrt
git stash # 如果改了本地的代码, 更新前, 先备份一下数据, 如果没有改就不用执行
git reset --hard
git pull
git stash pop # 和上面的stash 配套使用, 如果改了代码, 可能会提示冲突, 需要自己看看怎么解决. 
```
## 安装需要的第三方源
可以在这里安装第三方源, 例如 helloworld 之类的. 不需要可以跳过. 
具体请参考你安装的第三方源说明

## 更新并安装源
不管是第一次还是更新代码, 都需要更新一下源.
```
./scripts/feeds update -a
./scripts/feeds install -a
```
## 配置编译选项
这一步配置需要的功能.
第一次执行必须要操作, 如果是更新, 且不需要安装新软件包的时候, 可以跳过.
```
make menuconfig
```
这里挑几个重要的说明一下

- `target system, subtarget` 这里选择cpu 架构, 这一步非常重要.
- `target image` 这里选择最终生成的安装包格式以及文件系统格式, 可以参考[这篇文章的第5节](https://www.cnblogs.com/Magiclala/p/18418732)
- 大部分要安装的软件都在`LuCI > Application` 里面
- 如果是要运行在 `esxi` 虚拟机里面, 除了 `target image` 选择 `vmdk` 之外, 还需要安装 `open-vm-tool`, 不然关机没办法优雅关机.
- 默认ip 地址是 192.168.1.1, 用户名root, 无密码.

### 修改默认ip 地址
现在有很多光猫的子网默认就是 192.168.1.1. 导致刷完系统后, 需要拔掉光猫配置.
其实可以编译的时候改一下这个默认地址.
1. `make menuconfig` 主页按 Y 勾选 `Image configuration`
2. 勾选后, 选择`Image configuration` 进入配置
3. 勾选 `Use preinit IP configuration as default LAN IP` 和 `Preinit configuration options`
4. 选择 `Preinit configuration options` 进入配置
5. 最后面有下面有三个ip地址, 分别是ip地址, 子网掩码, 广播地址, 根据需要修改即可.

[参考文档](https://git.openwrt.org/?p=openwrt/openwrt.git;a=blob;f=package/base-files/image-config.in;hb=HEAD#l76)

在编译完成后, 如果修改了任何编译选项, 都需要重新执行编译.
## 编译
这一步要花很久, 耐心等待. 电脑不要休眠, 可能会中断编译. 中断了也不用担心, 在执行一次命令就行了, 会从上次中断的地方继续.
```
make V=s -j$(nproc)
```
编译完成的镜像在 bin/targets/xxx/yyy 下面, 具体根据选择的目标不同而有所不同.

## 虚拟机的faq
1. 启动失败
生成的vmdk 如果直接导入 esxi 执行, 会提示 `scsi0:0 的磁盘类型 7 不受支持`
原因是生成的磁盘类型是 ide 的, 把硬盘的控制器改为ide 即可. 
如果改了也启动不了, 可以看看是不是开了安全 efi, 关掉再试试.
2. 虚拟机磁盘太小了, 啥也装不了
把 openwrt 关机, 然后在esxi 中修改硬盘大小. 然后重新开机后, 用命令或者ui 挂载剩余空间就可以了.

