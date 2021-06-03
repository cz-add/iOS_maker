## 前言

网上关于组件化的理论很多而且已经比较成熟，理论方面请参看这篇集合文章iOS组件化。

### 一、组件化的初衷。

*   有利于代码模块的封装和复用。
*   对不同的业务模块可以进行物理隔离（通过git私有 仓库权限控制），进一步提升代码的稳定性和安全性。
*   项目整体结构层次分明，便于后期维护。
*   便于项目功能细分，颗粒划分更细，分配工作更合理，项目时间节点更容易掌控，便于进行敏捷开发。
*   便于进行单元测试。

### 二、组件化开发过程。

### 1、要组件化必须进行解耦。

我们谈解耦，并不是完全解除代码之间的耦合，通过学习和实践这是不合理也不可能的。我们解耦的目的其实是为了解除代码模块相互间的依赖，或者说我们的目的就是让代码模块变得单向依赖，像一个插头一样可以自由拔插。
（合理 不合理的 图）

### 2、模块化与解耦理论模块化与解耦

因为个人精力有限，此篇主要是记录组件化架构的实践，故这里只以功能模块划分组件，模块化可以根据自身项目自己封装对应模块。

### 3、组件化架构设计

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/160500_976f125d_9027123.jpeg "组件化1.jpg")

### 三、组件化架构实现。

### 1、目前业界常见的模块间通讯方案大致如下几种：

基于路由 URL 的 UI 页面统跳管理（url-block)。
基于反射的远程接口调用封装(target-action)。
基于面向协议思想的服务注册方案（protocol-class)。
基于通知的广播方案（NSNotification)。

可以使用一种或几种，我这里选用了基于反射的远程接口调用封装(target-action)和基于路由 URL 的 UI 页面统跳管理（url-block)来进行封装。

### 2、路由的实现：

网上成熟方案很多JLRoute就是一个非常好的路由框架，但是个人感觉JLRoute依然比较庞大，功能也不够单一。因此自己实现了一个简单的路由框架PTRouter。

> PTRouter，支持了注册scheme。注册scheme这一特性，可以更方便的调用诸如第方分享，或是统计SDK集成使用。PTRouter，为了实现功能单一性，只扩充了controller跳转的功能。数据的传入通过参数传递(底层是通过依赖注入实现)。通过block只返回跳转是否成功,比如打开设置页：

```
[PTRouter openURL:@"InnerJump://setting/browse?userID=007" callback:^(BOOL result) {
       if (!result) {
           [SVProgressHUD showInfoWithStatus:@"打开失败"];
       }
   }];
```

> PTRouter 支持同步 & 异步获取返回值，其中异步转同步内部通过semaphore实现

```
+ (void)openURL:(NSString *)url callback:(void (^)(BOOL result))callback;
+ (BOOL)openURL:(NSString *)url;
```

> 另外openURL除了支持url中带参数，也支持参数放在字典中

```
+ (void)openURL:(NSString *)url param:(NSDictionary<NSString*,id> * __nullable)param callback:(void (^)(BOOL result))callback;
+ (BOOL)openURL:(NSString *)url param:(NSDictionary<NSString *,id> * __nullable)param;
```

### 3、公共组件例如：第三方分享，数据统计，Bug分析等需要在app进入时就注册的，个人觉得应该放在工程主干中进行对应注册管理：

> PTAppLaunchHelper APP启动时触发自动注册组件

```
[PTAppLaunchHelper.shared autoInitModule];//根据AutoInitialize.plist 自动初始化组件
```

对应注册信息集中写在plist中，这样做一目了然，便于维护管理。个人使用runtime来动态注册组件。 PTAppLaunchHelper有两个函数:

*   autoInitModule 用来初始化组件。该函数会读取AutoInitialize.plist中的classes，通过runtime自动初始化协议完成初始化:

```
//init modules with AutoInitialize
- (void)autoInitModule;
```

[图片上传失败...(image-81bf26-1598413859499)]

*   autoRegistURL
    用来自动注册路由，该函数会读取AutoRegistURL.plist完成路由注册。其中controller代表类名，params代表默认参数，如果openURL传的参数与默认参数不符合，路由会报错

```
//init url with AutoRegistURL
- (void)autoRegistURL;
```

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/160509_eb3374d4_9027123.jpeg "组件化2.jpg")

> PTAppEventBus 生命周期监听组件 PTAppEventBus通过接收系统通知来获取app生命周期事件，收到生命周期事件后改变对应属性的值。默认提供了didEnterBackground等八个属性，可以使用响应式函数来监听。

```
- (void)observeWithBlock:(PTObservingBlock)block {

   if (self.owner && self.keyPath) {
       [self.owner addObserver:self.owner forKey:self.keyPath withBlock:block];
   }
   else {
       NSLog(@"owner =  %@， keypath = %@",self.owner,self.keyPath);
       NSString *reason = [NSString stringWithFormat:@"Object does not set owner or keypath"];
       @throw [NSException exceptionWithName:NSInvalidArgumentException
                                      reason:reason
                                    userInfo:nil];

       return;
   }
}
```

> PTAppEventBus使用前需要调用

```
- (void)start;
```

> 如果这些不够，需要监听更多的事件（例如app横竖屏状态），可以通过分类给PTAppEventBus的添加对应属性属性，操作如下：

```
NSMutableDictionary *defaultMap = [NSMutableDictionary dictionaryWithDictionary:[PTAppEventBus defaultNotificationMap]];
[defaultMap setObject:KDidChangeStatusBarOrientation forKey:UIApplicationWillChangeStatusBarOrientationNotification];
[PTAppEventBus.shared startWithNotificationMap:defaultMap];//开启EventBus，开启后组件可收到App生命周期事件
```

### 总结：

本文的组件通过pod 私有库进行集成。解藕部分还有待改进，后续有时间会继续修改。

[原文地址](https://zhuanlan.zhihu.com/p/183809375)