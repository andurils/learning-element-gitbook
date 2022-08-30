# 3.10 Card 卡片

### 0x00 简介

组件 `Card` 作为容器多用于信息聚合展示。 本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。`packages/card/src/main.vue` 文件是组件源码实现。 🔗 [组件文档 Card](https://element.eleme.cn/#/zh-CN/component/card) 🔗 [github源码 main.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/card/src/main.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 组件源码

#### template 模板内容

模板创建一个 `<div>` 元素父节点，通过 prop `shadow`动态添加卡片阴影样式。

若 `shadow` 值为 **null、undefined、空字符串**，表达式 `"shadow ? 'is-' + shadow + '-shadow' : 'is-always-shadow'"`始终返回`is-always-shadow`；其余返回 `is-[shadowType]-shadow`。

父节点包含2个子节点：`header`和`body`。

`header`节点是一个包裹着名为`header`的具名`slot`的`<div>` 元素，也可以传入prop `header`。`header`部分是可选的，只有为 具名`header` 插槽分发内容 或者定义 prop `header`的值，该节点才会被渲染（`v-if="$slots.header || header"`）。 若两者都设置了，展示内容为插槽分发内容优先级较高。因为 prop `header` 是做为插槽的`后备内容`。

`body`节点是一个包裹着匿名的`slot`的`<div>` 元素节点，class 值为`el-card__body`，用于内容自定义，支持通过 prop `bodyStyle`进行样式自定义。

{% code overflow="wrap" lineNumbers="true" %}
```html
<template>
  <div class="el-card" :class="shadow ? 'is-' + shadow + '-shadow' : 'is-always-shadow'">
    <!-- header -->
    <div class="el-card__header" v-if="$slots.header || header">
      <slot name="header">{{ header }}</slot>
    </div>
    <!-- body -->
    <div class="el-card__body" :style="bodyStyle">
      <slot></slot>
    </div>
  </div>
</template> 
```
{% endcode %}

#### attributes 属性

组件定义了3 个prop : `header` 、`contentPosition` 、`shadow` 。

```js
props: {
  header: {},
  bodyStyle: {},
  shadow: {
    type: String
  }
},
```

**header**

设置 header 文本内容（**卡片标题**），字符串类型。但仍可以传入其他类型：数组、 函数等，显示内容为对象转成字符串。

**bodyStyle**

设置 body 的样式，用于传入自定义样式。因为 body 的 class 定义如下，所以 bodyStyle 默认值等同于 `{ padding: '20px' }` 。

{% code title="card.scss" overflow="wrap" lineNumbers="true" %}
```css
/* packages\theme-chalk\lib\card.scss */
.el-card__body { 
    padding: 20px; 
}
```
{% endcode %}

**shadow**

设置阴影显示时机,没有定义`defalut`。当 `shadow`值未定义时 `undefined`， 表达式 `"shadow ? 'is-' + shadow + '-shadow' : 'is-always-shadow'"`始终返回`is-always-shadow`，等同于设置默认值`always`。 可选值为 `always / hover / never`，生成样式 `is-[shadowType]-shadow`。 只有 `is-always-shadow` 和 `is-hover-shadow` 样式有定义，其余都为无效样式（等同于设置 `never`）。

### 0x02 组件样式

#### src/card.scss

组件样式源码 `packages\theme-chalk\card\link.scss` 使用混合指令 `b`、`e`、 `when` 嵌套生成组件样式。

```scss
// 生成 .el-card
@include b(card) {
  // ...
  // 生成 .el-card.is-always-shadow
  @include when(always-shadow) {
    // ...
  }
  // 生成 .el-card.is-hover-shadow:hover, .el-card.is-hover-shadow:focus
  @include when(hover-shadow) {
    &:hover,
    &:focus {
      // ...
    }
  }
  // 生成 .el-card__header
  @include e(header) {
    // ...
  }
  // 生成 .el-card__body
  @include e(body) {
    // ...
  }
}
```

#### lib/card.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\card.scss`，内容格式如下。

```css
.el-card {
  // ...
}
.el-card.is-always-shadow,
.el-card.is-hover-shadow:focus,
.el-card.is-hover-shadow:hover {
  // ...
}
.el-card__header {
  // ...
}
.el-card__body {
  padding: 20px;
} 
```

***

### 0x03 📚参考

["components-slots",vuejs.org](https://cn.vuejs.org/v2/guide/components-slots.html)
