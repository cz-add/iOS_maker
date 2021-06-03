![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/160124_94f8e0e6_9027123.png "MJRefresh1.png")

MJRefresh几乎是我们开发工作中必用的一款三方库，它提供一套非常简单实用的拖拽执行回调事件的解决方案。下面是官方提供的框架图。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/160134_ca32040b_9027123.png "MJRefresh2.png")

其中最常用的几个默认视图类分别是：

```
下拉刷新控件：MJRefreshNormalHeader
上拉加载控件：MJRefreshAutoNormalFooter、MJRefreshBackNormalFooter
左滑加载控件：MJRefreshNormalTrailer
```

下面将对这些类，自上而下地进行分析。

## 公共基类控件

### MJRefreshComponent

通过框架图可以看出所有视图都源于同一个基类—— `MJRefreshComponent` ，它为子类提供了公用的属性和事件，主要有：

*   回调对象和回调方法
*   拖拽状态定义和控制
*   通过KVO，对事件（控件偏移、内容尺寸、手势状态）添加监听（回调响应交给子类实现）
*   其他：
    *   拖拽百分比
    *   根据拖拽比例自动切换透明度

`MJRefreshComponent` 还为子类搭建了基本的逻辑框架：

#### 视图创建

```
// 1.初始化
- (instancetype)initWithFrame:(CGRect)frame{;}

// 2.准备工作
- (void)prepare{;}

// 3.视图即将被父视图加入
- (void)willMoveToSuperview:(UIView *)newSuperview{
    // 滚动视图初始值的记录
    // 一些值的更新
    // 监听事件的更新
}

// 4.布局
- (void)layoutSubviews{
    [self placeSubviews];
}
```

### 滚动视图状态回调

```
// 当偏移值发生变化
- (void)scrollViewContentOffsetDidChange:(NSDictionary *)change{}
// 当内容大小发生变化
- (void)scrollViewContentSizeDidChange:(NSDictionary *)change{}
// 当点击手势状态发生变化
- (void)scrollViewPanStateDidChange:(NSDictionary *)change{}
```

### 状态设置

```
// 状态设置
- (void)setState:(MJRefreshState)state{;}
```

### 常用方法

```
// 进入刷新状态
- (void)beginRefreshing{;}
// 结束刷新状态
- (void)endRefreshing{;}
```

### 其他

```
// 自动切换透明度
- (void)setAutoChangeAlpha:(BOOL)autoChangeAlpha{;}
- (BOOL)isAutoChangeAlpha{;}
- (void)setAutomaticallyChangeAlpha:(BOOL)automaticallyChangeAlpha{;}

// 根据拖拽进度实时设置透明度
- (void)setPullingPercent:(CGFloat)pullingPercent{;}
```

## 下拉刷新控件（Header）

下拉刷新控件包含四个类：

*   MJRefreshHeader
    *   MJRefreshStateHeader
        *   MJRefreshNormalHeader
    *   MJRefreshGifHeader

### MJRefreshHeader

`MJRefreshHeader` 类是一个包含了完整的下拉刷新功能逻辑的空白视图，子类 `MJRefreshStateHeader` 和 `MJRefreshGifHeader` 只需要再添加一些额外的图片和文字，就能提升使用体验和保持代码的简洁易读性。

#### 实现过程

#### 1.初始化

创建视图，设置高度和位置。

```
- (void)prepare {
    [super prepare];
    // 设置存储key
    // 设置Header的高度
}

- (void)placeSubviews {
    [super placeSubviews];    
    // 设置Header的位置（y坐标）
}
```

2.偏移变化： `- scrollViewContentSizeDidChange`

当用户拖拽滚动控件，是其偏移值发生改变时，会回调 `- scrollViewContentOffsetDidChange:(NSDictionary *)change` 方法，在不同的状态下执行对应的逻辑。如果滚动视图已经将Header滚动至屏幕外，则不处理后续逻辑。

`- scrollViewContentOffsetDidChange:(NSDictionary *)change` 方法中，有一些关键的变量值，分别是:

*   当前滚动的偏移值：offsetY
*   头部控件刚好出现的偏移值：happenOffsetY
*   即将刷新的临界点：normal2pullingOffsetY

通过对这些变量值的比较，可以计算出拖拽动作应该被设置为何种状态。

*   控件正在被拖拽
    *   当拖拽时的偏移量大于临界值，且原状态为闲置时，将状态置为即将刷新
    *   当拖拽时的偏移量小于临界值，且原状态为即将刷新时，将状态重置会闲置
*   控件未被拖拽，且当前状态为松手进行刷新
    *   执行开始刷新的方法
*   控件未被拖拽，且为达到执行刷新回调的临界点

```
if (self.scrollView.isDragging) { // 如果正在拖拽
    self.pullingPercent = pullingPercent;

    // 当拖拽时的偏移量大于临界值，且原状态为闲置时，将状态置为即将刷新
    if (self.state == MJRefreshStateIdle && offsetY < normal2pullingOffsetY) {
        // 转为即将刷新状态
        self.state = MJRefreshStatePulling;
    }
    // 当拖拽时的偏移量小于临界值，且原状态为即将刷新时，将状态重置会闲置
    else if (self.state == MJRefreshStatePulling && offsetY >= normal2pullingOffsetY) {
        // 转为普通状态
        self.state = MJRefreshStateIdle;
    }
}
// 原状态为即将刷新，且手已松开
else if (self.state == MJRefreshStatePulling) {
    // 开始刷新
    [self beginRefreshing];
}
// 未达到刷新的偏移量，且手已松开
else if (pullingPercent < 1) {
    // 记录header露出的百分比
    self.pullingPercent = pullingPercent;
}
```

这里需要注意的是，当Header的状态处于 `MJRefreshStateRefreshing` 正在刷新，且控件还在滚动时，会执行 `- resetInset` 方法，目的是记录刷新结束后需要调整的上边距值 `insetTDelta` ，同时避免 CollectionView 在使用根据 Autolayout 和 内容自动伸缩 Cell, 刷新时导致的 Layout 异常渲染问题。

#### 3.状态设置

```- (void)setState:(MJRefreshState)state{
    _state = state;

    // 加入主队列的目的是等setState:方法调用完毕、设置完文字后再去布局子控件
    MJRefreshDispatchAsyncOnMainQueue([self setNeedsLayout];)
}
```

视图刷新被加入了异步队列的主线程中，是为了尽量等空间的属性设置完毕后再进行布局的刷新。

#### 4.开始刷新

执行 `- beginRefreshing` 方法，设置状态为 `MJRefreshStateRefreshing` 刷新中。 方法调用流程如下：

```1.开始刷新方法调用
- (void)beginRefreshing{
   // ...
   self.state = MJRefreshStateRefreshing;
   // ...
}

2.设置状态为正在刷新中
- (void)setState:(MJRefreshState)state{
    MJRefreshCheckState

    // 根据状态做事情
    if (state == MJRefreshStateIdle) {
        //... 
    } else if (state == MJRefreshStateRefreshing) {
        [self headerRefreshingAction];
    }
}
3.执行刷新动作
- (void)headerRefreshingAction {
    // 主要代码
    [UIView animateWithDuration:MJRefreshFastAnimationDuration animations:^{
        if (self.scrollView.panGestureRecognizer.state != UIGestureRecognizerStateCancelled) {
            CGFloat top = self.scrollViewOriginalInset.top + self.mj_h;
            // 增加滚动区域top
            self.scrollView.mj_insetT = top;
            // 设置滚动位置
            CGPoint offset = self.scrollView.contentOffset;
            offset.y = -top;
            [self.scrollView setContentOffset:offset animated:NO];
        }
    } completion:^(BOOL finished) {
        [self executeRefreshingCallback];
    }];
}
```

`- headerRefreshingAction` 方法为滚动视图设置了新的 `inset` 和 `offset` ，使得Header能在滚动视图的顶部停留，用于展示刷新文字动画之类的。

#### 5.结束刷新

结束刷新需要使用者在耗时操作结束后，主动调用 `- endRefreshing` 方法。 方法调用流程如下：

```
1.结束刷新方法调用
- (void)endRefreshing{
    MJRefreshDispatchAsyncOnMainQueue(self.state = MJRefreshStateIdle;)
}
2.设置状态为闲置
- (void)setState:(MJRefreshState)state{
    MJRefreshCheckState

    // 根据状态做事情
    if (state == MJRefreshStateIdle) {
        if (oldState != MJRefreshStateRefreshing) return;

        [self headerEndingAction];
    } else if (state == MJRefreshStateRefreshing) {
        // ...
    }
}
3.执行结束动作
- (void)headerEndingAction {;}
```

`- headerEndingAction` 方法将滚动视图的 `inset` 重置为刷新状态前的值，将header又隐藏了起来

## 上拉加载控件（Footer）

下拉刷新控件包含七个类：

*   MJRefreshFooter
    *   MJRefreshBackFooter
        *   MJRefreshBackNormalFooter
        *   MJRefreshBackGifFooter
    *   MJRefreshAutoFooter
        *   MJRefreshAutoNormalFooter
        *   MJRefreshAutoGifFooter

### MJRefreshFooter

`MJRefreshFooter` 类不能直接被使用，它仅定义了少量的基础属性和方法，例如构造方法、初始化控件高度，以及无数据加载情况下的处理。

能够直接使用的上拉加载控件是，由 `MJRefreshFooter` 衍生出的两个子类， `MJRefreshBackFooter` 和 `MJRefreshAutoFooter` ，这两个控件的不同之处在于：

```
MJRefreshBackFooter
MJRefreshAutoFooter
```



### MJRefreshBackFooter

#### 实现过程

#### 1.初始化

当MJRefreshBackFooter即将被加入父视图时，会走 `- willMoveToSuperview:` 方法，并在方法体内调用 `- scrollViewContentSizeDidChange:` 方法。该方法获取了父视图高度和父视图内容的高度，取二者中较大的数，作为Footer的纵坐标值，确保Footer的位置正好隐藏在视图或内容的最底部。

```- (void)scrollViewContentSizeDidChange:(NSDictionary *)change
{
    [super scrollViewContentSizeDidChange:change];

    // 内容的高度
    CGFloat contentHeight = self.scrollView.mj_contentH + self.ignoredScrollViewContentInsetBottom;
    // 表格的高度
    CGFloat scrollHeight = self.scrollView.mj_h - self.scrollViewOriginalInset.top - self.scrollViewOriginalInset.bottom + self.ignoredScrollViewContentInsetBottom;
    // 设置位置和尺寸
    self.mj_y = MAX(contentHeight, scrollHeight);
}
```

2.偏移变化： `- scrollViewContentSizeDidChange`

当用户在滑动控件使offset发生变化时，会触发 `MJRefreshKeyPathContentOffset` 的监听事件 —— `- scrollViewContentOffsetDidChange` 。 `MJRefreshBackFooter` 在 `- scrollViewContentOffsetDidChange` 方法里的代码逻辑与 `MJRefreshHeader` 是几乎相同的，这里不赘述。需要提一点的是， `MJRefreshBackFooter` 视图的临界值计算需要考虑内容的高度与滚动视图之间的高度差问题。

```
#pragma mark 获得scrollView的内容 超出 view 的高度
- (CGFloat)heightForContentBreakView
{
    CGFloat h = self.scrollView.frame.size.height - self.scrollViewOriginalInset.bottom - self.scrollViewOriginalInset.top;
    return self.scrollView.contentSize.height - h;
}

#pragma mark 刚好看到上拉刷新控件时的contentOffset.y
- (CGFloat)happenOffsetY
{
    CGFloat deltaH = [self heightForContentBreakView];
    // 内容和视图的高度差
    if (deltaH > 0) {
        //  内容高度 > 视图高度
        return deltaH - self.scrollViewOriginalInset.top;
    } else {
        // 内容高度 < 视图高度
        return - self.scrollViewOriginalInset.top;
    }
}
```

#### 3.状态设置

`MJRefreshBackFooter` 类的 `- setState` 方法的主要工作就是在开始刷新和结束刷新的时候，为滚动视图更新对应的 `offset` 和 `inset` 值。

### MJRefreshAutoFooter

#### 实现过程

#### 1.初始化

当MJRefreshAutoFooter即将被加入父视图时，会调用 `- willMoveToSuperview:` 方法，该方法获取了父视图的内容，作为Footer的纵坐标y的值，确保Footer的位置正好紧贴内容的底部。

```
- (void)willMoveToSuperview:(UIView *)newSuperview
{
    [super willMoveToSuperview:newSuperview];

    if (newSuperview) { // 新的父控件
        if (self.hidden == NO) {
            self.scrollView.mj_insetB += self.mj_h;
        }

        // 设置位置
        self.mj_y = _scrollView.mj_contentH;
    } else { // 被移除了
        if (self.hidden == NO) {
            self.scrollView.mj_insetB -= self.mj_h;
        }
    }
}
```

#### 2.刷新逻辑

`MJRefreshAutoFooter` 控件的位置是紧贴内容的，所以会存在两种情况： 1.当 **内容高度 < 控件高度** ，可以直接看到紧贴内容底部的Footer。这种情况下，加载的时机是在用户松手后调用的。

```
- (void)scrollViewPanStateDidChange:(NSDictionary *)change
{
    [super scrollViewPanStateDidChange:change];

    if (self.state != MJRefreshStateIdle) return;

    UIGestureRecognizerState panState = _scrollView.panGestureRecognizer.state;

    switch (panState) {
        // 手松开
        case UIGestureRecognizerStateEnded: {
            if (_scrollView.mj_insetT + _scrollView.mj_contentH <= _scrollView.mj_h) {
                // 内容 < 控件高度
                if (_scrollView.mj_offsetY >= - _scrollView.mj_insetT) { // 向上拽
                    self.triggerByDrag = YES;
                    [self beginRefreshing];
                }
            } else {
                // 内容 > 控件高度
                if (_scrollView.mj_offsetY >= _scrollView.mj_contentH + _scrollView.mj_insetB - _scrollView.mj_h) {
                    self.triggerByDrag = YES;
                    [self beginRefreshing];
                }
            }
        }
            break;

        case UIGestureRecognizerStateBegan: {
            [self resetTriggerTimes];
        }
            break;

        default:
            break;
    }
}
```

2.当 **内容高度 ≥ 控件高度** ，需要拖动视图到Footer的加载临界值，但此时不需要松开手，只要滚动视图的偏移量突破了临界值，就会触发加载方法。

```
- (void)scrollViewContentOffsetDidChange:(NSDictionary *)change
{
    [super scrollViewContentOffsetDidChange:change];

    if (self.state != MJRefreshStateIdle || !self.automaticallyRefresh || self.mj_y == 0) return;

    // 当autoTriggerTimes被设置成-1（滚动时无限加载）
    // 该方法保证拖动放手后，视图还在滚动的情况下，一直保持加载状态

    // 内容超出控件高度
    if (_scrollView.mj_insetT + _scrollView.mj_contentH > _scrollView.mj_h) {

        //  内容高度 - 控件高度 + 控件底部边距 + footer高度 * 百分比 - footer高度
        //  内容高度 - 控件高度 + 控件底部边距
        if (_scrollView.mj_offsetY >= _scrollView.mj_contentH - _scrollView.mj_h + self.mj_h * self.triggerAutomaticallyRefreshPercent + _scrollView.mj_insetB - self.mj_h) {
            // 防止手松开时连续调用
            CGPoint old = [change[@"old"] CGPointValue];
            CGPoint new = [change[@"new"] CGPointValue];
            if (new.y <= old.y) return;

            if (_scrollView.isDragging) {
                self.triggerByDrag = YES;
            }
            // 当底部刷新控件完全出现时，才刷新
            [self beginRefreshing];
        }
    }
}
```

从代码中可以看出，满足刷新的条件是： `拖动偏移量 ≥ 内容高度 - 控件高度 + 控件底部边距 + footer高度 * 刷新控件露出百分比 - footer高度` 当刷新控件露出百分比为默认值1.时，不等式可以简化为： `拖动偏移量 ≥ 内容高度 - 控件高度 + 控件底部边距`

#### 3.无限触发

`MJRefreshAutoFooter` 的另一特点就是无限触发，开发者可以设置属性自定义自动刷新的次数。

```
/** 自动触发次数, 默认为 1, 仅在拖拽 ScrollView 时才生效,

 如果为 -1, 则为无限触发
 */
@property (nonatomic) NSInteger autoTriggerTimes;
```

当滚动视图在持续地滚动时（内容高度≥控件高度），会不停地调用 `-scrollViewContentOffsetDidChange:` 方法，满足加载条件时从而不停的调用 `-beginRefreshing` 方法。

```
- (void)beginRefreshing{
    // 新的拖拽动作 && 剩余触发次数 && 是否无限触发
    if (self.triggerByDrag && self.leftTriggerTimes <= 0 && !self.unlimitedTrigger) {
        return;
    }

    [super beginRefreshing];
}
```

当前如果支持无限触发 `autoTriggerTimes == -1` ，那么在滚动视图停止滚动前，视图到达加载临界点时都会触发加载任务。

#### 4.状态设置

`-beginRefreshing` 在触发的时候，会将 `state` 设置成 `MJRefreshStateRefreshing` ，并执行加载数据的回调。 通常我们会在加载数据结束的回调方法中去调用 `-endRefreshing` 或 `-endRefreshingWithNoMoreData` 方法，此时 `state` 会被设置成 `MJRefreshStateIdle` 或 `MJRefreshStateNoMoreData` ，对应 `-setState:` 方法中的代码，我们可以看到无限触发次数是在此处进行了控制。

```
- (void)setState:(MJRefreshState)state{
    MJRefreshCheckState

    if (state == MJRefreshStateRefreshing) {
        // 执行加载数据回调
        [self executeRefreshingCallback];
    } else if (state == MJRefreshStateNoMoreData || state == MJRefreshStateIdle) {
        if (self.triggerByDrag) {
            if (!self.unlimitedTrigger) {
                self.leftTriggerTimes -= 1;
            }
            self.triggerByDrag = NO;
        }

        /** 结束刷新 */
        if (MJRefreshStateRefreshing == oldState) {

            // 当视图开启了分页显示，设置动画和回调
            if (self.scrollView.pagingEnabled) {
                CGPoint offset = self.scrollView.contentOffset;
                offset.y -= self.scrollView.mj_insetB;
                [UIView animateWithDuration:MJRefreshSlowAnimationDuration animations:^{
                    self.scrollView.contentOffset = offset;

                    if (self.endRefreshingAnimationBeginAction) {
                        self.endRefreshingAnimationBeginAction();
                    }
                } completion:^(BOOL finished) {
                    if (self.endRefreshingCompletionBlock) {
                        self.endRefreshingCompletionBlock();
                    }
                }];

                return;
            }

            // 结束刷新回调
            if (self.endRefreshingCompletionBlock) {
                self.endRefreshingCompletionBlock();
            }
        }
    }
}
```

## 左滑加载控件（Trailer）

`MJRefreshTrailer` 在实现逻辑上与 `MJRefreshBackFooter` 是完全一样的，只不过是将部分参数从垂直方向换成了水平方向。

## State、Normal 子类控件

State类型的控件 的主要特点是添加了不同状态的提示文字和刷新时间的显示。 Normal类型的控件 在 State类型控件 的基础上，添加了箭头图标和刷新的动画。

## Gif 子类控件

Gif类型的控件可以在拖拽和刷新时展示精美的动画来提升用户体验。 主要的两个方法是：

```
- (void)setImages:(NSArray *)images duration:(NSTimeInterval)duration forState:(MJRefreshState)state 
{ 
    if (images == nil) return; 

    self.stateImages[@(state)] = images; 
    self.stateDurations[@(state)] = @(duration); 

    /* 根据图片设置控件的高度 */ 
    UIImage *image = [images firstObject]; 
    if (image.size.height > self.mj_h) { 
        self.mj_h = image.size.height; 
    } 
}

- (void)setImages:(NSArray *)images forState:(MJRefreshState)state { 
    [self setImages:images duration:images.count * 0.1 forState:state]; 
}
```

刷新控件会根据图片的高度调整自身高度，同时会在没有自定义动画时长的情况下，根据动画的帧数自动设置完整播放一遍动画的时间。

### 拖拽动画

开发者可以通过拖拽百分比设置用户在拖拽时的动画，具体实现方式是通过计算当前拖拽的百分比在整体动画中对应的某个帧的图片来获取大致的下标。

```
// 通过拖拽百分比设置 Idle~Pulling状态之间的 对应的动画帧
- (void)setPullingPercent:(CGFloat)pullingPercent
{
    [super setPullingPercent:pullingPercent];
    NSArray *images = self.stateImages[@(MJRefreshStateIdle)];
    if (self.state != MJRefreshStateIdle || images.count == 0) return;
    // 停止动画
    [self.gifView stopAnimating];

    // 设置当前需要显示的图片
    NSUInteger index =  images.count * pullingPercent;
    if (index >= images.count) index = images.count - 1;
    self.gifView.image = images[index];
}
```

### 刷新动画

刷新动画会在视图状态处于“即将开始刷新”和“刷新中”进行，使用UIImageView的 `startAnimating` 对提前设置好的图片组逐帧播放，默认情况下是无限循环播放的。

```
- (void)setState:(MJRefreshState)state
{
    MJRefreshCheckState

    // 根据状态做事情
    if (state == MJRefreshStatePulling || state == MJRefreshStateRefreshing) {
        // 即将刷新 和 刷新中 状态的动画
        NSArray *images = self.stateImages[@(state)];
        if (images.count == 0) return;

        [self.gifView stopAnimating];
        if (images.count == 1) { // 单张图片
            self.gifView.image = [images lastObject];
        } else { // 多张图片
            self.gifView.animationImages = images;
            self.gifView.animationDuration = [self.stateDurations[@(state)] doubleValue];
            [self.gifView startAnimating];
        }
    } else if (state == MJRefreshStateIdle) {
        // 限制状态停止动画
        [self.gifView stopAnimating];
    }
}
```

## 总结

MJRefresh清晰整齐的架构为开发者提供了及其丰富的扩展性，而在通常没有定制需求的情况下，默认的控件已经十分够用了。
## 推荐👇：

如果你想一起进阶，不妨添加一下交流群[642363427](https://jq.qq.com/?_wv=1027&k=XADwQkqk)
