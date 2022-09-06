# 3.17 Button 按钮

### 0x00 简介

组件 `Button` 标记了一个（或封装一组）操作命令，响应用户点击行为，触发相应的业务逻辑。 本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。源码实现详见`packages/button/src/` 文件夹下  `button.vue`、 `button-group.vue` 等组件实现。 🔗 [组件文档 Button](https://element.eleme.cn/#/zh-CN/component/button) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/blob/dev/packages/button/src/)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

按钮组件有两部分 `<button>` 、 `<button-group>`，组件源码都在 `packages/button/src/` 文件夹下。在项目工程化机制下，每个组件对应各自的文件夹 `component-name`， 定义导出组件并为其扩展 `install` 方法，以 `commonjs2` 规范对每个组件单独打包构建，支持按需引入。

### 0x01 button-group 按钮组

`button-group.vue` 组件是包裹`button`按钮的容器。 组件创建一个 class 名为`el-button-group`的`<div>`元素容器，提供了匿名插槽，用于分发`button`组件使用内容。

```html
<template>
  <div class="el-button-group">
    <slot></slot>
  </div>
</template>
<script>
  export default {
    name: 'ElButtonGroup'
  };
</script>
```

### 0x02 button 组件

#### template 模板内容

组件创建一个class 名为`el-button`的原生`button`元素，内部由`loading 加载图标`、`icon 图标`以及`自定义按钮文本`等三部分组成。

```html
<template>
  <button
    class="el-button"
    @click="handleClick"
    :disabled="buttonDisabled || loading"
    :autofocus="autofocus"
    :type="nativeType"
    :class="[
      type ? 'el-button--' + type : '',
      buttonSize ? 'el-button--' + buttonSize : '',
      {
        'is-disabled': buttonDisabled,
        'is-loading': loading,
        'is-plain': plain,
        'is-round': round,
        'is-circle': circle
      }
    ]"
  >
    <i class="el-icon-loading" v-if="loading"></i>
    <i :class="icon" v-if="icon && !loading"></i>
    <span v-if="$slots.default"><slot></slot></span>
  </button>
</template>
```

**`button`元素根节点**

* 监听click事件，绑定了`handleClick`回调函数。
* 禁用状态由计算属性`buttonDisabled`、按钮加载状态`loading`属性值判定。
* 通过 `autofocus` 属性值设定当页面加载时按钮是否有输入焦点。
* 原生 `button` 类型设置。可选值 `button / submit / reset`：
  * `submit`:  此按钮将表单数据提交给服务器。如果未指定属性，或者属性动态更改为空值或无效值，则此值为默认值。
  * `reset`:  此按钮重置所有组件为初始值。
  * `button`: 此按钮没有默认行为。它可以有与元素事件相关的客户端脚本，当事件出现时可触发。
* 动态添加样式：
  * 根据 `type` 属性值添加类型样式`el-button--[primary/success/warning/danger/info/text]`。
  * 根据计算属性`buttonSize` 添加尺寸样式 `el-button--[medium/small/mini]`。
  * 根据计算属性`buttonDisabled`和prop的 `loading`、 `plain` 、`round` 、`circle`等属性，设置 `is-disabled`按钮禁用状态、 `is-loading`按钮加载状态、 `is-plain`朴素按钮、 `is-round`圆角按钮、 `is-circle`圆形按钮。

**内部元素节点**

* **loading 加载图标** ：若 `loading` 属性值为 `true`,按钮为加载状态。 渲染一个使用名称 `el-icon-loading` 的 Icon 图标。
* **icon 图标** ：若 `icon`属性设定了图标类名（`truthy`），且按钮不是加载状态，渲染该类名图标。
* **自定义按钮文本** ：提供了匿名插槽的 `<span>`元素，只有分发内容时才会渲染`v-if="$slots.default"`。

> 通过设置 Button 的属性来产生不同的按钮样式，推荐顺序为：`type` -> `plain` -> `round/circle` -> `size` -> `loading` -> `disabled`。

#### attributes 属性

组件 `prop` 定义如下：

```js
props: {
  type: {
    type: String,
    default: 'default'
  },
  size: String,
  icon: {
    type: String,
    default: ''
  },
  nativeType: {
    type: String,
    default: 'button'
  },
  loading: Boolean,
  disabled: Boolean,
  plain: Boolean,
  autofocus: Boolean,
  round: Boolean,
  circle: Boolean
},
```

| 参数          | 说明         | 类型      | 可选值                                                | 默认值    |
| ----------- | ---------- | ------- | -------------------------------------------------- | ------ |
| size        | 尺寸         | string  | medium / small / mini                              | —      |
| type        | 类型         | string  | primary / success / warning / danger / info / text | —      |
| plain       | 是否朴素按钮     | boolean | —                                                  | false  |
| round       | 是否圆角按钮     | boolean | —                                                  | false  |
| circle      | 是否圆形按钮     | boolean | —                                                  | false  |
| loading     | 是否加载中状态    | boolean | —                                                  | false  |
| disabled    | 是否禁用状态     | boolean | —                                                  | false  |
| icon        | 图标类名       | string  | —                                                  | —      |
| autofocus   | 是否默认聚焦     | boolean | —                                                  | false  |
| native-type | 原生 type 属性 | string  | button / submit / reset                            | button |

#### click 事件

监听组件的 `click` 事件 。当点击触发 `click` 事件，执行 `handleClick(evt)`方法，调用 `this.$emit('click', evt);` 触发当前实例上的`click`事件 。

```js
methods: {
  handleClick(evt) {
    // 触发当前实例上的事件 
    this.$emit('click', evt);
  }
}
```

#### 计算属性

**buttonSize**

用来获取按钮的尺寸，根据 `size`属性、使用`Form` 表单时`FormItem`的 计算属性 `elFormItemSize`，还有注册组件时全局属性设置 `this.$ELEMENT.size` 。

使用依赖注入的 `inject` 选项接收`FormItem`实例，获取其计算属性 `elFormItemSize`。

```js
// form-item 组件
inject: {
  elFormItem: {
    default: ''
  }
},

computed: {
  _elFormItemSize() {
    return (this.elFormItem || {}).elFormItemSize;
  },
  buttonSize() {
    return this.size || this._elFormItemSize || (this.$ELEMENT || {}).size;
  },
},
```

同样`form-item`组件中使用`inject` 选项接收`form`实例，用于计算元素尺寸 `elFormItemSize`,由 `form-item`和`form`各自的 `size`属性值计算可得 。

```js
// packages\form\src\form-item.vue
provide() {
  return {
    elFormItem: this
  };
}, 
inject: ['elForm'],

props: { 
  size: String
}, 
computed: { 
  _formSize() {
    return this.elForm.size;
  },
  elFormItemSize() {
    return this.size || this._formSize;
  }, 
},

// packages\form\src\form.vue 
provide() {
  return {
    elForm: this
  };
},

props: { 
  size: String, 
}, 
```

**buttonDisabled**

用来判断按钮的禁用状态，根据 `disabled`属性、`Form` 表单的`disabled`属性判定。

```js
inject: {
  elForm: {
    default: ''
  },
},

computed: {
  buttonDisabled() {
    return this.disabled || (this.elForm || {}).disabled;
  }
},
```

使用`inject` 选项接收`form`组件实例，获取其 **prop** `disabled`属性。

```js
// packages\form\src\form.vue
provide() {
  return {
    elForm: this
  };
},

props: { 
  disabled: Boolean, 
},
```

### 0x03 组件样式

#### src/button.scss

组件`button`、 `button-group` 样式的源码都定义在 `packages\theme-chalk\src\button.scss` 使用混合指令 `b`、`e`、`m`、`when`、 `utils-user-select`、 `button-size` 、 `button-variant` 嵌套生成组件样式。

```scss
// packages\theme-chalk\src\button.scss

// 生成 .el-button
@include b(button) {
  // ... 
  // @include utils-user-select(none); 控制用户能否选中文本
  // @include button-size(...);
  
  // 生成 .el-button+.el-button 
  & + & { /* ...  */ }
  // 生成 .el-button:focus,.el-button:hover
  &:hover, &:focus { /* ...  */ }
  // 生成 .el-button:active
  &:active { /* ...  */ }
  // 生成 .el-button::-moz-focus-inner 
  &::-moz-focus-inner { /* ...  */ }
  
  & [class*="el-icon-"] {
    // 生成 .el-button [class*=el-icon-]+span
    & + span { /* ...  */ }
  }
  
  @include when(plain) {
    // 生成 .el-button.is-plain:focus,.el-button.is-plain:hover
    &:hover,&:focus { /* ...  */ }
    // 生成 .el-button.is-plain:active
    &:active { /* ...  */ }
  }
  
  // 生成 .el-button.is-active
  @include when(active) { /* ...  */ }

  @include when(disabled) {
    // 生成 .el-button.is-disabled,.el-button.is-disabled:focus,.el-button.is-disabled:hover
    &,&:hover,&:focus { /* ...  */ }
    // 生成 .el-button.is-disabled.el-button--text
    &.el-button--text { /* ...  */ }

    &.is-plain {
      // 生成.el-button.is-disabled.is-plain, .el-button.is-disabled.is-plain:focus, .el-button.is-disabled.is-plain:hover
      &,&:hover,&:focus { /* ...  */ }
    }
  }
  // 生成 .el-button.is-loading
  @include when(loading) {
    // ... 
    
    // 生成 .el-button.is-loading:before
    &:before { /* ...  */ }
  }
  // 生成 .el-button.is-round
  @include when(round) { /* ...  */ }
  
  // 生成 .el-button.is-circle 
  @include when(circle) { /* ...  */ }
  
  /* ------------  */
  // 生成主题样式 primary/success/warning/danger/info
  @include m(primary) {
    // @include button-variant ...
  }
  // success/warning/danger/info ...
  /* ------------  */
  // 生成不同尺寸样式  medium/small/mini
  @include m(medium) {
    // @include button-size ...
    
    @include when(circle) { /* ...  */ }
  }
  // small/mini ...
  /* ------------  */
  
  // 生成 .el-button--text
  @include m(text) {
    // ...
    
    // 生成.el-button--text:focus,.el-button--text:hover
    &:hover,
    &:focus {
      // ...
    } 
    // 生成 .el-button--text:active
    &:active {
      // ...
    } 
    // 生成.el-button--text.is-disabled,.el-button--text.is-disabled:focus,.el-button--text.is-disabled:hover
    &.is-disabled,
    &.is-disabled:hover,
    &.is-disabled:focus {
      // ...
    }
  }
}

// 生成 .el-button-group 
@include b(button-group) {
  // ...
  // 生成 .el-button-group>.el-button
  & > .el-button {
    // ...
    
    // 生成 .el-button-group>.el-button+.el-button
    & + .el-button { /* ...  */ }
    // 生成 .el-button-group>.el-button.is-disabled
    &.is-disabled{ /* ...  */ }
    // 生成 .el-button-group>.el-button:first-child
    &:first-child { /* ...  */ }
    // 生成 .el-button-group>.el-button:last-child
    &:last-child { /* ...  */ }
    // 生成 .el-button-group>.el-button:first-child:last-child
    &:first-child:last-child {
      // ...
      
      // 生成 .el-button-group>.el-button:first-child:last-child.is-round
      &.is-round { /* ...  */ }
      // 生成 .el-button-group>.el-button:first-child:last-child.is-circle 
      &.is-circle { /* ...  */ }
    }
    // 生成 .el-button-group>.el-button:not(:first-child):not(:last-child)  
    &:not(:first-child):not(:last-child) { /* ...  */ }
    // 生成 .el-button-group>.el-button:not(:last-child)  
    &:not(:last-child) { /* ...  */ }
    // 生成 .el-button-group>.el-button:active,.el-button-group>.el-button:focus,.el-button-group>.el-button:hover
    &:hover,&:focus,&:active { /* ...  */ }
    // 生成 .el-button-group>.el-button.is-active
    @include when(active) { /* ...  */ }
  }
  
  & > .el-dropdown {
    // 生成 .el-button-group>.el-dropdown>.el-button
    & > .el-button { /* ...  */ }
  }

  @each $type in (primary, success, warning, danger, info) {
    .el-button--#{$type} {
      // 生成 .el-button-group .el-button--[primary/success/warning/danger/info]:first-child
      &:first-child { /* ...  */ }
      // 生成 .el-button-group .el-button--[primary/success/warning/danger/info]:last-child
      &:last-child { /* ...  */ }
      // 生成 .el-button-group .el-button--[primary/success/warning/danger/info]:not(:first-child):not(:last-child)
      &:not(:first-child):not(:last-child) { /* ...  */ }
    }
  }
}
```

`button` 组件样式定义的混合指令。

```js
// packages\theme-chalk\src\mixins\_button.scss

@mixin button-plain($color) {
  // ...

  &:hover,
  &:focus {
    // ...
  }

  &:active {
    // ...
  }

  &.is-disabled {
    &,
    &:hover,
    &:focus,
    &:active {
      // ...
    }
  }
}

@mixin button-variant($color, $background-color, $border-color) {
  // ...

  &:hover,
  &:focus {
    // ...
  }
  
  &:active {
    // ...
  }

  &.is-active {
    // ...
  }

  &.is-disabled {
    &,
    &:hover,
    &:focus,
    &:active {
      // ...
    }
  }

  &.is-plain {
    @include button-plain($background-color);
  }
}

@mixin button-size($padding-vertical, $padding-horizontal, $font-size, $border-radius) {
  // ...
  
  &.is-round {
    padding: $padding-vertical $padding-horizontal;
  }
}
```

### 0x04 📚参考

["button",MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/button)\
["依赖注入",vuejs.org](https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)
