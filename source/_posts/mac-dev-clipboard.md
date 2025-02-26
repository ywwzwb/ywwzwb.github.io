---
title: mac 剪贴板监控
date: 2025-02-26 10:11:22
tags: 
- mac
categories:
- 技术
- oc
---
最近做了一个监控剪贴版的 demo, 遇到一些坑, 顺带记录一下.
<!-- more -->

# 如何监控剪贴版变化

mac 并没有提供一个剪贴板更新的事件(AppKit 没有, UIKit 好像有一个). 所以只能通过轮询的方式来监控.
剪贴板有一个 `change count`, 可以轮询监控这个变化来监控到剪贴板变化了. 
那么我们就得到了一个demo
```objc
NSInteger oldChangeCount = [[NSPasteboard generalPasteboard] changeCount];
while(true) {
    do{
        NSInteger newChangeCount = [[NSPasteboard generalPasteboard] changeCount];
        if(newChangeCount == oldChangeCount) {
            break;
        }
        NSLog(@"pasteboard change!!");
        oldChangeCount = newChangeCount;
        NSLog(@"read string:%@", [[NSPasteboard generalPasteboard] stringForType:NSPasteboardTypeString]);
    }while(0);
    usleep(1000 * 100);// sleep 100 ms
}
```
这样我们就可以按照每 100 ms 的时间间隔轮询检查剪贴板变化了. 
# 问题
总所周知, 剪贴板是可以在登陆了同一个 Apple ID 的不同设备之间共享的, 那么只要有一个设备剪贴板有变化, 那么另一个设备也会收到通知.
看起来是不错, 但是这样会给我们的监控程序带来一个问题. 读取远程剪贴板是非常缓慢的过程, 而且会出现一个提示框. 在我这个场景下, 也不需要读取其他设备上的剪贴板.
于是乎, 上面的方案非常不优雅, 我们需要一个更好的方案. 
{% asset_img busy.png 非常慢 %}
经过一系列探索, 当然了, 这个肯定也是没有现成的 API 或者文档给你使用的. 
最初方案, 是计划通过CFPasteboard 的一些私有 API, 里面有一个非常有意思的接口 `CFPasteboardCopyFlavorsForItem`, 这个接口看起来可以读取剪贴板的元信息, 这点非常好, 但是由于不知道具体的数据类型, 以及返回类型的定义, 实在是难以下手. 
给这个函数打了一个断点, 发现在读取剪贴板类型时, 会调用这个接口, 而且更妙的是, 如果远程剪贴板, 那么类型列表中会有这样一个类型`com.apple.is-remote-clipboard`
{% asset_img remote.jpg %}
而且更好的是, 读取这个类型相对较快(我这里测试大约1秒左右, 完整读取剪贴板大约3秒左右), 也不会触发剪贴板同步的提示(暂时没看到).
所以我们只需要读取这个类型, 然后就可以判断是否是远程剪贴板, 如果是, 那么我们就跳过这个.
于是, 最终的代码如下:
```objc
NSInteger oldChangeCount = [[NSPasteboard generalPasteboard] changeCount];
while(true) {
    do{
        NSInteger newChangeCount = [[NSPasteboard generalPasteboard] changeCount];
        if(newChangeCount == oldChangeCount) {
            break;
        }
        NSLog(@"pasteboard change!!");
        oldChangeCount = newChangeCount;
        if([[NSPasteboard generalPasteboard] canReadItemWithDataConformingToTypes:@[@"com.apple.is-remote-clipboard"]]) {
            NSLog(@"remote object, skip");
            break;
        }
        NSLog(@"read string:%@", [[NSPasteboard generalPasteboard] stringForType:NSPasteboardTypeString]);
    }while(0);
    usleep(1000 * 100);// sleep 100 ms
}
```