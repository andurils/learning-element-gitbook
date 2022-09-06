# 3.28 Timeline 时间线

### 简介

时间轴组件`Timeline` 常用于垂直展示的时间流信息。 当有一系列信息需按时间排列时，使用时间轴进行视觉上的（正序或倒序）串联。本文将分析其源码实现，耐心读完，相信会对您有所帮助。🔗 [组件文档 Timeline](https://element.eleme.cn/#/zh-CN/component/timeline) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/tree/dev/packages/timeline/src)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 组件源码

时间轴功能提供了两个组件：`el-timeline`和 `el-timeline-item`。

各组件的 `prop` 声明，各属性功能说明详见[官方文档 Timeline#attributes](https://element.eleme.cn/#/zh-CN/component/timeline#timeline-attributes) 。

#### 根组件 main.vue

根组件是一个函数式组件，使用渲染函数/JSX 语法一个无序列表`<ul>` 元素。提供匿名插槽展示节点内容。

```js
// packages\timeline\src\main.vue
<script>
  export default {
    name: 'ElTimeline',
    props: {
      reverse: {
        type: Boolean,
        default: false
      }
    },
    // dead code
    provide() {
      return {
        timeline: this
      };
    },
    // 渲染函数
    render() {
      const reverse = this.reverse;
      const classes = {
        'el-timeline': true,
        'is-reverse': reverse
      };
      let slots = this.$slots.default || [];
      if (reverse) {
        slots = slots.reverse();
      }
      return (<ul class={ classes }>
        { slots }
      </ul>);
    }
  };
</script> 
```

属性 `reverse` 用于节点排序方向。通过`this.$slots`访问传入插槽的 元素类型为`vnode`节点数组，然后使用 方法`reverse()`将数组中元素的位置颠倒。

```js
let slots = this.$slots.default || [];
if (reverse) { 
  slots = slots.reverse();
}
```

> 此排序是元素节点位置顺序，不是节点内时间戳排序。

代码中存在不少 `dead code`,具体功能意义不明（感觉功能开发一半就停了）。

属性 `reverse`值为 `ture`,节点逆序时生成类名`is-reverse`，但是样式表中未定义此类。

```js
const classes = { 
 'is-reverse': reverse
};

// packages\theme-chalk\src\timeline.scss
@include b(timeline) {
  margin: 0;
  font-size: $--font-size-base;
  list-style: none;
  // 隐藏末节点轴线
  & .el-timeline-item:last-child {
    & .el-timeline-item__tail {
      display: none;
    }
  }
}
```

依赖注入代码即使注释掉也不影响组件功能。

```js
// packages\timeline\src\main.vue 
provide() {
  return {
    timeline: this
  };
},

// packages\timeline\src\item.vue
inject: ['timeline'],
```

#### 节点组件 item.vue

节点组件的模板用于渲染一个表示列表里的条目的`<li>` 元素。该元素节点下内容包含三部分：

* 垂直方向轴线。
* 节点/图标。
* 内容（含时间戳）。

```html
<template>
  <li class="el-timeline-item">
    <!-- 轴线 -->
    <div class="el-timeline-item__tail"></div>
    <!-- 节点/图标 -->
    <div v-if="!$slots.dot" class="el-timeline-item__node" >
      // 图标
    </div>
    <div v-if="$slots.dot" class="el-timeline-item__dot">
      <slot name="dot"></slot>
    </div>
    <!-- 信息内容 -->
    <div class="el-timeline-item__wrapper">
      // 内容
    </div>
  </li>
</template>  
```

渲染效果如下： ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/917ba6b27d084edc93b947e59ed14fb0\~tplv-k3u1fbpfcp-watermark.image?)

上一小节中介绍了根组件定义了样式用于设置末节点元素的轴线不显示。

```css
.el-timeline .el-timeline-item:last-child .el-timeline-item__tail {
  display: none;
}
```

**轴线**

类名`el-timeline-item__tail`的div元素使用绝对布局，在节点元素内部居左`left: 4px`，并通过定义`border-left` `height`样式属性来实现垂直轴线效果。

```css
.el-timeline-item__tail {
  position: absolute;
  left: 4px;
  height: 100%;
  border-left: 2px solid #e4e7ed;
}
```

**节点**

时间轴节点默为圆形实心点，颜色`#e4e7ed`。

* 属性`type`用于设置节点类型主题，生成不同颜色样式类`el-timeline-item__node--[primary/success/warning/danger/info]`。
* 属性`size` 用于设置预设节点尺寸(包含偏移量)，生成`el-timeline-item__node--[normal/large]`。
* 属性 `color` 用于元素的内联样式 `backgroundColor: color`， 设置节点颜色。
* 设置属性 `icon`值传入icon 类名就可直接直接使⽤图标。
* 提供了具名 `dot` 插槽用于自定义节点。
*

```html
<!-- 实心圆点/图标 -->
<div v-if="!$slots.dot"
  class="el-timeline-item__node"
  :class="[
    `el-timeline-item__node--${size || ''}`,
    `el-timeline-item__node--${type || ''}`
  ]"
  :style="{
    backgroundColor: color
  }"
>
  <i v-if="icon"
    class="el-timeline-item__icon"
    :class="icon"
  ></i>
</div>
<!-- 自定义 -->
<div v-if="$slots.dot" class="el-timeline-item__dot">
  <slot name="dot"></slot>
</div>
```

时间轴节点也是采用绝对布局，元素的尺寸和布局偏移在样式类`el-timeline-item__node--normal`和`el-timeline-item__node--large`定义。

```css
.el-timeline-item__node {
  position: absolute;
  background-color: #e4e7ed;
  border-radius: 50%; 
}
// 尺寸及布局偏移
.el-timeline-item__node--normal {
  left: -1px;
  width: 12px;
  height: 12px;
}
.el-timeline-item__node--large {
  left: -2px;
  width: 14px;
  height: 14px;
}
```

**信息内容**

时间轴内容包含时间戳和信息，通过属性`placement`可设置时间戳位置 `top / bottom`。

信息内容使用匿名插槽渲染。时间戳位置没有使用绝对布局或flex布局。当设置值`top`时，时间戳内容在信息内容之前；设置值`bottom`时，信息内容就会在时间戳内容之前。浏览器渲染时会遵守html排列顺序。

```html
<!-- 时间轴内容 -->
<div class="el-timeline-item__wrapper">
  <!-- 时间戳位置 top-->
  <div v-if="!hideTimestamp && placement === 'top'"
    class="el-timeline-item__timestamp is-top">
    {{timestamp}}
  </div>
  <!-- 内容 -->
  <div class="el-timeline-item__content">
    <slot></slot>
  </div>
  <!-- 时间戳位置 bottom-->
  <div v-if="!hideTimestamp && placement === 'bottom'"
    class="el-timeline-item__timestamp is-bottom">
    {{timestamp}}
  </div>
</div>
```

### 样式实现

组件样式源码 `packages\theme-chalk\src\timeline.scss`、`packages\theme-chalk\src\timeline-item.scss`使用混合指令嵌套生成组件样式。

```scss
// packages\theme-chalk\src\timeline.scss
// 生成 .el-timeline
@include b(timeline) {
  // ...
  
  // .el-timeline .el-timeline-item:last-child .el-timeline-item__tail
  & .el-timeline-item:last-child {
    & .el-timeline-item__tail {
      // ...
    }
  }
} 


// packages\theme-chalk\src\timeline-item.scss
// 生成 .el-timeline-item 
@include b(timeline-item) {
  // ... 

  // 生成  .el-timeline-item__wrapper
  @include e(wrapper) {
    // ...
  }
  // 生成 .el-timeline-item__tail
  @include e(tail) {
    // ...
  }
  // 生成 .el-timeline-item__icon
  @include e(icon) {
    // ...
  }
  // 生成 .el-timeline-item__node
  @include e(node) {
    // ...
    // 生成 .el-timeline-item__node--normal
    @include m(normal) {
      // ...
    }
    // 生成  .el-timeline-item__node--large   
    @include m(large) {
      // ...
    }
    // 生成 .el-timeline-item__node--[primary/success/warning/danger/info]
    @include m(primary) {
      // ...
    } 
    // success/warning/danger/info ....
    
  }
  // 生成 .el-timeline-item__dot
  @include e(dot) {
    // ...
  }
  // 生成 .el-timeline-item__content 
  @include e(content) {
    // ...
  }
  // 生成 .el-timeline-item__timestamp
  @include e(timestamp) {
    // ...
    // 生成 .el-timeline-item__timestamp.is-top 
    @include when(top) {
      // ...
    }
    // 生成 .el-timeline-item__timestamp.is-bottom
    @include when(bottom) {
      // ...
    }
  }
}
```
