```
//视图已经加载完了，可以进行ui的添加了
- (void)viewDidLoad {
    [superviewDidLoad];
  // Do any additional setup after loading the view.
  //初始化UILabel注意指定该对象的位置及大小
  UILabel *lb = [[UILabelalloc]initWithFrame:CGRectMake(0,20,300,200)];
  //设置文字
    lb.text =@"label测试我在学习中学些ui story水电费水电费未入围 i肉煨入味哦水电费水电费水电费";
  //设置背景色
    lb.backgroundColor = [UIColorcolorWithRed:0green:191.0/255.0blue:243.0/255.0alpha:1.0];
  //设置文字颜色
    lb.textColor = [UIColorwhiteColor];
  //文字大小，文字字体
    lb.font = [UIFontsystemFontOfSize:25];
  NSLog(@"系统字体名字：%@",lb.font.familyName);
  //打印文字字体列表
  NSArray *arrFonts = [UIFontfamilyNames];
  NSLog(@"系统字体列表：%@",arrFonts);
  //文字对齐
    lb.textAlignment =NSTextAlignmentJustified;
//    NSTextAlignmentLeft      = 0,    //居左对齐，默认
//    NSTextAlignmentCenter    = 1,    //居中对齐
//    NSTextAlignmentRight     = 2,    //居右对齐
//    NSTextAlignmentJustified = 3,    // Fully-justified. The last line in a paragraph is natural-aligned.
//    NSTextAlignmentNatural   = 4,    // Indicates the default alignment for script

  //换行模式
    lb.lineBreakMode =NSLineBreakByCharWrapping;
//    NSLineBreakByWordWrapping = 0,    //每一行的结尾以字或者一个完整单词换行（若不够一个单词的位置）
//    NSLineBreakByCharWrapping,//在每一行的结尾以字母进行换行
//    NSLineBreakByClipping,// Simply clip
//    NSLineBreakByTruncatingHead,// Truncate at head of line: "...wxyz"
//    NSLineBreakByTruncatingTail,// Truncate at tail of line: "abcd..."
//    NSLineBreakByTruncatingMiddle// Truncate middle of line:  "ab...yz"

  //指定行数，0为不限制行树，可以指定具体的数字
    lb.numberOfLines =0;
  //加圆角
    lb.layer.cornerRadius =30;
  //此行必须加，将原来的矩形角剪掉
    lb.clipsToBounds =YES;
  //加边框颜色，宽度，注意给layer加的颜色是CGColor类型
    lb.layer.borderColor = [[UIColorredColor]CGColor];
    lb.layer.borderWidth =1.0;

  //把label添加到视图上，并且会显示
    [self.viewaddSubview:lb];
}
```

Label的首行缩进一直是个很头疼的问题，现在IOS6只有有一个 attributedText的属性值得我们深究，可以达到我们自定义的行高，还有首行缩进，各种行距和间隔问题。下面这个是两个Label, 一个是UserName，另一个是Content文本多行信息

**创建标签**

```
@interface ViewController : UIViewController
@property ( weak , nonatomic ) IBOutlet UILabel *usernameLabel
@property ( weak , nonatomic ) IBOutlet UILabel *contentLabel;
@end
```

**视图展示层**

```
- ( void )viewDidLoad {
self . usernameLabel . text  =  @"用户名Jordan CZ: " ;
self . usernameLabel . adjustsFontSizeToFitWidth  =  YES ;
[ self . usernameLabel   sizeToFit ];
      self . contentLabel . text  =  @"首行缩进根据用户昵称自动调整 间隔可自定根据需求随意改变。。。。。。。" ;
self . contentLabel . adjustsFontSizeToFitWidth  =  YES ;
self . contentLabel . adjustsLetterSpacingToFitWidth  =  YES ;
[ self   resetContent ];
}
```

**自适应计算间距**

```
- ( void )resetContent{
NSMutableAttributedString *attributedString = [[ NSMutableAttributedString alloc ]initWithString : self . contentLabel . text ];
NSMutableParagraphStyle *paragraphStyle = [[ NSMutableParagraphStyle alloc ]init ];
paragraphStyle. alignment = NSTextAlignmentLeft ;
paragraphStyle. maximumLineHeight = 60 ;  //最大的行高 
paragraphStyle. lineSpacing = 5 ;  //行自定义行高度
[paragraphStyle setFirstLineHeadIndent : self . usernameLabel . frame . size .width + 5 ]; //首行缩进 根据用户昵称宽度在加5个像素
[attributedString addAttribute : NSParagraphStyleAttributeName value:paragraphStyle range : NSMakeRange ( 0 , [ self . contentLabel . text length ])];
self . contentLabel . attributedText = attributedString;
[ self . contentLabel sizeToFit ];
}
```

## UITextView的使用详解

```
//初始化并定义大小 
 UITextView *textview = [[UITextView alloc] initWithFrame:CGRectMake(20, 10, 280, 30)];
    textview.backgroundColor=[UIColor whiteColor]; //背景色
    textview.scrollEnabled = NO;    //当文字超过视图的边框时是否允许滑动，默认为“YES”
    textview.editable = YES;        //是否允许编辑内容，默认为“YES”
    textview.delegate = self;       //设置代理方法的实现类
    textview.font=[UIFont fontWithName:@"Arial" size:18.0]; //设置字体名字和字体大小;
    textview.returnKeyType = UIReturnKeyDefault;//return键的类型
    textview.keyboardType = UIKeyboardTypeDefault;//键盘类型
    textview.textAlignment = NSTextAlignmentLeft; //文本显示的位置默认为居左
    textview.dataDetectorTypes = UIDataDetectorTypeAll; //显示数据类型的连接模式（如电话号码、网址、地址等）
    textview.textColor = [UIColor blackColor];
    textview.text = @"UITextView详解";//设置显示的文本内容
    [self.view addSubview:textview];
```

**UITextView的代理方法如下:**

```
//将要开始编辑
- (BOOL)textViewShouldBeginEditing:(UITextView *)textView;

//将要结束编辑
- (BOOL)textViewShouldEndEditing:(UITextView *)textView;

//开始编辑
- (void)textViewDidBeginEditing:(UITextView *)textView;

//结束编辑
- (void)textViewDidEndEditing:(UITextView *)textView;

//内容将要发生改变编辑
- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString*)text;

//内容发生改变编辑
- (void)textViewDidChange:(UITextView *)textView;

//焦点发生改变
- (void)textViewDidChangeSelection:(UITextView *)textView;
```

**有时候我们要控件自适应输入的文本的内容的高度，只要在textViewDidChange的代理方法中加入调整控件大小的代理即可**

```

- (void)textViewDidChange:(UITextView *)textView{
  //计算文本的高度
 CGSize constraintSize;
    constraintSize.width = textView.frame.size.width-16;
    constraintSize.height = MAXFLOAT;
 CGSize sizeFrame =[textView.text sizeWithFont:textView.font
 constrainedToSize:constraintSize
 lineBreakMode:UILineBreakModeWordWrap];

 //重新调整textView的高度
    textView.frame =CGRectMake(textView.frame.origin.x,textView.frame.origin.y,textView.frame.size.width,sizeFrame.height+5);
}

```

**控制输入文字的长度和内容，可通调用以下代理方法实现**

```
- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString*)text
{
 if (range.location>=100)
    {
  //控制输入文本的长度
 return NO;
    }
 if ([text isEqualToString:@"\n"]) {
 //禁止输入换行
 return NO;
    }
 else
    {
 return YES;
    }
}
```

**UITextView退出键盘的几种方式**
**因为iphone的软键盘没有自带的退键盘键，所以要实现退出键盘需要自己实现，有如下几种方式：**
1）如果你程序是有导航条的，可以在导航条上面加多一个Done的按钮，用来退出键盘，当然要先实UITextViewDelegate。

```
- (void)textViewDidBeginEditing:(UITextView *)textView {

  UIBarButtonItem *done =    [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemDone
 target:self
 action:@selector(dismissKeyBoard)];

  self.navigationItem.rightBarButtonItem = done;

    [done release];
    done = nil;

}

- (void)textViewDidEndEditing:(UITextView *)textView { 
  self.navigationItem.rightBarButtonItem = nil; 
}

- (void)dismissKeyBoard { 
    [self.textView resignFirstResponder]; 
}
```

2）如果你的textview里不用回车键，可以把回车键当做退出键盘的响应键。
代码如下：

```
-(BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString*)text
{
 if ([text isEqualToString:@"\n"]) {
        [textView resignFirstResponder];
 return NO;
    }
  return YES;
}
```

3）还有你也可以自定义其他加载键盘上面用来退出，比如在弹出的键盘上面加一个view来放置退出键盘的Done按钮。
代码如下：

```
 UIToolbar * topView = [[UIToolbar alloc]initWithFrame:CGRectMake(0, 0, 320,30)];
    [topView setBarStyle:UIBarStyleBlack];

  UIBarButtonItem *btnSpace = [[UIBarButtonItem alloc]initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace
 target:self
 action:nil];

  UIBarButtonItem *doneButton = [[UIBarButtonItem alloc]initWithTitle:@"Done"
 style:UIBarButtonItemStyleDone
 target:self
 action:@selector(dismissKeyBoard)];

 NSArray * buttonsArray = @[btnSpace, doneButton];;
    [doneButton release];
    [btnSpace release];
    [topView setItems:buttonsArray];
    [textView setInputAccessoryView:topView];//当文本输入框加上topView
    [topView release];
    topView = nil;

-(IBAction)dismissKeyBoard
{
    [tvTextView resignFirstResponder];
}
```
