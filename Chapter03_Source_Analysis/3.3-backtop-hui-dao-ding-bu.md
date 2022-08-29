# 3.3 Backtop 回到顶部

### 0x00 简介

本文将深入分析组件 `Backtop` 源码，剖析其实现原理，耐心读完，相信会对您有所帮助。 [组件文档 Backtop](https://element.eleme.cn/#/zh-CN/component/backtop)

`packages/backtop/src/main.vue` 文件是组件源码实现。 [github 源码 main.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/backtop/src/main.vue)

### 0x01 DOM 结构

从模板内容看出组件就是一个使用了内置过渡动画组件 `transition`的`div`,同时支持 `slot` 功能，默认值为 名为 `el-icon-caret-top`的 `Icon` 图标。

```js
<template>
  <transition name='el-fade-in'>
    <div
      v-if='visible'
      @click.stop='handleClick'
      :style='{
        right: styleRight,
        bottom: styleBottom,
      }'
      class='el-backtop'
    >
      <slot>
        <el-icon name='caret-top'></el-icon>
      </slot>
    </div>
  </transition>
</template>
```

组件件 `transition` 实现淡入`fade-in`效果，`name`用于自动生成 CSS 过渡类名，将自动拓展为  `.el-fade-in-enter`，`.el-fade-in-enter-active`  等。

关于组件`transition` 过度样式，项目在`packages\theme-chalk\src\common\transition.scss`定义不同的进入和离开动画 ，设置持续时间和动画函数 。

`div` 元素 `class` 值为`el-backtop`，其可见性由属性 `visible` 进行控制，定义了`click`事件（ 事件修饰符 `stop` 用于阻止单击事件继续传播），使用计算属性构建内联样式，控制组件显示位置距离页面右边距、底部距离。

### 0x02 attributes 属性

组件 `prop` 提供了四个属性，指定验证要求、默认值等设置。

```js
props: {
    visibilityHeight: {
      type: Number,
      default: 200
    },
    target: [String], // 字符串
    right: {
      type: Number,
      default: 40
    },
    bottom: {
      type: Number,
      default: 40
    }
},
```

其中 `target` 类型是字符串， 传入触发滚动的对象的元素选择器，用于对象查找。若元素选择器查找不到，会提示报错 `target is not existed: ${this.target}`。

属性参数具体描述如下：

| 参数                | 说明                | 类型     | 默认值 |
| ----------------- | ----------------- | ------ | --- |
| target            | 触发滚动的对象           | string |     |
| visibility-height | 滚动高度达到此参数值才出现     | number | 200 |
| right             | 控制其显示位置, 距离页面右边距  | number | 40  |
| bottom            | 控制其显示位置, 距离页面底部距离 | number | 40  |

基于传入的 `right` 和 `bottom`属性值，计算属性 `styleBottom` 和 `styleRight` 返回`${number}px`格式化内容，用于组件的行内样式构建。

```js
computed: {
    styleBottom() {
      return `${this.bottom}px`;
    },
    styleRight() {
      return `${this.right}px`;
    }
},
```

### 0x03 组件生命周期

下面将分析组件的生命周期。

```js
<script>
// 引入节流函数
import throttle from 'throttle-debounce/throttle';

const cubic = (value) => Math.pow(value, 3);
const easeInOutCubic = (value) =>
  value < 0.5 ? cubic(value * 2) / 2 : 1 - cubic((1 - value) * 2) / 2;

export default {
  name: 'ElBacktop',
  // ...

  // 数据 property
  data() {
    return {
      el: null, // 触发滚动的对象
      container: null,  // 触发滚动的对象
      visible: false  // 组件是否可见
    };
  },

  mounted() {
    // 组件初始化
    this.init();
    // 判断组件是否可见
    this.onScroll();
    // 监听scroll使用节流函数（throttle）
    this.throttledScrollHandler = throttle(300, this.onScroll);
    // 添加scroll事件监听
    this.container.addEventListener('scroll', this.throttledScrollHandler);
  },

  methods: {
    // 组件初始化
    init() {
      // ...
    },
    // 判断组件是否可见
    onScroll() {
      // ...
    },
    // 点击按钮触发的事件
    handleClick(e) {
      // ...
    },
    // 返回页面顶部的操作
    scrollToTop() {
      // ...
    }
  },
  // 实例销毁之前调用
  beforeDestroy() {
    // 删除scroll事件监听
    this.container.removeEventListener('scroll', this.throttledScrollHandler);
  }
};
</script>
```

#### mounted()

组件实例被挂载后，依次调用 `init()` 、`onScroll()`、添加 `onScroll` 事件监听。

#### init()

获取触发滚动的对象，默认将`document`作为触发滚动的对象。

若属性`target`有传入值，则从中查找匹配的元素，找到后将其更新为触发滚动的对象。若找不到抛出异常`target is not existed`。

```js
init() {
  // 载入的网页 DOM树
  this.container = document;
  // 获取文档对象（document）的根元素的只读属性-根元素<html>
  this.el = document.documentElement;
  // 若指定 target
  if (this.target) {
    // 引用的querySelector()方法查找匹配HTMLElement对象。
    // 如果找不到查找匹配项，则返回null。
    this.el = document.querySelector(this.target);
    // 找不到匹配项 抛出异常 target is not existed
    if (!this.el) {
      throw new Error(`target is not existed: ${this.target}`);
    }
    // 匹配的元素赋值给 container
    this.container = this.el;
  }
},
```

#### onScroll()

根据触发滚动的对象垂直滚动的像素数(scrollTop),判断是否超过属性`visibilityHeight`数值，若超过则将属性`visible`值设为 `true`,此时组件在页面可见。

```js
onScroll() {
  // 获取元素的内容垂直滚动的像素数。
  const scrollTop = this.el.scrollTop;
  // 滚动高度达到此参数值  操作按钮才出现
  this.visible = scrollTop >= this.visibilityHeight;
}
```

#### 事件监听

为触发滚动的对象添加 `onScoll`事件监听，当页面滚动时触发事件调用 `onScroll()`，用于组件的显示或隐藏。

```js
// 监听scroll使用节流函数（throttle）
this.throttledScrollHandler = throttle(300, this.onScroll);
// 添加scroll事件监听
this.container.addEventListener("scroll", this.throttledScrollHandler);
```

在 `onScroll()`方法调用上添加了节流函数， `onScroll()`方法 300 毫秒只执行一次。主要是`scroll`事件会被频繁地触发，但有些时候并不希望在事件持续触发的过程中那么频繁地去执行函数。

#### beforeDestroy

实例销毁之前调用，删除触发滚动的对象上`scroll`事件监听。

```js
beforeDestroy() {
    // 删除scroll事件监听
    this.container.removeEventListener('scroll', this.throttledScrollHandler);
}
```

### 0x04 events 事件

组件提供了 `click` 事件 。前面内容介绍 `@click.stop='handleClick'` 可知，点击组件后调用`handleClick`方法。

#### handleClick(e)

首先调用`scrollToTop()`返回页面顶部，然后调用 `this.$emit('click', e)` 触发当前实例上的`click`事件 。

```js
handleClick(e) {
  // 返回页面顶部
  this.scrollToTop();
  // 触发当前实例上的事件
  this.$emit('click', e);
},
```

#### scrollToTop()

该方法通过设置`target`对象的 `scrollTop`值，实现返回页面顶部效果。为了效果体验更好使用了 **缓动函数** 和 **requestAnimationFrame** 。

```js
scrollToTop() {
  // 指定对象
  const el = this.el;
  // 点击按钮时间戳 UTC毫秒数
  const beginTime = Date.now();
  console.log(beginTime);
  // 获取元素的内容垂直滚动的像素数。
  const beginValue = el.scrollTop;
  const rAF =
    window.requestAnimationFrame || ((func) => setTimeout(func, 16));

  const frameFunc = () => {
    const progress = (Date.now() - beginTime) / 500;  //设置速率 500 毫秒内返回页面顶部
    if (progress < 1) {
      el.scrollTop = beginValue * (1 - easeInOutCubic(progress));
      rAF(frameFunc);
    } else {
      // 超过 500 直接返回页面顶部
      el.scrollTop = 0;
    }
  };
  rAF(frameFunc);
}
```

#### 缓动函数

**缓动函数**  自定义参数随时间变化的速率。缓动函数允许将数学公式应用到动画。

> 现实生活中，物体并不是突然启动或者停止，当然也不可能一直保持匀速移动。就像我们打开抽屉的过程那样，刚开始拉的那一下动作很快，但是当抽屉被拉出来之后我们会不自觉的放慢动作。或是掉落在地板上的物体，一开始下降的速度很快，接着就会在地板上来回反弹直到停止。

```js
const cubic = (value) => Math.pow(value, 3);
const easeInOutCubic = (value) =>
  value < 0.5 ? cubic(value * 2) / 2 : 1 - cubic((1 - value) * 2) / 2;
```

等效的 `TypeScript` 代码实现。 变量 x 表示 0（动画开始）到 1（动画结束）范围内的值。

```js
function easeInOutCubic(x: number): number {
  return x < 0.5 ? 4 * x * x * x : 1 - pow(-2 * x + 2, 3) / 2;
}
```

在 CSS 中，使用 transition 和 animation 属性实现。效果详见  [cubic-bezier.com](http://cubic-bezier.com/#0.65,0,0.35,1) 。

```
.block {
    transition: transform 0.6s cubic-bezier(0.65, 0, 0.35, 1);
}
```

缓动函数用于点击组件触发`scrollToTop`方法，回滚到视口顶部时速率,使其看起来更加真实。

```js
scrollToTop() {
    // ...
    el.scrollTop = beginValue * (1 - easeInOutCubic(progress));
    // ...
}
```

#### requestAnimationFrame()

**window.requestAnimationFrame()**  告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。从而达到经过浏览器优化，动画更流畅。

`requestAnimationFrame()` 递归调用实现。

```js
const rAF = window.requestAnimationFrame || ((func) => setTimeout(func, 16));

const frameFunc = () => {
  // ...
  rAF(frameFunc);
  // ...
};
rAF(frameFunc);
```

### 0x05 theme-chalk 样式实现

#### BEM 命名规范

组件样式使用 **BEM** 命名规范，`BEM` 的命名范约`[block-name]__[element-name]--[modifier-name]`，也就是 `block(块) + element(元素) + modifier(修饰符)`。

```css
.block {
}
.block__element {
}
.block--modifier {
}
```

* `.block`  代表了更高级别的抽象或组件。
* `.block__element`  代表.block 的后代，用于形成一个完整的.block 的整体。
* `.block--modifier`代表.block 的不同状态或不同版本。

`BEM`命名让其有更多的描述和更加清晰的结构，从其名字可以知道某个标记的含义作用。让前端代码更容易阅读和理解，更容易协作，更容易控制，更加健壮和明确，而且更加严密。

#### src/backtop.scss

组件样式源码 `packages\theme-chalk\src\backtop.scss` 使用 `scss` 的混合指令（`mixin`）生成组件样式, 利用混合器，可以很容易地在样式表的不同地方共享样式。由于组件的 `DOM` 结构比较简单,只使用了 `@mixin b($block)` 用于生成 `block` 。

```scss
@import "mixins/mixins";
@import "common/var";

@include b(backtop) {
  position: fixed;
  background-color: $--backtop-background-color;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  color: $--backtop-font-color;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.12);
  cursor: pointer;
  z-index: 5;

  // 父选择器的标识符&
  &:hover {
    background-color: $--backtop-hover-background-color;
  }
}
```

#### src/common/var.scss

`packages\theme-chalk\src\common\var.scss` 定义了全局公共样式变量。

```scss
/* Backtop
--------------------------*/
/// color||Color|0
$--backtop-background-color: $--color-white !default;
/// color||Color|0
$--backtop-font-color: $--color-primary !default;
/// color||Color|0
$--backtop-hover-background-color: $--border-color-extra-light !default;

/* Color
-------------------------- */
/// color|1|Brand Color|0
$--color-primary: #409eff !default;
/// color|1|Background Color|4
$--color-white: #ffffff !default;
/// color|1|Border Color|3
$--border-color-extra-light: #f2f6fc !default;
```

#### src/mixins/config.scss

`packages\theme-chalk\src\mixins\config.scss` 定义了用于 BEM 命名规范的变量。

```scss
$namespace: "el"; // 命名空间  element缩写
$element-separator: "__"; // 定义元素分隔符
$modifier-separator: "--"; // 定义状态分隔符
$state-prefix: "is-"; // 状态前缀
```

#### src/mixins/mixins.scss

`packages\theme-chalk\src\mixins\mixins.scss` 中的定义 `b`、`e`、`m`方法。

```scss
/* BEM
 -------------------------- */
@mixin b($block) {
  // 全局参数声明 组件的class命名
  $B: $namespace + "-" + $block !global;

  .#{$B} {
    @content;
  }
}
@mixin e($element) {
  // ...
}
@mixin m($modifier) {
  // ...
}
```

执行 `@include b(backtop)`，声明全局变量 `$B: $namespace+'-'+$block !global;` =>`$B:el-backtop !global;`

使用 `#{}`插值直接使用变量。

```css
$B: el-backtop !global;

.#{$B} {
  @content;
}

// 编译后为
.el-backtop {
  // ...
}
```

使用 `@content` 向混合样式中导入内容,最终编译效果

```scss
.el-backtop {
  position: fixed;
  background-color: #fff;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  color: #409eff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.12);
  cursor: pointer;
  z-index: 5;

  &:hover {
    background-color: #f2f6fc;
  }
}
```

#### lib/backtop.css

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\backtop.css`，内容格式如下。

```css
.el-backtop {
  position: fixed;
  background-color: #fff;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  color: #409eff;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  -webkit-box-pack: center;
  -ms-flex-pack: center;
  justify-content: center;
  font-size: 20px;
  -webkit-box-shadow: 0 0 6px rgba(0, 0, 0, 0.12);
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.12);
  cursor: pointer;
  z-index: 5;
}
.el-backtop:hover {
  background-color: #f2f6fc;
}
```

### 0x06 使用示例

组件在使用时需注意 `target` 触发滚动的对象的 CSS 属性。

```js
<template>
  <div id="app" class="app">
    <div style="min-height:2400px">
      <el-backtop target="#app" @click="top"
        ><div
          style="{
        height: 100%;
        width: 100%;
        background-color: #f2f5f6;
        box-shadow: 0 0 6px rgba(0,0,0, .12);
        text-align: center;
        line-height: 40px;
        color: #1989fa;
      }"
        >
          UP
        </div>
      </el-backtop>
    </div>
  </div>
</template>

<style>
.app {
  height: 100vh;
  overflow-x: hidden;
}
</style>
```

### 0x07 📚 参考

['视口 viewport',MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Viewport\_concepts)\
['requestAnimationFrame',MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Viewport\_concepts)\
["缓动函数",easings.net](https://easings.net/cn)\
['BEM',bemcss.com](https://www.bemcss.com/)\
['BEM',segmentfault](https://segmentfault.com/a/1190000000391762)
