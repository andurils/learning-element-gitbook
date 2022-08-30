# 3.9 Divider 分割线

### 0x00 简介

组件 `Divider` 多用于区隔内容。 本文将深入分析组件源码，剖析其实现原理，耐心读完，相信会对您有所帮助。`packages/divider/src/main.vue` 文件是组件源码实现。 🔗 [组件文档 Divider](https://element.eleme.cn/#/zh-CN/component/divider) 🔗 [github源码 main.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/divider/src/main.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 组件源码

组件`divider/src/main.vue` **SFC(单文件组件 single-file components )**，声明了基于模板的函数式组件。

```html
<template functional>
    // ...
</template>
<script>
export default {
  name: 'ElDivider',
  props: {
    // ...
  }
};
</script>
```

#### attributes 属性

组件定义了2个prop : `direction` 、`contentPosition`

**direction**

设置分割线方向,默认值`horizontal`，可选值为 `horizontal / vertical`，定义了 `validator` 进行验证。如果传入其他值生成无效样式 `el-divider--${direction}`。

```js
direction: {
  type: String,
  default: 'horizontal',
  validator(val) {
    return ['horizontal', 'vertical'].indexOf(val) !== -1;
  }
},
```

**contentPosition**

设置分割线文案的位置,默认值`center`，可选值为 `left / right / center`，定义了 `validator` 进行验证。如果传入其他值生成无效样式 `is-${props.contentPosition}`。

```js
contentPosition: {
  type: String,
  default: 'center',
  validator(val) {
    return ['left', 'center', 'right'].indexOf(val) !== -1;
  }
}
```

#### template 模板内容

组件将创建一个 `<div>` 元素节点,用来绘制分割线。 该节点包含1个子节点 `<div>` ，内置了 `slot` 用于自定义分割线文案。 子节点只有组件是分割线水平方向，且提供插槽内容用于自定义分割线文案时才会渲染。

```html
  <div
    v-bind="data.attrs"
    v-on="listeners"
    :class="[data.staticClass, 'el-divider', `el-divider--${props.direction}`]"
  >
    <div
      v-if="slots().default && props.direction !== 'vertical'"
      :class="['el-divider__text', `is-${props.contentPosition}`]"
     >
      <slot />
    </div>
  </div>
```

再进一步分析组件之前，先了深入了解下函数式组件。

**函数式组件 context**

函数式组件需要的一切都是通过 `context` 参数传递，组件中使用了该参数的部分属性：

* `props`：提供所有 prop 的对象
* `slots`：一个函数，返回了包含所有插槽的对象
* `data`：传递给组件的整个数据对象
* `listeners`： 一个包含了所有父组件为当前组件注册的事件监听器的对象。这是 `data.on` 的一个别名。

其他属性 `parent`、`children`、`scopedSlots`、`injections` 详见[官方文档](https://cn.vuejs.org/v2/guide/render-function.html#%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BB%84%E4%BB%B6) 。

`context` 接口定义 [@2.6源码 types/options.d.ts#L136](https://github.com/vuejs/vue/blob/2.6/types/options.d.ts#L136)

```js
export interface RenderContext<Props=DefaultProps> {
  props: Props;
  children: VNode[];
  slots(): any;
  data: VNodeData;
  parent: Vue;
  listeners: { [key: string]: Function | Function[] };
  scopedSlots: { [key: string]: ScopedSlot };
  injections: any
}
```

其中数据对象 `data` 类型为 `VNodeData` [官方文档](https://cn.vuejs.org/v2/guide/render-function.html#%E6%B7%B1%E5%85%A5%E6%95%B0%E6%8D%AE%E5%AF%B9%E8%B1%A1) 。

`VNodeData` 接口定义 [@2.6源码 flow/vnode.js#L35](https://github.com/vuejs/vue/blob/2.6/flow/vnode.js#L35) 。

**class Vs staticClass**

从接口声明上看两者区别在于`staticClass`只能是字符串。 `class`支持所有其他格式，例如对象或数组。

```js
declare interface VNodeData {
  // ...
  staticClass?: string;
  class?: any;
  // ...
};
```

下面使用 `Vue.compile` 实时编译模板字符串，对比区别，代码详见 [模板编译](https://cn.vuejs.org/v2/guide/render-function.html#%E6%A8%A1%E6%9D%BF%E7%BC%96%E8%AF%91)。

指令 `v-bind` 用于 `class`  时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组。

```html
<div :class="['element','active']"></div> 
```

编译(render)内容 输出 `class`

```js
function anonymous(
) {
  with(this){return _c('div',{class:['element','active']})}
}
```

不使用指令 `v-bind` 定义 `class`字符串内容时。

```html
<div class="active"></div> 
```

编译(render)内容 输出 `staticClass`

```js
function anonymous(
) {
  with(this){return _c('div',{staticClass:"active"})}
}
```

#### 组件功能

组件将创建一个 `<div>` 元素父节点,通过背景色绘制分割线。  `el-divider` 定义元素背景色， 通过`el-divider--horizontal` 或 `el-divider--vertical` 定义元素宽高实现水平、垂直分割效果

基于模板的函数式组件，可以通过 `context`访问到其独立的上下文内容。 `v-bind="data.attrs"` 使用 `data.attrs` 传递任何 HTML attribute；`v-on="listeners"` 使用 `listeners`  (即 `data.on` 的别名) 传递任何事件监听器。

`:class="[data.staticClass, 'el-divider', 'el-divider--${props.direction}']"` 根据传入prop 值设置分割线方向样式，然后将静态class 和动态生成的class 合并 merge 。

组件包含1个子节点 `<div>` ，只有组件是分割线水平方向（`props.direction !== 'vertical'`），且提供插槽内容（`slots().default`）时才会渲染。根据`contentPosition` 生成文案位置样式, `is-left`、 `is-right`或`is-center`。

***

### 0x02 组件样式

#### src/divider.scss

组件样式源码 `packages\theme-chalk\src\divider.scss` 使用混合指令 `b`、 `m`、`e`、 `when` 嵌套生成组件样式。

```scss
// 生成 .el-divider
@include b(divider) {
  // ...
  // 生成 .el-divider--horizontal
  @include m(horizontal) {
    // ...
  }
  // 生成 .el-divider--vertical
  @include m(vertical) {
    // ...
  }
  // 生成 .el-divider__text
  @include e(text) {
    // ...
    // 生成 .el-divider__text.is-left
    @include when(left) {
      // ...
    }
    // 生成 .el-divider__text.is-center
    @include when(center)  {
      // ...
    }
    // 生成 .el-divider__text.is-right
    @include when(right)  {
      // ...
    }
  }
}
```

#### lib/divider.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\divider.scss`，内容格式如下。

```css
.el-divider {
  // ...
}
.el-divider--horizontal {
  // ...
}
.el-divider--vertical {
  // ...
}
.el-divider__text {
  // ...
}
.el-divider__text.is-left {
  // ...
}
.el-divider__text.is-center {
  // ...
}
.el-divider__text.is-right {
  // ...
} 
```

### 0x03 📚参考

["vm-attrs",vuejs.org](https://cn.vuejs.org/v2/api/#vm-attrs)\
["vm-listeners",vuejs.org](https://cn.vuejs.org/v2/api/#vm-listeners)\
["函数式组件",vuejs.org](https://cn.vuejs.org/v2/guide/render-function.html#%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BB%84%E4%BB%B6)\
[“class vs staticclass”，stackoverflow](https://stackoverflow.com/questions/57943450/is-there-any-difference-between-class-and-staticclass-in-vue-js)
