# 3.27 Skeleton 骨架屏

### 简介

骨架屏组件`Skeleton` 常用于在需要等待加载内容的位置提供一个占位图形组合,可以被 `Loading` 完全代替，但是在可用的场景下可以提供更好的视觉效果和用户体验。

* 网络较慢，需要长时间等待加载处理的情况下。
* 图文信息内容较多的列表/卡片中。
* 只在第一次加载数据的时候使用

本文将分析其源码实现，耐心读完，相信会对您有所帮助。🔗 [组件文档 Skeleton](https://element.eleme.cn/#/zh-CN/component/skeleton) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/blob/dev/packages/skeleton/)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。。

### 组件源码

组件的 `prop` 声明，各属性功能说明详见[官方文档skeleton#attributes](https://element.eleme.cn/#/zh-CN/component/skeleton#skeleton-attributes) 。

#### 根组件 index.vue

根组件定义了两个部分，用于加载占位和真实DOM之间的切换：

1. 用于展示骨架屏，提供具名`template`插槽用于自定义占位符。
2. 用于展示真实组件UI，使用匿名插槽。

组件使用了`v-bind="$attrs"`实现了透传Attributes，用于父子组件的Attributes 继承。

```html
// packages\skeleton\src\index.vue
<template>
  <div> 
    <template v-if="uiLoading">
      <!-- 展示骨架屏 -->
      <div :class="['el-skeleton', animated ? 'is-animated' : '', ]" v-bind="$attrs">
        <template v-for="i in count">
          <!-- 展示自定义占位符 -->
          <slot v-if="loading" name="template">
            <el-skeleton-item v-for="item in rows" variant="p"/>
          </slot>
        </template>
      </div>
    </template>
    <template v-else>
      <!-- 展示真实 UI -->
      <slot v-bind="$attrs"></slot>
    </template>
  </div>
</template>
```

骨架屏占位元素渲染通过内部属性`uiLoading`进行控制，该属性逻辑稍后详细介绍。

属性`animated` 用于生成样式类`is-animated`，控制占位元素是否动画效果。

```css
.el-skeleton.is-animated .el-skeleton__item {
   background: linear-gradient(90deg,#f2f2f2 25%,#e6e6e6 37%,#f2f2f2 63%);
   background-size: 400% 100%;
   animation: el-skeleton-loading 1.4s ease infinite
}

@keyframes el-skeleton-loading {
  0% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0 50%;
  }
}
```

#### 占位元素渲染

默认情况下组件会渲染一条记录（属性`count`默认值为1），包含四个占位元素（属性`rows`默认值为4），默认为标签`p`样式。

```html
<template v-for="i in count">
  <!-- 展示自定义占位符 -->
  <slot v-if="loading" name="template">
    <el-skeleton-item
      v-for="item in rows"
      :key="item"
      :class="{
        'el-skeleton__paragraph': item !== 1,
        'is-first': item === 1,
        'is-last': item === rows && rows > 1,
      }"
      variant="p"
    />
  </slot>
</template>
```

首行会被渲染一个长度 33% 的段首;多行时末行会被渲染一个长度 61% 的段尾。

```css
.el-skeleton__p.is-last {
  width: 61%;
}
.el-skeleton__p.is-first {
  width: 33%;
}
```

渲染效果如下图：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47b6efc7862a42c39125f52a93a0e690\~tplv-k3u1fbpfcp-watermark.image?)

当渲染多条数据时，若未未自定义占位元素，会出现相邻记录的段尾和段首会出现在一行中（因为使用了`<template>` 标签），渲染结果如下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6b504e6af054d83bd2a2341cc7e5d26\~tplv-k3u1fbpfcp-watermark.image?)

> 当需要使用骨架屏渲染列表时,才会将 `count` 设置为多条。

#### 组件生命周期&事件

组件的显示状态`uiLoading`根据属性`throttle`和`loading`传入值动态设置。 当属性 `throttle` 值大于0，开启延迟渲染占位元素，侦听器会进行不同逻辑处理。

```js
watch: {
  loading: {
    handler(loading) {
      // 直接更新加载状态
      if (this.throttle <= 0) {
        this.uiLoading = loading;
        return;
      }
      // 延迟更新加载状态
      if (loading) { 
        clearTimeout(this.timeoutHandle);
        // 延迟执行
        this.timeoutHandle = setTimeout(() => {
          this.uiLoading = this.loading;
        }, this.throttle);
      } else {
        this.uiLoading = loading;
      }
    },
    immediate: true
  }
},
data() {
  return {
    uiLoading: this.throttle <= 0 ? this.loading : false
  };
}
```

> 当 API 的请求回来的特别快，往往骨架占位刚刚被渲染，真实的数据就已经回来了，用户的界面会突然一闪，此时为了避免这种情况，就需要通过 `throttle` 属性来避免这个问题。

#### 占位元素组件 item.vue

占位元素组件`el-skeleton-item`支持多种标签 `p/text/h1/h3/text/caption/button/image/circle/rect` 的样式。

```js
// packages\skeleton\src\item.vue
<template>
  <div :class="['el-skeleton__item', `el-skeleton__${variant}`]">
    <img-placeholder v-if="variant === 'image'" />
  </div>
</template>
```

各标签样式渲染效果如下，支持样式自定义：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8faebb85d054471a328fb86b236b493\~tplv-k3u1fbpfcp-watermark.image?)

当需要渲染标签`image`样式时使用 svg 图标组件`img-placeholder`。

```html
// packages\skeleton\src\img-placeholder.vue
<template>
  <svg viewBox="0 0 1024 1024"  xmlns="http://www.w3.org/2000/svg">
    <path
      d="M64 896V128h896v768H64z m64-128l192-192 116.352 116.352L640 448l256 307.2V192H128v576z m224-480a96 96 0 1 1-0.064 192.064A96 96 0 0 1 352 288z"
    />
  </svg>
</template>
```

组件渲染效果如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9064e03f6804765ad65744905e5189c\~tplv-k3u1fbpfcp-watermark.image?)

### 样式实现

组件样式源码 `packages\theme-chalk\src\skeleton.scss` 使用混合指令嵌套生成组件样式。

```scss
// packages\theme-chalk\src\skeleton.scss

// mixin指令  动画效果 
@mixin skeleton-color() {
  // ... 
  animation: #{$namespace}-skeleton-loading 1.4s ease infinite;
}
// 定义 @keyframes  el-skeleton-loading
@keyframes #{$namespace}-skeleton-loading {
  // ... 
}
// 生成 .el-skeleton
@include b(skeleton) {
  // ... 
  
  // 生成 .el-skeleton__first-line,.el-skeleton__paragraph
  @each $unit in (first-line, paragraph) {
    @include e($unit) {
      // ... 
    }
  }
  // 生成 .el-skeleton.is-animated .el-skeleton__item
  @include when(animated) {
    .#{$namespace}-skeleton__item {
      // 混入 占位元素的动画效果
      @include skeleton-color();
    }
  }
}
```

组件样式源码 `packages\theme-chalk\src\skeleton-item.scss` 使用混合指令嵌套生成组件样式。

```scss
// packages\theme-chalk\src\skeleton-item.scss

// mixin指令 圆形样式
@mixin circle-size($size) {
  // ...
}

@include b(skeleton) {
  // 生成 .el-skeleton__item
  @include e(item) {
    // ...
  }
  // 生成  .el-skeleton__circle
  @include e(circle) {
    border-radius: 50%;
    @include circle-size($--avatar-medium-size);
    
    // 生成 .el-skeleton__circle--lg
    @include m(lg) {
      @include circle-size($--avatar-large-size);
    }
    // 生成 .el-skeleton__circle--md
    @include m(md) {
      @include circle-size($--avatar-small-size);
    }
  }
  
  // 生成 .el-skeleton__p
  @include e(p) {
    // ...
    
    // 生成 .el-skeleton__p.is-last
    @include when(last) {
      width: 61%;
    }
    // 生成 .el-skeleton__p.is-first
    @include when(first) {
      width: 33%;
    }
  }
  
  // 生成  .el-skeleton__[button/text/h1/h3/h5/caption]
  @include e(button) {
    // ...
  }
  // 省略 ...
  
  // 生成 .el-skeleton__image  
  @include e(image) {
    // ...
    
    // 生成 .el-skeleton__image svg
    svg {
      // ...
    }
  } 
}
```

按照样式声明，也可以使用标签 `h5` 样式。圆形虽然定义了不同尺寸，但是组件未提供接口。

### 📚参考&关联阅读

['透传Attributes',vuejs](https://v2.cn.vuejs.org/v2/api/#vm-attrs)
