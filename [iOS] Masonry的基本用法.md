### 介绍
Masonry 是一个轻量级的布局框架，拥有自己的描述语法 采用更优雅的链式语法封装自动布局，简洁明了 并具有高可读性；

简单的用法示例：

```
[view1 mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.equalTo(superview).with.insets(padding);
}];
```

### 属性
支持的属性：

```
@property (nonatomic, strong, readonly) MASConstraint *left;
@property (nonatomic, strong, readonly) MASConstraint *top;
@property (nonatomic, strong, readonly) MASConstraint *right;
@property (nonatomic, strong, readonly) MASConstraint *bottom;
@property (nonatomic, strong, readonly) MASConstraint *leading;
@property (nonatomic, strong, readonly) MASConstraint *trailing;
@property (nonatomic, strong, readonly) MASConstraint *width;
@property (nonatomic, strong, readonly) MASConstraint *height;
@property (nonatomic, strong, readonly) MASConstraint *centerX;
@property (nonatomic, strong, readonly) MASConstraint *centerY;
@property (nonatomic, strong, readonly) MASConstraint *baseline;

```
与NSLayoutAttrubute对照表如下：

| Masonry       | NSAutoLayout              |           说明 |
|:------------- |:-------------------------:| -------------:|
| left          | NSLayoutAttributeLeft     |           左侧 |
| top           | NSLayoutAttributeTop      |           上侧 |
| right         | NSLayoutAttributeRight    |           右侧 |
| bottom        | NSLayoutAttributeBottom   |           下侧 |
| leading       | NSLayoutAttributeLeading  |  正常情况下等同于 left |
| trailing      | NSLayoutAttributeTrailing |  正常情况下等同于 right |
| width         | NSLayoutAttributeWidth    |            宽 |
| height        | NSLayoutAttributeHeight   |            高 |
| centerX       | NSLayoutAttributeCenterX  |       横向中点 |
| centerY       | NSLayoutAttributeCenterY  |       纵向中点 |
| baseline      | NSLayoutAttributeBaseline |       对齐基线 |
| leftMargin      | NSLayoutAttributeLeftMargin |       左边的 Margin|
| rightMargin      | NSLayoutAttributeRightMargin |     右边的 Margin |
| topMargin      | NSLayoutAttributeTopMargin |       顶部的 Margin |
| bottomMargin      | NSLayoutAttributeBottomMargin |       底部的 Margin |
| leadingMargin      | NSLayoutAttributeLeadingMargin |       前导（基本等于left）Margin |
| trailingMargin      | NSLayoutAttributeTrailingMargin |       后尾（基本等于tail）Margin |
| centerXWithinMargins      | NSLayoutAttributeCenterXWithinMargins |       中心X坐标Margin |
| centerYWithinMargins      | NSLayoutAttributeCenterYWithinMargins |       中心Y坐标Margin |

### 约束
上面没个属性都是一个 Block，其返回值都是`MASConstraint *`,而每个 MASConstraint 都有一些约束动作，其也是一个 Block 并且返回值仍是一个`MASConstraint *`。

#### 设置偏移量

| 调用方式 | 参数 | 效果 |
|:-- |:--:|----:|
| insets(MASEdgeInsets insets)| MASEdgeInsets |设置 view 的四边缩小大小，等于缩放效果 |
| sizeOffset(CGSize offset)| CGSize | frame 的 size 相当于参考量的偏移大小 |
| centerOffset(CGPoint offset)| CGPoint | center 相对于参考量的偏移大小 |
| offset(CGFloat offset)| CGFloat |所指属性相对于参考量的偏移大小 |

MASEdgeInsets 是 UIEdgeInsets 的 typedef:

```
typedef struct UIEdgeInsets {
    CGFloat top, left, bottom, right;  
} UIEdgeInsets;

```

在 Masonry 里面，offset 只是做了“加法”运算

![](http://note.youdao.com/yws/public/resource/bb93f1a1fb98c7fb1ec45283db978a11/4F2ACF8587504D66A54B35E6944D3BB5)

#### 设置优先级

|调用方式|参数|效果|
|:-- |:--:|----:|
|priority|MASLayoutPriority|其实就是 float 的 UILayoutPriority，设置属性的优先级|
|priorityLow|无| 等于 priority(MASLayoutPriorityDefaultLow)|
|priorityMedium|无|等于 priority(MASLayoutPriorityDefaultMedium)|
|priorityHigh|无|等于 priority(MASLayoutPriorityDefaultHigh)|

当约束出现冲突时，就需要设置优先级来解决

```
[label1 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.width.and.height.equalTo(@100);
        
        make.left.greaterThanOrEqualTo(content.mas_left);
        make.right.lessThanOrEqualTo(content.mas_right);
        make.top.greaterThanOrEqualTo(content.mas_top);
        make.bottom.lessThanOrEqualTo(content.mas_bottom);
        
        _leftConstraint = make.centerX.equalTo(content.mas_left).with.offset(50).priorityHigh(); // 优先级要比边界条件低
        _topConstraint = make.centerY.equalTo(content.mas_top).with.offset(50).priorityHigh(); // 优先级要比边界条件低
        
    }];
```

#### 关系计算

|调用方式|参数|效果|
|:-- |:--:|----:|
|equalTo(id attr)|	CGFloat|	设置属性等于某个数值|
|greaterThanOrEqualTo((id attr))	|CGFloat|	设置属性大于或等于某个数值|
|lessThanOrEqualTo(id attr)	|CGFloat	|设置属性小于或等于某个数值|
|multipliedBy(CGFloat multiplier)	|CGFloat|	设置属性乘以因子后的值|
|dividedBy(CGFloat divider)|	CGFloat	|设置属性除以因子后的值|

#### Content 优先级
Content Compression Resistance 在父 view 空间不够的情况下，Priority 越高内容越不会被挤压

```
//设置 content compression 为1000
[label2 setContentCompressionResistancePriority:UILayoutPriorityRequired                                    forAxis:UILayoutConstraintAxisHorizontal];
```
Content Hugging 在父 view 空间足够的情况下，Priority 越高 view 随着内容的变化而变化，优先级低的 view 不随内容变化而变化

```
//设置 content hugging 为750
[label2 setContentHuggingPriority:UILayoutPriorityDefaultHigh
                               forAxis:UILayoutConstraintAxisHorizontal];
```

#### 自定义 baseline
UIButton 默认中心对齐排列,自定义 view 的 baseline 在底部，可以通过重写viewForBaselineLayout 方法，来设置中心线；

```
// 返回自定义的baseline的view
- (UIView *)viewForBaselineLayout {
    return _imageView;
}

```



###参考

[Masonry](https://github.com/SnapKit/Masonry)