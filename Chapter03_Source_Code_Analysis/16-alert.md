# 3.16 Alert 警告

### 0x00 简介

组件 `Alert` 用于警告提示，展现需要关注的信息。本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。组件源码实现详见`packages/alert/src/main.vue` 。 🔗 [组件文档 Alert](https://element.eleme.cn/#/zh-CN/component/alert) 🔗 [gitee源码 main.vue](https://gitee.com/ElemeFE/element/blob/dev/packages/alert/src/main.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 组件源码

#### template 模板内容

组件模板创建一个class名为`el-alert`的`<div>` 元素根节点 `1️⃣`，包含两个子节点 ：左侧的`Icon图标``2️⃣`、右侧的`文字内容区域` `3️⃣`。 组件使用内置 `transition` 实现`el-alert-fade` 淡入淡出效果。

组件DOM结构渲染如下 👇: ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31971bc1293d4e1194fdbb895e3ecb09\~tplv-k3u1fbpfcp-watermark.image)

```html
<template>
  <transition name="el-alert-fade">
    <div
      class="el-alert"
      :class="[typeClass, center ? 'is-center' : '', 'is-' + effect]"
      v-show="visible"
      role="alert"
    >
      <!-- icon 图标 -->
      <i class="el-alert__icon" :class="[ iconClass, isBigIcon ]" v-if="showIcon"></i>
      <!-- 文字内容 包含关闭按钮 -->
      <div class="el-alert__content">
        <!-- 标题 -->
        <span class="el-alert__title" :class="[ isBoldTitle ]" v-if="title || $slots.title">
          <slot name="title">{{ title }}</slot>
        </span>
        <!-- 辅助性文字介绍 -->
        <p class="el-alert__description" v-if="$slots.default && !description"><slot></slot></p>
        <p class="el-alert__description" v-if="description && !$slots.default">{{ description }}</p>
        <!-- 关闭按钮 -->
        <i class="el-alert__closebtn" :class="{ 'is-customed': closeText !== '', 
            'el-icon-close': closeText === '' }" v-show="closable" @click="close()">
          {{closeText}}
        </i>
      </div>
    </div>
  </transition>
</template>
```

**1️⃣ 根节点**

* 根据传入_prop_的属性值动态添加class。
  * 计算属性`typeClass` 根据 `type` 属性值生成不同类型样式 class ， `el-alert--[success/warning/info/error]`。
  * 根据 `center` 属性值生成`is-center`，让图标文字水平居中。
  * 根据 `effect` 属性值生成主题样式 `is-[light/dark]`。
* `v-show="visible"` 使用 _Data Property_ 的 `visible`属性控制组件显示隐藏。组件虽为页面中的非浮层元素不会自动消失，但支持手动关闭隐藏。
* `role="alert"` 表示当前元素的类型,用于 `ARIA` 无障碍访问设置。

根节点元素内部使用flex布局。两个子节点交叉轴的中点对齐。

```css
.el-alert { 
    display: flex;
    /* 交叉轴的中点对齐 */
    align-items: center; 
}
```

`is-center` 通过设定属性`justify-content` 定义了主轴上的对齐方式，实现两个子节点水平居中。

```css
.is-center {
  justify-content: center;
}
```

**2️⃣ Icon图标**

该节点是一个Icon 图标元素,表示某种状态时提升可读性，图标节点是否渲染可见根据 `showIcon`属性值决定 `v-if="showIcon"` 。

* 静态样式 `el-alert__icon` 定义图标默认尺寸。
* 使用计算属性 `iconClass` `isBigIcon` 动态添加样式。
  * 计算属性`iconClass` 根据 `type` 属性值生成不同类型 icon class ， `el-icon-[success/warning/info/error]`。
  * 计算属性`isBigIcon` 根据 辅助性文字内容是否设置，生成图标大尺寸样式`is-big`。

Icon 不同大小尺寸效果 👇: ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ebdc46e2889462b836e1f18a259b417\~tplv-k3u1fbpfcp-watermark.image)

**3️⃣ 文字内容区域**

该节点是一个class 名为`el-alert__content`的`<div>` 元素，内容包含三个节点： `标题` `4️⃣` 、 `辅助性文字` `5️⃣`、 `关闭按钮` `6️⃣` 。

文字内容区域 DOM 结构渲染如下 👇:

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5add985891b244bc85ab05fc29f7642f\~tplv-k3u1fbpfcp-watermark.image)

> `标题` 、 `辅助性文字` 节点提供了具名插槽和匿名插槽，分发内容时需要指明 `slot#name`，否则内容都会分发给匿名插槽。

**4️⃣ 标题**

该节点是一个class 名为`el-alert__title`的`<span>` 元素,包裹着具名插槽 `title`。

只有在 `title`属性值是 `truthy` 或者向该插槽分发内容时，该节点才会被渲染。`title` 是插槽的`后备内容`。若两者都设置了（传入`title`值而且向该插槽提供内容），渲染展示内容为插槽分发内容。

使用计算属性 `isBoldTitle` 动态添加样式。 `isBoldTitle` 根据辅助性文字内容是否设置，生成样式`is-big` 加粗title字体,用于区分与辅助性文字的主次。

```
.is-bold {
  font-weight: 700;
}
```

**5️⃣ 辅助性文字**

该节点是一个class 名为`el-alert__description`的`<p>`段落元素。该节点提供了匿名插槽或 `description`属性值，但是两者只能同时设置一项，才能正常显示。

当然可以改成如下逻辑，默认显示`description`属性值，插槽分发内容后优先显示后者。

```html
<p class="el-alert__description" v-if="description || $slots.default">
    <slot> {{ description }} </slot>
</p> 
```

> 基于很多组件的逻辑实现： prop属性大多作为插槽的后备内容， 暂时还不知为啥提供这么对立条件，有他无我！

**6️⃣ 关闭按钮**

节点默认是一个名为`el-icon-close`的Icon 图标,提供`close`事件来设置关闭时的回调。通过 `closable`属性决定是否可关闭。

若设置`closeText`属性值自定义关闭按钮，此时节点为包裹文本的 `<i>`元素。

关闭按钮位置偏移至右上角，样式`el-alert__closebtn`定义如下：

```css
.el-alert__closebtn { 
  position: absolute;
  top: 12px;
  right: 15px; 
} 
```

#### attributes 属性

组件定义了8个prop 。

| 参数          | 说明        | 类型      | 可选值                        | 默认值   |
| ----------- | --------- | ------- | -------------------------- | ----- |
| title       | 标题        | string  | —                          | —     |
| type        | 类型        | string  | success/warning/info/error | info  |
| description | 辅助性文字     | string  | —                          | —     |
| closable    | 是否可关闭     | boolean | —                          | true  |
| center      | 文字是否居中    | boolean | —                          | true  |
| close-text  | 关闭按钮自定义文本 | string  | —                          | —     |
| show-icon   | 是否显示图标    | boolean | —                          | false |
| effect      | 选择提供的主题   | string  | light/dark                 | light |

`effect` 提供了自定义验证函数,提供了默认值`light`。 若传入`effect`值不是以下`light/dark`其中之一，会生成无效的样式`is-[effect]`，导致组件样式无法正常加载。

```js
props: { 
  effect: {
    type: String,
    default: 'light',
    validator: function(value) {
      return ['light', 'dark'].indexOf(value) !== -1;
    }
  }
},
```

#### 计算属性

**typeClass**

根据 `type` 属性值生成不同类型主题样式（背景颜色、文字颜色） `el-alert-[success/warning/info/error]`。

```js
computed: {
  typeClass() {
    return `el-alert--${ this.type }`;
  }, 
}
```

**iconClass**

计算属性`iconClass` 根据 `type` 属性值生成不同类型 icon class `el-icon-[success/warning/info/error]`。

```js
const TYPE_CLASSES_MAP = {
  'success': 'el-icon-success',
  'warning': 'el-icon-warning',
  'error': 'el-icon-error'
};

computed: { 
  iconClass() {
    return TYPE_CLASSES_MAP[this.type] || 'el-icon-info';
  }, 
}
```

若传入`type`值不是以下`success/warning/error`其中之一，都会生成`el-icon-info`，所以`type`默认值等于`info`。

**isBigIcon**

根据辅助性文字内容是否设置，只有在 `title`属性值是 `truthy` 或者向该插槽分发内容时，生成样式`is-big` 加粗title字体,用于区分与辅助性文字的主次。

```js
computed: { 
  isBigIcon() {
    return this.description || this.$slots.default ? 'is-big' : '';
  },
}
```

Icon 根据组件内容多少（高度也会调整），动态调整尺寸展示更加协调 👇: ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ebdc46e2889462b836e1f18a259b417\~tplv-k3u1fbpfcp-watermark.image)

**isBoldTitle**

根据辅助性文字内容是否设置，只有在 `title`属性值是 `truthy` 或者向该插槽分发内容时，生成图标大尺寸样式`is-big`

```js
computed: { 
  isBoldTitle() {
    return this.description || this.$slots.default ? 'is-bold' : '';
  }
}
```

#### close 事件

组件监听按钮的 `click` 事件 。当点击关闭按钮时，触发 `click` 事件，执行 `close()`方法，设置`visible`属性值 `false`，隐藏组件；调用 `this.$emit('close')` 触发当前实例上的`close`事件 。

```js
close() {
  // 隐藏组件
  this.visible = false;
  // 触发当前实例上的事件 
  this.$emit('close');
}
```

### 0x02 组件样式

#### src/alert.scss

组件样式源码 `packages\theme-chalk\src\alert.scss` 使用混合指令 `b`、`e` 、`m`、`when` 嵌套生成组件样式。

```scss

//  .el-alert
@include b(alert) {
  // ...
  
  @include when(light) {
    //  .el-alert.is-light .el-alert__closebtn
    .el-alert__closebtn { /* ... */ }
  }

  @include when(dark) {
    //  .el-alert.is-dark .el-alert__closebtn
    .el-alert__closebtn { /* ... */ }
    
    //  .el-alert.is-dark .el-alert__description
    .el-alert__description { /* ... */ }
  }
  
  //  .el-alert.is-center
  @include when(center) { /* ... */ }

  // success  warning  error 格式类似
  @include m(success) {
  
    // .el-alert--success.is-light
    &.is-light {
      // ...
      
      //.el-alert--success.is-light .el-alert__description
      .el-alert__description { /* ... */ }
    }
    // .el-alert--success.is-dark
    &.is-dark { /* ... */ }
  }
  
  // info
  @include m(info) {
    //.el-alert--info.is-light
    &.is-light { /* ... */ }
    
    // .el-alert--info.is-dark
    &.is-dark { /* ... */ }
    
    // .el-alert--info .el-alert__description
    .el-alert__description { /* ... */ }
  }
 

  // .el-alert__content
  @include e(content)  { /* ... */ }
  
  // .el-alert__icon
  @include e(icon) {
    // ...
   
    // .el-alert__icon.is-big
    @include when(big)  { /* ... */ }
  }
  // .el-alert__title
  @include e(title) {
    // ...
    
    // .el-alert__title.is-bold
    @include when(bold)  { /* ... */ }
  }
  
  // .el-alert .el-alert__description
  & .el-alert__description  { /* ... */ }
  
  // .el-alert__closebtn
  @include e(closebtn) {
    // ...
    
    // .el-alert__closebtn.is-customed
    @include when(customed)  { /* ... */ }
  }
}

.el-alert-fade-enter, .el-alert-fade-leave-active  { /* ... */ }
```

#### lib/alert.css

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\alert.scss`，内容格式如下。

```css
.el-alert { /* ...  */ } 
.el-alert.is-light .el-alert__closebtn { /* ...  */ } 
.el-alert.is-dark .el-alert__closebtn,
.el-alert.is-dark .el-alert__description  { /* ...  */ } 
.el-alert.is-center  { /* ...  */ } 

/* ...success...  */ 
.el-alert--success.is-light  { /* ...  */ } 
.el-alert--success.is-light .el-alert__description  { /* ...  */ } 
.el-alert--success.is-dark { /* ...  */ } 
/* ...warning...  */ 
/* ...error...  */  
/* ...info...  */  
.el-alert--info.is-light { /* ...  */ } 
.el-alert--info.is-dark { /* ...  */ } 
.el-alert--info .el-alert__description { /* ...  */ } 

.el-alert__content { /* ...  */ } 
.el-alert__icon  { /* ...  */ } 
.el-alert__icon.is-big { /* ...  */ } 
.el-alert__title { /* ...  */ } 
.el-alert__title.is-bold  { /* ...  */ } 
.el-alert .el-alert__description { /* ...  */ } 
.el-alert__closebtn { /* ...  */ } 
.el-alert__closebtn.is-customed { /* ...  */ } 
.el-alert-fade-enter, .el-alert-fade-leave-active { /* ...  */ }  
```

### 0x03 📚参考

["ARIA",MDN](https://developer.mozilla.org/zh-CN/docs/Web/Accessibility/ARIA)
