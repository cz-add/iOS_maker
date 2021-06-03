### 约束的优先级

AutoLayout添加的约束中也有优先级（Priority），优先级的数值1~1000，分为两种情况：

*   一种情况是我们经常添加的各种约束，默认值1000（最大值）优先执行，条件允许的话系统会自动满足我们的约束需求。
*   第二种就是固有约束（intinsic content size）严格说这种更像UILabel和UIButton的一种属性，但是在Autolayout中需要满足属性取值与约束优先级属性结合才能完成图形的绘制

当UILabel显示的内容过长或太短，控件就会被拉伸和压缩，当我们不想让控件被拉伸压缩时，就需要设置控件的固有约束（intinsic content size）来实现我们的需求。固有约束分为两种：

*   扛拉伸优先级（Content Hugging Priority）：默认251，优先级越高越不易被拉伸
*   防压缩优先级（Content Compression Resistance Prority）：默认750，优先级越高越不易被压缩

### 例子

两个Label并排放置，左边Label根据内容自适应，右Label距离左Label中间有固定距离，右Label距离右边距有固定距离，在不设置优先级的情况下，左Label要么被拉伸要么被压缩，如下图
![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/152311_4258c73b_9027123.png "先1.png")


我们设置左边Label的抗压缩优先级和抗拉伸优先级都大于右边Label，效果如图

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/152300_5d2824a0_9027123.png "先2.png")


由于左边Label的抗压缩和抗拉伸优先级都高于右边Label，而且其他约束的优先级（1000）也都高于右边label的固有宽度优先级，所以系统选择拉伸或者压缩了右边的Label，实现了我们的需求。
