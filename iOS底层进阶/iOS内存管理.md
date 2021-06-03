#### 1 似乎每个人在学习 iOS 过程中都考虑过的问题

*   alloc retain release delloc 做了什么?
*   autoreleasepool 是怎样实现的?
*   __unsafe_unretained 是什么?
*   Block 是怎样实现的
*   什么时候会引起循环引用，什么时候不会引起循环引用?

所以我将在本篇博文中详细的从 ARC 解释到 iOS 的内存管理，以及 Block 相关的原理、源码。

#### 2 从 ARC 说起

说 iOS 的内存管理，就不得不从 ARC(Automatic Reference Counting / 自动引用计数) 说起， ARC 是 WWDC2011 和 iOS5 引入的变化。ARC 是 LLVM 3.0 编译器的特性，用来自动管理内存。

与 Java 中 GC 不同，ARC 是编译器特性，而不是基于运行时的，所以 ARC 其实是在编译阶段自动帮开发者插入了管理内存的代码，而不是实时监控与回收内存。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/160711_62db4717_9027123.jpeg "内存管理1.jpg")

ARC 的内存管理规则可以简述为：

*   每个对象都有一个『被引用计数』
*   对象被持有，『被引用计数』+1
*   对象被放弃持有，『被引用计数』-1
*   『引用计数』=0，释放对象

#### 3 你需要知道

*   包含 NSObject 类的 Foundation 框架并没有公开
*   Core Foundation 框架源代码，以及通过 NSObject 进行内存管理的部分源代码是公开的。
*   GNUstep 是 Foundation 框架的互换框架

GNUstep 也是 GNU 计划之一。将 Cocoa Objective-C 软件库以自由软件方式重新实现

某种意义上，GNUstep 和 Foundation 框架的实现是相似的

通过 GNUstep 的源码来分析 Foundation 的内存管理

#### 4 alloc retain release dealloc 的实现

4.1 GNU – alloc

查看 GNUStep 中的 alloc 函数。

GNUstep/modules/core/base/Source/NSObject.m alloc:
```
+ (id) alloc 

{ 

return [self allocWithZone: NSDefaultMallocZone()]; 

}  

+ (id) allocWithZone: (NSZone*)z 

{ 

return NSAllocateObject (self, 0, z); 

}  
```

GNUstep/modules/core/base/Source/NSObject.m NSAllocateObject:

```
struct obj_layout { 

NSUInteger retained; 

}; 

NSAllocateObject(Class aClass, NSUInteger extraBytes, NSZone *zone) 

{ 

int size = 计算容纳对象所需内存大小; 

id new = NSZoneCalloc(zone, 1, size); 

memset (new, 0, size); 

new = (id)&((obj)new)[1]; 

}  
```
NSAllocateObject 函数通过调用 NSZoneCalloc 函数来分配存放对象所需的空间，之后将该内存空间置为 nil，最后返回作为对象而使用的指针。

我们将上面的代码做简化整理：

GNUstep/modules/core/base/Source/NSObject.m alloc 简化版本:

```
struct obj_layout { 

NSUInteger retained; 

}; 

+ (id) alloc 

{ 

int size = sizeof(struct obj_layout) + 对象大小; 

struct obj_layout *p = (struct obj_layout *)calloc(1, size); 

return (id)(p+1) 

return [self allocWithZone: NSDefaultMallocZone()]; 

}  
```

alloc 类方法用 struct obj_layout 中的 retained 整数来保存引用计数，并将其写入对象的内存头部，该对象内存块全部置为 0 后返回。

一个对象的表示便如下图：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/160722_b7e8022a_9027123.png "内存管理2.png")


4.2 GNU – retain

GNUstep/modules/core/base/Source/NSObject.m retainCount:


```
- (NSUInteger) retainCount 

{ 

return NSExtraRefCount(self) + 1; 

} 

inline NSUInteger 

NSExtraRefCount(id anObject) 

{ 

return ((obj_layout)anObject)[-1].retained; 

}  
```

GNUstep/modules/core/base/Source/NSObject.m retain:

```
- (id) retain 

{ 

NSIncrementExtraRefCount(self); 

return self; 

} 

inline void 

NSIncrementExtraRefCount(id anObject) 

{ 

if (((obj)anObject)[-1].retained == UINT_MAX - 1) 

[NSException raise: NSInternalInconsistencyException 

format: @"NSIncrementExtraRefCount() asked to increment too far”]; 

((obj_layout)anObject)[-1].retained++; 

}  
```

以上代码中， NSIncrementExtraRefCount 方法首先写入了当 retained 变量超出最大值时发生异常的代码(因为 retained 是 NSUInteger 变量)，然后进行 retain ++ 代码。

4.3 GNU – release

和 retain 相应的，release 方法做的就是 retain --。

GNUstep/modules/core/base/Source/NSObject.m release

```
- (oneway void) release 

{ 

if (NSDecrementExtraRefCountWasZero(self)) 

{ 

[self dealloc]; 

} 

} 

BOOL 

NSDecrementExtraRefCountWasZero(id anObject) 

{ 

if (((obj)anObject)[-1].retained == 0) 

{ 

return YES; 

} 

((obj)anObject)[-1].retained--; 

return NO; 

}  
```

4.4 GNU – dealloc

dealloc 将会对对象进行释放。

GNUstep/modules/core/base/Source/NSObject.m dealloc:

```
- (void) dealloc 

{ 

NSDeallocateObject (self); 

} 

inline void 

NSDeallocateObject(id anObject) 

{ 

obj_layout o = &((obj_layout)anObject)[-1]; 

free(o); 

}  
```

4.5 Apple 实现

在 Xcode 中 设置 Debug -> Debug Workflow -> Always Show Disassenbly 打开。这样在打断点后，可以看到更详细的方法调用。

通过在 NSObject 类的 alloc 等方法上设置断点追踪可以看到几个方法内部分别调用了：

retainCount

```
__CFdoExternRefOperation 
CFBasicHashGetCountOfKey  
```

retain

```
__CFdoExternRefOperation 
CFBasicHashAddValue  
```

release

```
__CFdoExternRefOperation 
CFBasicHashRemoveValue  
```

可以看到他们都调用了一个共同的 __CFdoExternRefOperation 方法。

该方法从前缀可以看到是包含在 Core Foundation，在 CFRuntime.c 中可以找到，做简化后列出源码：

CFRuntime.c __CFDoExternRefOperation:

```
int __CFDoExternRefOperation(uintptr_t op, id obj) { 

CFBasicHashRef table = 取得对象的散列表(obj); 

int count; 

switch (op) { 

case OPERATION_retainCount: 

count = CFBasicHashGetCountOfKey(table, obj); 

return count; 

break; 

case OPERATION_retain: 

count = CFBasicHashAddValue(table, obj); 

return obj; 

case OPERATION_release: 

count = CFBasicHashRemoveValue(table, obj); 

return 0 == count; 

} 

}  
```

所以 __CFDoExternRefOperation 是针对不同的操作，进行具体的方法调用，如果 op 是 OPERATION_retain，就去掉用具体实现 retain 的方法。

从 BasicHash 这样的方法名可以看出，其实引用计数表就是散列表。

key 为 hash(对象的地址) value 为 引用计数。

下图是 Apple 和 GNU 的实现对比：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/133453_df73e1d3_9027123.jpeg "内存管理3.jpg")
#### 5 autorelease 和 autorelaesepool

在苹果对于 NSAutoreleasePool 的文档中表示：

每个线程(包括主线程)，都维护了一个管理 NSAutoreleasePool 的栈。当创先新的 Pool 时，他们会被添加到栈顶。当 Pool 被销毁时，他们会被从栈中移除。

autorelease 的对象会被添加到当前线程的栈顶的 Pool 中。当 Pool 被销毁，其中的对象也会被释放。

当线程结束时，所有的 Pool 被销毁释放。

对 NSAutoreleasePool 类方法和 autorelease 方法打断点，查看其运行过程，可以看到调用了以下函数：

```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init]; 

// 等同于 objc_autoreleasePoolPush 

id obj = [[NSObject alloc] init]; 

[obj autorelease]; 

// 等同于 objc_autorelease(obj) 

[NSAutoreleasePool showPools]; 

// 查看 NSAutoreleasePool 状况 

[pool drain]; 

// 等同于 objc_autoreleasePoolPop(pool)  
```

[NSAutoreleasePool showPools] 可以看到当前线程所有 pool 的情况：

```
objc[21536]: ############## 

objc[21536]: AUTORELEASE POOLS for thread 0x10011e3c0 

objc[21536]: 2 releases pending. 

objc[21536]: [0x101802000] ................ PAGE (hot) (cold) 

objc[21536]: [0x101802038] ################ POOL 0x101802038 

objc[21536]: [0x101802040] 0x1003062e0 NSObject 

objc[21536]: ############## 

Program ended with exit code: 0  
```

在 objc4 中可以查看到 AutoreleasePoolPage：

```
objc4/NSObject.mm AutoreleasePoolPage 

class AutoreleasePoolPage 

{ 

static inline void *push() 

{ 

生成或者持有 NSAutoreleasePool 类对象 

} 

static inline void pop(void *token) 

{ 

废弃 NSAutoreleasePool 类对象 

releaseAll(); 

} 

static inline id autorelease(id obj) 

{ 

相当于 NSAutoreleasePool 类的 addObject 类方法 

AutoreleasePoolPage *page = 取得正在使用的 AutoreleasePoolPage 实例; 

} 

id *add(id obj) 

{ 

将对象追加到内部数组 

} 

void releaseAll() 

{ 

调用内部数组中对象的 release 方法 

} 

}; 

void * 

objc_autoreleasePoolPush(void) 

{ 

if (UseGC) return nil; 

return AutoreleasePoolPage::push(); 

} 

void 

objc_autoreleasePoolPop(void *ctxt) 

{ 

if (UseGC) return; 

AutoreleasePoolPage::pop(ctxt); 

}  
```

AutoreleasePoolPage 以双向链表的形式组合而成(分别对应结构中的 parent 指针和 child 指针)。

thread 指针指向当前线程。

每个 AutoreleasePoolPage 对象会开辟4096字节内存(也就是虚拟内存一页的大小)，除了上面的实例变量所占空间，剩下的空间全部用来储存autorelease对象的地址。

next 指针指向下一个 add 进来的 autorelease 的对象即将存放的位置。

一个 Page 的空间被占满时，会新建一个 AutoreleasePoolPage 对象，连接链表。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/133505_ee269dff_9027123.jpeg "内存管理4.jpg")

#### 6 __unsafe_unretained

有时候我们除了 __weak 和 __strong 之外也会用到 __unsafe_unretained 这个修饰符，那么我们对 __unsafe_unretained 了解多少?

__unsafe_unretained 是不安全的所有权修饰符，尽管 ARC 的内存管理是编译器的工作，但附有 __unsafe_unretained 修饰符的变量不属于编译器的内存管理对象。赋值时即不获得强引用也不获得弱引用。

来运行一段代码：

```
id __unsafe_unretained obj1 = nil; 

{ 

id __strong obj0 = [[NSObject alloc] init];  

obj1 = obj0;  

NSLog(@"A: %@", obj1); 

}  

NSLog(@"B: %@", obj1);  
```



对代码进行详细分析：
```
id __unsafe_unretained obj1 = nil; 

{ 

// 自己生成并持有对象 

id __strong obj0 = [[NSObject alloc] init]; 

// 因为 obj0 变量为强引用， 

// 所以自己持有对象 

obj1 = obj0; 

// 虽然 obj0 变量赋值给 obj1 

// 但是 obj1 变量既不持有对象的强引用，也不持有对象的弱引用 

NSLog(@"A: %@", obj1); 

// 输出 obj1 变量所表示的对象 

} 

NSLog(@"B: %@", obj1); 

// 输出 obj1 变量所表示的对象 

// obj1 变量表示的对象已经被废弃 

// 所以此时获得的是悬垂指针 

// 错误访问  
```

所以，最后的 NSLog 只是碰巧正常运行，如果错误访问，会造成 crash

在使用 __unsafe_unretained 修饰符时，赋值给附有 __strong 修饰符变量时，要确保对象确实存在
