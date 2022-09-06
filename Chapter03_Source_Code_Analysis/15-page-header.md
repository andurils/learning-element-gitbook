# 3.15 PageHeader 页头

### 0x00 简介

组件 `PageHeader` 用于用户需要快速理解当前页是什么时。相较于面包屑组件，使用页头组件的页面的路径比较简单。本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。组件源码实现详见`packages/page-header/src/main.vue` 。 🔗 [组件文档 PageHeader](https://element.eleme.cn/#/zh-CN/component/page-header) 🔗 [gitee源码 main.vue](https://gitee.com/ElemeFE/element/blob/dev/packages/page-header/src/main.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 组件源码

#### template 模板内容

组件模板创建一个class名为`el-page-header`的`<div>` 元素根节点，包含两个子节点 `左侧标题区域`、`右侧内容区域`。

两个子节点都提供了具名插槽，分发内容时需要指明 `slot#name`，否则内容会被丢弃。

```html
<template>
  <div class="el-page-header">
    <div class="el-page-header__left" @click="$emit('back')">
      <i class="el-icon-back"></i>
      <div class="el-page-header__title">
        <slot name="title">{{ title }}</slot>
      </div>
    </div>
    <div class="el-page-header__content">
      <slot name="content">{{ content }}</slot>
    </div>
  </div>
</template>
```

**左侧标题区域**

该节点是一个class 名为`el-page-header__left`的`<div>` 元素,并添加了`click`事件监听。\
当点击左侧区域后触发`click`事件，调用内建的 `$emit` 并传入事件名称`back` 来触发回调。

该节点区域内包含两个元素节点：

1. 一个名为`el-icon-back`的Icon图标 。
2. 一个class 名为`el-page-header__title`的`<div>` 元素，提供了具名插槽`title`。

具名插槽`title`提供后备内容，prop的 `title` 属性，`title` 默认值通过国际化方法设置 `t('el.pageHeader.title')` ，使用中文语言时值为`返回`。

**右侧内容区域**

该节点是一个class 名为`el-page-header__content`的`<div>` 元素，提供了具名插槽`content`。具名插槽`title`提供后备内容，prop的 `content` 属性。

#### attributes 属性

组件定义了2 个prop : `title`、`content`。

属性`title` 默认值使用工厂函数调用国际化方法获取翻译文本`t('el.pageHeader.title')`。

```js
// 引入国际化处理方法
import { t } from 'element-ui/src/locale';
export default { 
  props: {
    title: {
      type: String, 
      default() {
        return t('el.pageHeader.title');
      }
    }, 
  }
};
```

使用中文语言时值为`返回`。

```js
// 中文语言包  src\locale\lang\zh-CN.js
export default {
  el: { 
    pageHeader: {
      title: '返回'
    }, 
  }
}; 
```

#### 样式设置

`el-page-header` `el-page-header__left` 元素使用 flex 默认布局。

```css
.el-page-header { 
  display: flex; 
}
.el-page-header__left { 
  display: flex; 
}
```

左侧标题区域`el-page-header__left`和右侧内容区域`el-page-header__content`的分隔符通过CSS样式实现，通过设置 左侧标题区域的 伪类 `::after`。

```css
.el-page-header__left::after {
  content: "";
  position: absolute;
  width: 1px;
  height: 16px;
  right: -20px;
  top: 50%; 
  transform: translateY(-50%);
  background-color: #dcdfe6;
}
```

组件渲染效果如下 👇：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d94054934bb2408292b1ef83be0229c5\~tplv-k3u1fbpfcp-watermark.image)

***

### 0x02 组件样式

#### src/page-header.scss

组件样式源码 `packages\theme-chalk\src\page-header.scss` 使用混合指令 `b`、`e` 嵌套生成组件样式。

```scss
// 生成 .el-page-header
@include b(page-header) {
  // ...
  
  // 生成 .el-page-header__left
  @include e(left) {
    // ...
    
    // 生成 .el-page-header__left::after
    &::after {
      // ...
    }
    
    // 生成 .el-page-header__left .el-icon-back 
    .el-icon-back {
      // ...
    }
    
    // 生成 .el-page-header__title
    @include e(title) {
      // ...
    }
  }
  // 生成 .el-page-header__content
  @include e(content) {
    // ...
  }
}
```

#### lib/page-header.css

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\page-header.scss`，内容格式如下。

```css
.el-page-header { /* ...  */ } 
.el-page-header__left { /* ...  */ } 
.el-page-header__left::after { /* ...  */ } 
.el-page-header__left .el-icon-back { /* ...  */ } 
.el-page-header__title { /* ...  */ } 
.el-page-header__content { /* ...  */ } 
```

### 0x03 📚参考

[“vm-emit”，vuejs.org](https://cn.vuejs.org/v2/api/#vm-emit)\
[“伪类 ::after”，MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::after)
