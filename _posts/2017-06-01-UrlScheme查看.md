---
layout:     post
title:      "查看App之间的跳转协议"
subtitle:   "解决%log打印日志文件不存在的问题"
date:       2017-6-1 12:00:00
author:     "yasin"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 安全
---

[参考文章](https://github.com/yohunl/FlexInjected)

一直想做一个App能跳转到我想去的App里面，网上找urlScheme也就是那么几个，而且大多数只能打开App不能分析进入里面的功能，索性我自己弄一个截获跳转协议的工具好了。在很久以前就已经搭建好了theos的环境，不过好久没动过了。

### 创建工程

创建工程还是老步骤
1. /opt/theos/bin/nic.pl
2. 选择[11.] iphone/tweak
3. 填你的工程名，包名，作者，要hook的bundleid这里我们填com.apple.UIKit这样能对所有的app生效

### Hook函数

ios下跳转到其他App主要是`UIApplication`的这两个方法，先试一下能不能行

```
%hook UIApplication
-(BOOL)openURL:(NSURL*)url{
	UIAlertView * alert = [[UIAlertView alloc]initWithTitle:@"Welcome" message:url.absoluteString delegate:nil cancelButtonTitle:@"Thanks" otherButtonTitles:nil];
	[alert show];
	return %orig;
}

- (void)openURL:(NSURL*)url options:(NSDictionary<NSString *, id> *)options completionHandler:(id)completion{
	UIAlertView * alert = [[UIAlertView alloc]initWithTitle:@"Welcome" message:url.absoluteString delegate:nil cancelButtonTitle:@"Thanks" otherButtonTitles:nil];
	[alert show];
	%orig;
}

%end
```

分享文章到QQ试了试urlscheme就出来了。

![截屏分享到QQ]({{ site.url }}/img/in-post/showsuccess.png)

### 从接收端入手

前是从发送端入手，现在我们来看看接收端，从外部打开app主要是这几个方法，最后是个是ios9出的通用链接（Universal Links）进入的回调方法。全部都把url弹出来，在微信和QQ之间测试，发现QQ竟然换掉了系统的AlertView，当页面消失的时候就会收回弹框

```
%hook AppDelegate

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation{
    UIAlertView * alert = [[UIAlertView alloc]initWithTitle:@"Welcome" message:url.absoluteString delegate:nil cancelButtonTitle:@"Thanks" otherButtonTitles:nil];
    [alert show];
    return %orig;
}

-(BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options{
    UIAlertView * alert = [[UIAlertView alloc]initWithTitle:@"Welcome" message:url.absoluteString delegate:nil cancelButtonTitle:@"Thanks" otherButtonTitles:nil];
    [alert show];
    return %orig;
}

-(BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url{
    UIAlertView * alert = [[UIAlertView alloc]initWithTitle:@"Welcome" message:url.absoluteString delegate:nil cancelButtonTitle:@"Thanks" otherButtonTitles:nil];
    [alert show];
    return %orig;
}

-(BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler{
    UIAlertView * alert = [[UIAlertView alloc]initWithTitle:@"Welcome" message:userActivity.webpageURL.absoluteString delegate:nil cancelButtonTitle:@"Thanks" otherButtonTitles:nil];
    [alert show];
    return %orig;
}

%end
```

代码写完了还是通过，`make` `package` `install`

试了试可以看到app间的跳转协议了，但是这样很不方便，当我正常使用的时候很麻烦，每次都要点击关闭这个弹框，这里我们就需要做一个选择开启的功能。
在网上找了找可以把插件和App打包成deb安装，这样通过App来控制，不过很麻烦，都要自己来处理。

后来在网上找到一篇教程https://github.com/yohunl/FlexInjected 里面讲到一个[PreferenceLoader](https://github.com/DHowett/preferenceloader)可以很方便的在系统设置里面给我们的插件添加一个开关，可以选择对哪些App启用插件。而且这个插件是很多越狱软件依赖插件，基本都装的有，我们只管用就行了。

这个地方要新建一个tweak工程，通过新建的这个工程来控制。
[PreferenceLoader](https://github.com/DHowett/preferenceloader)的使用非常简单。只需要在工程根目录新建一个layout文件件，按照要求放入需要的文件就行了.

`layout/Library/Application Support`下面放上面生成的`dylib`文件，这个文件`dylib`在`.theos/_/Library/MobileSubstrate/DynamicLibraries`里面，反正我是在这里弄得，其他地方好像也有，我就不去找了。

![dylib文件在这里]({{ site.url }}/img/in-post/dynamiclibraries.png)

`layout/Library/PreferenceLoader/Preferences`里面是关于设置界面的一些配置，一个`Plist`文件和图标，图标就不说了。

`Plist`主要是这几个地方要改成你自己的
![Plist文件修改]({{ site.url }}/img/in-post/plist.png)

最后弄完是这样的
![Application Support]({{ site.url }}/img/in-post/F2D93245-2CBA-46B6-8FBB-12F4D7528906.png)

![PreferenceLoader]({{ site.url }}/img/in-post/preferences.png)

都弄好了就运行试试吧，出现插件不生效的情况可以用`socat`看看打印的日志信息，看看到底有没有成功加载插件。





