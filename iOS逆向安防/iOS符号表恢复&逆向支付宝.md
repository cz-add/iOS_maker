![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153534_5144f1d8_9027123.jpeg "1.jpg")

# 前言

符号表历来是逆向工程中的“必争之地”，而iOS应用在上线前都会裁去符号表，以避免被逆向分析。

本文会介绍一个自己写的工具，用于恢复iOS应用的符号表。

直接看效果,支付宝恢复符号表后的样子:

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153253_83a66489_9027123.jpeg "2.jpg")

文章有点长，请耐心看到最后，亮点在最后。

# 为什么要恢复符号表

逆向工程中，调试器的动态分析是必不可少的，而 Xcode + lldb 确实是非常好的调试利器, 比如我们在Xcode里可以很方便的查看调用堆栈，如上面那张图可以很清晰的看到支付宝登录的RPC调用过程。

实际上，如果我们不恢复符号表的话，你看到的调试页面应该是下面这个样子：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153303_924cee1b_9027123.jpeg "3.jpg")

同一个函数调用过程，Xcode的显示简直天差地别。

原因是，Xcode显示调用堆栈中符号时，只会显示符号表中有的符号。为了我们调试过程的顺利，我们有必要把可执行文件中的符号表恢复回来。

# 符号表是什么

我们要恢复符号表，首先要知道符号表是什么，他是怎么存在于 Mach-O 文件中的。

符号表储存在 Mach-O 文件的 __LINKEDIT 段中，涉及其中的符号表（Symbol Table）和字符串表（String Table）。

这里我们用 MachOView 打开支付宝的可执行文件，找到其中的 Symbol Table 项。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153315_695cdab7_9027123.jpeg "4.jpg")

符号表的结构是一个连续的列表，其中的每一项都是一个 `struct nlist`。

```
//  位于系统库 <macho-o/nlist.h> 头文件中
struct nlist {
  union {
  //符号名在字符串表中的偏移量
    uint32_t n_strx;	
  } n_un;
  uint8_t n_type;
  uint8_t n_sect;
  int16_t n_desc;
  //符号在内存中的地址，类似于函数指针
  uint32_t n_value;
};
 ```

这里重点关注第一项和最后一项，第一项是符号名在字符串表中的偏移量，用于表示函数名，最后一项是符号在内存中的地址，类似于函数指针（这里只说明大概的结构，详细的信息请参考官方Mach O文件格式的文档）。

也就是说如果我们知道了符号名和内存地址的对应关系，我们是可以根据这个结构来逆向构造出符号表数据的。

知道了如何构造符号表，下一步就是收集符号名和内存地址的对应关系了。



#获取OC方法的符号表

因为OC语言的特性，编译器会将类名、函数名等编译进最后的可执行文件中，所以我们可以根据Mach-O文件的结构逆向还原出工程里的所有类，这也就是大名鼎鼎的逆向工具 class-dump 了。class-dump 出来的头文件里是有函数地址的：
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153325_aae3f9a0_9027123.jpeg "5.jpg")

所以我们只要对class-dump的源码稍作修改，即可获取我们要的信息。

#符号表恢复工具

整理完数据格式，又理清了数据来源，我们就可以写工具了。


我们来看看怎么用这个工具：

1.下载源码编译
```
git clone --recursive https://github.com/tobefuturer/restore-symbol.git
cd restore-symbol && make
./restore-symbol
```
2.恢复OC的符号表，非常简单
```
./restore-symbol ./origin_AlipayWallet -o ./AlipayWallet_with_symbol
```

origin_AlipayWallet 为Clutch砸壳后，没有符号表的 Mach-O 文件
-o 后面跟输出文件位置

3.把 Mach-O 文件重签名打包，看效果

文件恢复符号表后，多出了20M的符号表信息
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153334_a2c2bdea_9027123.jpeg "6.jpg")

Xcode里查看调用栈
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153342_c2081ef8_9027123.jpeg "7.jpg")

可以看到，OC函数这部分的符号已经恢复了，函数调用栈里已经能看出大致的调用过程了，但是支付宝里，采用了block的回调形式，所以还有很大一部分的符号没能正确显示。

下面我们就来看看怎么样恢复这部分block的符号。

# 获取block的符号信息

还是同样的思路，要恢复block的符号信息，我们必须知道block在文件中的储存形式。

## block在内存中的结构

首先，我们先分析下运行时，block在内存中的存在形式。block在内存中是以一个结构体的形式存在的，大致的结构如下：
```
struct __block_impl {
  /**
  block在内存中也是类NSObject的结构体，
  结构体开始位置是一个isa指针
  */
  Class isa;
  
  /** 这两个变量暂时不关心 */
  int flags;
  int reserved;
  
  /**
  真正的函数指针！！
  */
  void (*invoke)(...);
  ...
}
```

说明下block中的isa指针，根据实际情况会有三种不同的取值，来表示不同类型的block：

1.  _NSConcreteStackBlock

    栈上的block，一般block创建时是在栈上分配了一个block结构体的空间，然后对其中的isa等变量赋值。

2.  _NSConcreteMallocBlock

    堆上的block，当block被加入到GCD或者被对象持有时，将栈上的block复制到堆上，此时复制得到的block类型变为了_NSConcreteMallocBlock。

3.  _NSConcreteGlobalBlock

    全局静态的block，当block不依赖于上下文环境，比如不持有block外的变量、只使用block内部的变量的时候，block的内存分配可以在编译期就完成，分配在全局的静态常量区。

第2种block在运行时才会出现，我们只关注1、3两种，下面就分析这两种isa指针和block符号地址之间的关联。

##block isa指针和符号地址之间的关联

分析这部分需要用到IDA这个反汇编软件, 这里结合两个实际的小例子来说明：

###1._NSConcreteStackBlock

假设我们的源代码是这样很简单的一个block：
```
@implementation ViewController
- (void)viewDidLoad {
    int t = 2;
    void (^ foo)() = ^(){
        NSLog(@"%d", t); //block 引用了外部的变量t
    };
    foo();
}
@end
```
编译完后，实际的汇编长这个样子：
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153350_7186204e_9027123.jpeg "8.jpg")

实际运行时，block的构造过程是这样：

1.  为block开辟栈空间
2.  为block的isa指针赋值（一定会引用全局变量：`_NSConcreteStackBlock`）
3.  获取函数地址，赋值给函数指针

所以我们可以整理出这样一个特征：

***重点来了!!!***

***凡是代码里用到了栈上的block，一定会获取`__NSConcreteStackBlock`作为isa指针，同时会紧接着获取一个函数地址，那个函数地址就是block的函数地址。***

结合下面这个图，仔细理解上面这句话
（这张图和上面那张图是同一个文件，不过裁掉了符号表）
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153401_9f3a92b7_9027123.jpeg "9.jpg")

利用这个特征，逆向分析时我们可以做如下推断：

在一个OC方法里发现引用了`__NSConcreteStackBlock`这个变量，那么在这附近，一定会出现一个函数地址，这个函数地址就是这个OC方法里的一个block。

比如上面图中，我们发现 viewDidLoad 里，引用了`__NSConcreteStackBlock`,同时紧接着加载了 sub_100049D4 的函数地址，那我们就可以认定sub_100049D4是viewDidLoad里的一个block, sub_100049D4函数的符号名应该是 viewDidLoad_block.

### 2\. _NSConcreteGlobalBlock

全局的静态block，是那种不引用block外变量的block，他因为不引用外部变量，所以他可以在编译期就进行内存分配操作，也不用担心block的复制等等操作，他存在于可执行文件的常量区里。

不太理解的话，看个例子：

我们把源代码改成这样：
```
@implementation ViewController
- (void)viewDidLoad {
   
    void (^ foo)() = ^(){
        //block 不引用外部的变量
        NSLog(@"%d", 123);
    };
    foo();
}
@end
```
那么在编译后会变成这样：
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153410_cf68a438_9027123.jpeg "10.jpeg")

那么借鉴上面的思路，在逆向分析的时候，我们可以这么推断

1.  在静态常量区发现一个_NSConcreteGlobalBlock的引用
2.  这个地方必然存在一个block的结构体数据
3.  在这个结构体第16个字节的地方会出现一个值，这个值是一个block的函数地址

###3\. block 的嵌套结构

实际在使用中，可能会出现block内嵌block的情况：
```
- (void)viewDidLoad {
  dispatch_async(background_queue ,^{
    ...
    dispatch_async(main_queue, ^{
      ...     
    });
  });
}
```


所以这里block就出现了父子关系，如果我们将这些父子关系收集起来，就可以发现，这些关系会构成图论里的森林结构，这里可以简单用递归的深度优先搜索来处理，详细过程不再描述。

## block符号表提取脚本（IDA+python）

整理上面的思路，我们发现搜索过程依赖于IDA提供各种引用信息，而IDA是提供了编程接口的，可以利用这些接口来提取引用信息。

IDA提供的是Python的SDK，最后完成的脚本也放在仓库里[search_oc_block/ida_search_block.py](https://github.com/tobefuturer/restore-symbol/blob/master/search_oc_block/ida_search_block.py)。

##提取block符号表

这里简单介绍下怎么使用上面这个脚本

1.  用IDA打开支付宝的 Mach-O 文件
2.  等待分析完成！ 可能要一个小时
3.  Alt + F7 或者 菜单栏 `File -> Script file...`
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153420_19619085_9027123.jpeg "11.jpg")

4.  等待脚本运行完成，预计30s至60s，运行过程中会有这样的弹窗
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153426_9cfbfe5c_9027123.jpeg "12.jpg")

5.  弹窗消失即block符号表提取完成
6.  在IDA打开文件的目录下,会输出一份名为`block_symbol.json`的json格式block符号表
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153431_3f905dd2_9027123.jpeg "13.jpg")

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153501_96979250_9027123.jpeg "14.jpg")

#恢复符号表&实际分析

用之前的符号表恢复工具，将block的符号表导入Mach-O文件
```
./restore-symbol ./origin_AlipayWallet -o ./AlipayWallet_with_symbol -j block_symbol.json
```

-j 后面跟上之前得到的json符号表

最后得到一份同时具有OC函数符号表和block符号表的可执行文件

这里简单介绍一个分析案例, 你就能体会到这个工具的强大之处了。

1.  在Xcode里对 `-[UIAlertView show]` 设置断点
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153509_3fdffcc1_9027123.jpeg "15.jpg")

2.  运行程序，并在支付宝的登录页面输入手机号和*错误的密码*，点击登录
3.  Xcode会在‘密码错误’的警告框弹出时停下，左侧会显示出这样的调用栈

#一张图看完支付宝的登录过程*

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/153515_4ff37118_9027123.jpeg "16.jpg")

