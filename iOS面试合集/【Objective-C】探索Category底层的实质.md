无论一个类设计的多么完美，在未来的需求演进中，都有可能会碰到一些无法预测的情况。那怎么扩展已有的类呢？一般而言，继承和组合是不错的选择。但是在Objective-C 2.0中，又提供了category这个语言特性，可以动态地为已有类添加新行为。如今category已经遍布于Objective-C代码的各个角落，从Apple官方的framework到各个开源框架，从功能繁复的大型APP到简单的应用，catagory无处不在。本文对category做了比较全面的整理，希望对读者有所裨益。　　

　　**Objective-C中类别特性的作用如下：**

　　（1）可以将类的实现分散到多个不同文件或多个不同框架中（补充新的方法）。

　　（2）可以创建私有方法的前向引用。

　　（3）可以向对象添加非正式协议。

　　**Objective-C中类别特性的局限性如下：**

　　（1）类别只能想原类中添加新的方法，且只能添加而不能删除或修改原方法，不能向原类中添加新的属性。

　　（2）类别向原类中添加的方法是全局有效的而且优先级最高，如果和原类的方法重名，那么会无条件覆盖掉原来的方法。　

　**一、Category的底层实现**
　　 Objective-C 通过 Runtime 运行时来实现动态语言这个特性，所有的类和对象，在 Runtime 中都是用结构体来表示的，Category 在 Runtime 中是用结构体 category_t 来表示的，下面是结构体 category_t 具体表示：
```typedef struct category_t {
    const char *name;//类的名字 主类名字
    classref_t cls;//类
    struct method_list_t *instanceMethods;//实例方法的列表
    struct method_list_t *classMethods;//类方法的列表
    struct protocol_list_t *protocols;//所有协议的列表
    struct property_list_t *instanceProperties;//添加的所有属性
} category_t;
```
　　从category的定义也可以看出category的可为（可以添加实例方法，类方法，甚至可以实现协议，添加属性）和不可为（无法添加实例变量）。

　　我们将结合 runtime 的源码探究下 Category 的实现原理。打开 runtime 源码工程，在文件 objc-runtime-new.mm 中找到以下函数：
```
void _read_images(header_info **hList, uint32_t hCount)
{
    ...
        _free_internal(resolvedFutureClasses);
    }
 
    // Discover categories.
    for (EACH_HEADER) {
        category_t **catlist =
            _getObjc2CategoryList(hi, &count);
        for (i = 0; i < count; i++) {
            category_t *cat = catlist[i];
            Class cls = remapClass(cat->cls);
 
            if (!cls) {
                // Category's target class is missing (probably weak-linked).
                // Disavow any knowledge of this category.
                catlist[i] = nil;
                if (PrintConnecting) {
                    _objc_inform("CLASS: IGNORING category \?\?\?(%s) %p with "
                                 "missing weak-linked target class",
                                 cat->name, cat);
                }
                continue;
            }
 
            // Process this category.
            // First, register the category with its target class.
            // Then, rebuild the class's method lists (etc) if
            // the class is realized.
            BOOL classExists = NO;
            if (cat->instanceMethods ||  cat->protocols
                ||  cat->instanceProperties)
            {
                addUnattachedCategoryForClass(cat, cls, hi);
                if (cls->isRealized()) {
                    remethodizeClass(cls);
                    classExists = YES;
                }
                if (PrintConnecting) {
                    _objc_inform("CLASS: found category -%s(%s) %s",
                                 cls->nameForLogging(), cat->name,
                                 classExists ? "on existing class" : "");
                }
            }
 
            if (cat->classMethods  ||  cat->protocols
                /* ||  cat->classProperties */)
            {
                addUnattachedCategoryForClass(cat, cls->ISA(), hi);
                if (cls->ISA()->isRealized()) {
                    remethodizeClass(cls->ISA());
                }
                if (PrintConnecting) {
                    _objc_inform("CLASS: found category +%s(%s)",
                                 cls->nameForLogging(), cat->name);
                }
            }
        }
    }
 
    // Category discovery MUST BE LAST to avoid potential races
    // when other threads call the new category code before
    // this thread finishes its fixups.
 
    // +load handled by prepare_load_methods()
 
    ...
}
```
　　我们可以知道在这个函数中对 Category 做了如下处理：

　　(1)将 Category 和它的主类（或元类）注册到哈希表中;

　　(2)如果主类（或元类）已实现，那么重建它的方法列表;

　　**Category的实现原理：**

- 在编译时期，会将分类中实现的方法生成一个结构体 method_list_t 、将声明的属性生成一个结构体 property_list_t ，然后通过这些结构体生成一个结构体 category_t 。
- 然后将结构体 category_t 保存下来
- 在运行时期，Runtime 会拿到编译时期我们保存下来的结构体 category_t
- 然后将结构体 category_t 中的实例方法列表、协议列表、属性列表添加到主类中
- 将结构体 category_t 中的类方法列表、协议列表添加到主类的 metaClass 中



**二、为何Category中的方法优先级高于原类中的方法？**
　　category_t 中的方法列表是插入到主类的方法列表前面（类似利用链表中的 next 指针来进行插入），所以这里 Category 中实现的方法并不会真正的覆盖掉主类中的方法，只是将 Category 的方法插到方法列表的前面去了。运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法，就会停止查找，这里就会出现覆盖方法的这种假象了。
```
// 这里大概就类似这样子插入
newproperties->next = cls->data()->properties;
cls->data()->properties = newproperties;，
```
**三、为何Category中不能添加实例变量？**
　　通过结构体 category_t ，我们就可以知道，在 Category 中我们可以增加实例方法、类方法、协议、属性。这里没有 objc_ivar_list 结构体，代表我们不可以在分类中添加实例变量。
　　因为在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这个就是 Category 中不能添加实例变量的根本原因。

