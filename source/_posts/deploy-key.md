---
title: deploy key 的使用
date: 2020-05-04 18:58:02
tags: github
categories:
- 技术
- 其他
---
如果你和我一样, 想要使用自动提交, 那么你肯定需要了解了解以下deploy key 的使用.
deploy key 本质上是一个 ssh 公钥, 和你的个人公钥一样, github 使用这个公钥来辨识用户.
<!-- more -->
## 是什么
既然和个人公钥一样, 那么使用deploy key 的意义何在? 最主要还是权限管理问题, 我列举了一个表格
个人密钥具可以用来代表用户, 用户下面的所有仓库都有读写权限. 
自动提交系统通常部署在其他服务器上, 如果不小心被其他人盗走了私钥, 则会导致难以承受的后果. 
相反, deploy key需要手动授权给指定的仓库, 对其他的仓库没有权限, 这样就大大降低了泄漏的成本, 而且, 由于这种自动提交的库, 往往是自动生成的数据, 即使万一不小心被别人恶意覆盖数据, 也可以重新生成.
## 怎么用
1. 首先, 和普通公钥对一样, 你需要先实现准备好公私钥对.
2. 在github 中找到特定的仓库, 点击 setting->deploy key->Add deploy key.
   ![](
https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/deploy-key/deploy-key01.png)
3. 在弹出的界面中输入之前准备好的公钥, 如果需要写入权限, 则钩上write access.

搞定收工.


