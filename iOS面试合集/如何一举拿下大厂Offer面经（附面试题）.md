**准备**

其实我很早就开始准备了，准确来说也不算准备，只是一直在总结iOS相关方面的知识，但是还是有大把时间可以自己学习一些感兴趣的方向。从过完年回来，我就有计划的复习和总结了一些知识。

看过的书籍，这里并不是泛泛的读一遍，而是详细理解了大多数内容，通俗一点就是可以用自己的话将相应的知识解读出 来 。《Android开发艺术探索》（这本书真心不错，我反复读了4、5遍）、《iOS群英传》（比较接近开发使用，因为做过一些应用开发，读起来还是比较简单的，读了2遍）、《剑指offer》（感觉面试中碰到的算法，70%都能找到相应的题目，保证所有的题都可以手写出来就行）。4个月精读了以上书籍，还有其他的都是简单了解，这里就不列举了，读完这些书，应该可以让你上一个层次吧（妈妈再也不用担心我面试啦…）。

刷题，主要是LeetCode（大概刷了300道题左右，每天3-6道，坚持下来，需要多复习，因为很多题过一段时间会忘记），还有看过一些牛课网。

看别人的面试经验，主要在网上，这里我列举两个比较好的。

*   1、 iOS客户端面试题集锦
*   2、 iOS阿里面试题锦集

**投递简历**

一份好的简历是非常有必要的，需要突出你的重点和闪光点，具体怎么写简历可以参考

## [iOS面试高薪，进阶 你会这些呢嘛？（持续更新中）](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/iOS%E9%9D%A2%E8%AF%95%E5%90%88%E9%9B%86/iOS%E9%9D%A2%E8%AF%95%E9%AB%98%E8%96%AA%EF%BC%8C%E8%BF%9B%E9%98%B6%20.md)

**CodeKK说简历**

有了一份好简历，接下来就是投递简历，一般是：拉钩+BOSS直聘+内推，从我这次面试机会来看，三者比例是2:2:1，如果被刷掉也不要灰心，现在大公司基本上各个部门都有自己的hr，可以在拉头和BOSS上多投递一些，万一其他部门看中你呢？

面试经历

这里我仅仅记录一些问过的题目（能记住的），答案我就不写出来，基本上都可以在网上找到相应的答案。

**一面**

1、iOS一些优化方案

2、最常用的版本控制工具是什么，能大概讲讲原理么

3、UNIX常用命令

4、c语言在iOS开发中的重要性

5、源代码管理工具的作用

**二面**

二面面试官是Eva？反正应该不是做iOS的，iOS的相关知识问的也不多，大多是项目上的东西。

atomic的多线程安全

聊项目，都具体做了什么。

nonatomic在自己管理内存的环境

**三面**

应该是Eva吧，主要了解一些个人的情况，以及一些项目，最后问了期望的薪资，然后当场就给了offer。

**快手**

**一面**

问了关于数据库的一些问题，SQLite的相关操作，没办法，我在华为唯一一个做的和iOS相关的项目，但是不太擅长数据库。

网络相关的问题，网络的五层模型，又问了TCP和TIP，还有iOS相关的长连接，这里问的比较深。

开始iOS相关的知识，视觉控制器的生命周期（view的生命周期）内存告急的处理（手动释放不可见视图的内存和成员变量）

第一面这就算过关了等待二面。

**二面**

问了项目相关的问题，这部分根据自己的项目经验，由于大家的经验都不同，这里我就不详细说了。

设定一个场景，怎么去实现相应的功能，因为快手这个部门想做社交，因此这里是问我是如何实现微信的联系人页面（包括与服务端有什么样的交互）

最后也是一个算法，写出所有数组的子序列

二面面试官是这个组的Eva，跟我讲了现在这个组的发展情况和快手现在的情况，由于快手成长很快，所以不能仅仅依靠一个APP，还需要在其他方面进行一些尝试，而这个组的任务就是在一些方面做一些尝试，大概就是这个样子。

**三面**

HR上来很亲切，问了我一些面试的情况，难不难之类的，然后又聊了聊我大学和研究生情况，我只想说我“too simple , too naive “，大概了解我后，只跟我聊我的不足，以此来压低我期望的薪水。说了一下薪资期望，加了微信，让我回去等待，说发offer大概是2周时间，因为需要走各种审批流程，让我不要着急。

快手是一个很年轻的公司，技术还是需要一定的积累，希望不要像小咖秀一样昙花一现。

**美团外卖**

**一面**

1、简历上写的项目问了一遍，然后开始问知识点。

2、volley的源代码，在图片缓存部分讨论了挺长时间，http中缓存机制，

3、视觉控制器的生命周期

4、数据库

5、多线程（NSTread、NSOPeration、GCDA+block）

6、http协议get post的区别

7、手机适配一些方案

8、真机调试、项目上线注意事项

9、静态方法是否能被重写

这些大概聊了1个半小时，开始的时候还有些紧张，慢慢聊开了，就好多了，面试官的语速有点快，老是需要面试官重复一遍，我也不经意间语速也变快了，不过能看出来面试官还是很厉害的。

**二面**

2次握手和3次挥手的原因，以及为什么需要这样做。

1、id和nill代表什么（nill和NULL的区别）

2、向一个nill对象发送消息会发生什么？

3、进程与线程区别

5、写一个NSString类的实现

6、http中的同步和异步

聊了一些项目上做的东西,问了问职业规划

由于二面面试官不是做iOS，本来面试我的人临时开会去了，所以这一轮面试没怎么问iOS相关知识，不过二面面试官一直是微笑，所以这一轮很轻松，更像是一起讨论问题。

面试完已经是下午4:30了，由于面试当天是星期五，而周五美团的会议比较多，所以等了会，二面面试官说三面面试官在开会，面试另约时间，我还是说这次一次面试完吧，这一等就等了2个半小时，期间hr跟我说三面面试官是个大牛。

**三面**

我认为iOS做的优秀的几个地方，然后又根据我说的问了问比较深入问题。

1、iOS是如何进行资源管理的。

2、Python比较重要的几个特性

3、网络五层结构，每一层协议，由于我网络不是很好，还问了一些其他的问题（例如MAC地址和ip地址的区别等）。

为什么离开原来公司，以及职业规划，然后因为面试完大概就晚上8点了，就先让我回去，下周让HR跟我联系，我想这是应该通过面试了吧。

美团技术还是很厉害的，从面试官的水平就可以看出来，尤其是外卖核心部门，办公环境是不错，但是感觉就是有点乱，不知道是不是因为今天面试的人很多，基本上一直有很多人来回走动，有一些嘈杂。

**阿里**

梦寐以求的阿里终于找我来面试了，没办法谁让马云爸爸太厉害，我投递的是杭州的天猫，是做虚拟现实的小组（刚听到这个名字感觉和自己不太相符），这是我到面试完后，才知道的，面试官也跟我说iOS上的需求可能不会很多，更多的是AR技术在iOS上的应用，包含OpenGL等技术。

**一面**

询问了我博客上写的一些东西，从项目立意谈起，到设计，再到详细的技术实现，可谓是面面俱到，由于自己写的博客还是比较熟悉，回答的还不错。

1、GLSurfaceView的相关知识，OpenGL，Shader，绘制流程。

2、询问当前做的项目，以及到具体的实现和优化。

3、多进程间的通讯，Binder机制。

4、询问看过哪些框架源码，EventBus，Volley讲了一下。

大概聊了一个小时左右，聊得还可以，基本上都回答上了，中间给了我很多建议，不懂的地方，也会仔细跟我讲解一番，其实有一半的时间都是跟我聊产品，为什么这个产品好，怎样做才能迎合市场，然后怎么设计整个产品等，感觉跟我现在水平不是一个层次的，果然，第二天就给我发了一封邮件，说我现在暂时不太合适投递的岗位。

**面试结果**

除了阿里淘宝外，其他的公司基本都拿到offer。

**最后总结**

自己对于互联网有一些小小的见解：随着资本的冷却，整个互联网市场也逐渐的冷静下来。iOS应用开发从一开始能说几个四大组件的名词，能随便写个监听事件，就能拿到高达上万的月薪的时代了。

归根到底并不是工作难找了，而是iOS应用开发工程师这个职位已经趋于正常，再也不是没什么技术也能拿高工资的香饽饽。当然这个也不是绝对的，对于中高级的开发人员来说，市场还是比较缺少的，尤其是知名企业对于招聘员工来说，不仅要求有过硬的技术，还要求有高素质，好的教育背景等等。

总的来说，高工资可以给你，但是前提条件是你要足够优秀，或者说让面试官觉得你很优秀。

**笔试**

对于社招的同学来说，基本上不需要笔试，但是也有公司是需要的，例如 今日头条和网易都有笔试。笔试都是比较基础的一些知识，Python、iOS等方面的，一般不会有网络，计算机等方面的笔试，一般情况下大家都能答出来。

**一、二面**

近一段时间的面试经历来说，一、二面的问题没有什么很大的区别（公司基本上都有3面技术面，但是也有例外，我在美团就是2面技术面。），基本上都是一线开发人员。主要考察你是否有牢固的基础知识和是否在平常开发中能熟练使用。

是否能讲解清楚你所做的项目，以及使用到的相关知识。

1、iOS基础知识

2、Python基础知识，大概是多线程，线程安全，集合类，JVM，类相关知识等。

3、iOS一些源码的阅读

4、优秀的第三方框架源码阅读

**三、四面**

一般公司都是三轮技术面，但是也有四轮技术面的，不过不多。很多公司基本上每一轮面试官都会记录他所询问的问题，以便给下一轮面试官作参考，还有就是避免对同一个知识点多次询问。所以到了这轮面试，基本上不会再询问比较基础的知识。

**会从两个方面考察，**

*   1、广度：比较新的技术（多线程，插件化等），http协议，数据库，iOS（一般不会询问之前面试官问过的问题）。
*   2、深度：一般会通过1或2个问题来考察，例如：项目中的贡献，所做的优化。设计能力，基本上不多，这个要看面试的岗位，因为我这里面试的只是高级开发，并不是架构。

工作中的亮点和突出。

**HR面**

基本上到了这轮，你就算通过面试了。hr会询问一些你的经历，最主要的还是和你商定薪资待遇。在这轮，大家应该要对自己的薪资水平有一个大体的了解，一般都是在原来的工资基础上增长20%~30%的样子，当然，如果你在面试过程中表现非常优秀，也可以不受这个限制。当然如果公司诚心要你，就算你要的工资过高，hr也会委婉的告诉你，不会直接把你pass。

> 作为一个开发者，有一个学习的氛围跟一个交流圈子特别重要这是一个我的iOS交流群：642363427，不管你是小白还是大牛欢迎入驻 ，分享BAT,阿里面试题、面试经验，讨论技术， 大家一起交流学习成长！

文章来源于网络，如有侵权，请联系小编删除。


