---
title: Hexo 搭建记录
date: 2019-05-08 13:16:34
tags: 
- hexo
categories:
- 技术
- 其他
---
本站使用hexo 搭建, 仅以此文记录搭建过程.

## 最终期望效果
现在通过 github 部署博客源代码, 使用 action 自动发布, 再也不用担心我的服务器挂掉了
<!-- more -->
~~1. 本地编写文章, 也可以通过控制页面创建文章~~
~~2. 附带一个简单后台, 提供基本的控制, 如新建文章等~~
~~3. 使用 git push 到自己的仓库~~
~~4. git push hook 将文件推送到 hexo 指定目录
~~5. hexo 将 md 渲染 为 html~~
~~6. hexo 自动将文章推送到github~~
~~7. 带有评论功能~~
~~8. 支持docker 部署~~

## 进度记录

- [x] `Hexo` 环境搭建(2018-05-08)
- [x] ~~图片上传系统~~(自建服务器有风险, 现在使用阿里云和公共图床)
- [x] ~~简单的维护功能, 例如推送, 开启服务器~~(废弃掉了)
- [x] ~~git webhook~~(使用 action 自动推送 2025-02-23)
- [x] github 网站配置(2018-05-08)
- [x] NexT 主题
  - [x] 缺少分类页面(2018-05-08)
  - [x] 缺少关于页面(2018-05-08)
- [x] 自动渲染
- [x] 自动推送
- [x] 评论系统
  - [x] ~~LeanCloud 应用创建(2018-05-08)~~ 废弃掉了
  - [x] ~~valine 集成~~ 废弃掉了
  - [x] 使用 [utterances](https://theme-next.js.org/docs/third-party-services/comments#Utterances) 集成了评论系统, 会自动添加到 issue 中
  - [x] 评论系统回复通知, 现在回复也能通过 issue 自动通知了, 感谢大佬开源
- [x] ~~docker 集成~~ 不需要了
## 下面的内容已经过时, 不建议使用

## Hexo 环境搭建

我是在 `docker` 中创建的 `Hexo` 环境, 处于容器小型化的考虑, 使用 `alpine` 作为基础.
### 必要依赖
首先需要安装依赖, 主要是 `git` 和 `nodejs`, 需要注意的是,  `nodejs`  包不含 `npm` 工具, 需要使用 `nodejs-npm`

安装完依赖环境后就可以安装 `hexo` 了, 这一步可以参考[官方文档](https://hexo.io/zh-cn/docs/)
完整安装命令如下
``` bash
apk update
apk upgrade
apk add git
apk add nodejs-npm
```
### github 部署依赖
另外, 由于需要提交代码到 github 仓库, 使用 `ssh` 公钥认证系统可以免密码认证, 那就需要安装 `openssh`

``` bash
# 安装openssh
apk add openssh
# 生成公私钥对
ssh-keygen -t rsa -b 4096 -C "hexo-server"
```
### 其他杂项
#### 时区
如果你发现在新建文章时, 时间不对, 可能需要调整服务器的时区, `alpine` 设置时区需要安装 `tzdata`
``` bash
apk add tzdata
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
```
#### ssh 远程登录
由于我是在docker 建立的容器, 初始没有设置密码, 如果需要远程登录, 比需要先设置密码

同时, 为了方便, 还可以使用公钥认证.

``` bash
# 生成hostkey
ssh-keygen -A
# 修改密码
echo "root:xxxx" | chpasswd
# 创建.ssh 目录
mkdir -p ~/.ssh
echo "你的公钥" >> ~/.ssh/authorized_keys
sed -i "s/#PermitRootLogin.*/PermitRootLogin without-password/" /etc/ssh/sshd_config
sed -i "s/#PasswordAuthentication.*/PasswordAuthentication no/" /etc/ssh/sshd_config
```

需要注意的是, docker 的  alpine 不建议使用 `openrc` 来使 `sshd 自启动`, [见此回复](https://github.com/gliderlabs/docker-alpine/issues/42#issuecomment-109796726)

所以如果需要启动 ssh 可以使用 下面这两条命令的一个就行了, 需要开机启动就加入到你的启动脚本中即可.
``` bash
# 阻塞式
/usr/sbin/sshd -D
# 非阻塞式
/usr/sbin/sshd
```

#### rss 支持
这一步可以跳过, 如果需要支持rss订阅功能, 你需要额外安装rss 插件
``` bash
npm install hexo-generator-feed --save
```

## Hexo 配置

### github 仓库关联

关联后, 可以使用 `hexo deploy` 将文章发布到github
需要修改 `_config.yml` 并设置git 地址. 
``` yml
deploy:
  type: git
  repo: git@github.com:xxx/xxxx.github.io.git
  branch: master
```
然后将之前生成的公钥(路径:`~/.ssh/id_rsa.pub`)拷贝出来. 在 `xxx.github.io` 仓库的设置为 deploy key 即可.
{% asset_img deploy-key01.png github 设置 %}


### 关于readme.md 文件

git 仓库一般在根目录都有一个 readme 文件, 如果你需要添加 readme 到 git 仓库中, 只需要在source 中添加一个 readme.md 文件, 同时在配置文件中. 将此文件跳过渲染即可.
``` yml
skip_render:
  - readme.md
```

### 使用自己的域名

首先需要在域名的控制面板中, 添加一个域名, 并将域名cname 设置为你的github域名
如果使用自己的域名, 在source 中建立一个文件 CNAME, 并在其中填入你自己的域名即可
同时, 这个文件也需要跳过渲染, 同readme.md 类似.
``` yml
skip_render:
  - CNAME
```

## 使用 NexT 主题

### 安装

你可以在[官方文档](https://theme-next.org/docs/getting-started/)中查看详细教程.
基本来说就是克隆代码到 theme 文件夹中, 然后在 hexo 的配置文件中将 theme 设置为 next 即可.

``` yml
theme: next
```
### 配置

这些配置不改也可以用, 主要是一些附加功能.

修改配置需要编辑 /theme/next/_config.yml 文件
开启 CC 标签
``` yaml
creative_commons:
  license: by-nc-sa
  sidebar: true
  post: false
  language: deed.zh
```
开启 `fontawesome` 的图标支持
``` yml
vendors:
fontawesome: //cdn.jsdelivr.net/npm/font-awesome@4/css/font-awesome.min.css
```

显示 github 图标, 这里需要注意一下, 如果你按照这个配置设置, 就必须要开启 `fontawesome`, 否则就看不到图标了

``` yaml
social:
  GitHub: https://github.com/xxx || github
social_icons:
  enable: true
  icons_only: true
  transition: false
```

显示分类页面

在命令行输入:
``` bash
hexo new page categories
```
修改 source/categories/index.md, 页面的类型设置为 categories, 同时, 通常这个页面也不需要评论, 一并关闭了.

全文如下:
``` text
---
title: 分类
date: 2019-05-08 22:47:57
type: "categories"
comments: false
---
```

## git push hook 的设置

我自己使用的私人仓库是自建的 `bitbucket`服务器, 其他环境没有测试过.
以下均以 `bitbucket` 来说明
1. 开启访问权限
和 `github` 的 `deploy key` 类似, `bitbucket` 也有类似的, 叫 `access key`, 也可以设置读写权限, 这里我们只需要读取就行了.
{% asset_img repository.png g %}
1. 添加webhook
   `webhook` 原理很简单, `bitbucket` 会在用户触发某个动作, 例如push动作, 之后, 自动调用某一个 url, 在其中传入特定的参数, web服务器接收到这个请求之后, 再执行相应的动作就可以了.
   我们可以使用golang 来完成这个简单的任务.
``` golang
package main

import (
	"log"
	"net/http"
	"os/exec"

	"github.com/gorilla/mux"
)

// WebhookHandler handle web hook
func WebhookHandler(w http.ResponseWriter, r *http.Request) {
	log.Print("X-Event-Key:", r.Header.Get("X-Event-Key"))
	if r.Header.Get("X-Event-Key") != "repo:refs_changed" {
		// only handle push event
		w.Write([]byte("ok\n"))
		return
	}
	cmdout, err := exec.Command("/bin/sh", "./update.sh").CombinedOutput()
	if err != nil {
		log.Print("update failed:", err, "\n, output: ", string(cmdout))
		w.WriteHeader(500)
		w.Write([]byte("failed\n"))
		return
	}
	log.Print("update succeed: \n", string(cmdout))
	w.Write([]byte("ok\n"))
}
func main() {
	r := mux.NewRouter()
	r.HandleFunc("/", WebhookHandler)
	log.Fatal(http.ListenAndServe(":8123", r))
}
```
3. webhook 在收到请求时, 会触发同目录下的 update.sh 脚本, 我们在这个脚本中执行自动更新代码, 自动编译, 自动部署即可.
``` bash
cd "your clone path"
# pull code
git clean --force -d -x
git reset --hard
git pull
# hexo build and deploy
hexo generate --deploy
```
## 评论系统

评论系统使用 [valine](https://valine.js.org/), 评论后会提交到 [LeanCloud](https://leancloud.cn/)
使用这位同学提供的评论的回复推送: [赵俊的博客](http://www.zhaojun.im/hexo-valine-admin/)
喜大普奔, 有回复系统了, 可惜没有审核, 希望别凉了

## HTTPS
github 会自动为你生成https证书( github 赛高)
## 杂谈
### CC 协议的选择
CC 协议规定了文章的授权方式以及是否允许商业使用, 与开源代码协议类似
可以参考[这篇文章](https://zhuanlan.zhihu.com/p/20641764)选择.