---
layout:     post
title:      "ReactiveCocoa快速入门"
subtitle:   "ReactiveCocoa（其简称为RAC）是由Github工程师们开发的一个应用于iOS和OS X开发的函数响应式编程新框架。ReactiveCocoa为开发者带来了函数式编程和响应式编程的思想。"
date:       2017-01-18
author:     "yasin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 开发
---

[ReactiveCocoa地址](https://github.com/ReactiveCocoa/ReactiveCocoa)

从RAC 5.0开始，RAC进行了巨大的变化，现在有了4个独立的库

[ReactiveObjC](https://github.com/ReactiveCocoa/ReactiveObjC)

[ReactiveSwift](https://github.com/ReactiveCocoa/ReactiveSwift) 

[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 

[ReactiveObjCBridge](https://github.com/ReactiveCocoa/ReactiveObjCBridge) 

### 导入

用`cocoapods`导入

```
    target 'ReactiveCocoa_Demo' do
      pod 'ReactiveCocoa'
    end
```

导入头文件`#import <ReactiveCocoa.h>`

### 基本使用
我们先试试`UIButton`

```
    [[btn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {
        NSLog(@"ddd");
    }];
```
点击button会打印"ddd"说明，调用了block里面的内容。
看着可能有点复杂，但是不熟的时候可以拆开写

先创建一个`RACSignal`对象，再添加要执行的block。
```
    RACSignal *racSignal = [btn rac_signalForControlEvents:UIControlEventTouchUpInside];

    [racSignal subscribeNext:^(id x) {
        NSLog(@"ddd");
    }];
```
`RACSignal`是RAC里面一个非常重要的概念，建议先掌握了`RACSignal`的基本用法后在去了解他的原理。

再来看看连续多个block的情况，比如已经创建了一个`UITextField`叫`tf`

```
    RACSignal *tfRac = [tf rac_textSignal];

    [[tfRac filter:^BOOL(NSNumber*length) {
        return YES;
    }] subscribeNext:^(id x) {
        NSLog(@"%@", x);
    }];

```

这样当UITextField里面内容变化的时候就会打印出UITextField的内容。

### filter

`filter`是一个过滤器里面能加一些判定条件用于判断是否继续往下面的执行。这样就表示当内容是123的时候才打印

```
    [[tfRac filter:^BOOL(id value) {
        if([value isEqualToString:@"123"]){
            return YES;
        }
        return NO;
    }] subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
```

### map

还有一个常用的操作`map`，在`map`的`block`里面可以更改要往下传的数据，比如下面的例子，textfield的内容被传入`map`，在`block`里面计算了文字长度之后，把文字长度传了下去。后面的block收到的参数就是这个长度值了。

```
    RACSignal *tfRac = [tf rac_textSignal];

    [[[tfRac
       map:^id(NSString*text){
           return @(text.length);
       }]
      filter:^BOOL(NSNumber*length){
          return[length integerValue] > 3;
      }]
     subscribeNext:^(id x){
         NSLog(@"%@", x);
     }];
```

### RAC宏
在ReactiveCocoa有一个`RAC`宏非常好用,他可以直接把信号的输出应用到对象的属性上。参数1是要绑定的对象，参数2是要绑定的属性。下面的例子，当输入值为"123456"的时候背景颜色就会变成黄色了。

```
    tf.backgroundColor = [UIColor redColor];
    RACSignal *tfRac = [tf rac_textSignal];
    RAC(tf,backgroundColor) = [tfRac map:^id(id value) {
        if([value isEqualToString:@"123456"]){
            return [UIColor yellowColor];
        }else{
            return [UIColor redColor];
        }
    }];
```

### 信号聚合
在用些时候需要聚合信号,`RACSignal`提供了`combineLatest:reduce:`这个方法可以把任意数量的信号聚合起来。

```
    //信号1
    RACSignal *tfRac = [tf rac_textSignal];
    [tfRac subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];

    //信号2
    RACSignal *tf1Rac = [tf1 rac_textSignal];
    [tf1Rac subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];

    //聚合信号
    [[RACSignal combineLatest:@[tfRac, tf1Rac]
                      reduce:^id(id x, id y){

                          if([x isEqualToString:y]){
                              NSLog(@"一样");
                              return @1;
                          }else{
                              NSLog(@"不一样");
                              return @0;
                          }
                      }] subscribeNext:^(id x) {
                          NSLog(@"%@",x);
                      }];
```

### 附加操作
有些时候我们可能需要在收到信号后进行一些附加操作，可以添加一个`doNext:`block。

```
    [[tfRac doNext:^(id x) {
    	NSLog(@"附加操作");
    }] subscribeNext:^(id x) {

    }];
```

参考资料

[ReactiveCocoa - 基础篇](http://www.cnblogs.com/tangchangjiang/p/5598079.html)

[ReactiveCocoa入门教程——第一部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part1)

[ReactiveCocoa入门教程——第二部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part2)

[最快让你上手ReactiveCocoa之基础篇](http://www.jianshu.com/p/87ef6720a096)

[最快让你上手ReactiveCocoa之进阶篇](http://www.jianshu.com/p/e10e5ca413b7)

