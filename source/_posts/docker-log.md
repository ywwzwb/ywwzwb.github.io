---
title: docker 随笔
date: 2020-05-05 13:58:00
tags: 
- docker
categories:
- 技术
- docker
---
这是在学习docker 过程中的一些随笔.

<!-- more -->

## 关于 docker 的虚拟化.
总所周知, docker 的虚拟化和虚拟机最大的不同就是docker 容器是直接在宿主机的操作系统上运行的, 不需要一整套的虚拟化硬件.
这个时候, 通常会有一个配图:
{% asset_img docker.png docker %}
{% asset_img virtualization.png 虚拟机 %}
上图是docker 的虚拟化, 下图是传统虚拟机, 可以看到传统虚拟机在虚拟机里面安装整套的操作系统, 虚拟机需要准备虚拟的cpu, 内存, 硬盘等设配.
而docker则是直接运行在宿主机上, 使用 Linux 的各种黑科技将进程隔离起来.
### 那么问题来了
在 Linux 的宿主机上很好理解, 因为 docker 容器里面的进程通常也是基于 Linux 的, 所以我们只需要做好进程之间的隔离就好了, 进程可以直接在宿主机上执行. 
那么 windows 呢? mac 呢? windows 和 mac 这种并不是 Linux 操作系统的如何怎么办? 由于操作系统内核的不同, 进程并不能直接在宿主机上运行.
那问题来了, 这个时候怎么办?
以mac 操作系统为例, 我们[查阅文档可知](https://docs.docker.com/docker-for-mac/docker-toolbox/)
mac 版本的 Docker for desktop 是基于 HyperKit 来开发的, HyperKit 可以理解为一个虚拟机. 这样, 一切就很清晰了.

{% asset_img docker-on-mac macOS 下的系统结构 %}


## docker 的多段构建 
基本上来说, 我们编译需要用到的工具在运行大多都用不到, 如果一起都打包一个镜像中, 就显得特别臃肿.
例如, 我们有一个 hello world 项目, 使用golang 编写. 
``` golang
package main

import "fmt"
func main() {
	fmt.Println("hello, world")
}
```
如果写一个普通的 DockerFile, 大概是这样.
``` Dockerfile
FROM golang:1.9-alpine
WORKDIR /go/src/helloworld/
COPY ./ ./
RUN go build && \
	cp /go/src/helloworld/helloworld /bin/helloworld
CMD ["/bin/helloworld"]
```
这样构建之后, 大概尺寸有 244MB
显然, 对于 hello, world 这种程序来说太大了.
事实上这里面占大头的是 golang 自身的构建环境, 而运行的时候我们并不需要这些.
那么我们可以采取分段构建, 让运行时尽量的小, 那么最后的 DockerFile 大概长这样
```Dockerfile
FROM golang:1.9-alpine
WORKDIR /go/src/helloworld/
COPY ./ ./
RUN go build

FROM alpine:latest
COPY --from=0 /go/src/helloworld/helloworld /bin/helloworld
CMD ["/bin/helloworld"]
```
最后构建出来的镜像只有 7.47MB. 真香.
当然, 有时候为了调试, 你可能需要单独构建编译部分, 这个时候就只需要在 FROM xxx 之后加上 as 就好了.
```Dockerfile
FROM golang:1.9-alpine as builder
...
```
这样在构建的时候, 可以使用 --target 来指定需要构建的段.

``` bash
docker build --target builder -t hello:builder .
```
构建后面的段时, 前面的段会自动构建, 不用担心依赖问题.
在上面的 Dockerfile 中, 我们看到有一句 
``` Dockerfile
COPY --from=0 /go/src/helloworld/helloworld /bin/helloworld` 
```
其中的 `from=0` 表示从第一个构建出来的镜像中拷贝文件, 
如果我们用了 `FROM xxx as xxx` 的时候, 也可以用这个别名
例如:
```
COPY --from=builder /go/src/helloworld/helloworld /bin/helloworld` 
```
除了从自己构建出来的镜像中拷贝文件, 也可以从其他镜像中拷贝文件.
``` Dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```
如果镜像不存在, 会自动下载, 放心好了.