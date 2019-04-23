
# 1.AOP-面向切面编程

oop - Object Oriented Programming
aop - Aspect Oriented Programming
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

```
Class class = NSClassFromString(@"ViewController");
ViewController *vc = [[class alloc] init];
SEL selector = NSSelectorFromString(@"getDataList");
[vc performSelector:selector];
```

# 4.消息传递方式

## 通知
```
//添加对登录情况的观察
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(loginSuccess) name:@"login" object:nil];

//发送通知
[[NSNotificationCenter defaultCenter] postNotificationName:@"login" object:nil];
```
## block

## target action

## KVO(观察者模式)

* 应用AOP技术，自动为被观察对象创造一个派生类，并将被观察对象的isa 指向这个派生类。对派生类的方法进行了override，并添加了通知代码。
```
//案例：AFNetWorking库对下载进度的观察
//添加对进度的观察
NSProgress *progress；
[progress addObserver:self
           forKeyPath:NSStringFromSelector(@selector(fractionCompleted))
              options:NSKeyValueObservingOptionNew
              context:NULL];
              
//实现observeValueForKeyPath方法捕获通知
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context {
   if ([object isEqual:self.downloadProgress]) {
        if (self.downloadProgressBlock) {
            self.downloadProgressBlock(object);
        }
    }
    else if ([object isEqual:self.uploadProgress]) {
        if (self.uploadProgressBlock) {
            self.uploadProgressBlock(object);
        }
    }
}
```

## 代理（设计模式）
```
比如tableview，由@Protocol，委托方(tableview)，代理方(当前viewcontroller)组成；
```

# 5.定时器

## GCD:

```
dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_queue_create("my queue", 0));
dispatch_source_set_timer(timer, dispatch_time(DISPATCH_TIME_NOW, 0), NSEC_PER_SEC, 0);
__weak ViewController *blockSelf = self;
dispatch_source_set_event_handler(timer, ^()
{
    [blockSelf myTimerAction];
});
dispatch_resume(timer);

```

## NSTimer:

* 一种方式是主线程中进行NSTimer操作，使用NSRunLoopCommonModes，防止主线程的runloop切换到其他模式导致timer失效。
```
 NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timer:) userInfo:nil repeats:YES];
 [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

* 一种方式是创建子线程，使用子线程的runloop（每个线程都有一个自己的runloop）
```
 NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];
 [thread start];
 - (void)newThread
 {
     @autoreleasepool
     {
         [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(addTime) userInfo:nil repeats:YES];
         [[NSRunLoop currentRunLoop] run];
     }
 }
```

# 6.异常崩溃

## 上线前

```
将发生崩溃的设备连接Xcode，选择window－> devices －> 选择自己的手机 -> view device logs 就可以查看手机上所有的崩溃信息了。
```

## 上线后

* 使用友盟或者通讯bugly的sdk，自动上传分析；

* 代码实现捕获并提交服务器
```
NSSetUncaughtExceptionHandler(&uncaughtExceptionHandler);
void uncaughtExceptionHandler(NSException *exception) {
    NSLog(@“Stack Trace: %@“, [exception callStackSymbols]);
    
    CrashLogSender *logSender = [[CrashLogSender alloc]init];
    [logSender sendCrashLog:exception.description];
}
```

# 7.数据存储

## CoreData

* CoreData.framework对原始SQLite数据库API访问的封装

## NSUserDefaults

* 存储NSxxx类型
```
[[NSUserDefaults standardUserDefaults] setObject:@"admin"forKey:@"user_name"];
```
* 存储其他系统类型需要转换成NSData存储
```
NSData *objColor = [NSKeyedArchiver archivedDataWithRootObject:[UIColor redColor]];
[[NSUserDefaults standardUserDefaults] setObject:objColor forKey:@"myColor"];
```
* 存储自定义结构体需要实现NSCoding协议

## NSFileManager


# 8. 模型与字典数组的转化

```
MJ框架与KVC的底层实现不同点:

1 KVC是通过遍历字典中的所有key,然后去模型中寻找对应的属性.

2 MJ框架是通过先遍历模型中的属性,然后去字典中寻找对应的key,所以用MJ框架的时候,模型中的属性和字典可以不用一一对应,同样能达到给模型赋值的效果.
```

## KVC(key value coding)

```
//模型
@interface Person : NSObject
@property (nonatomic, strong) NSString *name;
@property (nonatomic, assign) NSInteger age;
@end

//转化
NSDictionary *dic = @{@"name":@"revon", @"age":@"19"};
Person *p = [Person setValuesForKeysWithDictionary:dic];
```

## MJExtension

* 模型Person中，有一个数组，数组里边，装的就是Person对象
```
@interface Person : NSObject
@property (nonatomic, strong) NSString *name;
@property (nonatomic, assign) NSInteger age;
@property (nonatomic, strong) NSArray *stuarray;
@end

+ (NSDictionary *)mj_objectClassInArray{
    return @{@"stuarray" : @"Person"};//前边，是属性数组的名字，后边就是类名
}

//转化
NSDictionary *dic = @{@"name" : @"a",
                          @"stuarray" : @[@{@"name" : @"b", @"age" : @"18"}, @{@"name" : @"c", @"age" : @"140"}]};
Person *p = [Person mj_objectWithKeyValues:dic];
```

* 数组直接转为模型
```
//data.decryptData是一个数组array
//NSMutableArray *array = @[@{@"name" : @"b", @"age" : @"18"}, @{@"name" : @"c", @"age" : @"140"}];
NSMutableArray<Person *> *persionList = [[NSMutableArray alloc] init];
persionList = [Person mj_objectArrayWithKeyValuesArray:data.decryptData];
```

# 9.多线程同步

## 互斥锁

* @synchronized（）
```
//函数被多线程调用时，锁定self，读取修改self的变量；
- (void)saleTicket
{
    @synchronized(self) {
        // 先取出总数
        NSInteger count = self.ticketCount;
        if (count > 0) {
            self.ticketCount = count - 1;
            NSLog(@"%@卖了一张票，还剩下%zd张", [NSThread currentThread].name, self.ticketCount);
        } else {
            NSLog(@"票已经卖完了");
            break;
        }
    }
}
```
* NSLock、NSCondition
```
NSLock *lock = [[NSLock alloc] init];
[lock lock] //上锁
[lock unlock]//解锁
```

## 栅栏
```
//DISPATCH_QUEUE_CONCURRENT 并行队列，非有序
//DISPATCH_QUEUE_SERIAL 串行队列，有序
dispatch_queue_t h264DataQueue = dispatch_queue_create("com.h264.saveArray", DISPATCH_QUEUE_CONCURRENT);

//这里使用dispatch_barrier_async异步添加，不必等待代码块执行完成，防止接受message卡住，而且能保证添加入的顺序是完整的
- (void)Thread1
{
    dispatch_barrier_async(self.h264DataQueue, ^{
        [self.h264DataList addObject:message];
    });
}

//从头部提取出数据
- (void)Thread2
{
    __block NSData *message = nil;
        dispatch_barrier_sync(self.h264DataQueue, ^{
          message = [self.h264DataList firstObject];
          [self.h264DataList removeObject:message];
        });
}

```

# 10.frame & bounds

```
 frame: 该view在父view坐标系统中的位置和大小。（参照点是，父亲的坐标系统）
 bounds：该view在本地坐标系统中的位置和大小。（参照点是，本地坐标系统，以0,0点为起点）
```

# 11.ARC & MRR

```
ARC(Automatic Reference Counting)自动引用计数
MRR(manual retain-release) 手动管理内存
```

# 12.MVC,MVP,MVVM,基于router组件方案（VIPER,蘑菇街）
```

```

# 13.#import #include

    import处理了重复引用的问题，不用再去自己进行重复引用处理.

# 14.成员变量和属性
```
@interface Person : NSObject
{
    NSString *_sex;//成员变量，使用要用->，成员变量的默认修饰是@protected只能内部用。
}
@property (nonatomic, copy) NSString *name;//属性,最好都用属性，方便。
@end
```

# 15.Category、extension
```
可以把不同的功能组织到不同的category里;
模拟多继承（另外可以模拟多继承的还有protocol）
```
```
extension可以添加实例变量，而category不可以.
必须有一个类的源码才能为一个类添加extension
```

# 16.viewcontroller
```
viewDidLoad 这时候view已经有了，最适合创建一些附加的view和控件了。
有一点需要注意的是，viewDidLoad会调用多次（viewcontroller可能多次载入view),最好使用懒加载，判断是否为空.
```

# 17.进程通讯方式

## 1.URL Scheme
应用场景，想要进行qq，微信，微博分享跳转功能，需在LSApplicationQueriesSchemes配置这些第三方应用，白名单。

## 2.UIPasteboard
应用场景，复制淘口令到剪切板。

## 3.socket

# 18.私有api
```
LSApplicationWorkspace
```

# 19.代码混淆
```
```

# 20.运行时编程

## 1.动态方法交换

## 2.category类别添加新属性

## 3.获取类的属性，方法等。
```
objc_getClass
```

# 21.多线程

## 1.NSThread

## 2.GCD

## 3.NSOperationQueue

# 22.uiviewcontroller
    init-loadview-viewdidload-viewwillappear-viewdidappear-viewwilldisappear-viewdiddisappear-dealloc

# 23.






