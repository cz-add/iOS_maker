## 一、Swift泛型

　　泛型能够让开发者编写自定义需求已经任意类型的灵活可用的的函数和类型。能够让我们避免重复的代码。用一种清晰和抽象的方式来表达代码的意图。
```
func swapTwoStrings(_ a: inout String, _ b: inout String) {    let temporaryA = a    a = b    b = temporaryA}  func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {    let temporaryA = a    a = b    b = temporaryA}
```
　　从以上代码来看，它们功能代码是相同的，只是类型上不一样，这时我们可以使用泛型，从而避免重复编写代码。

　　泛型使用了占位类型名（在这里用字母 T 来表示）来代替实际类型名（例如 Int、String 或 Double）。

```
func swapTwoValues<T>(_ a: inout T, _ b: inout T)
```

　　**备注：这个函数功能是用来交换两个同样类型的值，但是这个函数用T占位符来代替实际的类型。并没有指定具体的类型，但是传入的a,b必须是同一个类型T。在调用这个函数d的时候才能指定T是哪种具体的类型。还有函数名后面跟的那个<T>是函数定义的一个占位符类型名，并不会查找T的具体类型。使用该函数只要保证传入的两个参数是同一个类型，就不用根据传入参数的类型不同，而写不同的方法。**

　　看泛型在代码用，是如何使用的。

```
override func viewDidLoad() {        super.viewDidLoad()         //swift泛型        var num1 = 100        var num2 = 200        print("交换前数据:  \(num1) 和 \(num2)")        swapTwoValue(&num1, &num2)        print("交换后数据:  \(num1) 和 \(num2)")                 var str1 = "ABC"        var str2 = "abc"        print("交换前数据:  \(str1) 和 \(str2)")        swapTwoValue(&str1, &str2)        print("交换后数据:  \(str1) 和 \(str2)")    }         func swapTwoValue<T>(_ a : inout T,_ b : inout T) {        let temp = a        a = b        b = temp    }
```

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/152553_066878dc_9027123.png "Swift1.png")


## 二、类型约束

　　泛型约束大致分为以下几种：

**　　（1）继承约束，泛型类型必须是某个类的子类型**

**　　（2）协议约束，泛型类型必须遵循某些协议**

**（3）条件约束，泛型类型必须满足某种条件** 

```
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {    // 这里是泛型函数的函数体部分}
```

###  　　1、继承约束

```
//定义一个父类，动物类class Animal{    //动物都会跑    func run(){        print("Animal run")    }}//定义狗类，继承动物类class Dog: Animal {    override func run(){//重写父类方法        print("Dog run")    }}//定义猫类，继承动物类class Cat: Animal {    override func run(){//重写父类方法        print("Cat run")    }}//定义泛型函数，接受一个泛型参数，要求该泛型类型必须继承Animalfunc AnimalRunPint<T:Animal>(animal:T){    animal.run() //继承了Animal类的子类都有run方法可以调用}AnimalRunPint(animal: Dog())AnimalRunPint(animal: Cat())
```
![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/152603_596d98e3_9027123.png "Swift2.png")

### 　　2、协议约束

```
//定义泛型函数，为泛型添加协议约束，泛型类型必须遵循Equatable协议func findIndex<T: Equatable>(array: [T], valueToFind: T) -> Int? {    var index = 0    for value in array {        if value == valueToFind {//因为遵循了Equatable协议，所以可以进行相等比较            return index        } else {            index += 1        }    }    return nil}//在浮点型数组中进行查找，Double默认遵循了Equatable协议let doubleIndex = findIndex(array: [3.14159, 0.1, 0.25], valueToFind: 9.3)if let index = doubleIndex {    print("在浮点型数组中寻找到9.3，寻找索引为\(index)")} else {    print("在浮点型数组中寻找不到9.3")}//在字符串数组中进行查找，String默认遵循了Equatable协议let stringIndex = findIndex(array: ["Mike", "Malcolm", "Andrea"], valueToFind: "Andrea")if let index = stringIndex {    print("在字符串数组中寻找到Andrea，寻找索引为\(index)")} else {    print("在字符串数组中寻找不到Andrea")}
```

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/152703_a52974fc_9027123.png "Swift3.png")

### 　　3、条件约束

```
//添加泛型条件约束，C1和C2必须遵循Stackable协议，而且C1和C2包含的泛型类型要一致func pushItemOneToTwo<C1: Stackable, C2: Stackable>( stackOne: inout C1, stackTwo: inout C2)    where C1.ItemType == C2.ItemType{//因为C1和C2都遵循了Stackable协议，才有ItemType属性可以调用    let item = stackOne.pop()    stackTwo.push(item: item)}//定义另外一个结构体类型，同样实现Stackable协议，实际上里面的实现和Stack一样struct StackOther<T>: Stackable{    var store = [T]()    mutating func push(item:T){//实现协议的push方法要求        store.append(item)    }    mutating func pop() -> T {//实现协议的pop方法要求        return store.removeLast()    }}//创建StackOther结构体，泛型类型为Stringvar stackTwo = StackOther<String>()stackTwo.push(item: "where")//虽然stackOne和stackTwo类型不一样，但泛型类型一样，也同样遵循了Stackable协议pushItemOneToTwo(stackOne: &stackOne, stackTwo: &stackTwo )print("stackOne = \(stackOne.store), stackTwo = \(stackTwo.store)")//打印：stackOne = ["hello"], stackTwo = ["where", "swift"]
```

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/152727_2aa230ee_9027123.png "Swift4.png")

