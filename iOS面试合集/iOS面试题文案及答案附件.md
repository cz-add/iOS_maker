![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/143906_836183bb_9027123.png "面试题文案1.png")


###1，分类和扩展有什么区别？可以分别用来做什么？分类有哪些局限性？分类的结构体里面有哪些成员？

①类别中原则上只能增加方法（能添加属性的的原因只是通过runtime能添加属性的的原因只是通过runtime的objc_setAssociatedObject和objc_getAssociatedObject方法解决无setter/getter的问题而已）；
②类扩展不仅可以增加方法，还可以增加实例变量（或者属性），只是该实例变量默认是@private类型的（
用范围只能在自身类，而不是子类或其他地方）；
③类扩展中声明的方法没被实现，编译器会报警，但是类别中的方法没被实现编译器是不会有任何警告的。这是因为类扩展是在编译阶段被添加到类中，而类别是在运行时添加到类中。
④类扩展不能像类别那样拥有独立的实现部分（@implementation部分），也就是说，类扩展所声明的方法必须依托对应类的实现部分来实现。
⑤定义在 .m 文件中的类扩展方法为私有的，定义在 .h 文件（头文件）中的类扩展方法为公有的。类扩展是在 .m 文件中声明私有方法的非常好的方式。

最重要的还是类扩展是在编译阶段被添加到类中，而类别是在运行时添加到类中。
分类方法未实现，编译器也不会报警告。
分类方法与原类中相同会优先调用分类。

分类的结构体
```
typedef struct objc_category *Category;
struct objc_category {
  char *category_name                          OBJC2_UNAVAILABLE; // 分类名
  char *class_name                             OBJC2_UNAVAILABLE; // 分类所属的类名
  struct objc_method_list *instance_methods    OBJC2_UNAVAILABLE; // 实例方法列表
  struct objc_method_list *class_methods       OBJC2_UNAVAILABLE; // 类方法列表
  struct objc_protocol_list *protocols         OBJC2_UNAVAILABLE; // 分类所实现的协议列表
}

```
###2，讲一下atomic的实现机制；为什么不能保证绝对的线程安全（最好可以结合场景来说）？

atomic是在setter和getter方法里会使用自旋锁spinlock_t来保证setter方法和getter方法的线程的安全。可以看做是getter方法获取到返回值之前不会执行setter方法里的赋值代码。如果不加atomic，可能在getter方法读取的过程中，再别的线成立发生setter操作，从而出现异常值。

加上atomic后，setter和getter方法是线程安全的，原子性的，但是出了getter方法和setter方法后就不能保证线程安全了
```
@property (atomic, strong) NSArray*                arr;
//thread A
for (int i = 0; i < 10000; i ++) {
    if (i % 2 == 0) {
        self.arr = @[@"1", @"2", @"3"];
    }
    else {
        self.arr = @[@"1"];
    }
}

//thread B
for (int i = 0; i < 100000; i ++) {
    if (self.arr.count >= 2) {
        NSString* str = [self.arr objectAtIndex:1];
    }
}

```
上面的例子线程B里面可能会因为数组越界而引起crash，因为加入在B线程里判断self.arr.count >= 2的时候数组是self.arr = @[@“1”, @“2”, @“3”];但是当调用[self.arr objectAtIndex:1]可能self.arr的值已经在线程A里被更改为了@[@“1”]，此时数组越界了。因此，虽然self.arr是atomic的，还是会出现线程安全问题。

###3，被weak修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable么？里面的结构可以画出来么？
被weak修饰的对象在被释放时候会置为nil，不同于assign；

Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。

1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
2、添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。

