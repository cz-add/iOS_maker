### 这是一篇学习swift的笔记

Objective-C是很好的语言，Runtime机制、消息机制等也是爱不释手。 Swift一直在更新，闲暇时间学一遍。学习的Blog：《从零开始学swift》**以下代码全部在playground进行的尝试**

## 变量

let 是常量 var 是变量 不能修改的使用常量可以提高程序的可读性。

```
var str = "Hello, playground"
print(str)

let constA:Int = 12
let constB = 12
let constC = 12; let constD:Float = 12
```
元组：关系数据库中的基本概念，元组表中的一条记录，每列就是一个字段。因此在二维表里，元组也称为记录。元组是Swift中特有的。

```
var abc = (10, 12, 30, "abc")
var score = (id:"1001", name:"张三", english_score:30, chinese_score:90)
score.english_score = 100
score
```
### 运算符

*   引用号（.）：实例调用属性、方法等操作符。
*   问号（?）：用来声明可选类型。
*   感叹号（!）：对可选类型值进行强制拆封。
*   箭头（->）：说明函数或方法返回值类型。
*   冒号运算符（:）：用于字典集合分割“键值”对。
*   ..< 运算符是一个非包函范围运算符，不包括上限值。
*   运算符 ... ：类似上面，包括上限值。
*   ?? 操作符： a ?? b 如果a不为nil，返回a里面的值，否则返回b的值

操作符要求类型一致（int8 和 int16是不同的）

```
let short:Int16 = 277
let sshort:Int8 = 100
//Int8(short) + sshort
short + Int16(sshort)
```
`..<`运算符
```
 let arr = ["abc", "bcd"]
        for i in 0..<arr.count {
            if someC[i] != anotherC[i] {
                return false
            }
```
### for in

遍历容器。可以用变量遍历，也可以用元组遍历。 如下

```
var stList = [String]()
stList.insert("abc", atIndex: 0)
stList.insert("bcd", atIndex: 1)
stList.insert("cde", atIndex: 0)

var stList2:[String] = ["a", "B", "c"];

for st in stList2 {
    print(st)
}

for (index, value) in stList.enumerate() {
    print("index\(index):\(value)")
}
```

### 属性

#### 属性类型

*   存储属性：存储数据，分为常量属性（let）和变量属性（var），如下面的name和number。
*   计算属性：不存储数据，通过计算其他属性返回数据，如下面的workYears。

当声明属性时，声明必须为它们**设置初始值**，或者在初始化时设置初始值。 如果不希望为属性设置初始值，必须声明它们作为可选。

```
var strLin = "loyinglin" //设置初始值
var str:String? //声明为可选
```
#### 读写器

*   get
*   set
*   willSet
*   didSet 见下代码。

```
class Person {
    var name:String = "loying"
    var number:Int = 2010551110
    private var mYear = 2016 {
        willSet(year) {
            print("willSet\(year)")
        }
        didSet(year) {
            print("didSet\(year)")
        }
    }
    var workYears:Int {
        get {
            print("get year")
        return 2016 - mYear
        }
        set(year) {
            if (year >= 0){
                mYear = 2016 - year
            }
        }
    }
}
```
### 访问权限控制

#### 访问权限类型

*   public：可以访问自己模块中的任何public实体，也可以访问其他模块中的public实体。
*   internal：只能访问自己模块的任何internal实体，不能访问其他模块中的internal实体。默认权限是internal。
*   private：只能在当前源文件中使用的实体，称为私有实体。

```
public class Person {
    private var name:String = "loying"
    internal var number:Int = 2010551111
}
```

#### 原则

*   统一性原则：A如果包含B，那么A的权限开放等级大于等于B。 （元组类型的访问级别遵循元组中字段最低级的访问级别）
*   设计原则：对外开放使用public，对外封闭使用internal或private。

### 结构体与类

结构体：值类型，每个实例没有独一无二的标识。

类：引用类型，每个实例有独一无二的标识。

### 可选链

可选链是一种可以调用属性、方法的过程，用于调用的对象可能为nil。 如果目标有值，调用就会成功；如果目标为nil，调用将返回nil。 多次请求或调用可以被链接成一个链，如果任意一个节点为nil将导致整条链失效。 ** 通过可选链调用方法时返回值总是可选类型的 **。 调用 Optional 对象方法前，必须拆包: 使用问号（?）声明的可选类型，在拆包时需要使用感叹号（!），这种拆包方式称为“显式拆包”； 使用感叹号（!）声明的可选类型，在拆包时可以不使用感叹号（!），这种表示方式称为“隐式拆包”。

```
var nilInt:String? = "abc"
let textNil = nilInt
if let test = nilInt {
    print("OK \(test)  \(textNil) \(textNil!)") //观察输出值
}
if (textNil != nil) {
    print("textNil \(textNil)")
}
```
### 函数

多种实现方式，具体如下

```
func plus(width:Int, height:Int)->Int { //普通函数
    return width + height
}
print("1 + 2 = \(plus(1, height: 2))")
plus((Int)(short), height: Int(sshort))

func plus2(W width:Int, H Height:Int)->Int { //参数别名
    return width + Height
}
plus2(W:12, H:23)

func plus3(width:Int, _ Height:Int)->Int { //不带参数名
    return width + Height
}
plus3(22, 33)

func plus4(width:Int, _ Height:Int = 10)->Int { //默认参数值
    return width + Height
}
plus4(20, 20)
plus4(20)

func plusMore(width:(Length:Int, Size:Int), _ mul:Int) -> (Length:Int, Size:Int) { //元组参数
    let ret = (Length:width.Length * mul,
        Size:width.Size * mul)
    return ret
}
plusMore((Length: 20, Size: 5), 3)
```

### 构造函数

*   便利构造函数：带convenience的init，必须调用指定构造函数
*   指定构造函数：不带convenience的init。 构造函数的主要作用是初始化实例，包括初始化存储属性和其它的初始化。

```
 class Person {
    var name:String = "loying"
    var number:Int = 2010551110

    init(id:Int) {
        number = id
    }
    convenience init() {
        self.init(id:0)
    }
}
```

### 继承

在Swift中，类是**单继承**。多重继承通过多个协议实现。

#### 属性、方法、下标

类可以继承另一个类的**方法、属性、下标**等特征。 子类继承父类后，可以**重写**父类的方法、属性、下标等特征。

#### 读写器

你可以将一个继承来的只读属性重写为一个读写属性，只需要你在重写版本的属性里提供 getter 和 setter 即可。但是，不可以将一个继承来的读写属性重写为一个只读属性。 你不可以为继承来的常量存储型属性或继承来的只读计算型属性添加属性观察器。这些属性的值是不可以被设置的，所以，为它们提供willSet 或didSet 实现是不恰当。

#### 重写静态属性

我们可以在子类中重写从父类继承来的属性，属性有实例属性和**静态属性**之分。 class修饰的属性可以被重写，static关键字就不能被重写。

#### 重写静态方法

静态方法使用class或static关键字，class修饰的静态方法可以被重写，static关键字就不能被重写。

#### 重写属性

下标是一种特殊属性。子类属性重写是重写属性的getter和setter访问器，对下标的重写也是重写下标的getter和setter访问器。

#### final

final声明的类不能被继承，final声明的属性、方法和下标不能被重写。

### 错误处理

#### Cocoa错误处理模式

构造函数的最后一个参数是NSErrorPointer（即NSError指针），那么在实际调用时候我们需要传递err变量地址（即&err），&是取地址符。当方法调用完成后，如果有错误则err变量会被赋值。

#### swift错误处理模式

使用do - try - catch 模式 使用了try?语句没有必要使用do-catch语句将其包裹起来。 所以使用try!打破错误传播链条时，应该确保程序不会发生错误。

```
do {
    let path:String? = NSBundle.mainBundle().pathForResource("text", ofType: "txt")

    let str = try NSString(contentsOfFile:path!, encoding:NSUTF8StringEncoding)
    print(str)
}
catch let err as NSError {
    err.description
}
```

### 捕获列表

捕获列表中的每个元素都是由weak或者unowned关键字和实例的引用（如self）成对组成。每一对都在方括号中，通过逗号分开

```
NSNotificationCenter.defaultCenter().addObserverForName("DoneModelChange", object: nil, queue: nil) { [unowned self](note) -> Void in
            self.tableView.reloadData()
        }
```
### 闭包

#### 尾随闭包

闭包表达式是函数最后一个参数，调用函数可以使用尾随闭包写法。 如下：

```
func plusAfterMul(plus:Int, factor1:Int, factor2:Int, funMul:(Int, Int)->Int)->Int {
    return plus + funMul(factor1, factor2)
}
plusAfterMul(1, factor1: 2, factor2: 3) {
    (a:Int, b:Int)->Int in
    var ret = 1
    var count = b
    while count > 0 {
        --count
        ret *= a
    }
    return ret
}
```

#### 单一表达式隐式返回

如下，排序函数的第二个参数的函数类型明确指出，一个布尔值必须由闭包返回。因为闭包体内含有一个表达式(s1 > s2)返回一个布尔值， 不会出现歧义，其返回关键字可以省略。

```
let count = [5, 10, -6, 75, 20]
var descending = sorted(count, { n1, n2 in n1 > n2 })
var ascending = sorted(count, { n1, n2 in n1 < n2 })
println(descending)
println(ascending)
```
### Any、AnyObject

AnyObject是一个协议，Any是零个协议。 AnyObject用于任何类实例，而Any用于任何变量。 AnyObject 可以代表任何 class 类型的实例。 看如下**例子** 声明一个 Int 和一个 String，按理说它们都应该只能被 Any 代表，而不能被 AnyObject 代表的。 实际上，元素其实已经变成了 NSNumber 和 NSString 。因为我们 import 了 UIKit 。因为我们显式地声明了需要 AnyObject，编译器认为我们需要的的是 Cocoa 类型而非原生类型，而帮我们进行了自动的转换。

```
import UIKit
let swiftInt: Int = 1
let swiftString: String = "miao"
var array: [AnyObject] = []
array.append(swiftInt)
array.append(swiftString)
```

在上面的代码中如果我们把 import UIKit 去掉，会得到无法适配 AnyObject 的编译错误。 可以把声明 array 的 [AnyObject] 换成 [Any]。

### 关联类型

Swift 允许相关类型，并可由关键字“typealias”协议定义内部声明。

*   类似typedef，可以给对象一个别名；
*   代表泛型；

```
protocol Container {
    typealias ItemType
    mutating func append(item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}
```

### 运算符重载

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/154138_d9e8d558_9027123.png "比.png")


Paste_Image.png

### 惰性初始化

*   简单表达式

`lazy var first = NSArray(objects: "1","2")`

*   闭包

`lazy var second:String = { return "second" }()`

不要忘记最后的*小括号*，只有加了小括号，闭包才会在调用的时立刻执行。

要类型声明`lazy var second:String` 这样Xcode会进行类型检查。

### 泛型函数、协议的应用

```
protocol Container {
    typealias ItemType
    mutating func append(item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}

struct Stack<T>: Container {
    var items = [T]()

    mutating func push(item: T) {
        items.append(item)
    }

    mutating func append(item: T) {
        self.push(item)
    }

    var count: Int {
        return items.count
    }

    subscript(i: Int) -> T {
        return items[i]
    }
}

func allItemsMatch<
    C1: Container, C2: Container
    where C1.ItemType == C2.ItemType, C1.ItemType:Equatable>(someC: C1, anotherC: C2) -> Bool {
        if someC.count != anotherC.count {
            return false
        }

        for i in 0..<someC.count {
            if someC[i] != anotherC[i] {
                return false
            }
        }

        return true
}

var stackOfStrings = Stack<String>()
stackOfStrings.push("abc")
stackOfStrings.push("qwe")

var arrayOfString = Stack<String>()
arrayOfString.push("abc")
arrayOfString.push("aaa")

if allItemsMatch(stackOfStrings, anotherC: arrayOfString) {
    print("ok")
}
else {
    print("not equal")
}
```
## 总结

学习的过程中，先对着《从零开始学swift》，把swift的代码实现一遍。 大概有300多行代码，写完之后语法基本熟悉了。 然后找一个oc的项目，把里面的oc代码全部用swift实现一遍。swift入门就结束了。 整个代码文件如下

```
//: Playground - noun: a place where people can play

import UIKit

//看右侧
var str = "Hello, playground"

//简单的打印
print(str)

let image = UIImage(named: "abc.jpeg")

let view = UIView(frame: CGRectMake(0, 0, 100, 100))

let constA:Int = 12
let constB = 12
let constC = 12; let constD:Float = 12

var abc = (10, 12, 30, "abc")
var score = (id:"1001", name:"张三", english_score:30, chinese_score:90)
score.english_score = 100
score

var strLin = "loyinglin"

strLin.insert("L", atIndex: strLin.endIndex)

//strLin.insertContentsOf("abc", at: strLin.endIndex)

//strLin.replaceRange(range, with: "abc")
(strLin as NSString).substringWithRange(NSMakeRange(0, 4))
strLin.substringWithRange(Range(start: strLin.startIndex.advancedBy(1), end: strLin.endIndex.advancedBy(-2)))

let short:Int16 = 277
let sshort:Int8 = 100
//Int8(short) + sshort
short + Int16(sshort)

var stList = [String]()
stList.insert("abc", atIndex: 0)
stList.insert("bcd", atIndex: 1)
stList.insert("cde", atIndex: 0)

var stList2:[String] = ["a", "B", "c"];

for st in stList2 {
    print(st)
}

for (index, value) in stList.enumerate() {
    print("index\(index):\(value)")
}

func plus(width:Int, height:Int)->Int {
    return width + height
}
print("1 + 2 = \(plus(1, height: 2))")
plus((Int)(short), height: Int(sshort))

func plus2(W width:Int, H Height:Int)->Int {
    return width + Height
}
plus2(W:12, H:23)

func plus3(width:Int, _ Height:Int)->Int {
    return width + Height
}
plus3(22, 33)

func plus4(width:Int, _ Height:Int = 10)->Int {
    return width + Height
}
plus4(20, 20)
plus4(20)

func plusMore(width:(Length:Int, Size:Int), _ mul:Int) -> (Length:Int, Size:Int) {
    let ret = (Length:width.Length * mul,
        Size:width.Size * mul)
    return ret
}
plusMore((Length: 20, Size: 5), 3)

func plusAfterMul(plus:Int, factor1:Int, factor2:Int, funMul:(Int, Int)->Int)->Int {
    return plus + funMul(factor1, factor2)
}
plusAfterMul(1, factor1: 2, factor2: 3) {
    (a:Int, b:Int)->Int in
    var ret = 1
    var count = b
    while count > 0 {
        --count
        ret *= a
    }
    return ret
}

enum Days:Int{
    case Mon
    case Tus, Wed, Thu, Fri
}

var bool = Days.Mon == Days.Thu

internal class Department {
    var dName:String = "department-1"
    let dNumber:Int = 123
}

private class Person {
    var name:String = "loying"
    var number:Int = 2010551110
    private var mYear = 2016 {
        willSet(year) {
            print("willSet\(year)")
        }
        didSet(year) {
            print("didSet\(year)")
        }
    }
    var Dept:Department?
    var YDept:Department!
    var workYears:Int {
        get {
            print("get year")
        return 2016 - mYear
        }
        set(year) {
            if (year >= 0){
                mYear = 2016 - year
            }
        }
    }

    init(id:Int) {
        number = id
        print("total person\(++Person.totalPerson) and \(self)")
    }

    convenience init() {
        self.init(id:0)
    }

    static var totalPerson = 0
}

private class Student: Person {
    var school:String {
        willSet(value) {
            print("school is \(value)")
        }
    }
    convenience init(sch:String) {
        self.init(sch:sch, id:123)
    }
    init(sch:String, id:Int) {
        school = sch
        super.init(id: id)
    }

    override var workYears:Int {
        get {
            return super.workYears
        }
        set(year) {
            super.workYears = year
        }
    }

    init() {
        school = "ly"
        super.init(id:123)
    }

    deinit {
        print("dealloc \(school)")
    }

}

private var st = Student(sch: "abc")
st.school
st.workYears

private var man = Person()
man = Person(id: 123)
man.number
man.mYear
man.workYears
man.workYears = 3
man.mYear
Person()

private var arrMen = [st, man]
for item in arrMen {
    let test = item as? Student
    if (test != nil) {
        (item as! Student).workYears
        print("item is student in \(test?.workYears)")
    }
    if let test2 = item as? Student {
        print("test2 \(test2)")
    }
}

man.Dept?.dNumber
man.Dept = Department()
man.Dept!.dNumber

man.YDept?.dNumber

var opStr:String?
print(opStr)
//print(opStr!)

var nilInt:String? = "abc"
let textNil = nilInt
if let test = nilInt {
    print("OK \(test)  \(textNil) \(textNil!)")
}
if (textNil != nil) {
    print("textNil \(textNil)")
}

struct DoubleArray: Equatable {
    let rows:Int, columns:Int
    var grid:[Int]
    init(row:Int, column:Int) {
        self.columns = column
        self.rows = row

        grid = Array(count: row * column, repeatedValue: 0)
    }

    subscript(row:Int, column:Int)->Int {
        return grid[row * columns + column]
    }
    mutating func setValue(row:Int, column:Int, value:Int) {
        grid[row * columns + column] = value
    }

    var count:Int {
        return grid.count
    }
}
func ==(lhs: DoubleArray, rhs: DoubleArray) -> Bool {
    return lhs.count == rhs.count
}

var dArr:DoubleArray = DoubleArray(row: 3, column: 3)
dArr[2, 2]
dArr.setValue(1, column: 1, value: 1)
dArr[1, 1]

extension Int {
    var lyDesc:String {
        var desc = ""
        switch (self) {
        case 4: desc = "Bad"
        case 8: desc = "Good"
        default:desc = "nothing"
        }
        return desc
    }

    func lyPrint() {
        print("ly Print self:\(self)")
    }
}

1.lyDesc
4.lyDesc
8.lyDesc

1234567.lyPrint()

var ab = {(a:Int, b:Int) -> Int in
    var ret = a + b
    return ++ret
}
ab(3, 4)

var moba:(Int, Int)->Int = {$0 * $1}
moba(4, 6)

do {
    let path:String? = NSBundle.mainBundle().pathForResource("text", ofType: "txt")

    let str = try NSString(contentsOfFile:path!, encoding:NSUTF8StringEncoding)
    print(str)
}
catch let err as NSError {
    err.description
}
str

var msg:String!
if msg != nil {

}
else {
    print("OJ\(msg)")
}

protocol Container {
    typealias ItemType
    mutating func append(item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}

struct Stack<T>: Container {
    var items = [T]()

    mutating func push(item: T) {
        items.append(item)
    }

    mutating func append(item: T) {
        self.push(item)
    }

    var count: Int {
        return items.count
    }

    subscript(i: Int) -> T {
        return items[i]
    }
}

func allItemsMatch<
    C1: Container, C2: Container
    where C1.ItemType == C2.ItemType, C1.ItemType:Equatable>(someC: C1, anotherC: C2) -> Bool {
        if someC.count != anotherC.count {
            return false
        }

        for i in 0..<someC.count {
            if someC[i] != anotherC[i] {
                return false
            }
        }

        return true
}

var stackOfStrings = Stack<String>()
stackOfStrings.push("abc")
stackOfStrings.push("qwe")

var arrayOfString = Stack<String>()
arrayOfString.push("abc")
arrayOfString.push("aaa")

if allItemsMatch(stackOfStrings, anotherC: arrayOfString) {
    print("ok")
}
else {
    print("not equal")
}
```
#推荐文章
[Swift中构造方法的解析](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/Swift/Swift%E4%B8%AD%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95%E7%9A%84%E8%A7%A3%E6%9E%90.md)
[Objective-C与Swift的贯通编程](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/iOS%E9%9D%A2%E8%AF%95%E5%90%88%E9%9B%86/Objective-C%E4%B8%8ESwift%E7%9A%84%E8%B4%AF%E9%80%9A%E7%BC%96%E7%A8%8B.md)
[Swift、OC分别实现用" | "隔开数组且只显示一行的小功能](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/Swift/Swift%E3%80%81OC%E5%88%86%E5%88%AB%E5%AE%9E%E7%8E%B0%E7%94%A8%22%20%7C%20%22%E9%9A%94%E5%BC%80%E6%95%B0%E7%BB%84%E4%B8%94%E5%8F%AA%E6%98%BE%E7%A4%BA%E4%B8%80%E8%A1%8C%E7%9A%84%E5%B0%8F%E5%8A%9F%E8%83%BD.md)
[Swift开发之泛型实例](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/Swift/Swift%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B3%9B%E5%9E%8B%E5%AE%9E%E4%BE%8B.md)
[【Swift实现代码】iOS架构模式之MVP](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/Swift/%E3%80%90Swift%E5%AE%9E%E7%8E%B0%E4%BB%A3%E7%A0%81%E3%80%91iOS%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%E4%B9%8BMVP.md)

**如果您觉得还不错，麻烦在文末 “点个赞” 或者 评论 “Mark”，谢谢您的支持**
