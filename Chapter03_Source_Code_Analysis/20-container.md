# 3.20 Container 布局容器

### 0x00 简介

前文分析过组件的 [布局栅格化(Grid Layout)](https://juejin.cn/post/6997040363546345486) ，通过基础的 24 分栏，迅速简便地创建布局。

本文将介绍用于布局的容器组件，使用 `Flexbox` 功能将其所控制区域设定为特定的布局，方便快速搭建页面的基本结构。本文将深入分析组件源码，剖析其实现原理，耐心读完，相信会对您有所帮助。 [组件文档](https://element.eleme.cn/#/zh-CN/component/container)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 布局容器

布局容器提供5个组件，支持多层嵌套，方便快速搭建页面的基本结构：

* `<el-container>`：布局容器，其下可嵌套 `<el-header>` `<el-footer>` `<el-aside>` `<el-main>` 或 `<el-container>` 本身，可以放在任何父容器中。当子元素中包含 `<el-header>` 或 `<el-footer>` 时，全部子元素会垂直上下排列，否则会水平左右排列。
* `<el-header>`：顶部容器，其下可嵌套任何元素，只能放在 `<el-container>` 中。
* `<el-aside>`：侧边栏容器，其下可嵌套任何元素，只能放在 `<el-container>` 中。
* `<el-main>`：主要区域容器，其下可嵌套任何元素，只能放在 `<el-container>` 中。
* `<el-footer>`：底部容器，其下可嵌套任何元素，只能放在 `<el-container>` 中。

> 以上组件采用了 flex 布局，使用前请确定目标浏览器是否兼容。此外，`<el-container>` 的子元素只能是后四者，后四者的父元素也只能是 `<el-container>`

以下代码通过多层嵌套可以实现常用的页面布局， 更多常用布局实现详见 [官方文档](https://element.eleme.cn/#/zh-CN/component/container#chang-jian-ye-mian-bu-ju) 。

```html
<el-container>
  <el-header>Header</el-header>
  <el-container>
    <el-aside width="200px">Aside</el-aside>
    <el-container>
      <el-main>Main</el-main>
      <el-footer>Footer</el-footer>
    </el-container>
  </el-container>
</el-container>
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/264f2db7f0b240a29831661a73b467df\~tplv-k3u1fbpfcp-watermark.image?)

### 0x02 代码实现

#### container 布局容器

组件 `container` 封装了 `<section>`元素，包含没有后备内容（默认值）的匿名插槽 。组件定义了`direction`的 prop 属性，用于子元素的排列方向。

```js
// packages\container\src\main.vue
<template>
  <section class="el-container" :class="{ 'is-vertical': isVertical }">
    <slot></slot>
  </section>
</template>
<script>
  export default {
    name: 'ElContainer',
    componentName: 'ElContainer',
    props: {
      direction: String
    },
    computed: {
      isVertical() {
        // ...
      }
    }
  };
</script>
```

若没有定义了`direction`属性值，组件通过`tag`判断子元素中包含 `<el-header>` 或 `<el-footer>` 时，全部子元素会垂直上下排列，否则会水平左右排列。 [componentOptions类型定义](https://github.com/vuejs/vue/blob/main/src/types/vnode.ts#L16)

```js
computed: {
  isVertical() {
    if (this.direction === 'vertical') {
      return true;
    } else if (this.direction === 'horizontal') {
      return false;
    }
    return this.$slots && this.$slots.default
      ? this.$slots.default.some(vnode => {
        const tag = vnode.componentOptions && vnode.componentOptions.tag;
        return tag === 'el-header' || tag === 'el-footer';
      })
      : false;
  }
} 
```

#### header 顶部容器

组件 `header` 封装了 `<header>`元素，包含一个 `slot` 。组件定义了`height`的 prop 属性，设置顶部容器高度，默认值`60px`。

```js
// packages\header\src\main.vue
<template>
  <header class="el-header" :style="{ height }">
    <slot></slot>
  </header>
</template>
<script>
  export default {
    name: 'ElHeader',
    componentName: 'ElHeader',
    props: {
      height: {
        type: String,
        default: '60px'
      }
    }
  };
</script>
```

#### aside 侧边栏容器

组件 `aside` 封装了 `<aside>`元素，包含一个 `slot` 。组件定义了`width`的 prop 属性，设置侧边栏宽度，默认值`300px`。

```js
// packages\aside\src\main.vue
<template>
  <aside class="el-aside" :style="{ width }">
    <slot></slot>
  </aside>
</template>
<script>
  export default {
    name: 'ElAside',
    componentName: 'ElAside',
    props: {
      width: {
        type: String,
        default: '300px'
      }
    }
  };
</script>
```

#### main 主要区域（内容）容器

组件 `main` 封装了 `<main>`元素，包含一个 `slot` 。

```js
// packages\main\src\main.vue
<template>
  <main class="el-main">
    <slot></slot>
  </main>
</template>
<script>
  export default {
    name: 'ElMain',
    componentName: 'ElMain'
  };
</script>
```

#### footer 底部容器

组件 `footer` 封装了 `<footer>`元素，包含一个 `slot`。组件定义了`height`的 prop 属性，设置顶部容器高度，默认值`60px`。

```js
// packages\footer\src\main.vue
<template>
  <footer class="el-footer" :style="{ height }">
    <slot></slot>
  </footer>
</template>
<script>
  export default {
    name: 'ElFooter',
    componentName: 'ElFooter',
    props: {
      height: {
        type: String,
        default: '60px'
      }
    }
  };
</script>
```

### 0x03 组件样式

组件样式源码使用 `scss` 的混合指令 `b`、 `when` 嵌套生成组件样式。

```scss
// packages\theme-chalk\src\common\var.scss
$--header-padding: 0 20px !default;
$--footer-padding: 0 20px !default;
$--main-padding: 20px !default;

// packages\theme-chalk\src\container.scss
@include b(container) {
  display: flex;
  flex-direction: row;
  flex: 1;
  flex-basis: auto;
  box-sizing: border-box;
  min-width: 0;

  @include when(vertical) {
    flex-direction: column;
  }
}

// packages\theme-chalk\src\header.scss
@include b(header) {
  padding: $--header-padding;
  box-sizing: border-box;
  flex-shrink: 0;
}

// packages\theme-chalk\src\footer.scss
@include b(footer) {
  padding: $--footer-padding;
  box-sizing: border-box;
  flex-shrink: 0;
} 

// packages\theme-chalk\src\main.scss
@include b(main) {
  // IE11 supports the <main> element partially https://caniuse.com/#search=main
  display: block;
  flex: 1;
  flex-basis: auto;
  overflow: auto;
  box-sizing: border-box;
  padding: $--main-padding;
}

// packages\theme-chalk\src\aside.scss
@include b(aside) {
  overflow: auto;
  box-sizing: border-box;
  flex-shrink: 0;
} 
```

使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成样式内容如下。

```css
/* packages\theme-chalk\lib\container.css */
.el-container {
  display: flex;
  flex-direction: row;
  flex: 1;
  flex-basis: auto;
  box-sizing: border-box;
  min-width: 0;
}

.el-container.is-vertical {
  flex-direction: column;
}

/* packages\theme-chalk\lib\main.css */
.el-main {
  display: block;
  flex: 1;
  flex-basis: auto;
  overflow: auto;
  box-sizing: border-box;
  padding: 20px;
}

/* packages\theme-chalk\lib\aside.css */
.el-aside {
  overflow: auto;
  box-sizing: border-box;
  flex-shrink: 0;
}

/* packages\theme-chalk\lib\header.css */
.el-header {
  padding: 0 20px;
  box-sizing: border-box;
  flex-shrink: 0;
}

/* packages\theme-chalk\lib\footer.css */
.el-footer {
  padding: 0 20px;
  box-sizing: border-box;
  flex-shrink: 0;
}
```

容器布局实现使用 CSS Flexbox,`flex-basis`、`flex-shrink`、`flex` 等属性的语法内容请阅读 [Flex 布局教程：语法篇 阮一峰](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)。

前文曾提到`<el-container>` 的子元素只能是后四者，后四者的父元素也只能是 `<el-container>`。因为 只有`container` 组件指定为 Flex 布局，其余组件若是想要Flex 布局生效，只能将组件作为 `container` 的子组件，当然 `container` 可以自包含。
