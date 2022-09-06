# 3.21 Progress 进度条

### 简介

组件 `Progress` 用于展示操作的当前进度，告知用户当前状态和预期。本文将深入分析组件源码，剖析其实现原理，耐心读完，相信会对您有所帮助。

组件源码文件 `packages/progress/src/progress.vue`。 🔗[gitee repo](https://gitee.com/ElemeFE/element/blob/dev/packages/progress/src/progress.vue) 🔗[组件文档 Progress](https://element.eleme.cn/#/zh-CN/component/progress)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 源码实现

#### Props

组件`prop`声明如下：

```js
<script>
  export default {
    name: 'ElProgress',
    props: {
      type: {
        type: String,
        default: 'line',
        validator: val => ['line', 'circle', 'dashboard'].indexOf(val) > -1
      },
      percentage: {
        type: Number,
        default: 0,
        required: true,
        validator: val => val >= 0 && val <= 100
      },
      status: {
        type: String,
        validator: val => ['success', 'exception', 'warning'].indexOf(val) > -1
      },
      strokeWidth: {
        type: Number,
        default: 6
      },
      strokeLinecap: {
        type: String,
        default: 'round'
      },
      textInside: {
        type: Boolean,
        default: false
      },
      width: {
        type: Number,
        default: 126
      },
      showText: {
        type: Boolean,
        default: true
      },
      color: {
        type: [String, Array, Function],
        default: ''
      },
      format: Function
    },
  };
</script>
```

各属性功能说明。

| 参数             | 说明                                          | 类型                    | 可选值                       | 默认值   |
| -------------- | ------------------------------------------- | --------------------- | ------------------------- | ----- |
| percentage     | 百分比（必填）                                     | number                | 0-100                     | 0     |
| type           | 进度条类型                                       | string                | line/circle/dashboard     | line  |
| stroke-width   | 进度条的高度，单位 px                                | number                | —                         | 6     |
| text-inside    | 进度条显示文字内置在进度条内（只在 type=line 时可用）            | boolean               | —                         | false |
| status         | 进度条当前状态                                     | string                | success/exception/warning | —     |
| color          | 进度条背景色（会覆盖 status 状态颜色）                     | string/function/array | —                         | ''    |
| width          | 环形进度条画布宽度（只在 type 为 circle 或 dashboard 时可用） | number                |                           | 126   |
| show-text      | 是否显示进度条文字内容                                 | boolean               | —                         | true  |
| stroke-linecap | circle/dashboard 类型路径两端的形状                  | string                | butt/round/square         | round |
| format         | 指定进度条文字内容                                   | function(percentage)  | —                         | —     |

`type` 、`percentage`、 `status` 供了自定义验证函数，验证属性值是否符合可选值或者可选范围。 当验证失败的时候，(开发环境构建版本的) Vue 将会产生一个控制台的警告。

#### Template

```js
<template>
  <div
    class="el-progress"
    :class="[
      'el-progress--' + type,
      status ? 'is-' + status : '',
      {
        'el-progress--without-text': !showText,
        'el-progress--text-inside': textInside,
      }
    ]"
    role="progressbar"
    :aria-valuenow="percentage"
    aria-valuemin="0"
    aria-valuemax="100"
  >
    <div class="el-progress-bar" v-if="type === 'line'">
       // ...
    </div>
    <div class="el-progress-circle" v-else>
      // ...
    </div>
    <div class="el-progress__text" v-if="showText && !textInside">
        // ...
    </div>
  </div>
</template>
```

组件根节点一个类名`el-progress`的 `<div>`元素 ，根据组件设定的属性值动态绑定组件样式：

* 进度条类型 `el-progress--[line/circle/dashboard]`
* 进度条状态 `is-[success/exception/warning]`
* 不显示进度条文字 `el-progress--without-text`
* 显示文字内置在进度条内 `el-progress--text-inside`

属性 `role` 、`aria-valuemin` 、`aria-valuenow`、`aria-valuemax` 实现 ARIA 无障碍网页应用， 更多内容详见 ["ARIA",MDN](https://developer.mozilla.org/zh-CN/docs/Web/Accessibility/ARIA)

根节点下包含 3 个子节点：

* 类名`el-progress-bar`的 `<div>`元素用于渲染线形进度条功能。
* 类名`el-progress-circle`的 `<div>`元素用于渲染环形/仪表盘进度条功能。
* 类名`el-progress__text`的 `<div>`元素渲染外显文字内容。

到此组件主要实现结构已经介绍完毕，接下来将分析各功能源码实现逻辑。

### 线形进度条

线形进度条，提供了进度百分比显示、文字内显/外显、文字内容格式化、自定义颜色等功能。

组件 DOM 结构如下图： ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1e8a2a19dcb4b088ba95725879c7c43\~tplv-k3u1fbpfcp-watermark.image?)

* 1️⃣ `el-progress` 组件根节点
* 2️⃣ `el-progress-bar` 线形进度条节点
  * 3️⃣ `el-progress-bar__outer` 进度条背景
  * 4️⃣ `el-progress-bar__inner` 进度显示
  * 5️⃣ `el-progress-bar__innerText` 内显文字
* 6️⃣ `el-progress__text` 外显文字

```html
<div class="el-progress">
  <div class="el-progress-bar" v-if="type === 'line'">
    <div class="el-progress-bar__outer" :style="{height: strokeWidth + 'px'}">
      <div class="el-progress-bar__inner" :style="barStyle">
        <div class="el-progress-bar__innerText" v-if="showText && textInside">
          {{content}}
        </div>
      </div>
    </div>
  </div>
  <div class="el-progress-circle" v-else>// ..</div>
  <div
    class="el-progress__text"
    v-if="showText && !textInside"
    :style="{fontSize: progressTextSize + 'px'}"
  >
    <template v-if="!status">{{content}}</template>
    <i v-else :class="iconClass"></i>
  </div>
</div>
```

#### 1️⃣

根据组件设定的属性值绑定组件样式： 进度条类型、进度条状态 、不显示进度条文字、文字显示位置（内置/外部）。

#### 2️⃣

当传入`type`值为`line`或不设置 `type`时，组件渲染成线形进度条。

#### 3️⃣

属性`strokeWidth`用于设置进度条背景的高度。

#### 4️⃣

计算属性`barStyle`用于控制进度条进度和颜色。

`barStyle`根据 `percentage` 属性值动态生成样式对象 `{ width: "80%" , backgroundColor: "" }`。 进度显示通过宽度百分比控制，此处的 `backgroundColor` 为自定义颜色值（会覆盖 status 状态颜色），默认情况为空。

```js
barStyle() {
  const style = {};
  style.width = this.percentage + '%';
  style.backgroundColor = this.getCurrentColor(this.percentage);
  return style;
},
```

***

#### 自定义颜色

方法 `getCurrentColor`用于自定义进度条颜色，通过判断属性 `color` 值的类型实现各自逻辑处理。

`color`  可以接受颜色字符串，函数和数组。

* 若值为函数，函数第一参数应传入 `percentage` 属性值 （格式`function(percentage)`），直接调用函数即可`this.color(percentage)`。
* 若值为字符串，直接返回属性值 `this.color`，`color` 默认值为`""`, 不设置该属性，默认情况下执行此逻辑。
* 若值为数组（字符串或对象数组），根据 `percentage` 所属范围，返回对应颜色。

若值为数组,则调用方法 `getLevelColor` 。

```js
getCurrentColor(percentage) {
  if (typeof this.color === 'function') {
    return this.color(percentage);
  } else if (typeof this.color === 'string') {
    return this.color;
  } else {
    return this.getLevelColor(percentage);
  }
},
```

方法 `getColorArray`，返回对象数组，定义不同数值范围的背景颜色。若字符串数组，方法根据数组长度，自动切分构建成对象数组 `[{ color: "#f56c6c", percentage: 20 }]`。 若传入对象数组，对象格式应为 `{ color: string, percentage: number }`。

```js
getColorArray() {
  const color = this.color;
  const span = 100 / color.length;
  return color.map((seriesColor, index) => {
    // 字符串数组，构建成对象数组
    if (typeof seriesColor === 'string') {
      return {
        color: seriesColor,
        percentage: (index + 1) * span
      };
    }
    return seriesColor;
  });
}

// [
//   { color: "#f56c6c", percentage: 20 },
//   { color: "#e6a23c", percentage: 40 },
//   { color: "#5cb87a", percentage: 60 },
//   { color: "#1989fa", percentage: 80 },
//   { color: "#6f7ad3", percentage: 100 },
// ]
```

方法 `getLevelColor`中调用方法`getColorArray`并排序，比对`percentage`属于哪一区间，返回对应的背景颜色。

```js
getLevelColor(percentage) {
  const colorArray = this.getColorArray().sort((a, b) => a.percentage - b.percentage);

  for (let i = 0; i < colorArray.length; i++) {
    if (colorArray[i].percentage > percentage) {
      return colorArray[i].color;
    }
  }
  return colorArray[colorArray.length - 1].color;
},
```

因为比较使用了`>`,例如 `percentage` 值 100 时, 就会执行`return colorArray[colorArray.length - 1].color;` 返回数组最后对象的颜色。

```html
<el-progress :percentage="percentage" :color="customColors"></el-progress>

<script>
  export default {
    data() {
      return {
        percentage: 100,
        customColors: [
          { color: "#f56c6c", percentage: 20 },
          { color: "#e6a23c", percentage: 40 },
          { color: "#5cb87a", percentage: 60 },
          { color: "#1989fa", percentage: 80 },
          { color: "#6f7ad3", percentage: 100 },
        ],
      };
    },
  };
</script>
```

***

#### 5️⃣

内显内容计算属性 `content`。默认按照 `[percentage]%`格式显示进度百分比。 若设置了属性`format`,则使用函数处理返回文字内容。

组件通过属性 `showText` 、 `textInside` 控制文字显示状态和位置。

```js
content() {
  if (typeof this.format === 'function') {
    return this.format(this.percentage) || '';
  } else {
    return `${this.percentage}%`;
  }
}
```

#### 6️⃣

外显内容跟内显一样，计算属性`content`。若设置了进度条状态`status`，则显示图标。

```html
<template v-if="!status">{{content}}</template>
<i v-else :class="iconClass"></i>
```

计算属性 `iconClass` 返回不同状态 `success/exception/warning`的 icon, 不同类型下`success/exception` 对应图标有所区别。

```js
iconClass() {
  if (this.status === 'warning') {
    return 'el-icon-warning';
  }
  if (this.type === 'line') {
    return this.status === 'success' ? 'el-icon-circle-check' : 'el-icon-circle-close';
  } else {
    return this.status === 'success' ? 'el-icon-check' : 'el-icon-close';
  }
},
```

使用计算属性`progressTextSize`控制字体大小。 当类型为`line`时，基于进度条高度-属性`strokeWidth`；当类型为`circle/dashboard`时，基于环形进度条画布宽度-属性`width`。

```js
progressTextSize() {
  return this.type === 'line'
    ? 12 + this.strokeWidth * 0.4
    : this.width * 0.111111 + 2 ;
},
```

#### 自定义文字格式

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7f685a5c70d47d69319d6ce782cd9bd\~tplv-k3u1fbpfcp-watermark.image?)

```html
// format 示例
<el-progress :percentage="100" :format="format"></el-progress>

<script>
  export default {
    methods: {
      format(percentage) {
        return percentage === 100 ? "满" : `${percentage}%`;
      },
    },
  };
</script>
```

### 组件样式

组件样式源码 `packages\theme-chalk\src\progress.scss` 使用 `scss` 的混合指令 `b`、 `e`、 `m`、 `when` 嵌套生成组件样式。

```

// 生成 .el-progress
@include b(progress) {
  // ...

  // 生成 .el-progress__text
  @include e(text) {
    // ...

    // 生成 .el-progress__text i
    i {
      // ...
    }
  }
  // 生成 .el-progress--circle, .el-progress--dashboard
  @include m((circle,dashboard)) {
    // ...

    // 生成
    // .el-progress--circle .el-progress__text,
    // .el-progress--dashboard .el-progress__text
    .el-progress__text {
      // ...

      // 生成
      // .el-progress--circle .el-progress__text i,
      // .el-progress--dashboard .el-progress__text i
      i {
        // ...
      }
    }
  }


  @include m(without-text) {
    // 生成 .el-progress--without-text .el-progress__text
    .el-progress__text {
      // ...
    }
    // 生成 .el-progress--without-text .el-progress-bar
    .el-progress-bar {
      // ...
    }
  }

  @include m(text-inside) {
    // 生成 .el-progress--text-inside .el-progress-bar
    .el-progress-bar {
      // ...
    }
  }

  // 状态样式
  @include when(success) {
    // 生成 .el-progress.is-success .el-progress-bar__inner
    .el-progress-bar__inner {
      // ...
    }

    // 生成 .el-progress.is-success .el-progress-bar__text
    .el-progress__text {
      // ...
    }
  }

  @include when(warning) {
    // ...
  }

  @include when(exception) {
    // ...
  }
}

// 生成 .el-progress-bar
@include b(progress-bar) {
  // ...

  // 生成 .el-progress-bar__outer
  @include e(outer) {
    // ...
  }
  // 生成 .el-progress-bar__inner
  @include e(inner) {
    // ...

    // 生成 .el-progress-bar__inner::after
    @include utils-vertical-center;
  }

  // 生成 .el-progress-bar__innerText
  @include e(innerText) {
    // ...
  }
}

// 该关键帧规则 代码中没有生效
@keyframes progress {
  0% {
    background-position: 0 0;
  }

  100% {
    background-position: 32px 0;
  }
}
```

在类`.el-progress-bar__inner`  定义了默认进度条颜色 `background-color: #409eff;`。

若指定了组件状态，会生成样式覆盖掉默认颜色。同时制定了外显文本的字体图标颜色。

```css
// success/exception/warning
.el-progress.is-success .el-progress-bar__inner {
  background-color: #67c23a;
}
.el-progress.is-success .el-progress__text {
  color: #67c23a;
}
```

若是使用属性`color`自定义颜色，会生成内联样式，会覆盖 status 状态颜色。

内显文本居右对齐，属性是在`.el-progress-bar__inner`的定义的。

```css
.el-progress-bar__inner {
  text-align: right;
  // ...
}
```

### 环形/仪表盘进度条

环形/仪表盘进度条实现主要使用了`<svg>`元素，会另开篇幅进行详细介绍。

#### 容器/画布

类名`el-progress-circle`的`div`元素创建环形进度条画布宽度（基于属性`width`）。

```html
<div class="el-progress">
  <div class="el-progress-bar" v-if="type === 'line'">// ...</div>
  <div
    class="el-progress-circle"
    :style="{height: width + 'px', width: width + 'px'}"
    v-else
  >
    <svg viewBox="0 0 100 100">
      <path
        class="el-progress-circle__track"
        :d="trackPath"
        stroke="#e5e9f2"
        :stroke-width="relativeStrokeWidth"
        fill="none"
        :style="trailPathStyle"
      ></path>
      <path
        class="el-progress-circle__path"
        :d="trackPath"
        :stroke="stroke"
        fill="none"
        :stroke-linecap="strokeLinecap"
        :stroke-width="percentage ? relativeStrokeWidth : 0"
        :style="circlePathStyle"
      ></path>
    </svg>
  </div>
  <div
    class="el-progress__text"
    v-if="showText && !textInside"
    :style="{fontSize: progressTextSize + 'px'}"
  >
    <template v-if="!status">{{content}}</template>
    <i v-else :class="iconClass"></i>
  </div>
</div>
```

环形/仪表盘进度条使用了`<svg>`元素，元素的`viewBox` 属性允许指定一个给定的一组图形伸展以适应特定的容器元素。`viewBox` 属性的值是一个包含 4 个参数的列表  `min-x`, `min-y`, `width` 、 `height`， 以空格或者逗号分隔开， 在用户空间中指定一个矩形区域映射到给定的元素，不允许宽度和高度为负值，0 则禁用元素的呈现。 此属性绝非三言两语能说清楚，在此不过多详述，推荐阅读 [理解 SVG viewport,viewBox,preserveAspectRatio 缩放](https://www.zhangxinxu.com/wordpress/2014/08/svg-viewport-viewbox-preserveaspectratio/)。

`<svg>`元素实现，使用了两个 `path` 元素创建背景环`el-progress-circle__track`和进度环`el-progress-circle__path`。

`svg`中的元素在浏览器渲染时会遵守 html 中有固定的排列顺序。先出现的元素会先绘制层级较低，而后绘制的元素层级依次增高，如果元素间的位置有重叠，则会现后绘制的元素会遮盖住先出现的元素（层级高遮盖层级低元素）。 渲染后图形叠加就实现了进度环效果。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e79232be5f73461eabced09122495c51\~tplv-k3u1fbpfcp-watermark.image?)

对比可知环形/仪表盘的背景环、以及进度环起始都不一样，组件使用了功能更加强大灵活 `<path>` 标签来实现，但理解起来有点难度。若只是环形进度条可以使用`<circle>`标签实现会更加简单。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/132abaf5e3b54ccfb99f90d23c747ede\~tplv-k3u1fbpfcp-watermark.image?)

#### 背景环 Path

接下先介绍几个基础且重要的计算属性。

属性 `relativeStrokeWidth`是圆形轮廓的宽度，值类型为 `percentage`, 是进度条的宽度跟画布宽度的百分比。 `strokeWidth` 默认值 6 , `width` 默认值 12, 所以 `relativeStrokeWidth` 默认值为 4.8 。

```js
relativeStrokeWidth() {
  return (this.strokeWidth / this.width * 100).toFixed(1);
},
```

属性 `radius` 是绘制圆形的半径，值类型为 `percentage`。

```js
radius() {
  if (this.type === 'circle' || this.type === 'dashboard') {
    // 画布一半 减去 轮廓宽度的一半
    return parseInt(50 - parseFloat(this.relativeStrokeWidth) / 2, 10);
  } else {
    return 0;
  }
},
```

属性 `radius` 是圆形的周长。

```js
perimeter() {
  return 2 * Math.PI * this.radius;
},
```

属性 `rate` 表示绘制圆形的弧长，类型 `dashboard`时只会绘制 `3/4` 周长长度的轮廓，也就是仪表盘进度条为什么有缺口。

```
rate() {
  return this.type === 'dashboard' ? 0.75 : 1;
},
```

背景环 `<path>` 元素中属性`d`创建一个圆形；属性`stroke-width`指定了圆形的轮廓的宽度，绑定计算属性`relativeStrokeWidth` ；属性`stroke`定义了给定图形元素的外轮廓的颜色`#e5e9f2`；属性 `fill` 用于填充轮廓内的形状的颜色，值为`none`无填充；计算属性 `trailPathStyle` 会生成 `stroke-dasharray`和`stroke-dashoffset`等属性用于设置描边的显示和偏移错位。

```js
<path
  class="el-progress-circle__track"
  :d="trackPath"
  stroke="#e5e9f2"
  :stroke-width="relativeStrokeWidth"
  fill="none"
  :style="trailPathStyle">
</path>
```

`fill` 属性填充效果如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/651dcbd5077f48989bbffb3d4bb97853\~tplv-k3u1fbpfcp-watermark.image?)

**d 属性**

属性`d`使用计算属性`trackPath`，返回创建圆环的一系列路径描述。

```js
trackPath() {
  const radius = this.radius;
  const isDashboard = this.type === 'dashboard';
  return `
    M 50 50
    m 0 ${isDashboard ? '' : '-'}${radius}
    a ${radius} ${radius} 0 1 1 0 ${isDashboard ? '-' : ''}${radius * 2}
    a ${radius} ${radius} 0 1 1 0 ${isDashboard ? '' : '-'}${radius * 2}
    `;
},
```

不同类型对应的路径描述。

```js
// circle
M 50 50
m 0 -47
a 47 47 0 1 1 0 94
a 47 47 0 1 1 0 -94

// dashbord
M 50 50
m 0 47
a 47 47 0 1 1 0 -94
a 47 47 0 1 1 0 94
```

路径描述中用到了`Moveto`、`Arcto`这两个指令。

> 指令是大小写敏感的；一个大写的命令指明它的参数是绝对位置，而小写的命令指明相对于当前位置的点。可以指定一个负数值作为命令的参数：负角度将是逆时针的，绝对 x 和 y 位置将视为负坐标。负相对 x 值将会往左移，而负相对 y 值将会向上移。

`Moveto` 命令用作开始一个路径。可以被想象成拎起绘图笔，落脚到另一处。在上一个点和这个指定点之间没有线段绘制。

* `M x,y`  在这里 x 和 y 是绝对坐标，分别代表水平坐标和垂直坐标。
* `m dx,dy`  在这里 dx 和 dy 是相对于当前点的距离，分别是向右和向下的距离。

`Arcto` 命令用作绘制椭圆弧(圆是特殊的椭圆)。 指令格式 `A/a (rx ry x-axis-rotation large-arc-flag sweep-flag x y)` ，

* `A (绝对)| a (相对)`
* `rx ry`是椭圆的两个半轴的长度。
* `x-axis-rotation` 是椭圆相对于坐标系的旋转角度，角度数而非弧度数。
* `large-arc-flag` 是标记绘制大弧(1)还是小弧(0)部分。
* `sweep-flag` 是标记向顺时针(1)还是逆时针(0)方向绘制。
* `x y` 是圆弧终点的坐标。 关于 `large-arc-flag`、`sweep-flag`参数对应的绘制情况。因为绘制的是半圆，参数`large-arc-flag`的绘制大小弧对组件没有影响。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/183505f6badd46ec917a411f054b6d6b\~tplv-k3u1fbpfcp-watermark.image?)

**`circle` 路径图解**

`circle`类型的路径指令解读如下：

1. `M 50 50` 绘制从点位 1️⃣ 也就是`圆心`开始，绝对坐标 `(50,50)`。
2. `m 0 47` 从点位 1️⃣ 沿 y 轴下移 47，也就是圆的半径长度,移动到点位 2️⃣。
3. `a 47 47 0 1 1 0 -94` 从点位 2️⃣ 到点位 3️⃣ 绘制一个半圆弧，从点位 2️⃣ 沿 y 轴上移 94（圆直径长度）就到点位 3️⃣。
4. `a 47 47 0 1 1 0 94` 从点位 3️⃣ 到点位 4️⃣(也就是点位 2️⃣)绘制一个半圆弧，从点位 3️⃣ 沿 y 轴下移 94 就到点位 4️⃣，形成一个闭合圆。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a38dfd922a83449ba4ed3cd6f7fa0999\~tplv-k3u1fbpfcp-watermark.image?)

**`dashbord` 路径图解**

`dashbord`类型的路径指令解读如下：

1. `M 50 50` 绘制从点位 1️⃣ 也就是`圆心`开始，绝对坐标 `(50,50)`。
2. `m 0 -47` 从点位 1️⃣ 沿 y 轴上移 47，也就是圆的半径长度,移动到点位 2️⃣。
3. `a 47 47 0 1 1 0 94` 从点位 2️⃣ 到点位 3️⃣ 绘制一个半圆弧，从点位 2️⃣ 沿 y 轴下移 94（圆直径长度）就到点位 3️⃣。
4. `a 47 47 0 1 1 0 -94` 从点位 3️⃣ 到点位 4️⃣(也就是点位 2️⃣)绘制一个半圆弧，从点位 3️⃣ 沿 y 轴上移 94 就到点位 4️⃣，形成一个闭合圆。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db78c3b4e4dc47afb0fa7d9848c3d0d2\~tplv-k3u1fbpfcp-watermark.image?)

不同类型的图形绘制起点(点位 2️⃣)，决定了进度条进度起点。当然`dashbord`类型到此还没有结束，它的缺口处理还没有。

**缺口实现**

`stroke-dasharray`和`stroke-dashoffset`等属性可以直接用作一个 CSS 样式表内部的属性。 计算属性  `trailPathStyle`  会生成一个样式对象，用于设置描边的显示和偏移错位。

```js
trailPathStyle() {
  return {
    strokeDasharray: `${(this.perimeter * this.rate)}px, ${this.perimeter}px`,
    strokeDashoffset: this.strokeDashoffset
  };
},
// circle
// {
//     stroke-dasharray: 295.31px, 295.31px;
//     stroke-dashoffset: -36.9137px;
// }

// dashbord
// {
//     stroke-dasharray: 221.482px, 295.31px;
//     stroke-dashoffset: -36.9137px;
// }
```

`stroke-dasharray`第一个属性值表是轮廓显示长度，`circle`为 `perimeter` 圆的周长，`dashbord`为 `0.75*perimeter`。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8976f367fc94fabae853036cf18ddb4\~tplv-k3u1fbpfcp-watermark.image?)

计算属性`strokeDashoffset`用于起点的偏移，正数为 x 值向左偏移，负数为 x 值向右偏移。

```js
strokeDashoffset() {
  const offset = -1 * this.perimeter * (1 - this.rate) / 2;
  return `${offset}px`;
},
```

相当于顺时针旋转圆，旋转缺口弧度的一半，形成一个对称结构。 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14e2db3673e74b7d8b0a1b4a9f066e4d\~tplv-k3u1fbpfcp-watermark.image?)

#### 进度环 Path

相较于背景环，进度环代码多了 `stroke-linecap`属性。

```html
<path
  class="el-progress-circle__path"
  :d="trackPath"
  :stroke="stroke"
  fill="none"
  :stroke-linecap="strokeLinecap"
  :stroke-width="percentage ? relativeStrokeWidth : 0"
  :style="circlePathStyle"
>
</path>
```

跟背景环一样路径创建，不同之处进度展示通过`stroke-dasharray` 第一个属性值进行控制，通过`绘制圆弧长度*percentage`来显示进度。同时加入了`transition`缓动效果，使视感更加真实。

```js
circlePathStyle() {
  return {
    strokeDasharray: `${this.perimeter * this.rate * (this.percentage / 100) }px, ${this.perimeter}px`,
    strokeDashoffset: this.strokeDashoffset,
    transition: 'stroke-dasharray 0.6s ease 0s, stroke 0.6s ease'
  };
},
```

属性 `stroke-linecap` 进度条边缘的形状为方形；\
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f089e3810b164700a3408e74dc677929\~tplv-k3u1fbpfcp-watermark.image?)

进度环的颜色使用计算属性`stroke`，支持自定义颜色、状态颜色，默认进度条背景色为`#20a0ff`。

```js
stroke() {
  let ret;
  if (this.color) {
    ret = this.getCurrentColor(this.percentage);
  } else {
    switch (this.status) {
      case 'success':
        ret = '#13ce66';
        break;
      case 'exception':
        ret = '#ff4949';
        break;
      case 'warning':
        ret = '#e6a23c';
        break;
      default:
        ret = '#20a0ff';
    }
  }
  return ret;
},
```

当 `percentage` 值为 0 时，`stroke-width`属性值为 0，元素将不绘制轮廓。

```js
:stroke-width="percentage ? relativeStrokeWidth : 0"
```

#### 外显文本

由前文可知，环形的内容显示为外显内容元素`el-progress__text`，计算属性`content`。若设置了进度条状态`status`，则显示图标。计算属性  `iconClass`  返回不同状态  `success/exception/warning`的 icon, 不同类型下`success/exception`  对应图标有所区别。

```html
<div
  class="el-progress__text"
  v-if="showText && !textInside"
  :style="{fontSize: progressTextSize + 'px'}"
>
  <template v-if="!status">{{content}}</template>
  <i v-else :class="iconClass"></i>
</div>
```

外显文本样式使用绝对定位，在画布水平垂直居中。

```css
.el-progress--circle .el-progress__text,
.el-progress--dashboard .el-progress__text {
  position: absolute;
  top: 50%;
  left: 0;
  width: 100%;
  text-align: center;
  margin: 0;
  -webkit-transform: translate(0, -50%);
  transform: translate(0, -50%);
}
.el-progress--circle .el-progress__text i,
.el-progress--dashboard .el-progress__text i {
  vertical-align: middle;
  display: inline-block;
}
```

### 📚 参考&&关联阅读

["SVG/Element/path",MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/path)\
["SVG/Attribute",MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute)
