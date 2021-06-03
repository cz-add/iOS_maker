**直接上代码**

**一、Swift实现**

**1、让 标签的字体显示灰色的实现**

```
        let str = getStr(of: object)
        let attrStr = NSMutableAttributedString.init(string: str)

        for i in 0..<str.count {

            let index = str.index(str.startIndex, offsetBy: i)
            print(str[index])

            if str[index] == "|" {
                attrStr.addAttribute(NSAttributedString.Key.foregroundColor, value:RGBA_COLOR_HEX(hex: 0xDBDBDB), range:NSRange.init(location:i , length: 1))
            }

        }

        testLable.attributedText = attrStr
```

**2、把数组用" | "隔开且最多只显示一行的算法getStr(of: object)**

```
    func getStr(of product: [String]) -> String {

        guard let madeAry = product else { return "" }
        if product.count == 1 {
            return product[0] as? String ?? ""
        }

        var str = ""
        var str_limit = ""
        for (index, item) in madeAry.enumerated() {
            if let a = item as? String {
                var l = ""
                if index == madeAry.count - 1 {
                    l = ""
                } else {
                    l = " | "
                }
                str_limit = str
                str = str + a + l
                let nsStr: NSString = str as NSString
                let item_width = nsStr.width(for: UIFont.nn_font(ofSize: 12))
                if item_width > ScreenWidth - 121 {
                    var end_str = l
                    if index == madeAry.count - 1,index > 0  {
                        end_str = " | "
                    }
                    str = String(str_limit.prefix(str_limit.count - end_str.count))
                    if str.count == 0 {
                        str = product[0] as? String ?? ""
                    }
                    break
                }
            }
        }

        return str
    }
```

**3、实现效果**

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/153638_2482d7f2_9027123.png "小1.png")

**二、OC实现**

**1、让 标签的字体显示灰色的实现**

```
        NSString * str = [self getStr:object];
        NSMutableAttributedString * attrStr = [[NSMutableAttributedString alloc] initWithString:str];
        for (int i = 0; i < str.length; i++) {
            NSString * temp = [str substringWithRange:NSMakeRange(i, 1)];
            if ([temp isEqualToString:@"|"]) {
                [attrStr addAttribute:NSForegroundColorAttributeName value: RGBA_COLOR_HEX(0xDBDBDB,1) range:NSMakeRange(i, 1)];
            }
        }

        self.testLable.text =  nil;
        self.testLable.attributedText = attrStr;
```

**2、把数组用" | "隔开且最多只显示一行的算法getStr**

```
- (NSString *)getMadeinStr:(NSArray *)object {
    if (object..count == 0){
        return @"";
    } else if (object.count == 1) {
        return object[0];
    }

    NSString * str = @"";
    NSString * str_limit = @"";
    for (int i = 0; i < object.count; i++) {
        NSString * l = @"";
        NSString * a = object[i];
        if (i == object.count - 1) {
            l = @"";
        } else {
            l = @" | ";
        }
        CGFloat width = (kScreenWidth-31)/2 - 20 ;
        str_limit = str;
        str = [NSString stringWithFormat:@"%@%@%@",str,a,l];

        CGFloat str_width = [str widthForFont: [UIFont systemFontOfSize:12]];

        CGFloat item_width =  str_width;
        if (item_width > width){

            NSString * endStr = l;
            if ((i == object.count - 1) && i > 0 ) {
                endStr = @" | ";
            }

            str = [str_limit substringWithRange:NSMakeRange(0, str_limit.length - endStr.length)];
            if (str.length == 0) {
                str = object[0];
            }

            break;
        }
    }

    return str;

}

//NSString计算宽度的分类
- (CGFloat)widthForFont:(UIFont *)font {
    CGSize size = [self sizeForFont:font size:CGSizeMake(HUGE, HUGE) mode:NSLineBreakByWordWrapping];
    return size.width;
}

- (CGSize)sizeForFont:(UIFont *)font size:(CGSize)size mode:(NSLineBreakMode)lineBreakMode {
    CGSize result;
    if (!font) font = [UIFont systemFontOfSize:12];
    if ([self respondsToSelector:@selector(boundingRectWithSize:options:attributes:context:)]) {
        NSMutableDictionary *attr = [NSMutableDictionary new];
        attr[NSFontAttributeName] = font;
        if (lineBreakMode != NSLineBreakByWordWrapping) {
            NSMutableParagraphStyle *paragraphStyle = [NSMutableParagraphStyle new];
            paragraphStyle.lineBreakMode = lineBreakMode;
            attr[NSParagraphStyleAttributeName] = paragraphStyle;
        }
        CGRect rect = [self boundingRectWithSize:size
                                         options:NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading
                                      attributes:attr context:nil];
        result = rect.size;
    } else {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        result = [self sizeWithFont:font constrainedToSize:size lineBreakMode:lineBreakMode];
#pragma clang diagnostic pop
    }
    return result;
}

```



