---
title: IP 地址介绍
date: 2019-12-06 16:34:00
tags: 
- IP
- 网络基础
categories:
- 技术
- 基础
---
互联网中, IP 地址意味着你电脑在互联网世界的门牌号, 不过很多时候, 我们并没有想象中的那么了解他.
<!-- more -->
## IP 地址是什么

了解计算机网络的同学都知道网络的7层结构.
如果忘掉的同学, 一起来回忆一下, 从下网上, 分别是物理层, 数据链路层, 网络层, 传输层, 表示层, 会话层, 应用层.
IP 地址是 IP 协议用于区分的数据标签.
由于IP 协议目前流行的有 IPv4 和 IPv6 两种, 所以地址也有两种. 他们两种之间互不兼容, 也就是说, 如果你电脑只配置了 IPv4 协议, 是无法**直接**(当然可以通过代理之类的间接连上)连接到一个IPv6 地址的, 反过来也不行.

## IPv4

IPv4 地址一共32位二进制, 我们通常习惯按每 8 位分隔一下, 然后将每 8 位使用十进制表示出来, 每一位就 从 0 到 255, 这种方式也叫点分十进制, 例如你的路由器地址可能是 192.168.1.1.
知道了 IP 地址的格式, 那么接下来就可以开始分配 IP 地址了. 
不过有些 IP 有特殊用途, 这部分要挑出来.
一共有哪些呢?
0.0.0.0-0.255.255.255 这个区段, 用来表示本网络, 不过只能用于设置源地址的时候有效. 例如我们通常在写一个网络服务时候, 监控的IP地址就写的 0.0.0.0 表示你要能连接到我任意一个 IP 地址, 就可以提供服务.

### 子网表示方法

0.0.0.0-0.255.255.255 这个区段叫做一个子网段
除此之外, 还有另一个表示方法: 我们使用 0 来表示 IP 地址中变化的位, 用 1 来表示不变的位, 那么这个网段就可以表示为 0.0.0.0/255.0.0.0 是不是有点熟悉? 没错, 这个 255.0.0.0 就是你经常看到的那个子网掩码. 
由于IP地址在划分网段时, 往往都末尾若干位保持变化变化, 前面一截保持不变, 所以我们可以使用数前面有多少个 1 的方式表示子网掩码, 例如 0.0.0.0/8 就表示这个子网从 0.0.0.0 开始, 前面8位保持不变. 

说完了子网, 我们继续回到特殊用途的 IP 地址中, 这次我们使用 a.b.c.d/e 的方式表示子网.

### 特殊的IPv4 地址

| 子网                                           |                                         说明                                          |
| :--------------------------------------------- | :-----------------------------------------------------------------------------------: |
| 0.0.0.0/8                                      |                                 本网络, 仅用于源地址                                  |
| 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16      |               私有网络, 也就是我们常用的局域网络. 可用的 IP 地址依次减少                |
| 100.64.0.0/10                                  | 运营商级NAT专用, 也就是给运营商用的局域网地址, 经常你使用光猫拨号后就有可能是这种地址 |
| 127.0.0.0/8                                    |                    本机环回地址, 经常用到的127.0.0.1 就是这里面的                     |
| 169.254.0.0/16                                 | 本地链路地址, 如果你电脑没插网线, 或者是局域网内没有开启 dhcp 服务就有可能是这个地址  |
| 192.0.0.0/24                                   |                    用于 IANA 的 IPv4 特殊用途地址表, 你应该用不到                     |
| 192.0.2.0/24,  198.51.100.0/24, 203.0.113.0/24 |                   用于写文档的时候用的虚假 IP 地址, 不应该实际使用                    |
| 192.88.99.0/24                                 |                                      废弃的字段                                       |
| 198.18.0.0/15                                  |                                       测试用途                                        |
| 224.0.0.0/4                                    |                                      多播用地址                                       |
| 240.0.0.0/4                                    |                   保留给未来用的, 不过 IPv6 快普及了, 估计用不着了                    |
| 255.255.255.255/32                             |                       这个网段就一个 IP 地址, 用来发送广播用的                        |

除了这些之外, 还有些IP 地址也是不分配给计算机的, 通常有两类, 子网末尾全为 0 和 全为 1 的, 例如  192.168.1.0/24 子网下的 192.168.1.0/32 和 192.168.1.255/32 这两个 IP 地址
子网中末尾位全为 0(二进制位) 的IP,  地址通常用来表示这个网段, 不用来分配给特定的计算机.
子网中末尾位全为 1(二进制位) 的IP 的地址通常用于广播, 也不分配. 
可以参考维基上的[说明](https://zh.wikIPedia.org/wiki/IPv4#%E4%BB%A50%E6%88%96255%E7%BB%93%E5%B0%BE%E7%9A%84%E5%9C%B0%E5%9D%80)
其他的IP地址就是正常分配给互联网上设备的, 也就是我们常说的公网IP地址.
这些特殊的IP 地址中, 有几个比较特殊, 我们详细讲讲

#### 环回地址

127.0.0.0/8 这个网段用来表示环回地址, 我们常用的 127.0.0.1 就在里面, 但是这个网段有好多 IP 地址, 其他的 IP 地址呢? 我们感觉从来没用过? 还有这玩意有啥用?
环回地址的作用基本有两个: 

1. 本地通信: 本机的两个服务之间通信, 可以直接使用这个地址, 不需要复杂的查询网卡 IP 地址的操作. 直接用就行了.
2. 检查协议: 可以用来检查上层协议是否工作正常, 排除网卡的问题.

另外,  使用环回地址是所有发网环回地址的数据包都会在数据链路层直接返回, 不会经过网卡, 即使网卡负载很高也不影响.

##### 其他地址去哪了

除了 127.0.0.1 之外的其他地址, 并不是所有操作系统都支持, 即使支持, 也需要手动配置, 写服务监听的时候, 也需要一起监听才行, 是在是多余.
所以我们一般用不到其他的 IP 地址. 像 IPv6 就聪明很多, 只分配了一个 ::1 给环回地址.

#### 本地链路地址

当你的电脑无法使用 `dhcp` 获取 IP 地址时, 你可能会看到电脑自己弄出了一个 169.254.0.0/16 网段的地址. 然后提示你无法访问互联网.
这个地址就是本地链路地址, 在 IPv4 下, 如果配置了正常的 IP 地址, 就不会有了, IPv6 则是一直都有本地链路地址.
如果你知道另一个计算机的本地链路地址, 并且你们在同一个局域网下, 也是可以使用本地链路地址正常通信的.

#### 多播/组播地址

IP 地址中, 有一部分是给组播用的. 组播有些时候也翻译为多播. 这两个词是一个意思, 都是 multicast 的翻译. 组播, 顾名思义, 你可以使用组播地址, 给一群计算机发送消息.
著名的 mdns(muticast DNS) 就是基于多播实现的. 
具体如何实现呢? 我这里有一个[简单的demo](https://gist.github.com/ywwzwb/c9ed8198df0330903d304f4fa1330f92)可以参考一下

## IPv6

IPv6 与 IPv4 最明显的区别就是长度不一样, IPv4 只有32位(二进制位)长, IPv6则有128位长, 足足是 IPv4 长度的4倍.
如果我们按顺序分配, 大约一共有2^128≈3.4×10^38 个IP 地址, 当然, 实际并不会按顺序分配, 也 IPv4 一样,  也有一部分 IP 地址有其他用途.
不过即使这样, 给每一个沙子都分配一个 IPv6 地址也还是绰绰有余的.
正是 IPv6 的大量地址, 万物互联, 物联网才得以发展.
当然, IPv6 除了地址数量多之外, 也有其他的特点, 例如与生带来的安全性, 高扩展性等等.

### IPv6 的表示方法

#### 16进制分隔

IPv6 地址很长, 如果还是使用点分十进制的方式表示, 就太长了.
比如一个IPv6 的地址可能是这样 202.12.23.93.4.91.99.4.128.33.23.65.77.45.23.123(我随便写的) 这也太长了.
IPv6 下一般使用:分隔, 也不用十进制了, 使用16进制表示地址, 并且每4位数分隔一次. 提及几乎缩短一倍.
例如:
2001:0db8:85a3:08d3:1319:8a2e:0370:7344
看起来还是很长对不对, 不过有很大一部分 IP 地址中间其实有连续的0. 例如
2001:0DB8:02de:0000:0000:0000:0000:0e13
对于这些0, 我们可以省略一下. 首先, 先导的 0 可以省略, 就和十进制下 0012 和 12 是相等的一样.
2001:DB8:2de:0:0:0:0:e13 这个是等价的, 是不是瞬间段短了很多.
还可以继续省略一下, 连续的 0 可以使用 :: 省略, 不过你只能使用一次.
2001:DB8:2de::e13 这样终于顺眼了很多了.
有些 IP 地址可以通过这样省略得很短, 例如本地环回地址 ::1
IPv6 的子网表示方法和 IPv4 类似, 不过由于地址过于冗长, 几乎都使用类似 2001:0DB8:02de::/48 的格式来表示.

#### IPv4 映射地址

IPv 6中有部分地址是由 IPv4 地址映射来的. 他们前缀是 ::ffff:xxxx:xxxx
这部分 IP 地址可以也可以写成 IPv6 和 IPv4 混合的形式.
例如 ::ffff:c0a8:5909 这个地址, 和 ::ffff:192.168.89.9 是等价的.

### IPv6 地址的分类

和 IPv4 类似, IPv6 中IP 地址也有分类

#### 单播地址

就是普通的 IPv6 地址, 数据发往一个特定的节点并处理

#### 多播地址

ff00::/8

和 IPv4 的多播类似. 数据会被一组节点接收数据.

#### 任播地址

任播像是多播和单播的结合, 和多播一样, 可以有一组节点加入任播的地址列表. 但是当第一个计算机返回之后, 数据就不会继续传输到其他节点了, 这个只有一个节点能响应的特性又像单播一样.

#### 特殊的IPv6 地址

##### 未指定地址

::/128
相当与 IPv4 的 0.0.0.0, 一般在服务监听的时候使用.

##### 本地环回地址

::1/128
不同与 IPv4 中有一堆却没啥用的环回地址, IPv6中只有一个

##### 链路本地地址

fe80::/10
和 IPv4 不一样的是, IPv6 下, 总是有链路本地地址.
本地链路地址只适用于单个链路, 不能跨子网.

##### 唯一本地地址

fc00::/7
这个类似于 IPv4 的私有网络地址, 可以跨链路, 可以跨子网. 
IPv6 目前了解较少, 以后深入学习后再补充
