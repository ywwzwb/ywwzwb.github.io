---
title: 如何在关联对象上使用 weak
date: 2017-07-31 15:16:46
tags: 
- objective-c
- ios
- mac
categories:
- 技术
- oc
---
要在 category 中定义属性, 唯一的办法就是使用关联对象. 
但是关联对象的存储方式只有 assign, retain, copy 三种, 并没有 weak. 想要使用 weak 属性就要自己想办法了.
<!-- more -->
例如
我们自定义类如下
``` objc
@interface MyObject : NSObject
@end

@implementation MyObject
@end
```
测试的时候, 在一个 viewcontroller 的 viewDidLoad 中, 我们定义一个 MyObject 对象, 并对其中的 weakObj 赋值.

``` objc
- (void)viewDidLoad {
    [super viewDidLoad
    self.myObj = [MyObject new];
    self.myObj.weakvalue = [NSObject new];
}
```
在 viewDidDisappear 中去获取值
``` objc
-(void)viewDidDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    NSLog(@"weak value is: %@", self.myObj.weakvalue);
}
```
如果我们 weakvalue 是一个使用 weak 定义的属性, 在 viewDidDisappear 中访问的时候将会变成nil.

那么如果需要使用关联对象, 需要如何存储这个 weakvalue 呢?

### 直接使用 assign

``` objc
void *weakValueKey = NULL;
-(void)setWeakvalue:(NSObject *)weakvalue {
    objc_setAssociatedObject(self, weakValueKey, weakvalue, OBJC_ASSOCIATION_ASSIGN);
}
-(NSObject *)weakvalue {
    return objc_getAssociatedObject(self, weakValueKey);
}
```
不用怀疑, 属性不会自动值为 nil, 如果在对象释放之后访问, 将会导致野指针错误
![崩溃了](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/weak-on-associate/943998-46ad4f6d3dcd8383.png)

### 解决方案
最简单的解决方案就是使用 block 包起来. 先来看代码
```
-(void)setWeakvalue:(NSObject *)weakvalue {
    __weak typeof(weakvalue) weakObj = weakvalue;
    typeof(weakvalue) (^block)() = ^(){
        return weakObj;
    };
    objc_setAssociatedObject(self, weakValueKey, block, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
-(NSObject *)weakvalue {
    id (^block)() = objc_getAssociatedObject(self, weakValueKey);
    return block();
}
```
运行结果
![image.png](https://zwb-hexo-image.oss-cn-chengdu.aliyuncs.com/weak-on-associate/943998-1b04f81c9c3d7d7d.png)

### 原理解析:

首先我们在 setter 方法里面使用了一个 weak 的局部变量 weakObj 来实现实际的 weak 对象储存. block 会捕获这个对象, 但是由于是 weak 对象, 并不会增加引用计数.
然后, 我们将这个 block 存放在关联对象中, 这样在 getter 中, 如果weakvalue 指向的对象还未释放, 则会正确返回, 如果已经释放, block 内部捕获的 weak 对象自然也会置空.

demo 在这里, [https://github.com/ywwzwb/AssociateWeakDemo](https://github.com/ywwzwb/AssociateWeakDemo)