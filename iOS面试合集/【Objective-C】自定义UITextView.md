代码支持：
 1、长按textView弹出换行操作；
　　2、自定义文字间距；
　　3、为textView添加placeholder文字；
　　直接贴代码：
　　1、.m文件
```
#import "TextView.h"

@interface TextView ()

{
    NSInteger b_index;
}
@end

@implementation TextView

CGFloat const UI_PLACEHOLDER_TEXT_CHANGED_ANIMATION_DURATION = 0.25;

- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
#if __has_feature(objc_arc)
#else
    [_placeHolderLabel release]; _placeHolderLabel = nil;
    [_placeholderColor release]; _placeholderColor = nil;
    [_placeholder release]; _placeholder = nil;
    [super dealloc];
#endif
}

- (void)awakeFromNib
{
    [super awakeFromNib];
    if (!self.placeholder) {
        _placeholder = @"";
    }
    
    if (!self.placeholderColor) {
        [self setPlaceholderColor:[UIColor lightGrayColor]];
    }
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textChanged:) name:UITextViewTextDidChangeNotification object:nil];
}

- (id)initWithFrame:(CGRect)frame
{
    if( (self = [super initWithFrame:frame]) )
    {
        _placeholder = @"";
        self.tintColor = [UIColor colorWithHexString:@"#B1B5BA"];
        self.returnKeyType = UIReturnKeyDone;
        [self setPlaceholderColor:[UIColor lightGrayColor]];
        
         //设置菜单

        UIMenuItem *menuItem = [[UIMenuItem alloc]initWithTitle:@"换行" action:@selector(selfMenu:)];
        UIMenuController *menuController = [UIMenuController sharedMenuController];
        [menuController setMenuItems:[NSArray arrayWithObject:menuItem]];
        [menuController setMenuVisible:NO];

        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textChanged:) name:UITextViewTextDidChangeNotification object:nil];
    }
    return self;
}

-(BOOL)canPerformAction:(SEL)action withSender:(id)sender{
    if (action == @selector(selfMenu:)) {
        return YES;

    }else if(action ==@selector(copy:) || action ==@selector(selectAll:) || action ==@selector(cut:) || action ==@selector(select:)){
        BOOL isAppear = [super canPerformAction:action withSender:sender];
        return isAppear;
    }
    return NO;

}

-(void)selfMenu:(id)sender{
    self.text = [self.text stringByAppendingString:@"\n"];
}

- (void)textChanged:(NSNotification *)notification
{
    if([[self placeholder] length] == 0)
    {
        return;
    }
    
    [UIView animateWithDuration:UI_PLACEHOLDER_TEXT_CHANGED_ANIMATION_DURATION animations:^{
        if([[self text] length] == 0)
        {
            [[self viewWithTag:999] setAlpha:1];
        }
        else
        {
            [[self viewWithTag:999] setAlpha:0];
        }
    }];
    
}

- (void)setText:(NSString *)text {
    [super setText:text];
    [self textChanged:nil];
}

- (void)drawRect:(CGRect)rect
{
    if( [[self placeholder] length] > 0 )
    {
        UIEdgeInsets insets = self.textContainerInset;
        if (_placeHolderLabel == nil )
        {
            
            _placeHolderLabel = [[UILabel alloc] initWithFrame:CGRectMake(insets.left+10,5,self.bounds.size.width - (insets.left +insets.right+20),1.0)];//insets.top
            _placeHolderLabel.lineBreakMode = NSLineBreakByWordWrapping;
            _placeHolderLabel.font = self.font;
            _placeHolderLabel.backgroundColor = [UIColor clearColor];
            _placeHolderLabel.textColor = self.placeholderColor;
            _placeHolderLabel.alpha = 0;
            _placeHolderLabel.tag = 999;
            [self addSubview:_placeHolderLabel];
        }
        _placeHolderLabel.text = self.placeholder;
        [_placeHolderLabel sizeToFit];//insets.top
        [_placeHolderLabel setFrame:CGRectMake(insets.left+10,5,self.bounds.size.width - (insets.left +insets.right),CGRectGetHeight(_placeHolderLabel.frame))];
        [self sendSubviewToBack:_placeHolderLabel];
    }
    
    if( [[self text] length] == 0 && [[self placeholder] length] > 0 )
    {
        [[self viewWithTag:999] setAlpha:1];
    }
    [super drawRect:rect];
}
- (void)setPlaceholder:(NSString *)placeholder{
    if (_placeholder != placeholder) {
        _placeholder = placeholder;
        [self setNeedsDisplay];
    }
    
}


-(void)textViewlineSpacing:(TextView *)textView{
    
    if (textView.text.length < 1) {
        textView.text = @"间距";
    }
    NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
    paragraphStyle.lineSpacing = 4;// 字体的行间距
    NSDictionary *attributes = @{
                                 
                                 NSFontAttributeName:[UIFont systemFontOfSize:13],
                                 
                                 NSParagraphStyleAttributeName:paragraphStyle
                                 
                                 };
    
    textView.attributedText = [[NSAttributedString alloc] initWithString:self.text attributes:attributes];
    textView.textColor = kTextColor_Black;
    if ([textView.text isEqualToString:@"间距"]) {
        textView.attributedText = [[NSAttributedString alloc] initWithString:@"" attributes:attributes];
    }
}

@end
```
2、.h文件
```
#import <UIKit/UIKit.h>

@interface TextView : UITextView

@property (nonatomic, strong) NSString *placeholder;
@property (nonatomic, strong) UIColor *placeholderColor;

@property (nonatomic, strong) UILabel *placeHolderLabel;


-(void)textChanged:(NSNotification*)notification;

-(void)textViewlineSpacing:(TextView *)textView;

@end
```
### 

分享：

工作之余，想进阶技术的同时找到志同道合的[**RD**，**欢迎点击RD**。](https://jq.qq.com/?_wv=1027&k=DaAyV952)

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/151016_9d41772b_9027123.jpeg "24396273-edf435cd4d7f39cf.jpg")

