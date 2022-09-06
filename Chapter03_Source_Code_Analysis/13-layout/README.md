# 3.13 Layout 布局

### 0x00 简介

组件提供了布局的栅格化(**Grid Layout**)系统，通过基础的 24 分栏，迅速简便地创建布局。该系统使用`Row`  和  `Col`  栅格组件进行创建。本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。

🔗 [组件文档 Layout](https://element.eleme.cn/#/zh-CN/component/layout) 🔗 [gitee 源码 行组件 row.js](https://gitee.com/ElemeFE/element/blob/dev/packages/row/src/row.js) 🔗 [gitee 源码 列组件 col.js](https://gitee.com/ElemeFE/element/blob/dev/packages/col/src/col.js)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 Grid Layout

栅格化提高了页面布局的一致性跟复用性，提升了整个设计开发流程的效率。同时使得网页的布局更加规范和简洁，提升用户体验。

网页栅格化神器 [Grid.Guide](http://grid.guide) ，可以自由设置最大宽度、列数以及留白边界自动动生成多种最佳栅格方案以供选择。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da1c8470289c4c2185a242393750785e\~tplv-k3u1fbpfcp-watermark.image)

**Bootstrap** 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多 12 列。

element 2 采用 **Ant Design** 的设计理念:在 12 栅格系统的基础上，将整个设计建议区域按照 24 等分的原则进行划分，解决在设计区域内大量信息收纳的问题。

***

**栅格化布局系统** 通过一系列的行（`row`）和列（`col`）来定义信息区块的外部框架，以保证页面的每个区域能够稳健地排布起来。下面就介绍一下栅格系统的工作原理：

* 通过  `row`  在水平方向建立一组  `column`（简写 col）。
* 内容应当放置于  `col`  内，并且只有  `col`  可以作为  `row`  的直接元素。
* 栅格系统中的列是指 1 到 24 的值来表示其跨越的范围。例如，三个等宽的列可以使用  `<col :span="8" />`  来创建。
* 如果一个  `row`  中的  `col`  总和超过 24，那么多余的  `col`  会作为一个整体另起一行排列。

接下来对组件行（`row`）和列（`col`）一一进行讲解。

### 0x02 Row 行组件

`packages/row/src/row.js` 使用渲染函数构建组件,支持组件渲染成自定义元素标签，主要用来作为`col`的容器。

#### render()

组件根据指定自定义元素渲染标签节点，由组件 prop 属性值动态计算添加 class 和 自定义样式（计算属性`style`），内部提供一个匿名插槽用于分发内容。

```js
render(h) {
  return h(this.tag, {
    class: [
      'el-row',
      this.justify !== 'start' ? `is-justify-${this.justify}` : '',
      this.align !== 'top' ? `is-align-${this.align}` : '',
      { 'el-row--flex': this.type === 'flex' }
    ],
    style: this.style
  }, this.$slots.default);
}
```

在 `JSX` 语法中，`h`  作为  `createElement`  的别名。第二个参数是一个包含模板相关属性的数据对象`VNodeData`,对象属性如下。

```js
{
  // 与 `v-bind:class` 的 API 相同，
  // 接受一个字符串、对象或字符串和对象组成的数组
  'class': {
    foo: true,
    bar: false
  },
  // 与 `v-bind:style` 的 API 相同，
  // 接受一个字符串、对象，或对象组成的数组
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // ...
}
```

若自定义元素标签为`<div>`，等同于如下 `template` 实现。

```html
<template>
  <div
    :style="style"
    :class="[
      'el-row',
      justify !== 'start' ? 'is-justify-' + justify : '',
      align !== 'top' ? 'is-align-' + align : '',
      { 'el-row--flex': type === 'flex'   }
    ]"
  >
    <slot></slot>
  </div>
</template>
```

#### 组件 props

组件定义了 5 个 prop : `tag` 、`gutter`、`type`、`justify`、`align`。

**tag**

支持组件渲染成自定义 html 标签,默认值为 `div`, 作为 `createElement` 方法的第一个参数。

```js
props: {
  tag: {
    type: String,
    default: 'div'
  },
},
```

**gutter**

栅格间隔设置,用来指定每一栏之间的间隔，默认间隔为 0。`col` 组件通过获取父组件`row`的 `gutter` 计算自己的左右 padding 。

```js
props: {
  gutter: Number,
},
```

**Flex 布局设置**

`type` 设置布局模式，可选 flex，仅在现代浏览器下有效。

```js
props: {
  type: String,
},
```

当 `type` 值为`flex`， `{ 'el-row--flex': type === 'flex' }` 会添加 class `el-row--flex`,开启 flex 布局。

```css
.el-row--flex {
  display: flex;
}
```

`justify` 用于设置 flex 布局下的水平排列方式，可选值`start/end/center/space-around/space-between`，生成的样式 `is-justify-[justify]`,用于设置 `justify-content` 属性。 其他值生成的样式无效。

`align` 用于设置 flex 布局下的垂直排列方式，可选值`top/middle/bottom`，生成的样式 `is-align-[align]`,用于设置 `align-items` 属性。 其他值生成的样式无效。

```js
props: {
  // flex 布局下的水平排列方式  justify-content
  justify: {
    type: String,
    default: 'start'
  },
  // flex 布局下的垂直排列方式  align-items
  align: {
    type: String,
    default: 'top'
  }
},
```

元素默认布局不会生成 flex 样式 `justify !== 'start' ? 'is-justify-' + justify : '',` ，`align !== 'top' ? 'is-align-' + align : '',` 。

系统基于 `Flex` 布局，允许子元素在父节点内的水平对齐方式 - 居左、居中、居右、等宽排列、分散排列。子元素与子元素之间，支持顶部对齐、垂直居中对齐、底部对齐的方式。

> Flex 布局，可以简便、完整、响应式地实现各种页面布局。其语法本文不做过多赘述，详见 [“Flex 布局教程”，阮一峰](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)。

#### 计算属性

计算属性 `style` 通过为  `row`  组件设置负值  `margin`  从而抵消掉为  `col`  组件设置的  `padding`，也就间接为“行（row）”所包含的“列（column）”抵消掉了`padding`。

```js
computed: {
  style() {
    const ret = {};
    // 通过gutter计算出实际左右 margin
    if (this.gutter) {
      ret.marginLeft = `-${this.gutter / 2}px`;
      ret.marginRight = ret.marginLeft;
    }
    return ret;
  }
},
```

### 0x03 Col 列组件

`packages/col/src/col.js` 使用渲染函数构建组件，支持渲染自定义元素标签。

#### render()

组件根据指定自定义元素渲染标签节点，由组件 prop 属性值动态计算添加 class 和 自定义样式，内部提供一个匿名插槽用于分发内容。

```js
render(h) {
    let classList = [];
    let style = {};

    // sytle 计算
    // class 计算

    return h(this.tag, {
      class: ['el-col', classList],
      style
    }, this.$slots.default);
  }
```

若自定义元素标签为`<div>`，等同于如下 `template` 实现。

```html
<template>
  <div :style="style" :class="['el-col', classList]">
    <slot></slot>
  </div>
</template>
```

#### 组件 props

定义了 10 个 prop ,具体功能详见中文注释。 `span` 默认值 24，对应栅格系统 24 分栏。

```js
props: {
  // 自定义元素标签
  tag: {
    type: String,
    default: 'div'
  },
  // 栅格占据的列数，总共24列，如果设置为0，则不渲染
  span: {
    type: Number,
    default: 24
  },
  // 栅格左侧的间隔格数
  offset: Number,
  // 栅格向右移动格数
  pull: Number,
  // 栅格向左移动格数
  push: Number,
  // 响应式布局设置
  // 响应式栅格数或者栅格属性对象 number/object (例如： {span: 4, offset: 4})
  // xs <768px  sm ≥768px  md ≥992px  lg ≥1200px  xl ≥1920px
  xs: [Number, Object],
  sm: [Number, Object],
  md: [Number, Object],
  lg: [Number, Object],
  xl: [Number, Object]
},
```

#### 计算属性

计算属性`gutter` 获取父组件 `row` 的 `gutter`值 。通过 `row` 组件的自定义 property 进行判断`parent.$options.componentName !== 'ElRow'`。

```js
computed: {
  gutter() {
    // 获取父实例 根据 compontName 属性 判断是组件 row
    let parent = this.$parent;
    while (parent && parent.$options.componentName !== 'ElRow') {
      parent = parent.$parent;
    }
    return parent ? parent.gutter : 0;
  }
},

// packages\row\src\row.js
export default {
  name: 'ElRow',
  // 自定义 property
  componentName: 'ElRow',
}
```

#### 组件 padding

若计算属性`gutter`值不为 0，计算 `col` 的左右 padding。

```js
render(h) {
    let style = {};
    // sytle 计算
    if (this.gutter) {
      style.paddingLeft = this.gutter / 2 + 'px';
      style.paddingRight = style.paddingLeft;
    }
  }
```

#### 组件 class

**分栏、间隔、左右偏移**

栅格、间隔、左右偏移的样式动态计算。

```js
// span 栅格占据的列数，通过 width 来实现
// offset 栅格左侧的间隔格数，通过 margin-left 实现
// push 栅格向右移动格数，通过 left 实现
// pull 栅格向左移动格数，通过 right 实现
["span", "offset", "pull", "push"].forEach((prop) => {
  if (this[prop] || this[prop] === 0) {
    classList.push(
      prop !== "span" ? `el-col-${prop}-${this[prop]}` : `el-col-${this[prop]}`
    );
  }
});
```

`col`组件样式 scss 实现。 由`.el-col-0`可知，当 span 设置为 0 时,组件 `display` 值为`none` ，不会被渲染。

```scss
[class*="el-col-"] {
  float: left;
  // 如何计算一个元素的总宽度和总高度
  box-sizing: border-box;
}
// 组件不渲染
.el-col-0 {
  display: none;
}

@for $i from 0 through 24 {
  // 生成 .el-col-0,.el-col-1, ... ,el-col-24
  .el-col-#{$i} {
    width: (1 / 24 * $i * 100) * 1%;
  }
  // 生成 .el-col-offset-0,.el-col-offset-1, ... ,el-col-offset-24
  .el-col-offset-#{$i} {
    margin-left: (1 / 24 * $i * 100) * 1%;
  }
  // 生成 .el-col-pull-0,.el-col-pull-1, ... ,el-col-pull-24
  .el-col-pull-#{$i} {
    position: relative;
    right: (1 / 24 * $i * 100) * 1%;
  }
  // 生成 .el-col-push-0,.el-col-push-1, ... ,el-col-push-24
  .el-col-push-#{$i} {
    position: relative;
    left: (1 / 24 * $i * 100) * 1%;
  }
}
```

**响应式布局**

响应式布局样式动态计算。预设四个响应尺寸：`xs` `sm` `md` `lg`。\
传入数字的话只会影响 `span`，还可以传入对象`{span: 4, offset: 4}` ,属性可选范围 `span/offset/pull/push`。

```js
// xs <768px 响应式栅格数或者栅格属性对象
// sm ≥768px 响应式栅格数或者栅格属性对象
// md ≥992px 响应式栅格数或者栅格属性对象
// lg ≥1200px 响应式栅格数或者栅格属性对象
// xl ≥1920px 响应式栅格数或者栅格属性对象
["xs", "sm", "md", "lg", "xl"].forEach((size) => {
  if (typeof this[size] === "number") {
    classList.push(`el-col-${size}-${this[size]}`);
  } else if (typeof this[size] === "object") {
    let props = this[size];
    Object.keys(props).forEach((prop) => {
      classList.push(
        prop !== "span"
          ? `el-col-${size}-${prop}-${props[prop]}`
          : `el-col-${size}-${props[prop]}`
      );
    });
  }
});
```

使用指令 `res` 生成 `@media` 媒体查询样式，其 scss 实现如下：

```scss
// 'xs', 'sm', 'md', 'lg', 'xl'
// 生成 @media only screen and (min-width: xxx px)
@include res(sm) {
  // 生成  .el-col-sm-0
  .el-col-sm-0 {
    display: none;
  }
  @for $i from 0 through 24 {
    // 生成 .el-col-sm-0,.el-col-sm-1, ... ,el-col-sm-24
    .el-col-sm-#{$i} {
      width: (1 / 24 * $i * 100) * 1%;
    }
    // 生成 .el-col-sm-offset-0,.el-col-sm-offset-1, ... ,el-col-sm-offset-24
    .el-col-sm-offset-#{$i} {
      margin-left: (1 / 24 * $i * 100) * 1%;
    }
    // 生成 .el-col-sm-pull-0,.el-col-sm-pull-1, ... ,el-col-sm-pull-24
    .el-col-sm-pull-#{$i} {
      position: relative;
      right: (1 / 24 * $i * 100) * 1%;
    }
    // 生成 .el-col-sm-push-0,.el-col-sm-push-1, ... ,el-col-sm-push-24
    .el-col-sm-push-#{$i} {
      position: relative;
      left: (1 / 24 * $i * 100) * 1%;
    }
  }
}
```

### 0x04 Column Gutter 实现原理

通过 `row` 和 `col` 组件，并通过 col 组件的  `span`  属性可以自由地组合布局。使用  `<el-col :span="12" />`  来创建二个等宽的列。 使用 `row` 组件提供  `gutter`  属性来指定每一栏之间的间隔，默认单位为 `px` 。

以下代码创建一个包含二个等宽的列的行，间隔为 24px。

```html
<el-row :gutter="24">
  <el-col :span="12"><div>col</div></el-col>
  <el-col :span="12"><div>col</div></el-col>
</el-row>
```

页面渲染后效果如下 👇：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e40d3d3205c4677bb48236198f8293a\~tplv-k3u1fbpfcp-watermark.image)

列与列的间隔距离等于属性值 `gutter`，首列左侧 和 尾列的右侧间隔值为 `gutter/2`。布局如下 👇：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24001c0d9b114d739c16ca80b1d188f6\~tplv-k3u1fbpfcp-watermark.image)

组件`col`中的计算属性 `gutter`获取父组件 `row` 的 `gutter` 值，并在 `render()` 中基于计算属性 `gutter`的值计算列的 `padding-left` `padding-right`,值为 `gutter / 2 + 'px'`。

```js
// packages\col\src\col.js
computed: {
  // 获取 el-row 的gutter值
  gutter() {
    // 父实例 根据 compontName 属性 判断是组件 el-row
    let parent = this.$parent;
    while (parent && parent.$options.componentName !== 'ElRow') {
      parent = parent.$parent;
    }
    return parent ? parent.gutter : 0;
  }
},
render(h) {
  let classList = [];
  let style = {};

  // 通过gutter计算自己的左右2个padding，用于分隔col
  if (this.gutter) {
    style.paddingLeft = this.gutter / 2 + 'px';
    style.paddingRight = style.paddingLeft;
  }

  // class 计算

  return h(this.tag, {
    class: ['el-col', classList],
    style
  }, this.$slots.default);
}
```

列与列的间隔距离等于属性值 `gutter` 等于 左列的 `padding-right`值 加上 右列的 `padding-left`值；首列左侧间隔值为 `padding-left` ； 尾列的右侧间隔值为 `padding-right`。效果如下 👇：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd2bc420fec242c08cacb0ed2508e2cb\~tplv-k3u1fbpfcp-watermark.image)

其它 3 等列、4 等列、……、24 等列依次类推，感觉是不是很好理解！\
**那组件`row`的计算属性 `style` 设置 `margin-left`  `margin-right`负值用意何在**？

```js
// packages\row\src\row.js
computed: {
  style() {
    const ret = {};
    if (this.gutter) {
      ret.marginLeft = `-${this.gutter / 2}px`;
      ret.marginRight = ret.marginLeft;
    }
    return ret;
  }
},
```

> 前文中提到，组件 `row` 的计算属性  `style`  通过为组件设置负值  `margin`  从而抵消掉为  `col`  组件设置的  `padding`，也就间接为“行（row）”所包含的“列（column）”抵消掉了`padding`。

接下来通过示例、图解的方式对其进行阐释。

在之前的示例中第一行第一列中插入一行（该行中包含两等宽列）,代码如下 👇：

```html
<!-- row1 -->
<el-row :gutter="24">
  <!-- col1 -->
  <el-col :span="12">
    <el-row :gutter="24">
      <el-col :span="12"><div>col</div></el-col>
      <el-col :span="12"><div>col</div></el-col>
    </el-row>
  </el-col>
  <!-- col2 -->
  <el-col :span="12"><div>col</div></el-col>
</el-row>
```

假设组件 `row` 没有设置负值， `margin`值为 0 。\
此时第一行第一列中嵌套的两列的间隔比默认的设置多出来一个 `padding` 值（`gutter/2`）。中间两列的间隔就 `gutter/2 * 3`。多嵌套一层，间隔就会增大 `gutter/2`。实现效果如下 👇：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98377a10af10458a8781cea4f87069a5\~tplv-k3u1fbpfcp-watermark.image)

若组件 `row` 的计算属性  `style`  通过为组件设置负值  `margin`，绝对值为 `gutter/2`。嵌套中的 `row`宽度会增加`gutter`,抵消掉为  `col`  组件设置的  `padding`，相当于此列没有设置 `padding`值。实现效果如下 👇：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa64562133914047b3a41f3bdde3be9a\~tplv-k3u1fbpfcp-watermark.image)

基于此逻辑，不管进行多少层级的嵌套，都能保证列与列之间的间隔一致。代码实际渲染效果如下 👇：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f225da3d43f41d581ed940890c4349d\~tplv-k3u1fbpfcp-watermark.image)

### 0x05 样式实现

#### Row 组件

组件样式源码 `packages\theme-chalk\src\row.scss` 使用混合指令 `b`、`m`、`utils-clearfix`嵌套生成组件样式。

```scss
// 生成 .el-row
@include b(row) {
  position: relative;
  box-sizing: border-box;
  // 使用清除浮动指令 生成 .el-row::after, .el-row::before
  @include utils-clearfix;

  // flex布局 生成 .el-row--flex
  @include m(flex) {
    display: flex;
    // 生成 .el-row--flex:after, .el-row--flex:before
    &:before,
    &:after {
      display: none;
    }
    // 对齐方式
    // 生成 .el-row--flex.is-justify-center
    @include when(justify-center) {
      justify-content: center;
    }
    // 生成 // 生成 .el-row--flex.is-justify-end
    @include when(justify-end) {
      justify-content: flex-end;
    }
    // 生成 .el-row--flex.is-justify-space-between
    @include when(justify-space-between) {
      justify-content: space-between;
    }
    // 生成 .el-row--flex.is-justify-space-around
    @include when(justify-space-around) {
      justify-content: space-around;
    }
    // 生成 .el-row--flex.is-align-middle
    @include when(align-middle) {
      align-items: center;
    }
    // 生成 .el-row--flex.is-align-bottom
    @include when(align-bottom) {
      align-items: flex-end;
    }
  }
}
```

使用  `gulpfile.js`编译  `scss`  文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成  `packages\theme-chalk\lib\row.css`，内容格式如下。

```css
.el-row {
  /*...*/
}
/*...clearfix...*/
.el-row::after,
.el-row::before {
  /*...*/
}
.el-row::after {
  /*...*/
}
/*...flex...*/
.el-row--flex {
  /*...*/
}
.el-row--flex:after,
.el-row--flex:before {
  /*...*/
}
/*...justify-content...*/
.el-row--flex.is-justify-center {
  /*...*/
}
.el-row--flex.is-justify-end {
  /*...*/
}
.el-row--flex.is-justify-space-between {
  /*...*/
}
.el-row--flex.is-justify-space-around {
  /*...*/
}
/*...align-items...*/
.el-row--flex.is-align-middle {
  /*...*/
}
.el-row--flex.is-align-bottom {
  /*...*/
}
```

***

#### Col 组件

组件样式源码 `packages\theme-chalk\src\col.scss` 生成组件样式实现分栏、间隔、偏移、响应式功能。

**分栏、间隔、偏移**

使用`@for`循环生成 `0` 至 `24` 对应样式：

* `span` 栅格占据的列数，通过 width 来实现 ,对应样式 `.el-col-[n]` 。
* `offset` 栅格左侧的间隔格数，通过 margin-left 实现 ,对应样式 `.el-col-offset-[n]` 。
* `push` 栅格向右移动格数，通过 left 实现 ,对应样式 `.el-col-push-[n]` 。
* `pull` 栅格向左移动格数，通过 right 实现,对应样式 `.el-col-pull-[n]` 。

组件栅格系统使用 24 分栏，所有每分栏宽度基准为 `(1 / 24 * 100) * 1%`。

```js
[class*="el-col-"] {
  float: left;
  // 如何计算一个元素的总宽度和总高度
  box-sizing: border-box;
}
// 组件不渲染
.el-col-0 {
  display: none;
}

@for $i from 0 through 24 {
  // 生成 .el-col-[0-24]
  .el-col-#{$i} {
    width: (1 / 24 * $i * 100) * 1%;
  }
  // 生成 .el-col-offset-[0-24]
  .el-col-offset-#{$i} {
    margin-left: (1 / 24 * $i * 100) * 1%;
  }
  // 生成 .el-col-pull-[0-24]
  .el-col-pull-#{$i} {
    position: relative;
    right: (1 / 24 * $i * 100) * 1%;
  }
  // 生成 .el-col-push-[0-24]
  .el-col-push-#{$i} {
    position: relative;
    left: (1 / 24 * $i * 100) * 1%;
  }
}
```

**响应式布局**

系统预设五个响应尺寸：`xs` `sm` `md` `lg` `xl`。使用指令 `res` 生成媒体查询，从而实现响应式设计。

```js
// 'xs', 'sm', 'md', 'lg', 'xl'
// 生成  @media only screen and (max-width: 767px) { ... }
@include res(xs) {
  // 生成  .el-col-xs-0
  .el-col-xs-0 {
    display: none;
  }
  @for $i from 0 through 24 {
    // 生成 .el-col-xs-[0-24]
    .el-col-xs-#{$i} {
      width: (1 / 24 * $i * 100) * 1%;
    }
    // 生成 .el-col-xs-offset-[0-24]
    .el-col-xs-offset-#{$i} {
      margin-left: (1 / 24 * $i * 100) * 1%;
    }
    // 生成 .el-col-xs-pull-[0-24]
    .el-col-xs-pull-#{$i} {
      position: relative;
      right: (1 / 24 * $i * 100) * 1%;
    }
    // 生成 .el-col-xs-push-[0-24]
    .el-col-xs-push-#{$i} {
      position: relative;
      left: (1 / 24 * $i * 100) * 1%;
    }
  }
}

// 生成 @media only screen and (min-width: 768px) { ... }
@include res(sm) { /*...*/ }
// 生成 @media only screen and (min-width: 992px) { ... }
@include res(md) { /*...*/ }
// 生成 @media only screen and (min-width: 1200px) { ... }
@include res(lg) { /*...*/ }
// 生成 @media only screen and (min-width: 1920px) { ... }
@include res(xl) { /*...*/ }
```

### 0x06 📚 参考

[“渲染函数 & JSX”，vuejs.org](https://cn.vuejs.org/v2/guide/render-function.html)\
["媒体查询",MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS\_layout/Media\_queries)\
[“@media”，MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media)
