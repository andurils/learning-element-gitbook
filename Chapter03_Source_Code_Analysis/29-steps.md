# 3.29 Steps 步骤条

### 简介

步骤条组件`Steps`是引导用户按照流程完成任务的分步导航条。当任务复杂或者存在先后关系时，将其分解成一系列步骤，从而简化任务，一半步骤不得少于 2 步。

本文将分析其源码实现，耐心读完，相信会对您有所帮助。🔗 [组件文档 Steps](https://element.eleme.cn/#/zh-CN/component/steps) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/tree/dev/packages/steps/src)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 组件源码

步骤条功能提供了两个组件：顶层组件`el-steps`和 子组件`el-step`。

各组件的 `prop` 声明，各属性功能说明详见[官方文档 Steps#attributes](https://element.eleme.cn/#/zh-CN/component/steps#steps-attributes) 。

#### 顶层组件 steps.vue

顶层组件基本上就是一个容器，包含着子组件`el-step`用到的共享状态。在`el-step`中直接通过`$parent`获取父实例，改变和同步其共享状态。

模板渲染成一个类名`el-steps`的简单div元素，同时使用匿名插槽渲染步骤元素。

* 声明`props` 用于外部传入的属性。
* 属性 `steps` 用于保存当前实例下子组件`el-step`的实例数组。
* 属性 `stepOffset` 用于设置步骤元素的间隔。
* 定义了侦听器，监听属性 `active`用于触发的自定义`change`事件，监听属性 `steps` 更新每个子组件实例步骤索引。

```js
// packages\steps\src\steps.vue
<template>
  <div
    class="el-steps"
    :class="[
       !simple && 'el-steps--' + direction,
       simple && 'el-steps--simple'
     ]">
      <slot></slot>
  </div>
</template>

<script> 
export default {  
  // props ... 
  data() {
    return {
      steps: [],
      stepOffset: 0
    };
  }, 
  watch: {
    active(newVal, oldVal) {
      this.$emit('change', newVal, oldVal);
    }, 
    steps(steps) {
      steps.forEach((child, index) => {
        child.index = index;
      });
    }
  }
};
</script> 
```

#### 步骤组件状态初始化

为了更好理解步骤组件一些状态初始化操作， 接下结合父子组件的生命周期流程进行讲解。

> 父子组件实例在创建时经历一系列的初始化步骤：`父beforeCreate -> 父created -> 父beforeMount ->子beforeCreate -> 子created -> 子beforeMount -> 子mounted-> 父mounted`。

首先，顶层组件`el-steps`初始化实例、 解析props、`data()`和侦听器等选项设置完毕。

其次，子组件`el-step`初始化实例、 解析props、`data()` 、计算属性、方法和侦听器等选项设置完毕。定义了`beforeCreate`钩子函数，当子组件实例初始化完成之后立即将该实例添加至父组件的属性`steps`数组中。

```js
// packages\steps\src\step.vue
beforeCreate() {
  this.$parent.steps.push(this);
},
```

再次，定义了`mounted`钩子函数，当子组件`el-step`被挂载之后，使用命令式的 `$watch()` 创建侦听器。用于监听属性`index`更改,执行侦听回调一次后，调用方法`unwatch()`就会停止该侦听器。

```js
// packages\steps\src\step.vue
data() {
  return {
    index: -1 // 实例数组steps中索引值
  };
},
mounted() { 
  const unwatch = this.$watch('index', val => {
    // 省略 ... 
    // 停止该侦听器
    unwatch();
  });
}
```

然后，顶层组件的属性`steps`数组长度大于0,会执行侦听回调，更新每个子组件实例的属性 `index`值，即实例在数组steps中索引值。

```js
// packages\steps\src\steps.vue
watch: { 
  steps(steps) { 
    steps.forEach((child, index) => { 
      child.index = index;
    });
  }
}
```

然后，触发子组件的`index`的属性侦听回调，此时会在子组件创建侦全局状态`active`、`processStatus`的侦听器，用于更新各个子组件的状态显示。用于触发回调方法`updateStatus`，参数是当前激活步骤的index。

```js
// packages\steps\src\step.vue
const unwatch = this.$watch('index', val => { 
  this.$watch('$parent.active', this.updateStatus, { immediate: true });
  this.$watch('$parent.processStatus', () => {
    const activeIndex = this.$parent.active;
    this.updateStatus(activeIndex);
  }, { immediate: true });

  // 停止该侦听器
  unwatch();
});
```

最后，执行方法 `updateStatus`，初始化子组件状态。因为定义了`immediate: true`，在侦听器创建时立即方法，所以会调用两次方法。

子组件有三个内部状态（隐式） `激活/已完成/未激活`，使用属性`internalStatus`表示内部状态值(显式)。

* `已完成` 当前激活步骤索引大于此步骤所在数组索引值，`internalStatus`值为属性`finishStatus`值。
* `激活` 当前激活步骤索引等于步骤所在数组索引值，若非首元素，则上一步骤元素的状态值不能为 `error`，也就是 `error`会中断步骤流程。 `internalStatus`值为属性`processStatus`值。
* `未激活` 当前激活步骤索引小于此步骤所在数组索引值，`internalStatus`值为`wait`。

根据当前子组件的索引，获取它的上一元素`prevChild`，调用`prevChild`的方法`prevChildcalcProgress`计算步骤间轴线样式，此方法稍后会详细介绍。

```js
// packages\steps\src\step.vue
updateStatus(val) {
  // 获取上一步骤
  const prevChild = this.$parent.$children[this.index - 1];

  if (val > this.index) {
    // 当前激活步骤索引大于此步骤索引值,该步骤状态为已完成
    this.internalStatus = this.$parent.finishStatus;
  } else if (val === this.index && this.prevStatus !== 'error') {
    // 上一步骤的状态是 'error'的化，之后步骤不会被激活
    this.internalStatus = this.$parent.processStatus;
  } else {
    // 未激活的状态为 'wait'
    this.internalStatus = 'wait';
  }
  // 存在上一步骤  步骤间轴线样式计算
  if (prevChild) prevChild.calcProgress(this.internalStatus); 
},
```

#### 步骤的状态

上面介绍了整个初始化的流程，也许大家会有疑惑，动态更新的属性`internalStatus`值作用是什么？

步骤组件提供了属性`status`值用于设置当前步骤的状态值；如果未设置， 就会根据属性`internalStatus`确定状态值。

计算属性`currentStatus`返回当前步骤的状态值，计算属性 `prevStatus` 返回上一步骤的状态值。

```js
props: 
  // ...
  status: String // 设置当前步骤的状态，不设置则根据 steps 确定状态
}, 
computed: {
  // 当前步骤的状态
  currentStatus() {
    return this.status || this.internalStatus;
  },
  // 上一步骤的状态
  prevStatus() {
    const prevStep = this.$parent.steps[this.index - 1];
    return prevStep ? prevStep.currentStatus : 'wait';
  },
}
```

### 步骤组件 step.vue

步骤条主要功能实现都在该组件中。

#### HTML

模板渲染成一个类名`el-step`的div元素，元素包含两部分内容

1. 用于图标、轴线的渲染。
2. 用于标题、描述的渲染。

```html
// packages\steps\src\step.vue
<template>
  <div class="el-step">
    <!-- 图标 & 轴线 -->
    <div class="el-step__head" >
      <div class="el-step__line" >
        // line
      </div> 
      <div class="el-step__icon">
        // icon
      </div>
    </div>
    <!-- 标题 & 描述 -->
    <div class="el-step__main">
      <div class="el-step__title">
        // title
      </div> 
      <div class="el-step__description">
        // description
      </div>
    </div>
  </div>
</template>
```

下图是不同状态步骤的展示效果：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2675e22009245959ad5c738e41159c5\~tplv-k3u1fbpfcp-watermark.image?)

步骤元素通过内联样式和动态类名渲染不同配置下组件样式。

```html
<div
  class="el-step"
  :style="style"
  :class="[
    !isSimple && `is-${$parent.direction}`,
    isSimple && 'is-simple',
    isLast && !space && !isCenter && 'is-flex',
    isCenter && !isVertical && !isSimple && 'is-center'
    ]">
  // ...
</div>
```

#### 根元素自定义类名

> 文档中提到当设置 `simple` 可应用简洁风格，该条件下 `align-center` 、 `direction` 、 `space` 都将失效。

接下将一一分析为什么设置会失效。

属性 `direction` 用于设置显示方向，生成类名 `is-vertical`或 `is-horizontal`。根据计算属性`isSimple`判断设置简洁风格时，只会生成类名 `is-simple`。

```js
!isSimple && `is-${$parent.direction}`,  // 生成类名 is-vertical/is-horizontal
isSimple && 'is-simple', // 生成类名 is-simple

// computed 是否简洁风格
isSimple() {
  return this.$parent.simple;
}, 
```

顶层组件中也会根据属性`simple`、`direction`生成根元素类名`el-steps--simple` 、`el-steps--vertical`或`el-steps--horizontal`。

```html
// packages\steps\src\steps.vue
<div
  class="el-steps"
  :class="[
      !simple && 'el-steps--' + direction,
      simple && 'el-steps--simple'
    ]">
    <slot></slot>
</div>
```

如果未设置间距 `space`和居中对齐 `alignCenter`，步骤条末元素会生成类名 `is-flex`用于自适应宽度。 如果设置了简洁风格，计算属性`space`中属性`space`设置无效。

```js
isLast && !space && !isCenter && 'is-flex', // 生成类名is-flex

// computed
// 是否步骤条末元素
isLast() {
  const parent = this.$parent;
  return parent.steps[parent.steps.length - 1] === this;
},
// 间隔设置
space() {
  const { isSimple, $parent: { space } } = this;
  return isSimple ? '' : space ;
},
// 标题描述是否居中对齐
isCenter() {
  return this.$parent.alignCenter;
},
```

非简洁模式和竖直方向下，设置居中对齐 `alignCenter`才会生效，生成类名`is-center`。

```js
isCenter && !isVertical && !isSimple && 'is-center' // 生成类名 is-center


// computed 是否竖直方向
isVertical() {
  return this.$parent.direction === 'vertical';
}, 
```

#### 根元素内联样式

顶层组件 `steps` 的根元素采用 flex布局。

```css
.el-steps { 
  display: flex;
}  
.el-steps--vertical {
  height: 100%; 
  flex-flow: column;
} 
```

计算属性`style`根据属性 `space` 设置`flex-basis`，指定了 flex 元素在主轴方向上的初始大小，也就是内容盒（content-box）的尺寸。相当于设置了步骤元素的 `width` 或 `height`。

> 当一个元素同时被设置了 `flex-basis` (除值为 `auto` 外) 和 `width` (或者在 `flex-direction: column` 情况下设置了`height`) , `flex-basis` 具有更高的优先级。

传入的属性 `space`值类型为 `number`时生成格式 `{20}px`。不设置根据步骤个数自动计算百分比实现自适应间距。

水平方向时，步骤末元素设置属性`max-width`值,其余设置属性`margin-right`值（属性`stepOffset`值没有相关计算或赋值，始终为 0）。

```js
//  computed 根元素样式
style: function() {
  const style = {};
  const parent = this.$parent;
  const len = parent.steps.length;

  const space = (typeof this.space === 'number'
    ? this.space + 'px' // 值为 number,生成 {20}px
    : this.space
      ? this.space
      : 100 / (len - (this.isCenter ? 0 : 1)) + '%'); // 未指定，则自适应间距

  style.flexBasis = space;
  if (this.isVertical) return style;
  if (this.isLast) {
    style.maxWidth = 100 / this.stepsCount + '%';
  } else {
    style.marginRight = -this.$parent.stepOffset + 'px';
  }
  return style;
}
```

属性 `space` 可设置值可以参考以下内容：

```css
/* 指定<'width'> */
flex-basis: 10em;
flex-basis: 3px;
flex-basis: auto;

/* 固有的尺寸关键词 */
flex-basis: fill;
flex-basis: max-content;
flex-basis: min-content;
flex-basis: fit-content;

/* 在 flex item 内容上的自动尺寸 */
flex-basis: content;

/* 全局数值 */
flex-basis: inherit;
flex-basis: initial;
flex-basis: unset;
```

下面通过运行实例对比分析下，各属性设置的渲染效果。

```html
<!-- space 未设置 -->
<el-steps :active="2" finish-status="success">
  <el-step title="步骤 1" description="这是一段很长很长很长的描述性文字"></el-step>
  <el-step title="步骤 2" description="这是一段很长很长很长的描述性文字"></el-step>
  <el-step title="步骤 3" description="这段就没那么长了"></el-step>
  <el-step title="步骤 4" description="这段就没那么长了"></el-step>
</el-steps>
<!-- space 未设置  align-center为true 居中对齐-->
<el-steps :active="2" finish-status="success" align-center>
  // ...
</el-steps>
<!-- space 100 即 100px -->
<el-steps :active="2" finish-status="success" :space="100">
  // ...
</el-steps>
```

步骤元素内容个数相同情况下，不同设置的效果如下: ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98aba78cce954f3eab95b16e6f021621\~tplv-k3u1fbpfcp-watermark.image?)

各步骤元素DOM结构如下（此处不考虑内部内容样式区别），除了第一个示例，其余每个步骤都是等分。 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e9527246c05422fbc5a43ff1db0ebaa\~tplv-k3u1fbpfcp-watermark.image?)

> 那问题来了，第一个示例中 space 未设置,组件会自适应宽度操作，那么末步骤元素发生了什么导致其宽度跟其他步骤元素不一致？

第一个示例DOM结构渲染如下，虽然都设置了`flex-basis: 33.3333%;`,但是末元素的未生效。

```html
<div class="el-steps el-steps--horizontal">
  <div class="el-step is-horizontal" style="flex-basis: 33.3333%; margin-right: 0px"></div>
  <div class="el-step is-horizontal" style="flex-basis: 33.3333%; margin-right: 0px"></div>
  <div class="el-step is-horizontal" style="flex-basis: 33.3333%; margin-right: 0px"></div> 
  <div class="el-step is-horizontal is-flex" style="flex-basis: 33.3333%; max-width: 25%"></div>
</div>
```

因为末元素添加了样式类名`is-flex`，覆盖了`flex-basis`，等同于 `flex: none` 或 `flex: 0 0 auto`。元素会根据自身宽高来设置尺寸。它是完全非弹性的：既不会缩短，也不会伸长来适应 flex 容器。

```css
.is-flex {
  flex-basis: auto !important; 
  flex-shrink: 0; 
  flex-grow: 0;
}
```

上文中提到非简洁模式下，只有未设置间距 `space`和居中对齐 `alignCenter`时，步骤条末元素会生成类名 `is-flex`。这也是第二、三示例末元素跟其他元素宽度一致原因。

```js
isLast && !space && !isCenter && 'is-flex', // 生成类名is-flex
```

竖直方向样式逻辑与水平一致（`alignCenter`设置无效）,此处不再赘述。

#### 图标 & 轴线

根据计算属性`currentStatus`生成当前步骤状态对应的主题样式 `is-[wait/process/finish/error/success]`

```html
<!-- 图标 & 轴线 -->
<div  class="el-step__head" :class="`is-${currentStatus}`">
  <div  class="el-step__line">
    // 轴线...
  </div> 
  <div class="el-step__icon"> 
    // 图标...
  </div>
</div>
```

**图标**

图标元素是类名`el-step__icon`的div元素，提供了具名`icon`插槽自定义步骤节点图标。插槽内容默认展示Icon图标或表示节点顺序圆环数字（从1开始）。简洁风格下，只有Icon设置生效。

当组件状态值为 `success` 或 `error` 时，使用系统提供图标。

```html
<div class="el-step__icon" :class="`is-${icon ? 'icon' : 'text'}`">
  <slot v-if="currentStatus !== 'success' && currentStatus !== 'error'" name="icon">
    <!-- 节点图标 -->
    <i v-if="icon" class="el-step__icon-inner" :class="[icon]"></i>
    <!-- 节点序号 -->
    <div class="el-step__icon-inner" v-if="!icon && !isSimple">{{ index + 1 }}</div>
  </slot>
  <i v-else :class="['el-icon-' + (currentStatus === 'success' ? 'check' : 'close')]"
    class="el-step__icon-inner is-status"
  >
  </i>
</div>
```

数字圆环的样式`is-text`设置的。

```css
:class="`is-${icon ? 'icon' : 'text'}`"

.el-step__icon.is-text {
  border-radius: 50%;
  border: 2px solid;
  border-color: inherit;
}
```

不同设置组件效果对比如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/942d495cdcbf4ecfb6282b8afcabf1ee\~tplv-k3u1fbpfcp-watermark.image?)

设置居中对齐 `alignCenter`时，图标的居中效果是通过样式控制的。

```css
.el-step.is-center .el-step__head {
  text-align: center; 
}
```

**轴线**

轴线元素是类名`el-step__line`的div元素，使用了绝对布局。

```html
<div class="el-step__line">
  <i class="el-step__line-inner" :style="lineStyle"></i>
</div>
```

通过偏移量、height、width等属性设置轴线位置、长度以及粗细。

```css
.el-step__line {
  position: absolute;
  border-color: inherit;
  background-color: #c0c4cc;
}
/* 水平方向 */
.el-step.is-horizontal .el-step__line {
  height: 2px;
  top: 11px;
  left: 0;
  right: 0;
}
/* 水平居中 */
.el-step.is-center .el-step__line {
  left: 50%;
  right: -50%;
}
/* 竖直方向 */
.el-step.is-vertical .el-step__line {
  width: 2px;
  top: 0;
  bottom: 0;
  left: 11px;
}
```

使用了伪类`:last-of-type` 设置最后一个步骤元素中轴线不显示。

```css
.el-step:last-of-type .el-step__line {
  display: none;
}
```

居中对齐下的轴线偏移量有些特殊，渲染效果如下。 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4478522d3b24932920b6dfe03326253\~tplv-k3u1fbpfcp-watermark.image?)

实际元素DOM结构如下： ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2390d30aebdd45bab70f565a1c42c515\~tplv-k3u1fbpfcp-watermark.image?)

轴线进度状态效果通过类名`el-step__line-inner`元素实现，该元素内联样式绑定属性`lineStyle`。

属性`lineStyle`通过方法`calcProgress`根据当前步骤的状态计算而来。 方法`updateStatus`作用在生命周期中详细介绍过。

当前步骤元素不是第一个，此时步骤的状态为已完成，就会更新其上一元素的 `lineStyle` 值，显示进度效果。

```js
updateStatus(val) { 
  // 存在上一步骤节点  计算进度
  if (prevChild) prevChild.calcProgress(this.internalStatus); 
},

calcProgress(status) {
  let step = 100;
  const style = {}; 
  
  style.transitionDelay = 150 * this.index + 'ms'; // 延迟响应过渡效果
  if (status === this.$parent.processStatus) { 
    step = this.currentStatus !== 'error' ? 0 : 0;
  } else if (status === 'wait') {
    step = 0;
    style.transitionDelay = (-150 * this.index) + 'ms'; // 负时会导致过渡立即开始
  }
  // 简洁风格无效
  style.borderWidth = step && !this.isSimple ? '1px' : 0;
  // 方向不同 赋值不同属性
  this.$parent.direction === 'vertical'
    ? style.height = step + '%'
    : style.width = step + '%';

  this.lineStyle = style;
}
```

#### 标题 & 描述

标题是类名`el-step__title`的div元素,提供了具名`title`插槽用于自定义标题，后备插槽内容为属性 `title`值。

描述类名`el-step__description`的div元素,提供了具名`description`插槽用于自定义描述性文字，后备插槽内容为属性 `description`值。简洁风格下，`description` 设置失效。

它们根据计算属性`currentStatus`生成当前步骤状态对应的主题样式 `is-[wait/process/finish/error/success]`。

```html
<!-- 标题 & 描述 -->
<div class="el-step__main">
  <div class="el-step__title" ref="title" :class="['is-' + currentStatus]">
    <slot name="title">{{ title }}</slot>
  </div>
  <!-- 简洁风格的箭头 -->
  <div v-if="isSimple" class="el-step__arrow"></div>
  <div  v-else class="el-step__description" :class="['is-' + currentStatus]">
    <slot name="description">{{ description }}</slot>
  </div>
</div>
```

#### 简洁风格

前面章节中介绍了简洁风格会导致很多设置无效，接下来整体的看下简洁风格的实现效果。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/101e8bcb60224c9a88ad36ce12569a1d\~tplv-k3u1fbpfcp-watermark.image?)

此时DOM结构渲染如下。

```html
<div class="el-steps el-steps--simple">
  <!-- 步骤 1 -->
  <div class="el-step is-simple">
    <div class="el-step__head">
      <div class="el-step__line">
        <i class="el-step__line-inner"></i>
      </div>
      <div class="el-step__icon is-icon">
        <i class="el-step__icon-inner el-icon-edit"></i>
      </div>
    </div>
    <div class="el-step__main">
      <div class="el-step__title">步骤 1</div>
      <div class="el-step__arrow"></div>
    </div>
  </div>
  <!-- 步骤 2 -->
  <!-- 步骤 3 -->
</div>
```

此时步骤元素使用flex布局，所以图标、标题、 箭头元素都在一行内展示。

```css
.el-step.is-simple {
  display: flex;
  align-items: center;
}

.el-step.is-simple .el-step__main {
  display: flex;
  align-items: stretch;
}
```

轴线元素DOM渲染了，但是没有设置宽、高、边框粗细，页面就无法展示。

```css
.el-step.is-horizontal .el-step__line {
  height: 2px; 
}
.el-step.is-vertical .el-step__line {
  width: 2px; 
}

calcProgress(status) {
  // ...
  // 简洁风格无效
  style.borderWidth = step && !this.isSimple ? '1px' : 0; 
}
```

箭头样式使用伪类`:before`、`:after`定义。

```css
.el-step.is-simple .el-step__arrow::after,
.el-step.is-simple .el-step__arrow::before {
  content: "";
  display: inline-block;
  position: absolute;
  height: 15px;
  width: 1px;
  background: #c0c4cc;
}
.el-step.is-simple .el-step__arrow::before { 
  transform: rotate(-45deg) translateY(-4px); 
  transform-origin: 0 0;
}
.el-step.is-simple .el-step__arrow::after { 
  transform: rotate(45deg) translateY(4px); 
  transform-origin: 100% 100%;
}
.el-step.is-simple:last-of-type .el-step__arrow {
  display: none;
}
```

### 样式实现

组件样式源码 `packages\theme-chalk\src\steps.scss` 使用混合指令嵌套生成组件样式。

```scss
// packages\theme-chalk\src\steps.scss

// 生成 .el-steps
@include b(steps) {
  // ...
  
  // 生成 .el-steps--simple
  @include m(simple) {
    // ...
  }
  // 生成 .el-steps--horizontal
  @include m(horizontal) {
    // ...
  }
  // 生成 .el-steps--vertical
  @include m(vertical) {
    // ...
  }
}
```

组件样式源码 `packages\theme-chalk\src\step.scss` 使用混合指令嵌套生成组件样式。

```scss
// packages\theme-chalk\src\step.scss

// 生成 .el-step
@include b(step) {
  // ...
  
  // 生成 .el-step:last-of-type .el-step__line
  @include pseudo(last-of-type) {
    @include e(line) {
      // ...
    } 
    // 生成 .el-step:last-of-type.is-flex 
    @include when(flex) {
      // ...
    }
    // 生成 .el-step:last-of-type .el-step__description,.el-step:last-of-type .el-step__main
    @include e((main, description)) {
      // ...
    }
  }
  // 生成.el-step__head
  @include e(head) {
    // ...
    
    // 生成 .el-step__head.is-[wait/process/finish/error/success]
    @include when(process) {
      // ...
    }
    // wait/finish/error/success ... 
  }
  // 生成 .el-step__icon 
  @include e(icon) {
    // ...
    
    // 生成 .el-step__icon.is-text
    @include when(text) {
      // ...
    }
    // 生成 .el-step__icon.is-icon
    @include when(icon) {
      // ...
    }
  }
  // 生成 .el-step__icon-inner
  @include e(icon-inner) {
    // ...
    
    // 生成 .el-step__icon-inner[class*="el-icon"]:not(.is-status)
    &[class*=el-icon]:not(.is-status) {
      // ...
    }

    // 生成 .el-step__icon-inner.is-status
    @include when(status) {
      // ...
    }
  }
  // 生成 .el-step__line
  @include e(line) {
    // ...
  }
  // 生成 .el-step__line-inner
  @include e(line-inner) {
    // ...
  }
  // 生成 .el-step__main
  @include e(main) {
    // ...
  }
  // 生成 .el-step__title
  @include e(title) {
    // ...
    
    // 生成  .el-step__title.is-[wait/process/finish/error/success]
    @include when(process) {
      // ...
    } 
    // wait/finish/error/success
  }
  // 生成 .el-step__description
  @include e(description) {
    // ...
    // 生成  .el-step__description.is-[wait/process/finish/error/success]
    @include when(process) {
      // ...
    }
    // wait/finish/error/success
 
  }
  // 生成 .el-step.is-horizontal
  @include when(horizontal) {
    // ...
    
    // 生成 .el-step.is-horizontal .el-step__line
    @include e(line) {
      // ...
    }
  }
  // 生成.el-step.is-vertical
  @include when(vertical) {
    // ...
    
    // 生成.el-step.is-vertical .el-step__head/main/title/line
    @include e(head) { /*...*/ }
    @include e(main) { /*...*/ } 
    @include e(title) { /*...*/ } 
    @include e(line) { /*...*/ }
     
    // 生成.el-step.is-vertical .el-step__icon.is-icon 
    @include e(icon) {
      @include when(icon) {
        // ...
      }
    }
  }
  
  @include when(center) { 
    // 生成.el-step.is-center .el-step__head/description/line
    @include e(head) { /*...*/ }
    @include e(description) { /*...*/ } 
    @include e(line) { /*...*/ }  
  }
  // 生成.el-step.is-simple
  @include when(simple) {
    // ...
    
    // 生成.el-step.is-simple .el-step__head/icon/main/title
    @include e(head) { /*...*/ }
    @include e(icon) { /*...*/ }
    @include e(main) { /*...*/ }
    @include e(title) { /*...*/ }
    
    
    @include e(icon-inner) {
      // 生成 .el-step.is-simple .el-step__icon-inner[class*="el-icon"]:not(.is-status) 
      &[class*=el-icon]:not(.is-status) {
        // ...
      }
      // 生成 .el-step.is-simple .el-step__icon-inner.is-status
      &.is-status {
        // ...
      }
    }
    
    // 生成 .el-step.is-simple:not(:last-of-type) .el-step__title 
    @include pseudo('not(:last-of-type)') {
      @include e(title) {
        // ...
      }
    }
    // 生成 .el-step.is-simple .el-step__arrow
    @include e(arrow) {
      // ...
      
      // 生成 .el-step.is-simple .el-step__arrow::after,.el-step.is-simple .el-step__arrow::after 
      &::before,
      &::after {
        // ...
      }
      // 生成 .el-step.is-simple .el-step__arrow::before  
      &::before {
        // ...
      }
      // 生成 .el-step.is-simple .el-step__arrow::after 
      &::after {
        // ...
      }
    }
    // 生成 .el-step.is-simple:last-of-type .el-step__arrow 
    @include pseudo(last-of-type) {
      @include e(arrow) {
        // ...
      }
    }
  }
} 
```

### 📚参考&关联阅读

['CSS/flex',MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex)\
['CSS/:last-of-type',MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:last-of-type)\
['生命周期',vuejs](https://staging-cn.vuejs.org/api/options-lifecycle.html)\
['watch',vuejs](https://staging-cn.vuejs.org/api/options-state.html#watch)
