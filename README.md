# Note

# 1.AOP-面向切面编程

设计目的：解决只有单继承的语言缺点。

优点：不入侵原有代码。

实用项目：MLeaksFinder，实现方式是load中更换UIViewController等的viewDidDisappear，在新方法中进行检查。

只要pod进工程即可，不需要修改原有代码。

# 2.NSProxy-代理

实用功能:

YYWeakProxy，内部保存真正的类，用在NSTimer解决循环引用;

TDProxyDemo，代理多个不同的类，方法选择器里做判断，解决objc不能多继承的问题；

# 3.反射机制

反射是一种强大的工具。它使您能够创建灵活的代码，这些代码可以在运行时装配，无需在组件之间进行源代表链接。

如很多私有api调用都要用到。

Class class = NSClassFromString(@"ViewController");

ViewController *vc = [[class alloc] init];

SEL selector = NSSelectorFromString(@"getDataList");

[vc performSelector:selector];

# 4.代理-设计模式

iOS中消息传递方式：通知、代理、block、target action、KVO；

其中代理如tableview，由@Protocol，委托方(tableview)，代理方(当前viewcontroller)组成；

# 5.定时器

GCD:

dispatch_source_t

dispatch_source_set_timer

NSTimer:

####

一种方式是主线程中进行NSTimer操作，使用NSRunLoopCommonModes，防止主线程的runloop切换到其他模式导致timer失效。

NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timer:) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

####

一种方式是创建子线程，使用子线程的runloop

NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];

在newThread函数中

[NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(addTime) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] run];

6.


























