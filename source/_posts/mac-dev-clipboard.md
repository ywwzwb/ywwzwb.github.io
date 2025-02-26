---
title: mac-dev-clipboard
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
总所周知, 剪贴板是可以在登陆了同一个 Apple ID 的不同设备之间共享的. 所以如果两个设备都运行了这个程序, 那么只要有一个设备剪贴板有变化, 那么另一个设备也会收到通知.
看起来是不错, 但是这样会给我们带来一个问题. 读取远程剪贴板是非常缓慢的过程, 而且会出现一个提示框.
这样非常不优雅, 所以需要一个更好的方案. 
![非常慢](busy.png)
经过一系列探索, 最终发现在读取剪贴板时, 可以读取到剪贴板的中的数据类型, 如果远程剪贴板, 那么会有这样一个字段`com.apple.is-remote-clipboard`
![](remote.jpg)
而且更好的是, 读取这个类型相对较快, 也不会触发剪贴板同步的提示.
所以我们只需要读取这个类型, 然后判断是否是远程剪贴板, 如果是, 那么我们就跳过这个.
```objc
NSInteger oldChangeCount = [[NSPasteboard generalPasteboard] changeCount];
while(true) {
    do{
        NSInteger newChangeCount = [[NSPasteboard generalPasteboard] changeCount];
        if(newChangeCount == oldChangeCount) {
            break;
        }
        NSLog(@"pastboard change!!");
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