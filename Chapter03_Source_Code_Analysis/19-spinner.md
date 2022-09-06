# 3.19 Spinner 加载中

### 0x00 简介

组件 `Spinner` 用于页面和区块的加载中状态,页面局部处于等待异步数据或正在渲染过程时，合适的加载动效会有效缓解用户的焦虑。 本文将深入分析组件源码，剖析其实现原理，耐心读完，相信会对您有所帮助。`packages/spinner/src/spinner.vue` 文件是组件源码实现。 🔗 [github 源码 spinner.vue](https://gitee.com/ElemeFE/element/blob/dev/packages/spinner/src/spinner.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 Spinner 组件

`Spinner.vue` 组件是一个隐藏组件，暂未有其他组件使用。

模板创建一个 class 名为`el-spinner`的`span`元素,包含了一个`<circle>` SVG 圆形。

```html
<template>
  <span class="el-spinner">
    <svg
      class="el-spinner-inner"
      viewBox="0 0 50 50"
      :style="{ width: radius/2 + 'px', height: radius/2 + 'px' }"
    >
      <circle
        class="path"
        cx="25"
        cy="25"
        r="20"
        fill="none"
        :stroke="strokeColor"
        :stroke-width="strokeWidth"
      ></circle>
    </svg>
  </span>
</template>
<script>
  export default {
    name: "ElSpinner",
    props: {
      type: String,
      radius: {
        type: Number,
        default: 100,
      },
      strokeWidth: {
        type: Number,
        default: 5,
      },
      strokeColor: {
        type: String,
        default: "#efefef",
      },
    },
  };
</script>
```

组件提供了以下 `prop` 属性。

| 参数          | 说明                            | 类型     | 默认值       |
| ----------- | ----------------------------- | ------ | --------- |
| type        | 无效参数                          | string | —         |
| radius      | svg 容器大小宽高值 `radius/2`，单位`px` | number | 100       |
| strokeWidth | 图形的轮廓的宽度                      | number | 5         |
| strokeColor | 图形的外轮廓的颜色                     | string | `#efefef` |

### 0x02 组件样式

#### src/spinner.scss

组件样式源码 `packages\theme-chalk\src\spinner.scss` 使用混合指令 `b` 嵌套生成组件样式。

```scss
// 生成 .el-time-spinner
@include b(time-spinner) {
  // ...
}
// 生成 .el-spinner
@include b(spinner) {
  // ...
}
// 生成 .el-spinner-inner
@include b(spinner-inner) {
  // ...

  // 生成 .el-spinner-inner .path
  & .path {
    // ...
  }
}
// 关键帧 CSS动画
@keyframes rotate {
  /* ...  */
}
@keyframes dash {
  /* ...  */
}
```

#### lib/spinner.css

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\spinner.css`，内容格式如下。

```css
.el-time-spinner {
  /* ...  */
}
.el-spinner {
  /* ...  */
}
.el-spinner-inner {
  animation: rotate 2s linear infinite;
  width: 50px;
  height: 50px;
}
.el-spinner-inner .path {
  stroke: #ececec;
  stroke-linecap: round;
  animation: dash 1.5s ease-in-out infinite;
}
@keyframes rotate {
  /* ...  */
}
@keyframes dash {
  /* ...  */
}
```

### 0x03 CSS 动画

默认的 `<circle>` 元素展示如下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48506cad5e8a4277b7ad705d822a94bc\~tplv-k3u1fbpfcp-watermark.image?)

使用 **`@keyframes`** [at-rule](https://developer.mozilla.org/zh-CN/docs/Web/CSS/At-rule)  规则通过在动画序列中定义关键帧（或 waypoints）的样式来控制 CSS 动画序列

定义 `<circle>`元素动画关键帧规则 `dash`

```css
@keyframes dash {
  0% {
    stroke-dasharray: 1, 150;
    stroke-dashoffset: 0;
  }
  50% {
    stroke-dasharray: 90, 150;
    stroke-dashoffset: -35;
  }
  100% {
    stroke-dasharray: 90, 150;
    stroke-dashoffset: -124;
  }
}

.el-spinner-inner .path {
  animation: dash 1.5s ease-in-out infinite;
}
```

添加动画效果后， `<circle>`元素展示如下：

![spinner1.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea0e1e27d9a14db3bb3632f567ef61fd\~tplv-k3u1fbpfcp-watermark.image?)

定义 `<svg>`元素动画关键帧规则 `rotate`，用于元素旋转。

```css
@keyframes rotate {
  100% {
    transform: rotate(360deg);
  }
}

.el-spinner-inner {
  animation: rotate 2s linear infinite;
}
```

添加动画效果后， `<svg>`元素展示如下：

![spinner\_svg.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2d94f67330b461780a73bcde86beffa\~tplv-k3u1fbpfcp-watermark.image?)

组件中不同元素的关键帧效果叠加,实现了 loding 动效。

![spinner\_c.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0433175516b74443bc0728795749b341\~tplv-k3u1fbpfcp-watermark.image?)

组件提供了 prop 用于自定义效果，代码如下：

```html
<template>
  <el-spinner
    :radius="300"
    :stroke-width="8"
    :stroke-color="'#E6A23C'"
  ></el-spinner>
</template>
```

组件效果如下：

![spinner\_ss.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e51f274a73b4906a95580a686bc1f2a\~tplv-k3u1fbpfcp-watermark.image?)

由于组件样式 `.path` 定义了 `stroke` 属性设置了默认值 `#ececec;`,导致`stroke-color`设置无效。需要移除该属性才能生效。

```css
@include b(spinner-inner) {
  // ...

  & .path {
    // stroke: #ececec;
    stroke-linecap: round;
    animation: dash 1.5s ease-in-out infinite;
  }
}
```

### 0x04 📚 参考

["@keyframes",MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@keyframes)
