---
title: Go Mod 初尝
date: 2020-11-10 16:17:00
tags: 
- golang
categories:
- 技术
- golang
---
众所周知, 在旧版本的 golang 中, 代码是需要放在 $GOPATH 目录下的, 其他的第三方依赖项目也是放在这个目录下, 难免有些混乱.
终于在新版本的 golang 中(golang 1.11 引入, 1.12 可以正式使用), 终于可以将项目代码与其他依赖分离出来了.
<!-- more -->
使用方式很简单, 几个步骤就可以了.
## 1. 启用 GOMODULE
命令行中输入 `go env -w GO111MODULE=on` 即可启用.
## 2. 创建项目并初始化.
在 $GOPATH 之外的任何地方, 新建项目, 并使用 `go mod init 项目名` 初始化项目.
创建好之后, 会自动新建一个 go.mod 文件, 这个文件用于存放依赖的名称, 版本等.
## 3. 使用
正常编写代码即可, 需要使用第三方依赖时, 直接编写即可, 用到的第三方依赖会在编译/运行时自动下载.
如果只是简单使用, 这样就完全足够了.
如果需要调整第三方库的版本, 可以手动修改 go.mod 文件即可.
## 4. 内部引用.
如果有项目内部的自模块需要引用, 也可以很方便的使用. 例如项目结构如下:
``` text
├── main.go
├── go.mod
└── tool
	└── tool.go
```
在main 中引用时, 填写 `import [模块名]/tool` 即可.
例如
``` golang
import (
	"gomodlearn/tool"
)
```
## 5. 关于访问加速
总所周知, 大部分第三方库都在国外服务器上, 直接使用往往都比较慢, 如果启用了 go mod, 而且你的golang 版本比较新(>= 1.13), 那么就可以使用模块代理功能加速访问. 
国内可以直接使用 [goproxy.cn](https://goproxy.cn/) 加速访问.
只需要在命令行中输入 
`go env -w GOPROXY=https://goproxy.cn,direct`
就可以享受超快的速度.
## 6. 进一步了解阅读
由于水平有限, 这边文章只是简单介绍, 甚至充满了各种错误.
要深入学习, 可以阅读这些文章:
[go mod 使用-掘金](https://juejin.im/post/6844903798658301960)
[go modules 官方文档](https://github.com/golang/go/wiki/Modules)
[干货满满的 Go Modules 和 goproxy.cn-掘金](https://juejin.im/post/6844903954879348750)
[goproxy.cn官网](https://goproxy.cn/)
欢迎留言讨论.