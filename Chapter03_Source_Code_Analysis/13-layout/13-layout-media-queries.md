# 媒体查询

### 0x00 简介

前文  [**📚 Layout 拾遗**](https://juejin.cn/post/6997419992023040030)  提到了栅格布局系统中响应式布局的实现方式`媒体查询`，本文将对其深入分析， 耐心读完，相信会对您有所帮助。

### 0x01 媒体查询

**媒体查询**（**Media queries**）是实现响应式设计的最常见方法，它可以让我们根据设备的大致类型（如打印、手机、平板或电脑等）或者特定的特征和设备参数（例如屏幕分辨率和浏览器视口宽度）来调整网站或应用。 例如，缩小小型设备上的字体大小，判断手机朝向（横屏或竖屏）等。

#### CSS 语法

媒体查询语法通常使用 `@media`(`@规则`),   可用于基于一个或多个媒体查询的结果来应用样式表。\
通过指定的媒体类型与正在显示文档的设备类型匹配，并且媒体查询中的所有表达式均为 `true`，将应用样式规则。

```css
// 媒体类型   媒体特性表达式
@media media-type and (media-feature-rule) {
  /* CSS rules go here */
}
```

#### 媒体类型

**媒体类型**（_Media types_）描述设备的一般类别。可以指定的媒体类型为：

* `all` 适用于所有设备。
* `print` 适用于在打印预览模式下。
* `screen` 主要用于屏幕(计算机屏幕、平板电脑、智能手机)。
* `speech` 主要应用于屏幕阅读器等发声设备。

> 在不使用 `not` 或 `only` 操作符的情况下，媒体类型是可选的，默认为 `all` 。

#### 媒体特性

**媒体特性**（_Media features_）描述了 user agent、输出设备，或是浏览环境的具体特征。媒体特性表达式是完全可选的，它负责测试这些特性或特征是否存在、值为多少。每条媒体特性表达式都必须用括号括起来。

最常用的特性是视口宽度，可以使用`min-width`（**最常用**）、`max-width`和`width`媒体特性，在视口宽度大于或者小于某个大小——或者是恰好等于某个大小——的时候，应用 CSS 样式规则。

#### 逻辑运算符

**逻辑操作符**（_logical operators_）  使用 `not`, `and`, `only`和`,` (**逗号**)  构建复杂的媒体查询。

* `and`  运算符用来把多个媒体属性组合起来，合并到同一条媒体查询中。只有当每个属性都为真时，这条查询的结果才为真。
* `not`运算符用于否定媒体查询，如果不满足这个条件则返回 true，否则返回 false。 如果出现在以逗号分隔的查询列表中，它将仅否定应用了该查询的特定查询。 `not`关键字不能用于否定单个媒体功能表达式，而只能用于否定整个媒体查询。
* `only` 运算符仅在整个查询匹配时才用于应用样式，并且对于防止老式浏览应用所选样式很有用。
* `,` 逗号将多个媒体查询合并为一个规则。 逗号分隔列表中的每个查询都与其他查询分开处理。 因此，如果列表中的任何查询为 true，则整个 `@media` 语句均返回 true。 换句话说，列表的行为类似于逻辑或`or`运算符。

> 使用 `not`, `only` 运算符，**必须指定媒体类型**。

> 在老式浏览器中将`@media only screen and (min-width: 992px) {...}` 简单地解释为`only`，因为没有叫 `only` 的设备，所以老式浏览器不会应用样式。 当不使用`only`时,将`@media screen and (min-width: 992px) {...}`简单地解释为`screen`，忽略查询的其余部分，应用样式。

### 0x02 断点

**断点**(_Breakpoints_) 是响应式设计的基石。 它是可自定义的宽度，使用它可以控制何时可以在特定视口或设备大小下调整布局。

#### 断点设置

参照了 Bootstrap 的 响应式设计，预设了五个响应尺寸：`xs`、`sm`、`md`、`lg`  和  `xl`。

| 断点          | 类中缀  | 分辨率     | 设备            |
| ----------- | ---- | ------- | ------------- |
| X-Small     | `xs` | <767px  | 超小屏幕   手机     |
| Small       | `sm` | ≥768px  | 小屏幕   平板      |
| Medium      | `md` | ≥992px  | 中等屏幕   桌面显示器  |
| Large       | `lg` | ≥1200px | 大屏幕   大桌面显示器  |
| Extra large | `xl` | ≥1920px | 超大屏幕   大桌面显示器 |

在文件 `theme-chalk\src\common\var.scss` 定义了这些断点。

```scss
/* Break-point
--------------------------*/
$--sm: 768px !default;
$--md: 992px !default;
$--lg: 1200px !default;
$--xl: 1920px !default;

// 断点Map
$--breakpoints: (
  "xs": (
    max-width: $--sm - 1,
  ),
  "sm": (
    min-width: $--sm,
  ),
  "md": (
    min-width: $--md,
  ),
  "lg": (
    min-width: $--lg,
  ),
  "xl": (
    min-width: $--xl,
  ),
);
```

### 0x03 组件响应式实现

#### @mixin res()

在文件 `theme-chalk\src\mixins\mixins.scss` 定义了指令 `res`构建 css,使用媒体查询来创建关键的分界点阈值。

```scss
/* Break-points
 -------------------------- */
// 断点Map
$--breakpoints: (
  "xs": (
    max-width: 767px,
  ),
  "sm": (
    min-width: 768px,
  ),
  "md": (
    min-width: 992px,
  ),
  "lg": (
    min-width: 1200px,
  ),
  "xl": (
    min-width: 1920px,
  ),
);

@mixin res($key, $map: $--breakpoints) {
  // 循环断点Map，如果存在则返回
  @if map-has-key($map, $key) {
    @media only screen and #{inspect(map-get($map, $key))} {
      @content;
    }
  } @else {
    @warn "Undefeined points: `#{$map}`";
  }
}
```

指令功能逻辑如下：

1. 指令 `res` 接受两个参数 `$key`(响应尺寸) 和 `$map`（断点 Map）。
2. 指令内部使用函数 `map-has-key($map,$key)` 作用是根据 `$key` 参数，返回 `$key`在 `$map` 中对应的 value 值。如果 `$key` 不存在 `$map`中，将返回 null 值（此时生成警告断点未定义 `Undefeined points` ）。
3. 若返回 true,使用函数 `map-get($map,$key)` 返回 `$key` 在 `$map` 中对应的 value 值。
4. 使用函数 `inspect($value)` 将其转成字符串，使用插值 `#{}`、`@content` 生成 媒体查询。

```scss
// 参数 $key  值为  xs
// 获取Map
map-get($map, $key) =>  ( max-width: 767px )
// 返回string
inspect(map-get($map, $key)) =>  '( max-width: 767px )'

@media only screen and #{inspect(map-get($map, $key))} { @content; }
// 编译结果
@media only screen and (max-width: 767px) { ... }
```

在栅格系统中，媒体查询通过断点构建 CSS。响应式布局五个响应尺寸样式如下：

```css
/* 超小屏幕（手机，小于 768px） */
@media only screen and (max-width: 767px) {
  ...;
}
/* 小屏幕（平板，大于等于 768px） */
@media only screen and (min-width: 768px) {
  ...;
}
/* 中等屏幕（桌面显示器，大于等于 992px） */
@media only screen and (min-width: 992px) {
  ...;
}
/* 大屏幕（大桌面显示器，大于等于 1200px） */
@media only screen and (min-width: 1200px) {
  ...;
}
/* 超大屏幕（大桌面显示器，大于等于 1920px） */
@media only screen and (min-width: 1920px) {
  ...;
}
```

### 0x04 基于断点的隐藏实现

`Element` 额外提供了一系列类名，用于在某些条件下隐藏元素。

#### 使用方式

首先，项目引入样式文件：

```js
import "element-ui/lib/theme-chalk/display.css";
```

然后将这些类名可以添加在任何 DOM 元素或自定义组件上。

* `hidden-xs-only` - 当视口在  `xs`  尺寸时隐藏
* `hidden-sm-only` - 当视口在  `sm`  尺寸时隐藏
* `hidden-sm-and-down` - 当视口在  `sm`  及以下尺寸时隐藏
* `hidden-sm-and-up` - 当视口在  `sm`  及以上尺寸时隐藏
* `hidden-md-only` - 当视口在  `md`  尺寸时隐藏
* `hidden-md-and-down` - 当视口在  `md`  及以下尺寸时隐藏
* `hidden-md-and-up` - 当视口在  `md`  及以上尺寸时隐藏
* `hidden-lg-only` - 当视口在  `lg`  尺寸时隐藏
* `hidden-lg-and-down` - 当视口在  `lg`  及以下尺寸时隐藏
* `hidden-lg-and-up` - 当视口在  `lg`  及以上尺寸时隐藏
* `hidden-xl-only` - 当视口在  `xl`  尺寸时隐藏

#### 实现原理

文件 `theme-chalk\src\common\var.scss` 定义了隐藏类断点 Map`$--breakpoints-spec` ，使用逻辑运算符 `and`，让媒体查询能跨越多个断点宽度。

```scss
$--breakpoints-spec: (
  "xs-only": (
    max-width: $--sm - 1,
  ),
  "sm-and-up": (
    min-width: $--sm,
  ),
  "sm-only": "(min-width: #{$--sm}) and (max-width: #{$--md - 1})",
  "sm-and-down": (
    max-width: $--md - 1,
  ),
  "md-and-up": (
    min-width: $--md,
  ),
  "md-only": "(min-width: #{$--md}) and (max-width: #{$--lg - 1})",
  "md-and-down": (
    max-width: $--lg - 1,
  ),
  "lg-and-up": (
    min-width: $--lg,
  ),
  "lg-only": "(min-width: #{$--lg}) and (max-width: #{$--xl - 1})",
  "lg-and-down": (
    max-width: $--xl - 1,
  ),
  "xl-only": (
    min-width: $--xl,
  ),
);
```

转译后内容如下，定了关于视口宽度的各种媒体查询规则。

```scss
$--breakpoints-spec: (
  "xs-only": (
    max-width: 767px,
  ),
  "sm-and-up": (
    min-width: 768px,
  ),
  "sm-only": "(min-width: 768px) and (max-width: 991px)",
  "sm-and-down": (
    max-width: 991px,
  ),
  "md-and-up": (
    min-width: 992px,
  ),
  "md-only": "(min-width: 992px) and (max-width: 1199px)",
  "md-and-down": (
    max-width: 1199px,
  ),
  "lg-and-up": (
    min-width: 1200px,
  ),
  "lg-only": "(min-width: 1200px) and (max-width: 1919px)",
  "lg-and-down": (
    max-width: 1919px,
  ),
  "xl-only": (
    min-width: 1920px,
  ),
);
```

使用指令 `res` 生成带前缀 `hidden`的隐藏类。隐藏类 CSS 规则 `display: none !important;` 用于隐藏元素。

```scss
// 类前缀
.hidden {
  // 遍历 Map $--breakpoints-spec
  @each $break-point-name, $value in $--breakpoints-spec {
    // 生成.hidden-#{$break-point-name} 如 hidden-xs-only  hidden-sm-only
    &-#{$break-point-name} {
      // 生成媒体查询
      @include res($break-point-name, $--breakpoints-spec) {
        display: none !important;
      }
    }
  }
}
```

最终生成 `display.css`，内容如下：

```scss
// 当视口在 `xs` 尺寸时
@media only screen and (max-width: 767px) {
  /* .hidden-xs-only  */
}
// 当视口在 `sm` 及以上尺寸时
@media only screen and (min-width: 768px) {
  /* .hidden-sm-and-up  */
}
// 当视口在 `sm` 尺寸时
@media only screen and (min-width: 768px) and (max-width: 991px) {
  /* .hidden-sm-only  */
}
// 当视口在 `sm` 及以下尺寸时
@media only screen and (max-width: 991px) {
  /* .hidden-sm-and-down  */
}
//  当视口在 `md` 及以上尺寸时
@media only screen and (min-width: 992px) {
  /* .hidden-md-and-up  */
}
// 当视口在 `md` 尺寸时
@media only screen and (min-width: 992px) and (max-width: 1199px) {
  /* .hidden-md-only  */
}
// 当视口在 `md` 及以下尺寸时
@media only screen and (max-width: 1199px) {
  /* .hidden-md-and-down  */
}
// 当视口在 `lg` 及以上尺寸时
@media only screen and (min-width: 1200px) {
  /* .hidden-lg-and-up  */
}
// 当视口在 `lg` 尺寸时
@media only screen and (min-width: 1200px) and (max-width: 1919px) {
  /* .hidden-lg-only  */
}
// 当视口在 `lg` 及以下尺寸时
@media only screen and (max-width: 1919px) {
  /* .hidden-lg-and-down  */
}
// 当视口在 `xl` 尺寸时
@media only screen and (min-width: 1920px) {
  /* .hidden-xl-only  */
}
```

### 0x05 📚 参考

[“使用媒体查询”，MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Media\_Queries/Using\_media\_queries)\
[“Media\_queries”，MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS\_layout/Media\_queries)\
["@规则",MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/At-rule)\
[screen sizes](https://screensiz.es/)\
[“sass 在线编辑器”，sassmeister.com](https://www.sassmeister.com/)
