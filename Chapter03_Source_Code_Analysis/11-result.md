# 3.11 Result 结果

### 0x00 简介

组件 `Result` 用于对用户的操作结果或者异常状态做反馈。 本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。组件源码实现详见`packages/result/src/` 文件夹下包含多个文件 `index.vue`、`icon-info.vue`、`icon-success.vue`、`icon-warning.vue`、`icon-error.vue`。 🔗 [组件文档 Result](https://element.eleme.cn/#/zh-CN/component/result) 🔗 [github源码](https://github1s.com/ElemeFE/element/blob/dev/packages/result/src/)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

***

### 0x01 svg 图标组件

图标通过封装一个 `<svg>`元素，定义 `viewBox`属性，通过 `<path>` 元素是用来定义形状。

```js
<template>
  <svg viewBox="0 0 48 48" xmlns="http://www.w3.org/2000/svg">
      <!--  创建图形形状元素 -->
      <path   ...  /> 
  </svg>
</template>
```

封装 `icon-info.vue`、`icon-success.vue`、`icon-warning.vue`、`icon-error.vue` 等单独组件实现不同类型的`svg`图标。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d6cd431061d400cb54547a6b2a2be6e\~tplv-k3u1fbpfcp-watermark.image)

图标颜色需要通过添加自定义的 class 样式修改。

### 0x02 组件源码

#### template 模板内容

模板创建一个 `<div>` 元素根节点，class 名为`el-result`，包含 4 个子节点 `自定义图标`、`自定义标题`、`自定义二级标题`、`自定义底部额外区域`。

每个节点都提供了各自具名`slot`, 在向具名插槽提供内容的时候需要指明 `slot#name`，否则内容无法正确分发，将被丢弃（未提供匿名插槽）。

```html
<template>
  <div class="el-result">
    <!-- 自定义图标 slot icon -->
    <div class="el-result__icon">
      <slot name="icon">
        <component :is="iconElement" :class="iconElement" />
      </slot>
    </div>
    <!-- 自定义标题 slot title --> 
    <div v-if="title || $slots.title" class="el-result__title">
      <slot name="title">
        <p>{{ title }}</p>
      </slot>
    </div>
    <!-- 自定义二级标题 slot subTitle --> 
    <div v-if="subTitle || $slots.subTitle" class="el-result__subtitle">
      <slot name="subTitle">
        <p>{{ subTitle }}</p>
      </slot>
    </div>
    <!-- 自定义底部额外区域 slot extra --> 
    <div v-if="$slots.extra" class="el-result__extra">
      <slot name="extra"></slot>
    </div>
  </div>
</template>
```

**自定义图标**

该节点是一个class 名为`el-result__icon`的`<div>` 元素包，裹着名为`icon`的具名`slot`。

插槽提供了后备内容，动态引用svg 图标组件。具体详见下文章节 **动态组件** 。

**自定义标题**

该节点是一个class 名为`el-result__title`的`<div>` 元素包，裹着名为`title`的具名`slot`。

只有在 `title` 是 `truthy` 或者向该插槽提供内容时，该节点才会被渲染。若两者都设置了（传入`title`值而且向该插槽提供内容），展示内容为插槽分发内容优先级较高。因为 `title` 是做为插槽的`后备内容`。

> **truthy**（真值）指的是在**布尔值**上下文中，转换后的值为真的值。所有值都是真值，除非它们被定义为 [假值](https://developer.mozilla.org/zh-CN/docs/Glossary/Falsy)（即除 `false`、`0`、`""`、`null`、`undefined` 和 `NaN` 以外皆为真值）

**自定义二级标题**

该节点是一个class 名为`el-result__subtitle`的`<div>` 元素包，裹着名为`title`的具名`subTitle`。

只有在 `subTitle` 是 `truthy` 或者向该插槽提供内容时，该节点才会被渲染。若两者都设置了（传入`subTitle`值而且向该插槽提供内容），展示内容为插槽分发内容优先级较高。因为 `subTitle` 是做为插槽的`后备内容`。

**自定义底部额外区域**

该节点是一个class 名为`el-result__extra`的`<div>` 元素包，裹着名为`extra`的具名`slot`。只有向该插槽提供内容时`v-if="$slots.extra"`，该节点才会被渲染。

#### 动态组件

上文名为`icon`的具名`slot`的 `后备内容`使用了`动态组件` ,根据传入不同的图标类型`icon`，动态引入对应类型svg图标组件。

通过 Vue 的 `<component>` 元素加一个特殊的 `is` attribute 来实现在不同组件之间进行动态切换。

```html
<component :is="iconElement" :class="iconElement" />
```

svg 组件引入注册。

```js
<script>
import IconSuccess from './icon-success.vue';
import IconError from './icon-error.vue';
import IconWarning from './icon-warning.vue';
import IconInfo from './icon-info.vue'; 


export default { 
  components: {
    [IconSuccess.name]: IconSuccess,
    [IconError.name]: IconError,
    [IconWarning.name]: IconWarning,
    [IconInfo.name]: IconInfo
  }, 
};
</script> 
```

**计算属性**

计算属性 `iconElement`根据传入`icon`值动态生成 图标名称。用于组件引用和class添加。

```js

const IconMap = {
  success: 'icon-success',
  warning: 'icon-warning',
  error: 'icon-error',
  info: 'icon-info'
};

export default { 
  // ...
  computed: {
    iconElement() {
      const icon = this.icon;
      return icon && IconMap[icon] ? IconMap[icon] : 'icon-info';
    }
  }
}; 
```

根据 `IconMap` 定义，若传入的`icon`值不是`success / warning / info / error`其中一个，返回默认值`icon-info` 等同于 `icon`值为 `info`。

`iconElement`值范围为 `icon-success`, `icon-warning`, `icon-error`, `icon-info`。

**svg 样式**

`iconElement`值 `icon-success`、 `icon-warning`、 `icon-error`、 `icon-info` 会用于svg图标颜色的自定义。不同类型的class定义了不同的颜色，效果如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5191e54899c24edf915f35a2b506d572\~tplv-k3u1fbpfcp-watermark.image)

#### attributes 属性

组件定义了3 个prop : `title` 、`subTitle` 、`icon` 。

```js
  props: {
    title: {
      type: String,
      default: ''
    },
    subTitle: {
      type: String,
      default: ''
    },
    icon: {
      type: String,
      default: 'info'
    }
  },
```

**title**

设置标题内容,默认值为 `''`。 具体功能详见上文章节 **自定义标题** 。

**subTitle**

设置二级标题内容,默认值为 `''`。 具体功能详见上文章节 **自定义二级标题** 。

**icon**

图标类型设置,用于引入指定类型的svg图标、设置图标颜色。 具体功能详见上文章节 **计算属性** 。

***

### 0x03 组件样式

#### src/result.scss

组件样式源码 `packages\theme-chalk\src\result.scss` 使用混合指令 `b`、`e` 嵌套生成组件样式。

```scss
 
// 生成 .el-result
@include b(result) {
  // ... 
  
  // 生成 .el-result__icon svg
  @include e(icon) {
    svg {
      // ... 
    }
  }
  
  // 生成 .el-result__title
  @include e(title) {
    // ...  
    
    // 生成 .el-result__title  p
    p {
      // ... 
    }
  }
  
  // 生成 .el-result__subtitle
  @include e(subtitle) {
    // ... 
    
    // 生成 .el-result__subtitle  p
    p {
      // ... 
    }
  }
  
  // 生成 .el-result__extra
  @include e(extra) {
    // ... 
  }
  
  // 生成 .el-result .icon-success
  .icon-success {
    // ... 
  }
  
  // 生成 .el-result .icon-error
  .icon-error {
    // ... 
  }
  
  // 生成 .el-result .icon-info
  .icon-info {
    // ... 
  }
  
  // 生成 .el-result .icon-warning
  .icon-warning {
    // ... 
  }
}
```

#### lib/result.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\result.scss`，内容格式如下。

```css

.el-result {
    // ... 
}

.el-result__icon svg {
    // ... 
}

.el-result__title {
    // ... 
}

.el-result__title p {
    // ... 
}

.el-result__subtitle {
    // ... 
}

.el-result__subtitle p {
    // ... 
}

.el-result__extra {
    // ... 
}

.el-result .icon-success {
    // ... 
}

.el-result .icon-error {
    // ... 
}

.el-result .icon-info {
    // ... 
}

.el-result .icon-warning {
    // ... 
}
```

***

### 0x04 📚参考

["动态组件",vuejs.org](https://cn.vuejs.org/v2/guide/components.html#%E5%8A%A8%E6%80%81%E7%BB%84%E4%BB%B6)\
["viewBox",MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/viewBox)\
["Truthy",MDN](https://developer.mozilla.org/zh-CN/docs/Glossary/Truthy)
