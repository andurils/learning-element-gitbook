# 3.22 Rate 评分

### 简介

评分组件 `Rate` 多用于展示评价或者对事物快速评级。本文将分析源码实现，耐心读完，相信会对您有所帮助。组件源码实现详见`packages/rate/src/main.vue` 。 🔗 [组件文档 Rate](https://element.eleme.cn/#/zh-CN/component/rate) 🔗 [gitee源码 main.vue](https://gitee.com/ElemeFE/element/blob/dev/packages/rate/src/main.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 源码实现

组件的 `prop` 声明，各属性功能说明详见 [官方文档rate#attributes](https://element.eleme.io/#/zh-CN/component/rate#attributes) 。

#### 组件DOM结构

组件的根节点是类名`el-rate`的div元素，子元素包含评分icon图标和辅助文字两部分。

精简之后的结构代码如下：

```html
<template>
  <div class="el-rate">
    <span v-for="(item, key) in max" class="el-rate__item">
      <i class="el-rate__icon"></i>
    </span>
    <span class="el-rate__text">{{ text }}</span>
  </div>
</template>
```

#### 评分icon图标

组件的主要逻辑都在评分icon图标部分，接下来将采用渐进式的讲解方式，对比分析各代码功能逻辑。

**基础展示**

评分icon图标基于属性`max`渲染一个类名`el-rate__item`的`span`元素列表生成。默认个数为 `5`。使用计算属性 `classes` 返回不同状态（选中/未选中）的 icon 类名。使用方法 `getIconStyle` 返回不同状态（选中/未选中）的 icon 颜色。

此时组件代码如下，没有鼠标交互事件，只能用作展示。

```html
<span
  v-for="(item, key) in max"
  class="el-rate__item" 
  :key="key">
  <i
    class="el-rate__icon"
    :class="[classes[item - 1]]" 
    :style="getIconStyle(item)"> 
  </i>
</span>
```

运行如下示例代码

```html
<template>
  <div>
    <div class="block">
      <span class="demonstration">默认不区分颜色</span>
      <el-rate v-model="value1"></el-rate>
    </div>
    <div class="block">
      <span class="demonstration">区分颜色</span>
      <el-rate v-model="value2" :colors="colors"> </el-rate>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      value1: 2,
      value2: 2,
      // 等同于 { 2: '#99A9BF', 4: { value: '#F7BA2A', excluded: true }, 5: '#FF9900' }
      colors: ['#99A9BF', '#F7BA2A', '#FF9900']  
    };
  },
};
</script>
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce780c828d834a88aafb06a33f7f5ef7\~tplv-k3u1fbpfcp-watermark.image?)

组件支持评分分级，通过 `iconClasses`、 `colors`、 `lowThreshold`、 `highThreshold`等属性控制显示逻辑内容。

> 评分默认被分为三个等级，可以利用颜色数组对分数及情感倾向进行分级（默认情况下不区分颜色）。三个等级所对应的颜色用`colors`属性设置，而它们对应的两个阈值则通过 `low-threshold` 和 `high-threshold` 设定。也可以通过传入颜色对象来自定义分段，键名为分段的界限值，键值为对应的颜色。

当组件用`v-model`传入了一个值， `v-model`的语法糖会把这个值当成`props`的`value`传到组件中。此时组件内部使用属性`currentValue`等同于属性`value`。

⚠️ **当后面添加鼠标事件后，鼠标移动会直接改变 `currentValue`,属性`currentValue`不等同于属性`value`**,

```js
data() {
  return { 
    currentValue: this.value, 
  };
},
watch: {
  value(val) {
    this.currentValue = val; 
  }
},
```

**基础展示的样式计算**

计算属性 `classes`基于组件的绑定值判断选中状态，生成的icon图标类名数组，由于遍历`MAX`是从`1`开始，所以调用时需`item-1`。

```js
classes() {
  let result = [];
  let i = 0;
  // 基于当前选中值 作为阈值
  let threshold = this.currentValue;
  // 半选逻辑 ...
  // 选中状态 
  for (; i < threshold; i++) {
    result.push(this.activeClass);
  }
  // 未选中状态 
  for (; i < this.max; i++) {
    result.push(this.voidClass);
  }
  return result;
},
```

方法`getIconStyle` 返回当前图标状态颜色。根据组件的只读状态，返回不同的未选中颜色。

```js
getIconStyle(item) {
  const voidColor = this.rateDisabled ? this.disabledVoidColor : this.voidColor;
  return {
    color: item <= this.currentValue ? this.activeColor : voidColor
  };
},
```

计算属性`activeClass`使用方法`getValueFromMap`，返回对应分段区间的选中状态icon类名。

```js
//computed
activeClass() {
  return this.getValueFromMap(this.currentValue, this.classMap);
}, 
// methods
getValueFromMap(value, map) { 
  const matchedKeys = Object.keys(map)
    .filter(key => {
      const val = map[key];
      // 区间最大值是否包含  true 不包括  false 包括
      const excluded = isObject(val) ? val.excluded : false;
      return excluded ? value < key : value <= key;
    })
    .sort((a, b) => a - b); // 排序 从小到大
  // 获取最小配置
  const matchedValue = map[matchedKeys[0]];
  // 根据类型获取值
  return isObject(matchedValue) ? matchedValue.value : (matchedValue || '');
},
```

计算属性 `classMap` 返回 3 个分段区间对应颜色的对象。

* 若`iconClasses`属性传入数组，共有 3 个元素，转换成对象，分段界限值基于`lowThreshold`、`highThreshold`、`max`，各区间为 `[0,lowThreshold]` 、`(lowThreshold,highThreshold)` 、`[highThreshold,max]`。
* 若传入对象，可自定义分段，键名为分段的界限值，键值为对应的icon类名。

```js
// 默认值 { 2: 'el-icon-star-on', 4: { value: 'el-icon-star-on', excluded: true }, 5: 'el-icon-star-on' }
classMap() {
  return Array.isArray(this.iconClasses)
    ? {
      [this.lowThreshold]: this.iconClasses[0],
      [this.highThreshold]: { value: this.iconClasses[1], excluded: true },
      [this.max]: this.iconClasses[2]
    } : this.iconClasses;
},
```

属性 `iconClasses`默认值 `['el-icon-star-on', 'el-icon-star-on', 'el-icon-star-on']`，组件默认各评分层级 icon 类型相同。

计算属性 `activeColor` 基于同样的处理逻辑，返回对应分段区间的选中状态icon颜色。

```js
activeColor() {
  return this.getValueFromMap(this.currentValue, this.colorMap);
},
colorMap() { 
  return Array.isArray(this.colors)
    ? {
      [this.lowThreshold]: this.colors[0],
      [this.highThreshold]: { value: this.colors[1], excluded: true },
      [this.max]: this.colors[2]
    } : this.colors;
},
```

计算属性`voidClass`根据是否只读状态，返回未选中状态icon类名。计算属性`rateDisabled`返回组件是否只读状态。

```js
// 未选中 icon 的类名  不同模式下：只读模式/可选
voidClass() {
  return this.rateDisabled ? this.disabledVoidIconClass : this.voidIconClass;
},
// 是否为只读 
rateDisabled() {
  return this.disabled || (this.elForm || {}).disabled;
} 

// elForm 注入
inject: {
  elForm: {
    default: ''
  }
},
```

> 若表单禁用设置为 true，则表单内组件上的 disabled 属性不再生效。

**支持半选**

接下来在类名`el-rate__icon`的`i`元素中添加一个类名`el-rate__decimal`的`i`元素，用于实现半选效果。

```html
<span
  v-for="(item, key) in max"
  class="el-rate__item" 
  :key="key">
  <i
    class="el-rate__icon"
    :class="[classes[item - 1]]" 
    :style="getIconStyle(item)"> 
    <i
      v-if="showDecimalIcon(item)"
      :class="decimalIconClass"
      :style="decimalStyle"
      class="el-rate__decimal">
    </i>
  </i>
</span>
```

方法`showDecimalIcon` 控制组件支持半选展示状态。

* 只读模式下，默认支持半选效果。
* 非只读模式下，小数位要不小于0.5才会显示半选效果。

```js
showDecimalIcon(item) {
  let showWhenDisabled = this.rateDisabled && this.valueDecimal > 0 && 
      item - 1 < this.value && item > this.value; 
      
  let showWhenAllowHalf = this.allowHalf &&
    this.pointerAtLeftHalf &&
    item - 0.5 <= this.currentValue &&
    item > this.currentValue;
  return showWhenDisabled || showWhenAllowHalf;
},
```

属性`pointerAtLeftHalf` 用于普通模式下控制半星展示，当绑定值包含小数位时为`true`。

```js
data() {
  return { 
    pointerAtLeftHalf: true,
  };
},
watch: {
  value(val) {
    this.pointerAtLeftHalf = this.value !== Math.floor(this.value);
  }
},
```

Icon类名使用计算属性`decimalIconClass`设置，跟上一小节的`activeClass`实现一样。

半选显示样式通过计算属性`decimalStyle`控制，返回包含选中颜色、宽度的样式对象。

* 只读模式下，按照小数位百分比 `valueDecimal` 显示宽度。
* 非只读模式下，小数位不管多大，只显示半颗星 `50%`。

```js
decimalStyle() {
  let width = ''; 
  if (this.rateDisabled) {  // 只读模式
    width = `${ this.valueDecimal }%`; 
  } else if (this.allowHalf) { 
    width = '50%'; // 半颗星
  } 
  return {
    color: this.activeColor,
    width
  };
},
// 当前值百分比  3.2 => 20(%)
valueDecimal() {
  return this.value * 100 - Math.floor(this.value) * 100;
},
```

不同模式下样式对比如下：

```html
<!-- 只读模式 -->
<el-rate v-model="value" disabled show-score> </el-rate>
<!-- 半选模式 -->
<el-rate v-model="value" allow-half show-score> </el-rate>
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dc3d73909434f688b0105e9ab9cc39d\~tplv-k3u1fbpfcp-watermark.image?)

**鼠标交互事件**

在元素添加一些事件和样式处理逻辑，用于支持鼠标滑动、点击事件。

```html
<span
  v-for="(item, key) in max"
  class="el-rate__item"
  @mousemove="setCurrentValue(item, $event)"
  @mouseleave="resetCurrentValue"
  @click="selectValue(item)"
  :style="{ cursor: rateDisabled ? 'auto' : 'pointer' }"
  :key="key">
  <i
    :class="[classes[item - 1], { 'hover': hoverIndex === item }]"
    class="el-rate__icon"
    :style="getIconStyle(item)">
    <i
      v-if="showDecimalIcon(item)"
      :class="decimalIconClass"
      :style="decimalStyle"
      class="el-rate__decimal">
    </i>
  </i>
</span>
```

`span`元素样式根据只读状态设置光标的类型。

```html
:style="{ cursor: rateDisabled ? 'auto' : 'pointer' }"
```

`mousemove`事件监听方法`setCurrentValue`,用于设置鼠标悬停元素索引 `hoverIndex`（从`1`开始）和`currentValue`，只读模式下无效。

在半选模式下，选择类名`el-rate__icon`的DOM元素作为目标，获取基于目标节点的内填充边在 X 轴方向上的偏移量 `offsetX`。

如果`offsetX`小于目标元素的内部宽度的一半，则认为光标在目标元素的左半边，`pointerAtLeftHalf` 值为 `true`，`currentValue`值为 `item-0.5`;反之`pointerAtLeftHalf` 值为 `false`，`currentValue`值为 `item`。所以选中效果就是`半星->整星` 或 `整星->半星` 的过程。

```js
setCurrentValue(value, event) {
  if (this.rateDisabled) {
    return;
  } 
  if (this.allowHalf) {
    let target = event.target;
    if (hasClass(target, 'el-rate__item')) {
      target = target.querySelector('.el-rate__icon');
    }
    if (hasClass(target, 'el-rate__decimal')) {
      target = target.parentNode;
    }
    this.pointerAtLeftHalf = event.offsetX * 2 <= target.clientWidth;
    this.currentValue = this.pointerAtLeftHalf ? value - 0.5 : value;
  } else {
    this.currentValue = value;
  }
  this.hoverIndex = value;
},
```

属性`hoverIndex` 用于icon图标鼠标悬停放大效果。

```css
:class="[{ 'hover': hoverIndex === item }]"

// 生成样式
.el-rate__icon.hover { 
  transform: scale(1.15);
}
```

`mouseleave`事件监听方法`resetCurrentValue`,用于重置`pointerAtLeftHalf`、`hoverIndex`和`currentValue`等属性值，只读模式下无效

```js
resetCurrentValue() {
  if (this.rateDisabled) {
    return;
  }
  if (this.allowHalf) {
    this.pointerAtLeftHalf = this.value !== Math.floor(this.value);
  }
  this.currentValue = this.value;
  this.hoverIndex = -1;
}
```

`clcik`事件监听方法`selectValue`用于触发组件自定义`change`事件，返回改变后的分值，只读模式下无效。 `$emit('input', val)` 用来更新组件`v-model` 值。

```js
selectValue(value) {
  if (this.rateDisabled) {
    return;
  }
  if (this.allowHalf && this.pointerAtLeftHalf) {
    this.$emit('input', this.currentValue);
    this.$emit('change', this.currentValue);
  } else {
    this.$emit('input', value);
    this.$emit('change', value);
  }
},
```

#### 辅助文字

类名`el-rate__text`的span元素用于显示辅助文字,是否显示由属性 `showText` 、`showScore`值控制，文字的颜色由属性 `textColor`控制。

```html
<span v-if="showText || showScore" 
    class="el-rate__text" :style="{ color: textColor }">
    {{ text }}
</span>
```

显示内容使用了计算属性 `text`。 当 `showScore=true`, 显示当前分数,将当前组件分值使用 scoreTemplate 模板进行内容格式化。 当 `showText=true`, 显示当前辅助文字,基于当前组件分值调用对应索引（属性 `texts`）的文字描述。

```js
text() {
  let result = '';
  if (this.showScore) {
    result = this.scoreTemplate.replace(/\{\s*value\s*\}/, this.rateDisabled
      ? this.value
      : this.currentValue);
  } else if (this.showText) {
    result = this.texts[Math.ceil(this.currentValue) - 1];
  }
  return result;
},
```

属性`showText`和`showScore`不能同时为真。

#### 键盘事件

回到根节点，添加如下代码。

```html
<div
  class="el-rate"
  @keydown="handleKey"
  role="slider"
  :aria-valuenow="currentValue"
  :aria-valuetext="text"
  aria-valuemin="0"
  :aria-valuemax="max"
  tabindex="0">
  // el-rate__item...
  // el-rate__text...
</div>
```

属性 `role` 、`aria-valuenow` 、`aria-valuetext`、`aria-valuemin`、`aria-valuemax`用于实现 ARIA无障碍网页应用。

属性 `tabindex="0"` 表示元素是可聚焦的，并且可以通过键盘导航来聚焦到该元素。

添加方法`handleKey`监听`keydown`事件，只读模式下无效。

* 当根节点元素聚焦后，使用方向键增加/减少分值，按下键`Up/Right`用于增加分值，按下键`Left/Down`用于减少分值。默认步进为`1`，当允许半选时，步进为`0.5`。
* 边界值处理，最小值（`0`）和最大值（`max`）。
* 最后触发组件定义`change`事件，返回改变后的分值；同时更新v-model值。

```js
handleKey(e) {
  if (this.rateDisabled) {
    return;
  }
  let currentValue = this.currentValue;
  const keyCode = e.keyCode;
  if (keyCode === 38 || keyCode === 39) { // Up / Right
    if (this.allowHalf) {
      currentValue += 0.5;
    } else {
      currentValue += 1;
    }
    // 阻止事件冒泡
    e.stopPropagation();
    e.preventDefault();
  } else if (keyCode === 37 || keyCode === 40) { // Left / Down
    if (this.allowHalf) {
      currentValue -= 0.5;
    } else {
      currentValue -= 1;
    }
    e.stopPropagation();
    e.preventDefault();
  }
  
  currentValue = currentValue < 0 ? 0 : currentValue;
  currentValue = currentValue > this.max ? this.max : currentValue;

  this.$emit('input', currentValue); // 更新v-model值
  this.$emit('change', currentValue);
},
```

> 组件需要指定一个`$emit('input')`来改变组件v-model值，否则无法更新用户传入的v-model值。

***

### 组件样式

组件样式源码 `packages\theme-chalk\src\rate.scss` 使用混合指令 `b`、`e` 嵌套生成组件样式。

```scss
// 生成 .el-rate 
@include b(rate) {
  // ..
  // 生成 .el-rate:active,.el-rate:focus
  &:focus, &:active {
    // ..
  }  
  // 生成 .el-rate__item
  @include e(item) {
    // ..
  }
  // 生成 .el-rate__icon
  @include e(icon) {
    // ..
    // 生成 .el-rate__icon.hover
    &.hover {
      transform: scale(1.15);
    } 
    // 不知具体作用
    .path2 {
      // ..
    }
  }
  // 生成 .el-rate__decimal
  @include e(decimal) {
    // ..
  }
  // 生成 .el-rate__text
  @include e(text) {
    // ..
  }
}
```

在`created`，增加逻辑如果value传入非数值,使用`$emit('input', 0)`，更新v-model值为0。

```js
created() {
  if (!this.value) {
    this.$emit('input', 0);
  }
}
```

***

### 📚参考&关联阅读

["ARIA",MDN](https://developer.mozilla.org/zh-CN/docs/Web/Accessibility/ARIA)\
["Element/mousemove\_event",MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/mousemove\_event)\
["tabindex",MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global\_attributes/tabindex)
