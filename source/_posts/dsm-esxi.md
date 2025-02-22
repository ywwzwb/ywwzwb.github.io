---
title: 在 esxi 下安装群晖系统
date: 2021-01-19 13:58:00
tags: 
- esxi
- 群晖
- 直通
- XPEnology
categories:
- 技术
- 其他
---

之前一直用的 hp gen8 裸装的黑裙, 最新终于有钱升级系统了, 搞了一套新的配置, 系统性能还可以, 打算用虚拟平台安装群晖.
<!-- more -->
## 硬件配置
1. cpu: e5-2670 v3, 10核20线程, 基本上够用了, 买自某宝, 卖家说是正式版, 用cpu-z 看了一下, 貌似确实是正式版, 价格 400 左右.
2. 内存: 16g * 2 regecc ddr4 内存, 一共500.
3. 主板: 来自某群友的星能主板, x99 芯片组, 带10个sata 接口, 1个pci-e 接口, 4个 8643 接口, 2个万兆, 2个千兆. 短时间内基本够用.  唯一的缺点是不带远程控制, 安装系统需要使用插上显示器才行.
4. 电源: 全汉的拆机电源, 非模块电源, 450w, 100 左右
5. 硬盘: 旧笔记本的拆机硬盘, 500g, 机械硬盘.
6. 机箱还在咕咕咕.

因为哥们儿的原因, 这些设备基本都是2手, 先凑合用着吧, 内存以后如果不够还可以扩充, 硬盘以后再从 gen8 上迁移过来.
## 操作系统选择
基本上几个方案: 
1. 直接安装黑裙
2. 更换系统为 unraid
3. pve/esxi + 虚拟化黑裙

首先, 直接安装黑裙就算了, 毕竟这套平台对黑裙实在是性能过剩.
unraid 之前也简单安装了, 试用了一下, 但是感觉使用诸多不便. 连在线文件管理都没有, 自带的虚拟机管理平台快照功能也不支持, 简直无语.
最后还是决定直接使用虚拟化平台, pve没有用过, vmware 系列的软件平时用的比较多, 就拿这个开刀了.
## esxi 下黑裙的安装
首先, 建议你读一下xpenology 的[这篇教程](https://xpenology.com/forum/topic/13061-tutorial-install-dsm-62-on-esxi-67)
其次, 你需要一个Windows电脑/虚拟机, 用来做转换文件之类的工作.
然后可以开始了.
### 需要准备的文件: 
####  synoboot vmdk
这个可以从 [V1.01 的loader](https://mega.nz/file/fdBWBJYB#P3MbGY2v_X_udUhaSgVBQZ74KNRf7vtjMCO39u1I91Y) 中找到, 下载完成后, 解压就可以看到了.
#### loader
这里使用[1.04b 的 loader](https://mega.nz/#F!Fgk01YoT!7fN9Uxe4lpzZWPLXPMONMA), loader 的选择可以参考[这篇文章](https://xpenology.com/forum/topic/13333-tutorialreference-6x-loaders-and-platforms/), 你可以去 [loader 的发布页面](https://xpenology.com/forum/topic/12952-dsm-62-loader/) 看看有没有更新.
#### 系统安装包
这个从[群晖官网下载](https://archive.synology.com/download/Os/DSM)即可, 我上面选的loader 是 918+ 的, 安装包也要选择这个型号才行. 1.04b 支持的系统版本是6.2.0, 6.2.3. 系统也需要正确选择, 不同版本loader 对群晖系统的兼容性列表在[这里](https://xpenology.com/forum/topic/13333-tutorialreference-6x-loaders-and-platforms/)
如果你照着我的教程, 那么6.2.3 的下载地址应该是[这个](https://global.download.synology.com/download/DSM/release/6.2.3/25426/DSM_DS918%2B_25426.pat)

#### XPEnology Tool 工具集
这其实是一堆工具的集合, 比如 notepad++, OSFMountPortable.
发布页面在[这里](https://xpenology.com/forum/topic/12422-xpenology-tool-for-windows-x64/)
你也可以点[这个链接直接下载](https://mega.nz/#F!BtViHIJA!uNXJtEtXIWR0LNYUEpBuiA)
后面会用到这个来修改img 文件.

#### 修改loader
解压上一步下载的XPEnology Tool, 
打开OSFMountPortable x64 文件夹中的OSFMountPortable 程序.
点击Mount new, 选择之前下载的loader, 询问分区时, 选择分区 0, 记得去掉readonly 之前的勾.
![挂载img](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/esxi-dsm/change%20loader.png)
最后点击完成. 文件管理器中会多出一个磁盘. 
打开磁盘中的grub 文件夹, 用notepad++ 打开grub.cfg文件. notepad ++ 在 工具包中的npp文件夹中.
找到 menuentry, 1.04b 的一共有三个, 将Baremetal 的两个段删除.
![编辑grub](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/esxi-dsm/edit-grub.png
)
另外, 在第29行处, 将 `set extra_args_918=''` 修改为 `set extra_args_918='DiskIdxMap=100004 SataPortMap=144'`
DiskIdxMap 和 SataPortMap 的值可以按需修改.
SataPortMap 是用于设定硬盘控制器的, 每一位代表一个控制器, 说明该控制器下接多少个硬盘, 比如, 144 表示有三个控制器(3位数), 第一个控制器下能添加一个1个硬盘, 剩余两个控制器, 每个能添加4个硬盘.
DiskIdxMap 用于设定硬盘顺序, 当插入多个硬盘时, 每一个硬盘的索引. 这个数据是16进制数据, 两位数为一组. 第一组 10 用来表示第一个控制器的第一个硬盘, 索引从10 开始, 也是就是十进制的16, 第二个00 表示第二个控制器的第一个硬盘索引从0 开始, 第三个控制器的硬盘从4 开始.
这里面比较特殊的是第一组控制器, 我们将它设置为10 是因为群晖默认情况下, 最多支持16个硬盘, 索引就是从 0 - 15, 将索引设置为 16 显然超出了这个范围, 就是为了隐藏这个硬盘, 因为我们第一个硬盘控制器的第一个硬盘(sata 0:0)会将它设置为引导盘, 不需要也不应该在磁盘管理器中显示出来. 
如果需要修改sn, mac 地址之类的, 也一并修改这个文件内容.
mac 地址地址需要拷贝出来, 之后创建虚拟机需要用到, 如果你没有修改应该是00:11:32:12:34:56.
保存文件后, 回到文件夹中的OSFMountPortable 程序中, 点击下方的dismount all & exit.
这时, loader 就修改完成了.

#### 将文件上传到虚拟机的存储中.
打开 esxi, 选择存储, 打开数据存储浏览器, 将loader 和 synoboot.vmdk 上传上去, 放在同一个目录下面.
### 创建虚拟机

1. 选择新建虚拟机
2. 操作系统选择 linux, 其他 linux4.x (64位)
3. 自定义设置中, 删掉默认的磁盘, 默认的scsi控制器, cd/dvd, usb设备.
4. cpu, 内存看心情, 后期不够可以加, 测试的话, 1个cpu, 2g内存就可以了.
5. 网络适配器型号改成e1000e, mac 地址改成之前grub.cfg 中的即可.
6. 现在添加硬盘, 选择现有硬盘, 选择刚才上传的synoboot.vmdk 即可, 控制器位置设定在0:0.
7. 打开虚拟机选项, 引导选项, 将启用uefi 安全引导的勾选框去掉.
8. 如果你需要直通整个AHCI设备的话, 就可以开始直通了.
9. 如果你只是测试用, 没有直通整个 AHCI 设备的话, 那么这时候再新建一个sata设备.之后就可以添加数据盘了, 大小随便, 放在这个新建的这个sata设备下即可.

![虚拟机配置](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/esxi-dsm/vm_config.png)

上图中系统版本选择错了, 最新的群晖系统, linux内核已经更新为4.4了.
### 开机试试
之后可以开机了. 之后虚拟机就不用管了.
打开群晖助手, 就可以找到你的机器了.

### 安装群晖
进入系统后, 点击设置, 手动安装, 选择之前下载的DSM_DS918+_xxx.pat
接下来就是自动安装了.
需要注意的是, 除非你洗白了, 否则不要开启quickconnect 服务. 
另外, 安装完成后, 记得在设置中, 将系统更新关闭.
## 其他优化
### 直通sata
如果要长期使用的话, 肯定是要添加大量硬盘到群晖的, 而如果直接在esxi 中初始化磁盘, 并在其中创建虚拟磁盘连接到群晖显然不太现实. 原因主要为以下几点

1. 虚拟盘连接到群晖是无法读取 smart 信息的, 这样如果硬盘损坏就完蛋.
2. 直通通常也会性能更高. 
3. 直通后, 即使硬盘损坏, 更换硬盘时, 不需要去esxi 中初始化, 直接替换硬盘即可, 如果群晖有设置shr, raid, 那么就可以自动开始恢复数据了.

不过也有缺点
1. 直通后, 无法对直通的硬盘打快照
2. 直通时, 需要完整的内存分配好, 比如设定的是2GB, 则虚拟机启动后, 就会占用2GB.
3. 直通往往是整个sata控制器, 如果你电脑只有一个sata控制器, 那么直通后, 你可能需要想想把你的esxi放哪.

那么我们开始直通sata 设备.
#### 首先, 需要在主板中开启虚拟化vt-d 支持
这个需要在bios 中设置, 并需要cpu 的支持.
#### 启用直通
默认情况下, esxi 是没有直通sata设备的, 你需要手动将硬盘直通到群晖.
#### 查询设备pid, vid
打开esxi 的管理平台, 依次打开管理->硬件->PCI 设备
右上角输入 ahci 或是 sata 筛选, 找到 sata 控制器后, 点击设备, 在下方会显示设备的 vid, pid.
![读取pid, vid](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/esxi-dsm/%E7%9B%B4%E9%80%9A1.png
)
找到将pid, vid记下来, 如果你由多个设备, 而你想直通其中一个怎么办? 如果型号不一样就好办, 否则就只有靠猜.
#### 修改passthrough
这一步需要用到ssh 登陆到命令行, 并使用vi 命令修改文件, 如果你不会, 可能需要先了解一下.
首先, 打开管理平台, 依次打开主机->操作->服务, 启用安全shell 就行了.
![启用ssh](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/esxi-dsm/%E7%9B%B4%E9%80%9A2.png)
接下来, 使用ssh 登陆到 esxi 命令行, 编辑/etc/vmware/passthru.map 文件, 在末尾添加上这样一行
```
vid pid d3d0 false
```
例如
![编辑passthru.map](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/esxi-dsm/%E7%9B%B4%E9%80%9A3.png)
保存退出后, 记得禁用安全shell.
回到管理平台, 重启主机.
#### 启用直通
重启主机后, 回到PCI 设备, 找到之前的 sata 控制器, 点击切换直通, 之后可能会报错, 不用理他, 修改完成后重启就好了.
![切换直通](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/esxi-dsm/%E7%9B%B4%E9%80%9A4.png)

#### 为虚拟机添加直通设备
首先, 需要将内存改为固定分配.
编辑虚拟机, 内存->预留所有客户机内存 (全部锁定)
之后, 点击添加其他设备, pci 设备就好了.

### 安装vmware-tool
安装vmware tool 后, 可以使用更快的vmxnet3 网卡, 可以安全关机, 也对虚拟机运行有一定优化, 推介安装.
github 上已经有人将 vmware tool 编译为spk 安装包了, 我们直接下载后安装就行了. 
注意, 需要根据不同的设备类型选择不同的安装包型号, 详情查看文档:
[https://github.com/leonardw/synology-open-vm-tools](https://github.com/leonardw/synology-open-vm-tools)
如果是 918+ 的话, [点这个链接就好了](https://github.com/leonardw/synology-open-vm-tools/releases/download/release-11.0.1-1/open-vm-tools_apollolake-6.1_11.0.1-1.spk).

### 启用转码(半洗白)
我们在上面设置的SN 地址虽然能开机, 但是如果你安装了 moments, dsvideo 后会发现无法转码, 无法生成缩略图, 这个时候, 可以考虑修改SN 与MAC 地址, 也称为半洗白.
#### 取得合法的 SN 与 mac 地址
首先, 你需要一个配对好的 sn 与 mac 地址. 可以参考这篇文章[获取群辉官方提供的免费序列，适用2019年9月份更新之后](https://post.smzdm.com/p/akm7wqx9/)的教程. 
简单来说通过群晖自带的 vitrual machine manager, 创建一个虚拟的群晖, 这个群晖是有正确的序列号的, 然后将这个虚拟群晖里面的 mac 地址与 sn 拷贝出来.
我在实际操作时, 发现这一步在虚拟机机上的群晖上不好使, 我启动后, vmm 里面的机器无法获取ip地址, 找了一台电脑直接安装群晖后倒是没问题, 你们可以也可自己试试.
当然, 这个 sn 自然是不能用于 QuickConnect的. 
#### 使用 SN 与 MAC 地址
拿到匹配的 sn 与 mac 地址后, 可以参考之前的步骤, 修改 img 文件, 重新用这个img 文件作为启动盘. 
也可以参考这篇文章 [修改群晖中的SN和MAC](https://www.jianshu.com/p/d1a09eaace3c) 直接修改. 不过需要注意的是, 教程中第六步中的 mount -t vfat synoboot1 /tmp/boot/ 这个命令需要修改. 
改为 `mount -t vfat sdxx /tmp/boot/`
sdxx 有可能是 sda1, sda2, sdb1 等, 你需要一个个去测试.
最后记得修改网卡的 mac 地址.
#### 验证 SN
修改好之后, 可以去信息中心查看 sn 是否正确. 如果没问题, 可以在命令行中输入这个命令, 检查是否是合法的 sn
``` bash
cat /usr/syno/etc/codec/activation.conf
```
如果结果返回类似下面的内容, 那么证明没问题了.
``` json
{"success":true,"activated_codec":["h264_dec","h264_enc","hevc_dec","mpeg4part2_dec"],"token":"xxx"}
```

## 参考资料
[ESXI 网卡等PCI设备硬件直通配置](https://blog.csdn.net/qiaohewei/article/details/108621848?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.control)
[ Tutorial: Install DSM 6.2 on ESXi 6.7](https://xpenology.com/forum/topic/13061-tutorial-install-dsm-62-on-esxi-67)
[ESXI 7.0直通板载SATA控制器](https://kzpu.com/archives/4269.html)
[HPE Gen10 Plus ESXi 7.0 虚拟化安装黑群晖 DS918+](https://www.opsit.cn/5859.html)
[黑群晖 DS918+ 修改引导参数隐藏引导盘和数据盘](https://www.opsit.cn/5931.html)
[获取群辉官方提供的免费序列，适用2019年9月份更新之后](https://post.smzdm.com/p/akm7wqx9/)
[修改群晖中的SN和MAC](https://www.jianshu.com/p/d1a09eaace3c)
[DS918+已有SN码 无法半洗白](http://www.nasyun.com/thread-72407-1-1.html)