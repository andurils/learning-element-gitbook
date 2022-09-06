# 3.23 Switch 开关

### 简介

开关选择器 `Switch` 用于表示开关状态/两种状态之间的切换时。本文将分析源码实现，耐心读完，相信会对您有所帮助。组件源码实现详见`packages/switch/src/component.vue` 。 🔗 [组件文档 Switch](https://element.eleme.cn/#/zh-CN/component/switch) 🔗 [gitee源码 component.vue](https://gitee.com/ElemeFE/element/blob/dev/packages/switch/src/component.vue)

组件`checkbox`单独使用可以表示两种状态之间的切换，和 `switch` 类似。两者在于切换 `switch` 会直接触发状态改变，而 `checkbox` 一般用于状态标记，需要和提交操作配合。

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 源码实现

组件的 `prop` 声明，各属性功能说明详见 [官方文档switch#attributes](https://element.eleme.cn/#/zh-CN/component/switch#attributes) 。

组件的根节点是类名`el-switch`的div元素，内部有四个子元素：

* `type="checkbox"`的input 元素。
* 关闭时的图标/文字描述（左侧），类名`el-switch__label--left`的span 元素。
* 组件核心，类名`el-switch__core`的span 元素。
* 打开时的图标/文字描述（右侧），类名`el-switch__label--right`的span 元素。

```html
<template>
  <div
    class="el-switch"
    :class="{ 'is-disabled': switchDisabled, 'is-checked': checked }"
    role="switch"
    :aria-checked="checked"
    :aria-disabled="switchDisabled"
    @click.prevent="switchValue"
  >
    <input class="el-switch__input" type="checkbox">
    <span  :class="['el-switch__label', 'el-switch__label--left']"></span>
    <span  class="el-switch__core" ref="core" ></span>
    <span  :class="['el-switch__label', 'el-switch__label--right']"></span> 
  </div>
</template>
```

组件页面渲染内容如下。 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/605d7f7eaddc40a4910f3262322c4f43\~tplv-k3u1fbpfcp-watermark.image?)

页面无法看到 input 元素，由其样式`.el-switch__input`可知宽/高/透明度都为0。页面无法聚焦到该元素。

```css
.el-switch__input {
  position: absolute;
  width: 0;
  height: 0;
  opacity: 0;
  margin: 0;
}
```

#### 根节点

根据组件的禁用状态以及打开状态绑定样式 `is-disabled`、`is-checked`。

```js
:class="{ 'is-disabled': switchDisabled, 'is-checked': checked }"
```

计算属性`checked`判定组件是否打开状态。组件扩展的 value 类型，不仅支持`boolean`类型，也支持`string/number`。当使用`string/number`类型参数时， 需要手动设定`activeValue` 和 `inactiveValue`属性值。

```js
checked() {
    return this.value === this.activeValue;
  },
```

计算属性`switchDisabled`返回组件是否禁用。

```js
computed: { 
  switchDisabled() {
    return this.disabled || (this.elForm || {}).disabled;
  }
},
// 注入 在form中使用 
inject: {
  elForm: {
    default: ''
  }
},
```

添加点击事件`@click="switchValue"`，用于实现组件点击后切换状态（当设置文字描述时，点击文字也能实现状态切换）。事件使用了修饰符 `prevent`，当触发的事件调用 `event.preventDefault()`，阻止自身默认事件的执行。

方法 `switchValue` 在非禁用状态下调用`handleChange`方法，实现开关状态的切换。

方法 `handleChange`中使用`this.$emit('input', val);`更新组件v-model值也就是value值。没有看到更新属性`value`值的操作。使用`this.$emit('change', val);`触发自定义状态变化事件。

```js
switchValue() {
  !this.switchDisabled && this.handleChange();
},
handleChange(event) {
  const val = this.checked ? this.inactiveValue : this.activeValue;
  this.$emit('input', val);
  this.$emit('change', val);
  // ...
},
```

> 组件使用v-model，需要使用`$emit('input')`来改变组件v-model值，否则无法更新用户传入的v-model值。 因为

属性 `role` 、`aria-checked` 、`aria-disabled`用于实现 ARIA无障碍网页应用。

#### 状态描述

组件支持使用文字或者icon图标对状态进行描述。

组件左侧为关闭状态描述元素，只有设置了属性`inactiveIconClass`或`inactiveText`才会显示。若设置属性`inactiveIconClass`会忽略 `inactiveText`。

组件右侧为打开状态描述元素，实现逻辑请参考`左侧`分析。

```html
<span
  :class="['el-switch__label', 'el-switch__label--left', !checked ? 'is-active' : '']"
  v-if="inactiveIconClass || inactiveText">
  <i :class="[inactiveIconClass]" v-if="inactiveIconClass"></i>
  <span v-if="!inactiveIconClass && inactiveText" :aria-hidden="checked">{{ inactiveText }}</span> 
// core
<span
  :class="['el-switch__label', 'el-switch__label--right', checked ? 'is-active' : '']"
  v-if="activeIconClass || activeText">
  <i :class="[activeIconClass]" v-if="activeIconClass"></i>
  <span v-if="!activeIconClass && activeText" :aria-hidden="!checked">{{ activeText }}</span>
</span>
```

#### 组件核心

span元素是开关的外层容器，圆形滑块是由`:after`伪元素实现。

```html
<span class="el-switch__core" ref="core" :style="{ 'width': coreWidth + 'px' }">
```

通过CSS设置高度、外边框圆角实现左右半圆效果。

```css
.el-switch__core { 
  display: inline-block;
  position: relative;
  height: 20px;  
  border-radius: 10px; 
  background: #dcdfe6;
}
```

定义打开状态下开关背景色。

```css
.el-switch.is-checked .el-switch__core {
  border-color: #409eff;
  background-color: #409eff;
} 
```

圆形滑块使用绝对定位，使用 `left`和`margin-left`定义打开状态下滑块位置（最右侧） 。 使用`transition: all .3s`定义了滑块动画时间以及背景颜色变化的时间。

```css
.el-switch__core:after {
  content: "";
  position: absolute;
  top: 1px;
  left: 1px;
  border-radius: 100%; 
  transition: all 0.3s;
  width: 16px;
  height: 16px;
  background-color: #fff;
}

.el-switch.is-checked .el-switch__core::after {
  left: 100%;
  margin-left: -17px;
}
```

属性 `coreWidth` 默认值为40。若设置了不同状态的开关背景色，在`mounted`中会调用方法`setBackgroundColor`。将元素样式`border-color`、 `background-color`更新为当前状态设置的颜色。

```js
props: {
  width: {
    type: Number,
    default: 40
  },
},
data() {
  return {
    coreWidth: this.width
  };
}, 
mounted() { 
  this.coreWidth = this.width || 40;
  if (this.activeColor || this.inactiveColor) {
    this.setBackgroundColor();
  } 
},
methods: { 
  setBackgroundColor() {
    let newColor = this.checked ? this.activeColor : this.inactiveColor;
    this.$refs.core.style.borderColor = newColor;
    this.$refs.core.style.backgroundColor = newColor;
  },
}
```

#### input 元素

关于 input 元素的功能和使用场景无法准确分析推测。

```html
<template>
  <div class="el-switch" >
    <input
      class="el-switch__input"
      type="checkbox"
      @change="handleChange"
      ref="input"
      :id="id"
      :name="name"
      :true-value="activeValue"
      :false-value="inactiveValue"
      :disabled="switchDisabled"
      @keydown.enter="switchValue"
    >
    // ...
  </div>
</template>
<script> 
  import Focus from 'element-ui/src/mixins/focus'; 

  export default { 
    mixins: [Focus('input'), Migrating, emitter], 
    props: {
      // ...
      name: {
        type: String,
        default: ''
      }, 
      id: String
    }, 
    watch: {
      checked() {
        this.$refs.input.checked = this.checked;
        // ...
      }
    },
    methods: {
      handleChange(event) {
        // ...
        this.$nextTick(() => {
          // set input's checked property
          // in case parent refuses to change component's value
          this.$refs.input.checked = this.checked;
        });
      }, 
    },
    mounted() {
      // ...
      this.$refs.input.checked = this.checked;
    }
  };
</script>
```

文档中组件暴露出的 `fous`方法就是用于获取input元素的焦点 `this.$refs["input"].focus()`。

```js
export default function(ref) {
  return {
    methods: {
      focus() {
        this.$refs[ref].focus(); // 获取焦点;
      }
    }
  };
}
```

将相关代码移除，组件也可正常运行。此处逻辑不做过多赘述！

#### 拾遗

在`created`中，进行了数值处理，若是v-model的值不是 `activeValue` `inactiveValue`任意属性值，使用`$emit('input',val)` 更新v-model值为`inactiveValue` ，让组件置位关闭状态。`!`和`~`运算符让number转变成boolen类型。

```js
created() {
  if (!~[this.activeValue, this.inactiveValue].indexOf(this.value)) {
    this.$emit('input', this.inactiveValue);
  }
},
```

当设置属性 `validateEvent`值为true 时，改变 switch 状态时是否触发表单的校验，触发组件 `formItem` 的方法 `onFieldChange`。具体校验逻辑等到组件`Form` 分析文章详细介绍。

```js
watch: {
  checked() {
    // ...
    if (this.validateEvent) {
      this.dispatch('ElFormItem', 'el.form.change', [this.value]);
    }
  }
},

//  method  packages\form\src\form-item.vue  
addValidateEvents() {
  const rules = this.getRules();

  if (rules.length || this.required !== undefined) {
    // ...
    this.$on('el.form.change', this.onFieldChange);
  }
},
```

### 组件样式

组件样式源码 `packages\theme-chalk\src\switch.scss` 使用混合指令 `b`、`e`、`when`、`m` 嵌套生成组件样式。

```scss
// 生成 .el-rate
@include b(switch) {
  // ... 
  @include when(disabled) {
    // 生成 .el-switch.is-disabled .el-switch__core,.el-switch.is-disabled .el-switch__label 
    & .el-switch__core,
    & .el-switch__label {
      // ...
    }
  }
  // 生成 .el-switch__label
  @include e(label) {
    // ...
    // 生成.el-switch__label.is-active
    @include when(active) {
      // ...
    }
    // 生成 .el-switch__label--left 
    @include m(left) {
      // ...
    }
    // 生成 .el-switch__label--right
    @include m(right) {
      // ...
    }
    // 生成 .el-switch__label * 
    & * {
      // ...
    }
  }
  // 生成 .el-switch__input
  @include e(input) {
    // ...
  }
  // 生成 .el-switch__core 
  @include e(core) {
    // ...
    // 生成 .el-switch__core:after
    &:after {
      // ...
    }
  } 
  
  @include when(checked) {
    // 生成  .el-switch.is-checked .el-switch__core
    .el-switch__core {
      // ...
      // 生成 .el-switch.is-checked .el-switch__core::after
      &::after {
        // ...
      }
    }
  }
  // 生成 .el-switch.is-disabled
  @include when(disabled) {
    // ...
  }
  
  @include m(wide) { 
    .el-switch__label {
      &.el-switch__label--left {
        // 生成 .el-switch--wide .el-switch__label.el-switch__label--left span
        span {
          // ...
        }
      }
      &.el-switch__label--right {
        // 生成 .el-switch--wide .el-switch__label.el-switch__label--right span
        span {
          // ...
        }
      }
    }
  }
  // 过渡样式使用场景未找到
  // 生成 .el-switch .label-fade-enter,.el-switch .label-fade-leave-active
  & .label-fade-enter,
  & .label-fade-leave-active {
    // ...
  }
}  
```
