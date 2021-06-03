![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134446_b6800d72_9027123.png "isa1.png")

instance的isa指针指向class。

当调用对象方法时，通过instance的isa指针找到class，最后找到对象方法的实现进行调用。

class的isa指针指向meta-class。

当调用类方法时，通过class的isa指针找到meta-class，最后找到类方法的实现进行调用。

## 二.class的superclass

#### 2.1 class对象的superclass指针

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134508_f2226696_9027123.png "isa2.png")

当Student的instance对象要调用Person的对象方法时，会先通过isa找到Student的class，然后通过superclass找到Person的class，最后找到对象方法的实现进行调用。

#### 2.2 meta-class对象的superclass指针

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134519_3388d4c5_9027123.png "isa3.png")

当Student的class要调用Person的类方法时，会先通过isa指针找到Student的meta-class，然后通过superclass找到Person的meta-class，最后找到类方法的实现进行调用

## 三.isa细节

从64bit开始，isa需要进行一次位运算，才能计算出真实地址

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134531_6abda4f2_9027123.png "isa4.png")

从源码中可以查到ISA_MASK的相关信息

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134545_0cdd7e24_9027123.png "isa5.png")

## isa、superclass总结

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134559_d42700ad_9027123.png "isa6.png")

*   实例对象(instance)的isa指向class
*   类对象(class)的isa指向meta-class
*   元类对象(meta-class)的isa指向基类的meta-class
*   类对象(class)的superclass指向父类的class
*   *   如果没有父类，superclass指针为nil
*   元类对象(meta-class)的superclass指向父类的meta-class
*   *   基类的meta-class的superclass指向基类的class
*   instance调用对象方法的轨迹
*   *   isa找到class，方法不存在，就通过superclass找父类
*   class调用类方法的轨迹
*   *   isa找到meta-class，方法不存在，就通过superclass找父类

## 四.class和meta-class的结构

```
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */
```

在runtime.h头文件中我们可以看到一个条件编译如果不是objc2的版本下，那么条件范围内的代码将不会被执行，所以在OC2.0中，这段代码已经过时了。而且通过OBJC2_UNAVAILABLE也可以看出，struct objc_class这个结构体在OC2.0中已经不可用了。



#### 4.1 窥探struct objc_class的结构

类和元类在内存中的结构，就是struct objc_class这样一个结构体，这个结构体在objc-runtime-new.h头文件中可以找到最新版本的定义。如下图结构体struct objc_class中部分代码所示：

```
struct objc_class : objc_object {
    Class isa;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable-方法缓存
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags-用于获取具体的类信息
    ...
    ...
    ...
}
```

注意：冒号语法是继承的意思，结构体objc_class继承自objc_object，这种语法是C++的语法。

```
class_rw_t *data() const {
    return bits.data();
}
```

```
class_rw_t* data() const {
    return (class_rw_t *)(bits & FAST_DATA_MASK);
}
```

#### 下图为老版本代码结构，可以参考新版本源码

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134609_aef53def_9027123.png "isa7.png")

struct objc_class结构体中包含上面这样一段代码，返回值是class_rw_t，是一个可读写的表单。

进入class_rw_t头文件中我们可以看到一段public的代码，返回值是class_rw_ext_t类型。如下图所示

```
class_rw_ext_t *ext() const {
    return get_ro_or_rwe().dyn_cast<class_rw_ext_t *>();
}

```
进入class_rw_ext_t头文件，我们可以看到方法列表、属性列表、协议列表的相关定义。如下图所示

```
struct class_rw_ext_t {
    const class_ro_t *ro;
    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;
    char *demangledName;
    uint32_t version;
};
```

进入到class_rw_ext_t结构体中class_ro_t类型的头文件中，我们可以看到instanceSize、ivars的定义。下图为头文件中的部分代码。

```
struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;

    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;
    ...
    ...
    ...
```

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134619_2297bc00_9027123.png "isa8.png")

总结：OC的类信息存放在哪里？

*   对象方法、属性、成员变量、协议信息，存放在class对象中。
*   类方法，存放在meta-class对象中。
*   成员变量的具体值，存放在instance对象。


