# 3.6 Avatar 头像

### 0x00 简介

组件`Avatar`用来代表用户或事物，支持图片、图标或字符等类型。

本文将深入分析组件 `Avatar` 源码，剖析其实现原理，耐心读完，相信会对您有所帮助。 [组件文档 Avatar](https://element.eleme.cn/#/zh-CN/component/avatar)

`packages/avatar/src/icon.vue` 文件是组件源码实现。 [github 源码 main.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/avatar/src/main.vue)

### 0x01 组件源码

从源码可知组件没有使用**template**  来创建 HTML，而是使用具有 JavaScript 的完全编程的能力的**渲染函数**，它比模板更接近编译器。

```js
<script>
export default {
  name: 'ElAvatar',
  // 组件属性prop
  props: {
    // ...
  },
  // 数据属性
  data() {
    // ...
  },
  // 计算属性
  computed: {
    // 组件动态 class
    avatarClass() {
       // ...
    }
  },
  methods: {
    // 加载失败处理方法
    handleError() {
      // ...
    },
    // 生成图标、图片或者字符等不同类型元素VNode
    renderAvatar() {
      // ...
    }
  },
  // 渲染虚拟DOM
  render() {
    // ...
  }
};
</script>
```

#### attributes 属性

组件定义了 8 个 `prop`。

```js
// 组件属性prop
  props: {
    size: {
      type: [Number, String],
      validator(val) {
        if (typeof val === 'string') {
          return ['large', 'medium', 'small'].includes(val);
        }
        return typeof val === 'number';
      }
    },
    shape: {
      type: String,
      default: 'circle',
      validator(val) {
        return ['circle', 'square'].includes(val);
      }
    },
    icon: String,
    src: String,
    alt: String,
    srcSet: String,
    error: Function,
    fit: {
      type: String,
      default: 'cover'
    }
  },
```

各属性详情描述如下：

| 参数     | 说明                                | 类型            | 可选值                                       | 默认值    |
| ------ | --------------------------------- | ------------- | ----------------------------------------- | ------ |
| icon   | 设置头像的图标类型，参考 Icon 组件              | string        |                                           |        |
| size   | 设置头像的大小                           | number/string | number / large / medium / small           | large  |
| shape  | 设置头像的形状                           | string        | circle / square                           | circle |
| src    | 图片头像的资源地址                         | string        |                                           |        |
| srcSet | 以逗号分隔的一个或多个字符串列表表明一系列用户代理使用的可能的图像 | string        |                                           |        |
| alt    | 描述图像的替换文本                         | string        |                                           |        |
| fit    | 当展示类型为图片的时候，设置图片如何适应容器框           | string        | fill / contain / cover /none / scale-down | cover  |

**icon**

因为组件图标内部使用 `<i class='iconNanme' />`形式，所以属性 `icon` 格式应为 `el-icon-[iconName]`。

**size**

当组件未设置 `size`属性时，组件 prop `size` 值为 `undefined`, 因为组件样式 `el-avatar` 定义了 `width`、 `height`、 `line-height` 等属性， 跟样式 `el-avatar--large` 中的定义相同，所以等效于 `size` 设置了 `large`。

若`size`字符串值不匹配下列字符串中的一个 `large / medium / small`，此时 `size`值为传入字符串内容，添加一个未定义的 class，组件大小为默认。

```
<el-avatar size="x-large"> </el-avatar>
//生成DOM  el-avatar--x-large 未在样式中定义，组件大小使用 .el-avatar 规则
<span class="el-avatar el-avatar--x-large el-avatar--circle"></span>
```

**shape**

`shape`设置了默认值 `circle`， 若字符串值不匹配下列字符串中的一个 `circle / square`，此时 `shape`值为传入字符串内容，添加一个未定义的 class，组件大小为默认。

```html
<el-avatar shape="dot"></el-avatar>
//生成DOM el-avatar--dot 未在样式中定义，组件样式使用 .el-avatar 规则
<span class="el-avatar el-avatar--dot"></span>
```

**srcSet**

`<Image>` 元素的  `srcset`属性的值是一个字符串，用来定义一个或多个图像候选地址，以  `,`分割，每个候选地址将在特定条件下得以使用。候选地址包含图片  URL 和一个可选的宽度描述符和像素密度描述符，该候选地址用来在特定条件下替代原始地址成为`src` 的属性。

**fit**

只有使用图片展示类型时， 使用  `fit`  属性定义才会生效，会在  `<Image>` 元素中添加内联样式`{object-fit:fill}`,各属性值描述：

* `contain` 被替换的内容将被缩放，以在填充元素的内容框时保持其宽高比。 整个对象在填充盒子的同时保留其长宽比，因此如果宽高比与框的宽高比不匹配，该对象将被添加“[黑边](https://zh.wikipedia.org/wiki/%E9%BB%91%E9%82%8A)”。
* `cover` 被替换的内容在保持其宽高比的同时填充元素的整个内容框。如果对象的宽高比与内容框不相匹配，该对象将被剪裁以适应内容框。
* `fill` 被替换的内容正好填充元素的内容框。整个对象将完全填充此框。如果对象的宽高比与内容框不相匹配，那么该对象将被拉伸以适应内容框。
* `none` 被替换的内容将保持其原有的尺寸。
* `scale-down` 内容的尺寸与  `none`  或  `contain`  中的一个相同，取决于它们两个之间谁得到的对象尺寸会更小一些。

#### 计算属性

计算属性 `avatarClass` 根据大小、图标、形状等设置动态添加样式，生成组件的样式列表。

```js
// 组件动态 class
avatarClass() {
  const { size, icon, shape } = this;
  // 默认组件容器样式
  let classList = ['el-avatar'];
  // size 指定large / medium / small
  if (size && typeof size === 'string') {
    classList.push(`el-avatar--${size}`);
  }
  // icon 设置
  if (icon) {
    classList.push('el-avatar--icon');
  }
  // shape 默认 circle
  if (shape) {
    classList.push(`el-avatar--${shape}`);
  }
  // 拼接
  return classList.join(' ');
}
```

由代码可知上文提到 `size`、 `shape` 属性值只要定义了，就会生成对应 class,没有进行无效检查。

#### error 事件(应为属性)

当组件使用图片类型时，若图像加载过程中发生错误时被触发 `onError` 事件， `<img onError={this.handleError} />` ， 即调用 `handleError` 方法。

```js
 // 数据属性
  data() {
    return {
      isImageExist: true  // 图片是否存在
    };
  },
  methods: {
    // 加载失败处理方法
    handleError() {
      const { error } = this;
      const errorFlag = error ? error() : undefined; // 错误标记
      // 错误标记不为 false 时，判定图片不存在
      if (errorFlag !== false) {
        this.isImageExist = false;
      }
    },
  }
```

根据 `handleError()` 方法定义可知，若 `error` 属性（类型为 `Function`）未定义, `errorFlag` 值将始终为 `undefined`,则更新 `isImageExist` 为 `false`,隐藏 `<Image>` 元素。展示该组件下包裹的子组件--展示一张备用图片。

```html
<el-avatar :size="60" src="https://empty">
  <img
    src="https://cube.elemecdn.com/e/fd/0fc7d20532fdaf769a25683617711png.png"
  />
</el-avatar>
```

官方提供 **图片加载失败的 fallback 行为** 示例代码。事件 `error` 没有任何效果，将`@error="errorHandler"` 改成 `:error="errorHandler"`,然后修改 `errorHandler()` 方法的返回值为 `false`;

```html
<template>
  <div class="demo-type">
--    <el-avatar :size="60" src="https://empty" @error="errorHandler">
++    <el-avatar :size="60" src="https://empty" :error="errorHandler">
      <img src="https://cube.elemecdn.com/e/fd/0fc7d20532fdaf769a25683617711png.png"/>
    </el-avatar>
  </div>
</template>
<script>
  export default {
    methods: {
      errorHandler() {
--        return true
++        return false
      }
    }
  }
</script>
```

此时 `error` 等于 `errorHandler`， `errorFlag` 值为 `false`, `isImageExist` 仍是 `true` , `<Image>`显示裂图效果，效果对比如下。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c48b1c9ecce439bb2c855f246831481\~tplv-k3u1fbpfcp-watermark.image)

#### 渲染函数 & JSX

组件使用渲染函数来渲染虚拟 DOM。因为使用 `JSX`语法，代码更加深入底层，可以更好的控制交互细节。

> vue 中 `h`  作为  `createElement`  的别名。在含有 JSX 的任何方法自动注入  `const h = this.$createElement`，`render()` 等效于 `render(h)`。

组件将创建一个 `<span>` 元素 VNode，动态添加 class，设置组件大小。调用方法 `renderAvatar()` 生成图标、图片或者字符等不同类型元素 VNode。

```js
// 渲染虚拟DOM
render() {
    const { avatarClass, size } = this;
    // size 为数值时 设置内联样式 覆盖默认尺寸
    const sizeStyle = typeof size === 'number' ? {
      height: `${size}px`,
      width: `${size}px`,
      lineHeight: `${size}px`
    } : {};

    return (
      <span class={ avatarClass } style={ sizeStyle }>
        {
          // 生成图标、图片或者字符等不同类型元素VNode
          this.renderAvatar()
        }
      </span>
    );
}
```

`renderAvatar()` 按照 图片类型`<Image>`、图标、插槽（自定义头像展示内容）等类型先后顺序进行判断，只会创建一种类型的元素内容。

```js
// 生成图标、图片或者字符等不同类型元素VNode
renderAvatar() {
  const { icon, src, alt, isImageExist, srcSet, fit } = this;
  // 优先创建图片类型
  if (isImageExist && src) {
    return <img
      src={src}
      onError={this.handleError}
      alt={alt}
      srcSet={srcSet}
      style={{ 'object-fit': fit }}/>;
  }
  // 其次图标类型
  if (icon) {
    return (<i class={icon} />);
  }
  // 最后插槽自定义
  return this.$slots.default;
}
```

### 0x02 组件样式

#### src/avatar.scss

组件样式源码 `packages\theme-chalk\src\avatar.scss` 使用混合指令 `b`、 `m` 嵌套生成组件样式。

```

// 生成 .el-avatar
@include b(avatar) {
  // ...

  // 生成 .el-avatar > img
  >img {
    // ...
  }

  // 生成 .el-avatar--circle
  @include m(circle) {
    // ...
  }

  // 生成 .el-avatar--square
  @include m(square) {
    // ...
  }

  // 生成 .el-avatar--icon
  @include m(icon) {
    // ...
  }

  // 生成 .el-avatar--large
  @include m(large) {
   // ...
  }

  // 生成 .el-avatar--medium
  @include m(medium) {
    // ...
  }

  // 生成 .el-avatar--small
  @include m(small) {
    // ...
  }
}
```

上文中讲到 `size`属性默认值`large`时，提到`el-avatar`  定义了  `width`、 `height`、 `line-height`  等属性， 跟样式  `el-avatar--large`  中的定义相同。

```scss
// .el-avatar
@include b(avatar) {
  // ...
  width: $--avatar-large-size;
  height: $--avatar-large-size;
  line-height: $--avatar-large-size;
  font-size: $--avatar-text-font-size;
}
// .el-avatar--large
@include m(large) {
  width: $--avatar-large-size;
  height: $--avatar-large-size;
  line-height: $--avatar-large-size;
}
```

#### lib/avatar.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\avatar.scss`，内容格式如下。

```css
.el-avatar {
  // ...
  width: 40px;
  height: 40px;
  line-height: 40px;
  font-size: 14px;
}
.el-avatar > img {
  // ...
}
.el-avatar--circle {
  // ...
}
.el-avatar--square {
  // ...
}
.el-avatar--icon {
  // ...
}
.el-avatar--large {
  width: 40px;
  height: 40px;
  line-height: 40px;
}
.el-avatar--medium {
  // ...
}
.el-avatar--small {
  // ...
}
```

### 0x03 📚 参考

[“渲染函数 & JSX”，vuejs.org](https://cn.vuejs.org/v2/guide/render-function.html)\
[“srcset”，MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLImageElement/srcset)\
[“object-fit”，MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/object-fit)\
[“Babel Preset JSX”](https://github.com/vuejs/jsx#installation)\
[“vuejs/babel-plugin-transform-vue-jsx”](https://github.com/vuejs/babel-plugin-transform-vue-jsx)
