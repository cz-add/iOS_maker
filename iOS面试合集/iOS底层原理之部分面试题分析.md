## Runtime Asssociate方法关联的对象，是否需要在dealloc中释放？

`不需要释放`

## 分析

我们知道当一个对象销毁的时候会调用 `dealloc` 方法，那么我们先看下 `dealloc` 都进行了哪些操作。

*   `dealloc` 函数调用了 `_objc_rootDealloc` 函数

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/145101_8eccb96d_9027123.png "底层1.png")

*   `_objc_rootDealloc` 函数调用 `rootDealloc` 函数

```
void
_objc_rootDealloc(id obj)
{
    ASSERT(obj);

    obj->rootDealloc();
}
```

*   `rootDealloc` 函数查看

```
inline void
objc_object::rootDealloc()
{
    if (isTaggedPointer()) return;  // fixme necessary?

    if (fastpath(isa.nonpointer  &&  
                 !isa.weakly_referenced  &&  
                 !isa.has_assoc  &&  
                 !isa.has_cxx_dtor  &&  
                 !isa.has_sidetable_rc))
    {
        assert(!sidetable_present());
        free(this);
    } 
    else {
        object_dispose((id)this);
    }
}
```

从 `rootDealloc` 函数中我们看到了判断isa相关属性的地方，实际上当一个对象存在会进入 `else` 中，即 `object_dispose` 函数

*   `object_dispose` 函数查看

```
id 
object_dispose(id obj)
{
    if (!obj) return nil;

    objc_destructInstance(obj);    
    free(obj);

    return nil;
}
```

通过 `objc_destructInstance` 函数找到对象，然后 `free` ，我们看 `objc_destructInstance` 函数

*   `objc_destructInstance` 函数

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/145117_2adf73e7_9027123.png "底层2.png")

重点查看 `_object_remove_assocations` 函数

*   `_object_remove_assocations` 函数分析

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/145130_b627f11e_9027123.png "底层3.png")

## 类、分类方法同名时调用顺序是怎样的？

当 `非+load` 方法同名时，分类的方法在类的方法前面（ `注意不是覆盖` ），因为 `分类的方法是在类realize之后 attach进去的` ，所以 `优先分类，其次类`

## 当 `+load` 方法同名时， `优先类，其次分类`

## 分类与类的扩展

## 分类

*   专门用来给类添加新的方法
*   不能添加属性，但是可以通过runtime动态添加属性（因为我们在前面的篇章中分析过，分类底层代码中有属性列表）
*   分类中 `@property` 定义的变量只会生成 `setter` 以及 `getter` 方法的声明，但是 `不会生成对应的方法实现以及带有下划线的成员变量`



## 类的扩展

```
@property
```

## 什么是Runtime？

runtime是由C和C++汇编实现的一套API，为OC语言添加了面向对象和运行时功能。

*   运行时：将数据类型的确定由编译阶段推迟到了运行阶段。我们平时所写的OC代码，最终转换为runtime的C语言代码。

## 方法的本质是什么？SEL、IMP是什么？两者之间的关系是什么？

## 方法的本质

方法的本质是 `消息的发送` ，涉及到消息发送的流程有

```
objc_msgSend
lookUpImpOrForward
resolveInstanceMethod
forwardingTargetForSelector
mesthodSignatureForSelector & forwardInvocation
```

## SEL、IMP

*   sel：方法编号，类比一本书的目录
*   imp：方法函数指针地址，类比一本书的页数
*   sel与imp关系：sel是方法编号，通过sel找到imp的函数指针地址，通过imp就能找到函数的实现

## 能否向编译后的类中添加实例变量？能否向运行时创建的类添加实例变量？

```
编译后实例变量存储到 ro 中，一旦编译完成，内存结构就完全确定了，无法再次修改
```

## [self class] 与 [super class]的区别

我们先看以下如下代码打印结果，其中self是LGTeacher类，LGTeacher继承于LGPerson，LGPerson继承于NSObject[图片上传失败...(image-e06edd-1604300283675)]

从打印结果中我们看到无论是 `[self class]` 还是 `[super class]` 的结果是一样的，为什么呢？

## 分析

*   我们知道任何方法调用都会隐藏两个参数，即 `(id self , sel _cmd)` ，其中 `self` 是消息接收者。对于 `[self class]` 来说，它的消息接收者是 `自身LGTeacher` 没什么可说的，所以打印的是 `LGTeacher` 。
*   首先我们要知道 `super` 只是关键字，它意思是说从父类调用方法，因此 `[super class]` 就是 `直接调用的就是父类的class` 方法，它的本质是 `objc_msgSendSuper` ，只是 `objc_msgSendSuper` 速度更快，直接跳过 `self` 。但需要注意的是， `[super class]` 的消息接受者依然是 `LGTeacher` ，所以最终打印的是 `LGTeacher` 。

## 内存偏移相关问题

我们先准备代码，定义 `IFPerson` 类，代码如下
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/145148_4c3ed9ca_9027123.png "底层4.png")


![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/145202_042c61b1_9027123.png "底层5.png")

我们再看ViewController代码

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/145509_6cddf742_9027123.png "底层6.png")




## `[(__bridge id)kc doSomething]` 为什么不会崩溃？

首先我们知道对于一个对象，它的指针地址指向的是 `isa` ，同时 `isa` 地址指向 `当前的class` ，所以 `kc` 指向的是 `IFPerson` 的 `isa` ，而 `person` 的指针指向的 `也是isa` ，这样它们都是 `isa` 从 `cache_t` 中查找 `doSomething` 方法，因此不会崩溃。

## 为什么 `[(__bridge id)kc doSomething]` 打印的结果是 `ViewController` ？

*   从打印结果中 `[person doSomething]` 打印出出来 `shifx` 是没有什么问题的，毕竟给person.name赋值 `shifx` ，但是 `[(__bridge id)kc doSomething]` 打印的结果是 `ViewController` 呢？要解决这个问题首先我们需要知道 `person` 能够找到 `name` 是 `指针从isa内存平移了8个字节` 移动到了 `name` 。那么对于 `kc` 来说，它也需要指针平移，但是为什么平移后的结果是 `viewController` 呢？这就需要明白 `栈地址是从高到低存储的，且是先进后出` ，由于前面先调用了 `[super viewDidLoad]` 方法，且 `viewDidLoad` 的隐藏参数是 `(id self, IMP _cmd)` ，所以 `self` 会先入栈，其次是 `cls` -> `kc` -> `person` ，出栈的顺序刚好相反，由于 `[(__bridge id)kc doSomething]` 时需要指针平移，自然指向了 `self（即ViewController）` ，所以打印的结果是 `ViewController` 。
*   为了验证我们上面分析是否正确，我们修改代码位置，将声明 `IFPerson *person = [[IFPerson alloc] init]` 放在 `[super viewDidLoad]` 之后，即
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/145535_a4e88dca_9027123.png "底层7.png")

此时我们按照我们上面的分析 `self` 会先入栈，其次是 `person` -> `cls` -> `kc` ，猜测 `[(__bridge id)kc doSomething]` 打印结果应该是 `IFPerson （person的isa指向其Class）` ，

可以看出我们的分析是正确的。
