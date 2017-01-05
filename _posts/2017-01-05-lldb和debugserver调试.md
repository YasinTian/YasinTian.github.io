---
layout:     post
title:      "lldb调试第三方App"
subtitle:   "lldb是苹果内置在Xcode中的调试器，一般调试我们自己开发的程序。但是越狱后结合debugserver我们就能调试任意的App。今天就先来学习一下环境的配置。"
date:       2017-01-05
author:     "yasin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 安全
---

在iOS逆向工程中，通过class-dump能得到函数头文件，但是一般情况下，参数都是`id`类型，这时候可以通过调试器打印出来看一下，起到事半功倍的作用。

## debugserver

在连接过`XCode`后，XCode会自动给手机安装上`debugserver`，`debugserver`是安装在手机端用于接受电脑端传过去的`lldb`命令的。执行后再把结果返回给`lldb`。
在正常的开发中，`debugserver`是没有`task_for_pid`的权限的，所以一般只能调试我们自己的App。但是这样对越狱开发一点用都没有了。但是通过配置我们可以提升`debugserver`的权限。

## 配置debugserver

### debugserver减肥

首先将debugserver拷贝到电脑中。命令是

```
scp root@(手机的ip地址):/Developer/usr/bin/debugserver (要拷贝到的路径)/debugderver
```

然后执行

```
cd debugderver文件的路径
lipo -thin arm64 debugserver -output debugserver
```

下载[http://iosre.com/ent.xml](http://iosre.com/ent.xml)这个文件放到debugderver文件所在的路径。
执行

```
/opt/theos/bin/ldid -Sent.xml debugserver
-S后面是没有空格的
```

如果长时间没有执行完毕，就只能用另一种方案了。
下载[http://iosre.com/ent.plist](http://iosre.com/ent.plist)这个文件
执行

```
codesign -s - --entitlements ent.plist -f debugserver
```

最后把修改过的debugserver拷贝回iOS

```
scp (debugserver的地址)/debugserver root@192.168.5.228:/usr/bin/debugserver
```

debugserver我们放在/usr/bin里面可以直接输入debugserver运行。

ssh连接手机，提升权限

```
chmod +x /usr/bin/debugserver
```

## 打开调试器

### 手机端

```
debugserver -x backboard *:1234 /Applications/MobileSMS.app
```

这句话表示，打开短信，并且注入调试器。等待来自任意ip的1234端口的`lldb`连接

### 电脑端

```
/Applications/Xcode.app/Contents/Developer/usr/bin/lldb
process connect connect://192.168.5.228:1234
```

这样就可以连上了。连接成功后
如果连接上是卡住的状态，好像是连接上回默认被断点卡住，lldb输入c命令就能好。
