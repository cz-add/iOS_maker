　Swift 被设计用来无缝兼容 Cocoa 和 Objective-C 。在 Swift 中，你可以使用 Objective-C 的 API（包括系统框架和你自定义的代码），你也可以在 Objective-C中 使用 Swift 的 API。这种兼容性使 Swift 变成了一个简单、方便并且强大的工具集成到你的 Cocoa 应用开发工作流程中。下面通过一个案例演示，实现Swift与Object-C的混合编程。

 作为一个开发者，有一个学习的氛围跟一个交流圈子特别重要，这有个iOS交流群：[642363427](https://jq.qq.com/?_wv=1027&k=dCPZiOpH)，不管你是小白还是大牛欢迎入驻 ，分享BAT,阿里面试题、面试经验，讨论技术！

步骤一：创建工程文件，名为Person。注意选择编程语言为Swift。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152018_bb90ee83_9027123.png "oc1.png")


步骤二：接下来就是要实现OC跟Swift的混合编程啦！首先创建一个Person类将他加入到工程中，语言选择为：Objective-C

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152349_ad5c6d40_9027123.png "oc2.png")


步骤三：单击Finsh按钮，会出现下图中的提示框，此处单击YES，系统会自动生成桥接文件。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152408_c3110b38_9027123.png "oc3.png")


这是可以看到，系统已经创建出一个名为Person-Bridging-Header.h文件啦！，然后选中该文件将#import "Person.h"包含进去

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152438_d3ad2c70_9027123.png "oc4.png")


这是我们拷贝下系统创建的桥接文件名，在工程中进行搜索，可以看到配置文件

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152452_71a15105_9027123.png "oc5.png")


步骤四：Person类创建好后，我们先不用去写代码，接下来再去创建一个House类，不过此类是Swift语言编写的。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152508_f01e0a2f_9027123.png "oc6.png")


在House类中，定义成员变量，初始化方法，以备Person类调用。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152523_b0849e0b_9027123.png "oc7.png")


为防止后期，连接时无法使用，此处对该文件进行编译，如下图。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/152550_abb9daca_9027123.jpeg "oc8.jpg")

步骤五：剩下来要做的工作就是编写代码啦!手写在Person类中使用前向声明调用House,然后声明几个成员变量，

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/150705_894f2052_9027123.png "oc9.png")


为之后测试做准备，在Person.m文件中去重写description方法，下图中的选中部分，是系统桥接时生成的文件。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/150717_9a9a7415_9027123.png "oc10.png")


步骤六：在控制器中使用Person和House

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/150731_283c2400_9027123.png "oc11.png")


步骤七：打印输出结果

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/150741_f791f9fe_9027123.png "oc12.png")

