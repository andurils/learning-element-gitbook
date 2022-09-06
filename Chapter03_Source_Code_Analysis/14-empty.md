# 3.14 Empty 空状态

### 0x00 简介

组件 `Empty` 用于空状态时的展示占位图。 使用场景多为当目前没有数据时，用于显式的用户提示； 或初始化场景时的引导创建流程。 本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。组件源码实现详见`packages\empty\src\index.vue` 。 🔗 [组件文档 Empty](https://element.eleme.cn/#/zh-CN/component/empty) 🔗 [gitee源码 index.vue](https://gitee.com/ElemeFE/element/blob/dev/packages/empty/src/index.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 组件源码

#### template 模板内容

`index.vue` 组件模板创建一个class名为`el-empty`的`<div>` 元素根节点，包含三个子节点 `空状态图片`、`描述文字`、`底部内容`。

每个节点都提供了插槽，需要注意的底部内容的插槽为匿名插槽，分发内容时需要指明 `slot#name`，否则内容都默认分发至 `底部内容` 节点内。。

```html
<template>
  <div class="el-empty">
    <!-- 空状态图片 slot image -->
    <div class="el-empty__image" :style="imageStyle">
      <img v-if="image" :src="image" ondragstart="return false">
      <slot v-else name="image">
        <img-empty />
      </slot>
    </div>
    <!-- 描述文字 slot  description -->
    <div class="el-empty__description">
      <slot v-if="$slots.description" name="description"></slot>
      <p v-else>{{ emptyDescription }}</p>
    </div>
    <!-- 底部内容 匿名插槽-->
    <div v-if="$slots.default" class="el-empty__bottom">
      <slot></slot>
    </div>
  </div>
</template>
```

**空状态图片**

该节点是一个class 名为`el-empty__image`的`<div>` 元素，使用计算属性 `imageStyle` 定义图片宽度。

若设置 `image` 属性传入图片 URL，优先渲染 `<img>` 元素，图片不支持拖放 `ondragstart="return false"`。

默认渲染`image`插槽的后背内容--内置的SVG图标组件 `<img-empty/>`，组件详情详见下文章节 **内置 svg 图标组件** 。

```js
// 组件引入
import ImgEmpty from './img-empty.vue'; 

export default {
  name: 'ElEmpty',
  components: {
    // name: 'ImgEmpty',
    [ImgEmpty.name]: ImgEmpty
  },
}
```

**描述文字**

该节点是一个class 名为`el-empty__description`的`<div>` 元素，提供了名为`description`的具名`slot`。

默认使用 `<p>` 段落元素包裹计算属性 `emptyDescription`渲染。 只有当向`description`插槽分发内容时 `v-if="$slots.description"`，使用插槽渲染内容。

计算属性 `emptyDescription` 返回 prop 的 `description`属性值 或 国际化空状态描述设置 `t('el.empty.description')`。优先返回 `description` 值。

**底部内容**

该节点是一个class 名为`el-empty__bottom`的`<div>` 元素，提供了匿名插槽。只有向该插槽分发内容时`v-if="$slots.default"`，该节点才会被渲染。

#### attributes 属性

组件定义了3 个prop : `image` 、`imageSize` 、`description` 。

```js
props: {
  // 图片URL
  image: {
    type: String,
    default: ''
  },
  // 设定图片宽度
  imageSize: Number,
  // 文本描述
  description: {
    type: String,
    default: ''
  }
},
```

#### 计算属性

**emptyDescription**

当 `description` 属性值是 `truthy` 时，返回`description` 属性值。否则默认返回国际化空状态描述设置 `t('el.empty.description')`。

```js
// 引入国际化处理方法
import { t } from 'element-ui/src/locale';

emptyDescription() {
  return this.description || t('el.empty.description');
},
```

前文 [组件源码国际化](https://juejin.cn/post/6992193878321266725#heading-5) 提到国际化功能也可以通过 `mixin` 方式提供给组件页面使用。

```js
import Locale from 'element-ui/src/mixins/locale';

export default {
  mixins: [Locale], 
  computed: {
    emptyDescription() {
      // return this.description || t('el.empty.description');
      return this.description || this.t('el.empty.description');
    }, 
  }
};
```

**imageStyle**

定义图片显示尺寸（宽度）。当 `imageSize` 属性值是 `truthy` 时，返回自定义宽度样式。

```js
imageStyle() {
  return {
    width: this.imageSize ? `${this.imageSize}px` : ''
  };
}
```

#### 内置 svg 图标组件

`img-empty.vue`图标组件封装一个 `<svg>`元素， 图片内容如下 👇

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4378062fea514276adec96a480e0d5e8\~tplv-k3u1fbpfcp-watermark.image)

***

### 0x02 组件样式

#### src/empty.scss

组件样式源码 `packages\theme-chalk\src\empty.scss` 使用混合指令 `b`、`e` 嵌套生成组件样式。

```scss
// 生成 .el-empty
@include b(empty) {
  // ...

  // 生成 .el-empty__image
  @include e(image) {
    // ...
    
    // 生成 .el-empty__image img 
    img {
      // ...
    }
    
    // 生成 .el-empty__image svg
    svg {
      // ...
    }
  }
  
  // 生成 .el-empty__description
  @include e(description) {
    // ...
    
    // 生成 .el-empty__description p
    p {
      // ...
    }
  }
  
  // 生成 .el-empty__bottom
  @include e(bottom) {
    // ...
  }
}
```

#### lib/empty.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\empty.scss`，内容格式如下。

```css
.el-empty { /* ...  */ }
/* image */
.el-empty__image { /* ...  */ }
.el-empty__image img, .el-empty__image svg { /* ...  */ }
.el-empty__image img { /* ...  */ }
.el-empty__image svg { /* ...  */ }
/* description */
.el-empty__description { /* ...  */ }
.el-empty__description p { /* ...  */ }
/* bottom  */
.el-empty__bottom { /* ...  */ } 
```
