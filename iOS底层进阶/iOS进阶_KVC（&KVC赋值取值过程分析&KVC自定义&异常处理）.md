## KVC（Key-value coding）

键值编码

### 基本使用

1.  能够对对象的私有成员进行取值赋值
2.  对数值和结构体型的属性进行的打包解包处理

实例： WTPerson.h
```
#import <Foundation/Foundation.h>

@interface WTPerson : NSObject{
//    @public  //@protect默认
    NSString * _name;
}

/** name  **/
//@property(nonatomic,strong)NSString * name;

@end
```

ViewController.m
```
#import "ViewController.h"
#import "WTPerson.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UITextField *text;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    WTPerson * p = [WTPerson new];
    //访问成员变量
    //p.name = @"wt";
    //NSLog(@"%@",p.name);

    //访问私有变量（必须要要设置为public才可访问）
    //p->_name = @"wt";
    //NSLog(@"%@",p->_name);

    //KVC(即使不用public修饰，也可以访问私有变量)
    [p setValue:@"wt" forKey:@"name"];
    NSLog(@"%@",[p valueForKey:@"name"]);

    [self.text setValue:[UIColor redColor] forKeyPath:@"_placeholderLabel.textColor"];
}
```


### KVC赋值取值过程分析和自定义及异常处理

#### 赋值过程

*   1、先找相关方法`set<Key>; _set<Key>; setIs<Key>;`
*   2、若是没有相关方法`+(BOOL)accessInstanceVariablesDirectly`判断是否可以直接访问成员变量
*   3、如果判断NO，直接执行KVC的`setValue:forUndefinedKey:(系统抛出一个异常，未定义key)`
*   4、如果是YES，继续找相关变量`_<key> _is<Key> <key> is<Key>`
*   5、方法或成员都不存在，`setValue:forUndefinedKey:`方法默认是抛出异常

#### 实例验证

WTPerson.h
```
#import <Foundation/Foundation.h>
@interface WTPerson : NSObject{
    @public  //@protect默认
    NSString * _name;
    NSString * _isName;
    NSString * name;
    NSString * isName;
}
@end
```
WTPerson.m
```
#import "WTPerson.h"
@implementation WTPerson

-(void)setName:(NSString *)name{
    NSLog(@"%s",__func__);
}

-(void)_setName:(NSString *)name{
    NSLog(@"%s",__func__);
}

-(void)setIsName:(NSString *)name{
    NSLog(@"%s",__func__);
}
@end
```

ViewController.m
```
#import "ViewController.h"
#import "WTPerson.h"

@interface ViewController ()
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    WTPerson * p = [WTPerson new]; 
    //验证KVC赋值过程
    [p setValue:@"wt" forKey:@"name"];

    NSLog(@"name = %@",p->name);
    NSLog(@"_name = %@",p->_name);
    NSLog(@"isname = %@",p->isName);
    NSLog(@"_isname = %@",p->_isName);
}

@end
```

*   运行程序，我们把`WTPerson.m`中的`-(void)setName:(NSString *)name`、`-(void)_setName:(NSString *)name`、`-(void)setIsName:(NSString *)name`三个方法依次注释，我们发现三个方法都会被依次执行。
*   然后我们把`WTPerson.h`中的`NSString * _name;`、`NSString * _isName;`、`NSString * name;`、`NSString * isName;`依次注释，我们会发现4个属性依次被赋值。

在`WTPerson.m`中我们让`accessInstanceVariablesDirectly`返回`NO`，则程序直接崩溃。

```
+ (BOOL)accessInstanceVariablesDirectly{
    return NO;
}
```

#### 取值过程

*   1、先找相关方法`get<Key>,key`
*   2、若没有相关方法，`+(BOOL)accessInstanceVariabkesDirectly`判断是否可以直接访问成员变量
*   3、如果是NO，直接执行KVC的`valueForUndefinedKey:`(系统抛出一个异常，未定义key)
*   4、如果是YES，继续找相关变量`_<key>、_is<Key>、<key>、is<Key>`
*   5、方法或成员都不存在，`valueForUndefineKey:`方法，默认是抛出异常

#### 实例验证

WTPerson.m
```
#import "WTPerson.h"

@implementation WTPerson

//- (NSString*) getName{
//    NSLog(@"%s",__func__);
//    return @"getName";
//}

- (NSString*) name {
    return @"name";
}

//+ (BOOL)accessInstanceVariablesDirectly{
//    return NO;
//}
@end
```

ViewController.m
```
#import "ViewController.h"
#import "WTPerson.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UITextField *text;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    WTPerson * p = [WTPerson new];

    //验证KVC取值过程
    NSLog(@"name = %@",[p valueForKey:@"name"]);
}

@end
```
取值方式与赋值方式大致相同。

### KVC自定义

#### 自定义KVC代码实现

创建分类`NSObject+KVC`

NSObject+KVC.h
```
#import <Foundation/Foundation.h>

@interface NSObject (KVC)

- (void)wt_setValue:(nullable id)value forKey:(NSString *)key;

- (id)wt_valueForKey:(NSString *)key;

@end
```

NSObject+KVC.m

```
#import "NSObject+KVC.h"
#import <objc/runtime.h>

@implementation NSObject (KVC)

- (id)wt_valueForKey:(NSString *)key{
    //判断是否合法
    if (key == nil && key.length ==0) {
        return nil;
    }

    //Key
    NSString * Key = key.capitalizedString;

    //先找相关方法 get<Key>,key
    NSString * getKey = [NSString stringWithFormat:@"get%@:",Key];

    if ([self respondsToSelector:NSSelectorFromString(getKey)]) {
        return [self performSelector:NSSelectorFromString(getKey)];
    }

    if ([self respondsToSelector:NSSelectorFromString(key)]) {
        return [self performSelector:NSSelectorFromString(key)];
    }

    if (![self.class accessInstanceVariablesDirectly]) {
        NSException * exception = [NSException exceptionWithName:@"NSUnknownKeyException" reason:@"setValue:forUndefineKey" userInfo:nil];
        @throw exception;
    }

    //再找相关变量
    //获取所有的成员变量
    unsigned int count = 0;
    Ivar * ivars = class_copyIvarList([self class], &count);
    NSMutableArray * arr = [[NSMutableArray alloc]init];
    for (int i = 0; i<count; i++) {
        Ivar var = ivars[i];
        const char * varName = ivar_getName(var);
        NSString *name = [NSString stringWithUTF8String:varName];
        [arr addObject:name];
    }

    //_<key> _is<Key> <key> is<Key>
    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"_%@",key]]) {

            return object_getIvar(self, ivars[i]);
        }
    }

    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"_is%@",Key]]) {

            return object_getIvar(self, ivars[i]);
        }
    }

    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"%@",key]]) {

            return object_getIvar(self, ivars[i]);
        }
    }

    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"is%@",Key]]) {
            return object_getIvar(self, ivars[i]);
        }
    }
    free(ivars);
    return nil;
}

- (void)wt_setValue:(nullable id)value forKey:(NSString *)key{

    //判断是否合法
    if (key == nil && key.length ==0) {
        return;
    }

    //Key
    NSString * Key = key.capitalizedString;

    //先找相关方法 set<Key>; _set<Key>; setIs<Key>;
    NSString * setKey = [NSString stringWithFormat:@"set%@:",Key];

    if ([self respondsToSelector:NSSelectorFromString(setKey)]) {
        [self performSelector:NSSelectorFromString(setKey) withObject:value];
        return;
    }

    NSString * _setKey = [NSString stringWithFormat:@"_set%@:",Key];

    if ([self respondsToSelector:NSSelectorFromString(_setKey)]) {
        [self performSelector:NSSelectorFromString(_setKey) withObject:value];
        return;
    }

    NSString * setIsKey = [NSString stringWithFormat:@"setIs%@:",Key];
    if ([self respondsToSelector:NSSelectorFromString(setIsKey)]) {
        [self performSelector:NSSelectorFromString(setIsKey) withObject:value];
        return;
    }

    if (![self.class accessInstanceVariablesDirectly]) {
        NSException * exception = [NSException exceptionWithName:@"NSUnknownKeyException" reason:@"setValue:forUndefineKey" userInfo:nil];
        @throw exception;
    }

    //再找相关变量
    //获取所有的成员变量
    unsigned int count = 0;
    Ivar * ivars = class_copyIvarList([self class], &count);
    NSMutableArray * arr = [[NSMutableArray alloc]init];
    for (int i = 0; i<count; i++) {
        Ivar var = ivars[i];
        const char * varName = ivar_getName(var);
        NSString *name = [NSString stringWithUTF8String:varName];
        [arr addObject:name];
    }

    //_<key> _is<Key> <key> is<Key>
    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"_%@",key]]) {
            object_setIvar(self, ivars[i], value);
            free(ivars);
            return;
        }
    }

    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"_is%@",Key]]) {
            object_setIvar(self, ivars[i], value);
            free(ivars);
            return;
        }
    }

    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"%@",key]]) {
            object_setIvar(self, ivars[i], value);
            free(ivars);
            return;
        }
    }

    for (int i = 0; i < count; i++) {
        NSString *keyName = arr[i];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"is%@",Key]]) {
            object_setIvar(self, ivars[i], value);
            free(ivars);
            return;
        }
    }

    [self setValue:value forUndefinedKey:Key];
    free(ivars);
}
@end
```

验证

ViewController.m

```
#import "ViewController.h"
#import "WTPerson.h"
#import "NSObject+KVC.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    WTPerson * p =[WTPerson new];
    [p wt_setValue:@"wt" forKey:@"name"];

    NSLog(@"name-KVC = %@",[p wt_valueForKey:@"name"]);
    NSLog(@"_name = %@",p->_name);
    NSLog(@"_isName = %@",p->_isName);
    NSLog(@"name = %@",p->name);
    NSLog(@"isName = %@",p->isName);
}
@end
```

在项目中 `commond+shift+o` 搜索`setValue:forKey`发现在`Foundation`框架下的`NSKeyValueCoding`文件下

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135425_fa1514cc_9027123.png "kvc1.png")


我们查看这个文件中的方法，发现这个文件中是一些分类的集合

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135434_0492b9c4_9027123.png "kvc2.png")


### KVC异常处理及正确性验证

#### KVC异常处理

*   1、赋值为空 `setNilValueForKey`
*   2、Key值不存在 `setValue:forUndefinedKey`

#### 正确性验证

validateValue 该方法的工作原理：

*   1、先找一下你的类中是否实现了方法 `-(BOOL)validate<Key>:error;`
*   2、如果实现了就会根据实现方法里面的自定义逻辑返回NO或者YES；如果没有实现这个方法，则系统默认返回YES

#### 示例代码

WTPerson…h
```
#import <Foundation/Foundation.h>

@interface WTPerson : NSObject

/** name  **/
@property(nonatomic,strong)NSString * name;

/** age  **/
@property(nonatomic,assign)int age;

@end
```

WTPerson.m
```
#import "WTPerson.h"

@implementation WTPerson

//对非对象类型，值不能为空
- (void) setNilValueForKey:(NSString *)key{
    NSLog(@"%@ 值不能为空",key);
}

//赋值的key不存在
- (void) setValue:(id)value forUndefinedKey:(NSString *)key{
    NSLog(@"key = %@值不存在",key);
}

//取值的key不存在
- (id) valueForUndefinedKey:(NSString *)key{
    NSLog(@"key = %@值不存在",key);
    return nil;
}

//正确性验证
- (BOOL) validateAge:(inout id  _Nullable __autoreleasing *)ioValue error:(out NSError * _Nullable __autoreleasing *)outError{
    NSNumber* value = (NSNumber*)*ioValue;
    NSLog(@"%@",value);
    if ([value integerValue] >= 0 && [value integerValue] <= 200) {
        return YES;
    }
    return NO;
}

@end
```
ViewController.m
```
#import "ViewController.h"
#import "WTPerson.h"
#import "WTContainer.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    WTPerson * p = [WTPerson new];

    //异常处理
    [p setValue:@18 forKey:@"name"];
    [p setValue:nil forKey:@"name"];
    NSLog(@"name = %@",p.name);

    [p setValue:nil forKey:@"age"];
    NSLog(@"age = %d",p.age);

    [p setValue:@"hello" forKey:@"name1"];

    NSLog(@"name = %@",[p valueForKey:@"name1"]);

    //万能容器
    WTContainer * container = [WTContainer new];

    [container setValue:@"wt" forKey:@"name"];
    [container setValue:@18 forKey:@"age"];

    NSLog(@"name = %@,age = %@",[container valueForKey:@"name"],[container valueForKey:@"age"]);

    //正确性验证
    NSNumber * value = @200;
    NSNumber * value1 = @199;

    if ([p validateValue:&value1 forKey:@"age" error:NULL]) {

        [p setValue:value1 forKey:@"age"];
    }

    NSLog(@"%@",[p valueForKey:@"age"]);
}

@end
```