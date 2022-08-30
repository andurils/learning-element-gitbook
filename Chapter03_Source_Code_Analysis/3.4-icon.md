# 3.4 Icon 图标

### 0x00 简介

本文将深入分析组件 `Icon` 源码，剖析其实现原理，耐心读完，相信会对您有所帮助。 [组件文档 Icon](https://element.eleme.cn/#/zh-CN/component/icon)

`packages/icon/src/icon.vue` 文件是组件源码实现。 [github 源码 icon.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/icon/src/icon.vue)

### 0x01 组件源码

组件 `Icon` 的源码应是最简单的。封装了**字体图标**的使用方法。提供了属性 `name`,用于指定使用的图标名称。

```js
<template>
  <i :class="'el-icon-' + name"></i>
</template>

<script>
  export default {
    name: 'ElIcon',

    props: {
      name: String
    }
  };
</script>
```

组件使用 `<el-icon name="iconName"></el-icon>` 等同于 `<i class="el-icon-iconName"></i>`，所以使用图标功能有两种实现方式。**官方更推荐后者也就是直接使用字体图标。**

在 `Icon`组件示例中直接使用了后者,没有调用`el-icon`组件 。

```html
<i class="el-icon-edit"></i>
<i class="el-icon-share"></i>
<i class="el-icon-delete"></i>
```

支持图标功能的其他组件内部实现大多使用字体图标方式，而不是调用`el-icon`组件,下面以图标按钮为例。

```js
// 图标按钮 使用方式
<el-button type="primary" icon="el-icon-search">搜索</el-button>

// button 源码  packages\button\src\button.vue
<template>
  <button class="el-button" >
    ...
    <i class="el-icon-loading" v-if="loading"></i>
    <i :class="icon" v-if="icon && !loading"></i>
    ...
  </button>
</template>
```

### 0x02 字体图标样式

#### 📁 src\fonts

用于存放字体图标使用字体文件。

`element-icons.ttf` 是 **TrueType 格式**的字体文件。 Windows 和 Mac 上常见的字体格式，是一种原始格式，因此它并没有为网页进行优化处理。\
`element-icons.woff` 是 **Web Open Font** 的字体文件是一个开放的 TrueType/OpenType 的压缩版，同时支持元数据包的分离。 **Web 字体中最佳格式,针对网页进行特殊优化**。

#### 📃 src\icon.scss

**@font-face 自定义字体库**

组件样式源码 `packages\theme-chalk\src\icon.scss` 首先自定义图标字体库。

```scss
// CSS @规则 指定自定义字体
@font-face {
  font-family: "element-icons";
  src: url("#{$--font-path}/element-icons.woff") format("woff"), /* chrome, firefox */
      url("#{$--font-path}/element-icons.ttf") format("truetype"); /* chrome, firefox, opera, Safari, Android, iOS 4.2+*/
  font-weight: normal;
  font-display: $--font-display;
  font-style: normal;
}
```

* `font-family`： 指定自定义字库名称为 `element-icons`。
* `src`：设置字体的加载路径和格式 `[ <url> [ format( <string> ) ]`，通过逗号分隔多个加载路径和格式。
* `[ <url> [ format( <string> ) ]`： `url`指定自定义的字体的存放路径 ；`string`指定自定义的字体的格式，主要用来帮助浏览器识别。 变量`$--font-path`值为 `fonts`。
* `font-weight`： 定义字体的粗细。`normal` 默认值定义标准的字符。
* `font-style`： 定义字体样式使用斜体、倾斜或正常字体。`normal` 默认值正常字体。
* `font-display`：决定了一个`@font-face`  在不同的下载时间和可用时间下是如何展示的。变量 `$--font-display` 值为`auto`,字体显示策略由用户代理定义。

**el-icon-element 样式**

通过 CSS 选择器，对`class`以`el-icon-`开始 或者 包含`el-icon-`的元素设置样式。通过`font-family`引用自定义字体`font-family: element-icons !important;`。

```scss
[class*=" el-icon-"],
[class^="el-icon-"] {
  font-family: element-icons !important;
  // ...
}

// ...

// margin 左右设置
.el-icon--right {
  margin-left: 5px;
}
.el-icon--left {
  margin-right: 5px;
}
```

**el-icon-iconName:before 图标设置**

显示图标使用 CSS 伪元素选择器`::before`进行添加，自身的`content`属性是对应的图标代码。 `::before` 匹配一个虚拟元素，主要被用于为当前元素增加装饰性内容的。显示的内容是其自身的`content`属性，默认是内联元素。

```scss
// 图标内容设置
.el-icon-iconName:before {
  content: "\eXXX";
}

// ...
```

前文讲到脚本 `build/bin/iconInit.js` ，解析  `packages/theme-chalk/src/icon.scss`，提取所有  `icon`  名字生成  `examples/icon.json`  图标集合文件。

```js
// build\bin\iconInit.js
// 遍历 nodes
nodes.forEach((node) => {
  // ...
  // node获取选择器名称匹配 .el-icon-iconName:before
  var reg = new RegExp(/\.el-icon-([^:]+):before/);
  // ...
});
```

`icon.json`  在官网入口文件`examples\entry.js`  中导入，挂载到  `Vue.prototype`。 用于`Icon图标`文档页生成所有的图标集合 。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8addb6b640543c1a44cd9a61c46d350\~tplv-k3u1fbpfcp-watermark.image)

**loding 旋转实现**

使用 **css animation** 实现 `loading` 图标不停旋转效果。

```scss
.el-icon-loading {
  animation: rotating 2s linear infinite;
}

@keyframes rotating {
  0% {
    transform: rotateZ(0deg);
  }
  100% {
    transform: rotateZ(360deg);
  }
}
```

#### 📃 lib\icon.scss

前文可知使用  `gulpfile.js`编译  `scss`  文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成  `packages\theme-chalk\lib\icon.css`。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0917527e22f4282808babbb8253429c\~tplv-k3u1fbpfcp-watermark.image)

### 0x04 原理图解

一图描述下使用图标时，发生了什么。

```html
<i class="el-icon-share"></i>
// 或
<el-icon name="share"></el-icon>
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1eae92fa8635463faa10456b73900773\~tplv-k3u1fbpfcp-watermark.image)

### 0x05 📚 参考

["@font-face",MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face)\
["::before",MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::before)\
["CSS selectors",W3C](https://www.w3.org/TR/2018/REC-selectors-3-20181106/#selectors)
