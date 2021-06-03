## 一、单例模式

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/153418_fe0cb814_9027123.png "进阶.png")

　　单例模式是设计模式中最简单的一种，甚至有些模式大师都不称其为模式，称其为一种实现技巧，因为设计模式讲究对象之间的关系的抽象，而单例模式只有自己一个对象。

　　关于单例，有三个重要的准则需要牢记：

　　1\. 单例必须是唯一的(要不怎么叫单例？) 在程序生命周期中只能存在一个这样的实例。单例的存在使我们可以全局访问状态。例如：NSNotificationCenter, UIApplication和NSUserDefaults。

　　2\. 为保证单例的唯一性，单例类的初始化方法必须是私有的。这样就可以避免其他对象通过单例类创建额外的实例。

　　3\. 考虑到规则1，为保证在整个程序的生命周期中值有一个实例被创建，单例必须是线程安全的。并发有时候确实挺复杂，简单说来，如果单例的代码不正确，如果有两个线程同时实例化一个单例对象，就可能会创建出两个单例对象。也就是说，必须保证单例的线程安全性，才可以保证其唯一性。



### 　　实例：创建单例的两种方式

```
import UIKit
//final修饰符：可以防止类被继承，还可以防止子类重写父类的属性、方法以及下标。该修饰符不能修饰结构体和枚举。
final class SingleClass: NSObject
{
    //使用static修饰符，定义一个静态常量。静态常量在实例调用结束后不会消失，并且保留原值，即其内存不会被释放。当下次调用实例时，仍然使用常量原有的值。
    static let shared = SingleClass()
    //为了保持一个单例的唯一性，单例的构造器必须是private的。以防止其他对象也能创建出单例类的实例
    private override init() {}
     
    func say()
    {
        print("Hello, CoolKeTang!")
    }
}
SingleClass.shared.say()
 
//第二种
final class SecondSingletonClass: NSObject
{
    //使用static修饰符，定义一个静态变量。
    static var shared: SecondSingletonClass
    {
        //借助结构体来存储类型变量（class var），并使用let修饰符来保证线程的安全
        struct Static
        {
            static let instance: SecondSingletonClass = SecondSingletonClass()
        }
        return Static.instance
    }
    //为了保持一个单例的唯一性，单例的构造器必须是私有的，以防止其它对象也能创建出单例类的实例
    private override init() {}
     
    func say()
    {
        print("Hello, CoolKeTang!")
    }
}
```
二、Swift的三种传值方式
　　第一种：属性传值
　　属性传值很简单，适用于 从第一级传入第二级(正向传递)
```
//在要进入的控制器定义属性
class SecViewController: UIViewController {
    var labT = ""
}
 
//在一级控制器中给二级控制器赋值
let secVC = SecViewController()
secVC.labT = "属性传值"
```
第二种：代理传值
（适用于逆向传值）二级到一级
```
//在二级控制器定义代理协议
protocol  SecViewControllerDelegate{
 
    func SecDelegateSendValue(str:String)
}
 
class SecViewController: UIViewController {
    var labT = ""
    // 设置代理属性
     var delegetes:SecViewControllerDelegate! = nil
    //调用代理方法
    if (self.delegetes != nil) {
            self.delegetes.SecDelegateSendValue(str: "代理传值")
        }
//在一级控制器遵循协议
class ViewController: UIViewController,SecViewControllerDelegate
//实现协议方法
func SecDelegateSendValue(str:String)
    {
        prolab.text = str
    }
//成为代理者
secVC.delegetes = self
```
第三种：闭包传值
(本质是OC中的代码块传值)
```
// 定义闭包
typealias callBackFunc = (_ text:String) ->Void
//创建闭包对象
var selCallBack:callBackFunc?
//调用闭包
        if (selCallBack != nil) {
            selCallBack!("代码块传值")
        }
//在一级界面 获取闭包传的值
let secVC = SecViewController()
        secVC.labT = "属性传值"
 
        secVC.selCallBack = {(str:String) in
 
           self.Blocklab.text = str
        }
```

