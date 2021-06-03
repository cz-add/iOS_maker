![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155301_0738ceff_9027123.jpeg "24396273-390e25407174b567.jpg")

```
//
//  ViewController.m
//  004-NSThread状态
//
//  Created by mac on 2018/4/27.
//  Copyright © 2018年 mac. All rights reserved.
//

#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {

    [self threadDemo];
//杀主线程
NSLog(@"走！！");
//注意：exit 方法没法杀掉主线程！但是会让主线程阻塞！！
//    [NSThread exit];
//    [[NSThread currentThread] start];
//    exitDemo
//    [self exitDemo];

}

-(void)exitDemo{
    [self performSelectorInBackground:@selector(starMainThread) withObject:nil];
    //注意！！！exit会让杀掉主线程！但是APP不会挂掉
    [NSThread exit];
}

-(void)starMainThread{
    [NSThread sleepForTimeInterval:1.0];
    //开启主线程
    [[NSThread mainThread] start];
}

-(void)threadDemo{
    //创建线程
    NSThread * t =[[NSThread alloc]initWithTarget:self selector:@selector(threadStatus) object:nil];
    //线程就绪（CPU翻牌）
    [t start];
}
-(void)threadStatus{
    for (int i = 0; i<20; i++) {
        //阻塞,当运行满足某个条件，会让线程“睡一会”
        //提示：sleep 方法是类方法，会直接休眠当前线程！！
        if(i == 8){
            NSLog(@"睡一会,2秒");
            [NSThread sleepForTimeInterval:2.0];
        }
        NSLog(@"%@  %d",[NSThread currentThread],i);

        //当线程满足某一个条件时，可以强行终止的
        //exit 类方法，终止当前线程！！
        if(i ==15){
            //一旦强行终止线程，后续的所有代码都不会被执行
            //注意：在终止线程之前，应该要释放之前分配的对象
            [NSThread exit];
        }
    }
    NSLog(@"能来吗？？答案：来不了");
}
@end
```
