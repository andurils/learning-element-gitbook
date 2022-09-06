# 3.30 Input 输入框

### 简介

输入框组件 `Input` 通过鼠标或键盘输入表单域内容，提供复合型型输入框，带搜索的输入框，还可以进行大小选择。 本文将分析其源码实现，耐心读完，相信会对您有所帮助。🔗 [组件文档 Input](https://element.eleme.cn/#/zh-CN/component/input) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/blob/dev/packages/input/src)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 模板HTML

组件的 `props` 声明，各属性功能描述详见[官方文档Input#attributes](https://element.eleme.cn/#/zh-CN/component/input#input-attributes) 。

组件根元素是一个类名为`el-textarea`或`el-input`的 div 元素，元素内容包含了三部分：

1. 封装原生 `input` 控件实现单行文本输入框。
2. 封装原生 `textarea` 控件多行文本输入文本域。

```html
// packages\input\src\input.vue
<template>
  <div :class="[type === 'textarea' ? 'el-textarea' : 'el-input',]" >
    <!-- 单行文本输入框 -->
    <template v-if="type !== 'textarea'">
      <!-- 输入框前置内容 -->
      <div class="el-input-group__prepend"></div> 
      <!-- 表单输入控件 -->
      <input>
      <!-- 输入框头部内容 -->
      <span class="el-input__prefix"></span>
      <!-- 输入框尾部内容 -->
      <span class="el-input__suffix"></span>
      <!-- 输入框后置内容 -->
      <div class="el-input-group__append"></div> 
    </template>
    <!-- 多行文本输入的文本域 -->
    <textarea></textarea>
  </div>
</template>
```

### 组件根元素

* 属性`type`值确定使用 `el-textarea` 或 `el-input`。
* 组件尺寸`inputSize`生成样式 `el-input--medium/small/mini`。
* 禁用状态`inputDisabled` 生成样式 `is-disabled`。
* 开启输入长度限制，输入超限时`inputExceed`生成样式 `is-disabled`。
* 复合型输入框样式`el-input-group`、`el-input-group--append/prepend`根据前置/后置插槽内容生成。
* 头部/尾部样式`el-input--prefix`、`el-input--suffix`根据插槽内容或功能属性值生成。
* 添加了鼠标移入移出事件，使用内部属性 `hover` 记录元素悬停状态。

```html
<div :class="[
  type === 'textarea' ? 'el-textarea' : 'el-input',
  inputSize ? 'el-input--' + inputSize : '',
  {
    'is-disabled': inputDisabled,
    'is-exceed': inputExceed,
    'el-input-group': $slots.prepend || $slots.append,
    'el-input-group--append': $slots.append,
    'el-input-group--prepend': $slots.prepend,
    'el-input--prefix': $slots.prefix || prefixIcon,
    'el-input--suffix': $slots.suffix || suffixIcon || clearable || showPassword
  }
  ]"
  @mouseenter="hovering = true"
  @mouseleave="hovering = false"
>
  // ...
</div>
```

### 单行文本输入

单行文本输入框通过封装原生 `input` 控件实现,支持控件的原生属性。组件通过组合以下多种元素实现文本框 `text`或密码框`password`等复合型输入框功能：

1. 输入框前置内容，提供具名插槽`prepend`,内容一般为标签或按钮。
2. 原生 input 表单输入控件，添加了自定义事件。
3. 输入框头部内容，可以通过 `prefix-icon` 或具名插槽`prefix`增加显示图标。
4. 输入框尾部内容，可以通过 `suffix-icon` 或具名插槽`suffix`增加显示图标。该元素也用于展示输入框清空Icon、密码显隐切换Icon以及展示输入字数统计。
5. 输入框后置内容，提供具名插槽`append`,内容一般为标签或按钮。

```html
<template v-if="type !== 'textarea'">
  <!-- 输入框前置内容 -->
  <div class="el-input-group__prepend" v-if="$slots.prepend">
    <slot name="prepend"></slot>
  </div>
  <!-- 表单输入控件 -->
  <input
    :tabindex="tabindex"  // 元素是否可以聚焦 键盘导航
    v-if="type !== 'textarea'"
    class="el-input__inner"
    v-bind="$attrs" // 透传 Attributes 例如 placeholder
    :type="showPassword ? (passwordVisible ? 'text': 'password') : type" // text/password  也支持其他原生input的type值
    :disabled="inputDisabled" // 是否禁用
    :readonly="readonly"  // 是否只读
    :autocomplete="autoComplete || autocomplete" // 自动补全
    ref="input"
    @compositionstart="handleCompositionStart" // 输入法编辑器 (IME) 事件
    @compositionupdate="handleCompositionUpdate"
    @compositionend="handleCompositionEnd"
    @input="handleInput" // 输入内容
    @focus="handleFocus" // 获取焦点
    @blur="handleBlur" // 失去焦点
    @change="handleChange" // 输入值变化
    :aria-label="label" // ARIA 无障碍属性
  >
  <!-- 输入框头部内容 -->
  <span class="el-input__prefix" v-if="$slots.prefix || prefixIcon">
    <slot name="prefix"></slot>
    <i class="el-input__icon" v-if="prefixIcon"  :class="prefixIcon"></i>
  </span>
  <!-- 输入框尾部内容 -->
  <span class="el-input__suffix" v-if="getSuffixVisible()">
    <span class="el-input__suffix-inner">
      <template v-if="!showClear || !showPwdVisible || !isWordLimitVisible">
        <slot name="suffix"></slot>
        <i class="el-input__icon" v-if="suffixIcon" :class="suffixIcon"></i>
      </template>
      // ...
    </span>
    // ...
  </span>
  <!-- 输入框后置内容 -->
  <div class="el-input-group__append" v-if="$slots.append">
    <slot name="append"></slot>
  </div>
</template>
```

组件渲染效果如下:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9131a61167a46c8b413256ebfe4379b\~tplv-k3u1fbpfcp-watermark.image?)

> 输入框头部/尾部都提供了具名插槽用于增加显示图标，当然也可以传入文本等其他内容，但不建议这么做。

当设置头部/尾部内容时, input 输入框通过属性 `padding` 提供了 `30px` 的宽度区域用于内容展示。头部/尾部元素使用绝对布局，将内容偏移覆盖至 padding 区域。

```css
.el-input__prefix {
  position: absolute;
  height: 100%;
  left: 5px;
  top: 0;  
}

.el-input__suffix {
  position: absolute;
  height: 100%;
  right: 5px;
  top: 0; 
}
.el-input--prefix .el-input__inner {
    padding-left: 30px; 
}
.el-input--suffix .el-input__inner { 
    padding-right: 30px; 
}
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2aeda49c84049b7bc89897be1a6573a\~tplv-k3u1fbpfcp-watermark.image?)

以下示例将自定义文本传入插槽。

```html
<el-input placeholder="请输入内容" v-model="input">
  <template slot="prefix">
    <span style="display: flex; align-items: center; height: 100%">头部内容</span>
  </template>
  <template slot="suffix">
    <span style="display: flex; align-items: center; height: 100%">尾部内容</span>
  </template>
</el-input>
```

示例渲染出现内容覆盖 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e46bbf86fdc416baeb301e8870ed58b\~tplv-k3u1fbpfcp-watermark.image?)

#### input 元素类型

组件默认使用 `text` 类型的input控件。当设置 `showPassword`可得到一个可切换显示隐藏的密码框，内部属性`passwordVisible`记录显隐状态，根据不同的状态使用 `password` 或 `text` 类型。

```js
:type="showPassword ? (passwordVisible ? 'text': 'password') : type"
```

属性 `type` 值也可设置为其他[原生input的type值](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input#input\_types)。

```html
<el-input v-model="input" placeholder="请输入内容" type="color"></el-input>
<el-input v-model="input" placeholder="请输入内容" type="date"></el-input>
<el-input v-model="input" placeholder="请输入内容" type="datetime-local"></el-input> 
<el-input v-model="input" placeholder="请输入内容" type="file"></el-input> 
<el-input v-model="input" placeholder="请输入内容" type="month"></el-input> 
<el-input v-model="input" placeholder="请输入内容" type="time"></el-input> 
<el-input v-model="input" placeholder="请输入内容" type="week"></el-input>
```

使用其他原生类型时组件渲染效果，但是一般不建议这么使用！一般组件类库都会提供对应的更加丰富的功能组件，使用库组件页面样式风格更加统一，提升用户交互体验。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/346aee57fabb4e589fca484f57393e3d\~tplv-k3u1fbpfcp-watermark.image?)

#### 输入法编辑器(IME)事件

输入法在中文、日文和韩文等少数语言中使用。以中文拼音输入法为例，输入的过程大致可以分为**组字（composition）** 和 **提交（commit）** 两阶段。比如想打“你好”两个字，会在输入框输入“nihao”的拼音，当输入第一个字母“n”时，组字过程就开始了。此时本地的 IME（input method editor） 软件（比如微软/搜狗拼音输入法）会为我们提供**组字框**和**候选列表**的 UI 组件。

> 关于输入法事件更多介绍，请阅读 [Web 键盘输入法应用开发指南 —— 输入法事件](https://xie.infoq.cn/article/49640d0f3c0ba0d793f6b7501)。

`compositionstart`、`compositionupdate`和`compositionend`是一组事件。

1. 首先是使用拼音输入法开始输入汉字时 `compositionstart` 被触发，组字框和候选列表相应出现；
2. 此后，每按一个新键，就会触发 `compositionupdate`，此时组字框和候选列表的内容也发生变化；
3. 当选择了候选列表中的某个字或词，或者敲击空格（中文输入法），`compositionend` 事件会触发，表明输入被提交。
4. 当取消输入时，比如使用鼠标单击页面空白处，就会终止当前输入，也会触发`compositionend` 事件。

属性`isComposing`值为 `true`表示用户正在输入。输入结束后（选择字词或者取消），调用方法`handleInput`，触发组件 `input` 事件。

```js
// @compositionstart="handleCompositionStart"
// @compositionupdate="handleCompositionUpdate"
// @compositionend="handleCompositionEnd"

handleCompositionStart() {
  this.isComposing = true; // 正在输入
},
handleCompositionUpdate(event) {
  const text = event.target.value;
  const lastCharacter = text[text.length - 1] || '';
  // 韩文字符编码 判断最后一个字符是否特殊的功能键 Process Key 
  this.isComposing = !isKorean(lastCharacter); 
},
handleCompositionEnd(event) {
  // 输入结束后，触发 input 事件
  if (this.isComposing) {
    this.isComposing = false;
    this.handleInput(event);
  }
},
```

#### input/change 事件

* 当 `<input>`、`<select>`、`<textarea>`  元素的 `value` 被修改时，会触发 `input` 事件。
* 当用户更改`<input>`、`<select>`、`<textarea>` 元素的值并提交这个更改时，`change` 事件在这些元素上触发。基于表单元素的类型和用户对标签的操作的不同，`change` 事件触发的时机也不同。

对于文本输入元素，比如 `<input type="text">`，每当元素的 `value` 改变，`input` 事件都会被触，`change` 事件在控件失去焦点后才会触发。

```js
// @input="handleInput" 
// @change="handleChange"

handleInput(event) {
  // 输入法下用户正在输入
  if (this.isComposing) return;

  // IE 11下 DatePicker组件 hack 写法，详见issues
  // https://github.com/ElemeFE/element/issues/8548
  // should remove the following line when we don't support IE
  if (event.target.value === this.nativeInputValue) return;
  
  // 触发触发当前实例上的input事件
  this.$emit('input', event.target.value);
 
  // Input 为受控组件 更新组件的绑定值
  this.$nextTick(this.setNativeInputValue);
},
handleChange(event) {
  // 触发触发当前实例上的change事件
  this.$emit('change', event.target.value);
},
```

对于需要使用输入法的语言， `v-model` 不会在输入法组合文字过程中得到更新，因为`compositionend`事件没触发是不会执行`handleInput`逻辑。

```js
if (this.isComposing) return;
```

#### focus/blur 事件

元素获取或失去焦点时，调用事件监听方法，更新内部属性`focused`记录元素焦点状态，同时触发实例自定义 `focus` 或 `blur` 事件。

```js
// @focus="handleFocus"
// @blur="handleBlur"

handleFocus(event) {
  this.focused = true;
  // 触发触发当前实例上的focus事件
  this.$emit('focus', event);
},
handleBlur(event) {
  this.focused = false;
  // 触发触发当前实例上的blur事件
  this.$emit('blur', event);
  // 开启输入时是否触发表单的校验  
  if (this.validateEvent) {
    // 触发组件`FormItem`的自定义`el.form.blur`事件。
    this.dispatch('ElFormItem', 'el.form.blur', [this.value]);
  } 
},
```

#### provide/inject 依赖注入

在 Form 组件中，每一个表单域由一个 Form-Item 组件构成，表单域中可以放置各种类型的表单控件，包括 Input、Select、Checkbox、Radio、Switch、DatePicker、TimePicker等。

表单`form`和表单域`form-item`使用`provide` 选项指定给后代组件的状态，避免了 **prop 逐级透传** 。

```js
// packages\form\src\form.vue
provide() {
  return {
    elForm: this
  };
},

// packages\form\src\form-item.vue
provide() {
  return {
    elFormItem: this
  };
}, 
inject: ['elForm'],
computed: { 
  // ...
  _formSize() {
    return this.elForm.size;
  },
  elFormItemSize() {
    return this.size || this._formSize;
  },
  sizeClass() {
    return this.elFormItemSize || (this.$ELEMENT || {}).size;
  }
}, 
```

组件 `input` 使用`inject`选项注入上层组件提供的数据，如尺寸、校验、禁用，用于组件内部状态的控制计算。

```js
// packages\input\src\input.vue
inject: {
  elForm: {
    default: ''
  },
  elFormItem: {
    default: ''
  }
},
computed: {
  // 表单域下组件的尺寸
  _elFormItemSize() {
    return (this.elFormItem || {}).elFormItemSize;
  },
  // 表单域下组件的校验状态 校验中/成功/失败
  validateState() {
    return this.elFormItem ? this.elFormItem.validateState : '';
  },
  // 是否在输入框中显示校验结果反馈图标
  needStatusIcon() {
    return this.elForm ? this.elForm.statusIcon : false;
  },
  // 表单域下组件的校验状态图标
  validateIcon() {
    return {
      validating: 'el-icon-loading',
      success: 'el-icon-circle-check',
      error: 'el-icon-circle-close'
    }[this.validateState];
  },
  // 组件尺寸大小
  inputSize() {
    return this.size || this._elFormItemSize || (this.$ELEMENT || {}).size;
  },
  // 组件禁用状态
  inputDisabled() {
    return this.disabled || (this.elForm || {}).disabled;
  },
}


// this.$ELEMENT 来源于组件库的全局注册
const install = function(Vue, opts = {}) { 
  Vue.prototype.$ELEMENT = {
    size: opts.size || '',
    zIndex: opts.zIndex || 2000
  }; 
  // ...
};
```

### 生命周期

在生命周期钩子`created`、`mounted`、`updated`中，添加监听事件，初始化组件状态。

```js
created() {
  // 监听当前实例上的自定义 inputSelect 事件，用于快速选中输入控件的所有内容
  this.$on('inputSelect', this.select);
}, 
mounted() {
  this.setNativeInputValue(); // 设置原生输入控件的value值
  this.resizeTextarea(); // 设置文本域的大小
  this.updateIconOffset(); // 输入框头部/尾部（图标）元素偏移
}, 
// 在数据更改导致的虚拟 DOM 重新渲染和更新完毕之后被调用。
updated() {
  // `updated` 不会保证所有的子组件也都被重新渲染完毕。
  //  使用 `vm.$nextTick` 确保整个视图都被重新渲染之后才会运行的代码
  this.$nextTick(this.updateIconOffset);
} 
```

组件对属性 `value`、 `nativeInputValue`、 `type`添加了侦听器，用于状态更新以及关联表单验证事件。

```js
watch: {
  value(val) {
    this.$nextTick(this.resizeTextarea);
    // 开启输入时是否触发表单的校验
    if (this.validateEvent) {
      // 触发组件`FormItem`的自定义`el.form.change`事件，告知表单字段内容发生改变。
      this.dispatch('ElFormItem', 'el.form.change', [val]);
    }
  },
  // 原生输入控制value值处理
  nativeInputValue() {
    this.setNativeInputValue();
  }, 
  type() {
    // 组件渲染为 input 或者 textarea 
    // 类型切换会导致 DOM 也会发生改变 ，所以使用 `vm.$nextTick`。
    this.$nextTick(() => {
      this.setNativeInputValue();
      this.resizeTextarea();
      this.updateIconOffset();
    });
  }
},
```

下面逐一介绍下代这些方法的功能和作用。

#### select()

方法 `select` 用于选中输入控件的所有内容。

```js
// methods
// 通过模板引用获取input/textarea实例
getInput() {
  return this.$refs.input || this.$refs.textarea;
}, 
// 选中输入控件的所有内容
select() {
  this.getInput().select();
},
```

#### setNativeInputValue()

方法`setNativeInputValue`用于设置原生控件的 `value` 属性，该属性时一个包含了文本框当前文字的`DOMString`。原生控件的`value`值默认是空字符串 (`""`).

计算属性`nativeInputValue` 用于将输入内容格式化成字符串。

```js
nativeInputValue() { 
  return this.value === null || this.value === undefined ? '' : String(this.value);
},
```

在方法`setNativeInputValue`中使用`nativeInputValue`更新属性`value`值。

```js
setNativeInputValue() {
  const input = this.getInput();
  if (!input) return;
  if (input.value === this.nativeInputValue) return;
  input.value = this.nativeInputValue;
},
```

#### resizeTextarea()

方法`resizeTextarea`用来设置文本域的大小，这个讲解 `<textarea>`详细介绍。

#### updateIconOffset()

方法`updateIconOffset` 根据前置/后置内容元素的 `offsetWidth`，在水平方向移动头部/尾部内容元素。

```js
updateIconOffset() {
  this.calcIconOffset('prefix'); //计算头部图标偏移
  this.calcIconOffset('suffix'); //计算尾部图标偏移
},
calcIconOffset(place) {
  // 根据 el-input__prefix/el-input__suffix 选中元素节点
  let elList = [].slice.call(this.$el.querySelectorAll(`.el-input__${place}`) || []); 
  let el = null;
  // 找到当前实例的头部/尾部元素节点
  for (let i = 0; i < elList.length; i++) {
    if (elList[i].parentNode === this.$el) {
      el = elList[i];
      break;
    }
  }
  // 映射关系 头部对应前置， 尾部对应后置
  const pendantMap = {
    suffix: 'append',
    prefix: 'prepend'
  };

  const pendant = pendantMap[place];
  // 根据对应插槽是否传入内容，若传入内容，插槽内容渲染，图标需要移动；否则清除样式 
  if (this.$slots[pendant]) {
    // 尾部元素移动为负值
    el.style.transform = `translateX(${place === 'suffix' ? '-' : ''}${this.$el.querySelector(`.el-input-group__${pendant}`).offsetWidth}px)`;
  } else {
    el.removeAttribute('style');
  }
}, 
```

### v-model

指令`v-model`常用于在表单输入元素（`<input>`、`<textarea>` 、 `<select>`）创建双向绑定数据绑定。它会根据控件类型自动选取正确的方法来更新元素。

它会根据所使用的元素类型自动使用对应的 DOM 属性和事件组合：

* 文本类型的 `<input>` 和 `<textarea>` 元素会绑定 `value` property 并侦听 `input` 事件；
* `<input type="checkbox">` 和 `<input type="radio">` 会绑定 `checked` property 并侦听 `change` 事件；
* `<select>` 会绑定 `value` property 并侦听 `change` 事件。

> `v-model` 会忽略任何表单元素上初始的 `value`、`checked` 或 `selected` attribute。它将始终将当前绑定的 JavaScript 状态视为数据的正确来源。你应该在 JavaScript 中使用`data` 选项来声明该初始值。

`v-model` 本质是个语法糖，以组件`el-input`为例

```html
<el-input v-model="searchText" />
```

上面的代码其实等价于下面这段 (编译器会对 `v-model` 进行展开)：

```html
<input 
  :value="searchText" 
  @input="newValue => searchText = newValue" 
/>
```

所以在组件 `input` 的 `props` 选项里声明 `value` 时必须的，这也是文档中为什么会说属性 `value/v-model` 都是绑定值

当组件`el-input`属性 `value` 最新值需要更新到父组件属性`searchText`时，就会使用`$emit`触发实例 `input` 事件，实现双向绑定。

```js
// @input="handleInput"  
handleInput(event) { 
  // ...
  
  // 触发当前实例上的input事件
  this.$emit('input', event.target.value); 
},
```

所以官方文档这句话 **Input 为受控组件，它总会显示 Vue 绑定值** 也就不难理解了。

### 单行文本输入 input

#### 后置内容

后置内容不止用于展示图标，还提供了内容清空、密码显隐、输入文字统计以及表单验证状态图标等内容。

```html
<!-- 后置内容 -->
<span class="el-input__suffix" v-if="getSuffixVisible()">
  <span class="el-input__suffix-inner">
    <template v-if="!showClear || !showPwdVisible || !isWordLimitVisible">
      // 图标
    </template>
    <!-- 可清空 -->
    <i v-if="showClear" class="el-input__icon el-icon-circle-close el-input__clear"
      @mousedown.prevent
      @click="clear"
    ></i>
    <!-- 密码切换 -->
    <i v-if="showPwdVisible" class="el-input__icon el-icon-view el-input__clear"
      @click="handlePasswordVisible"
    ></i>
    <!-- 输入字数显示 -->
    <span v-if="isWordLimitVisible" class="el-input__count">
      <span class="el-input__count-inner">
        {{ textLength }}/{{ upperLimit }}
      </span>
    </span>
  </span>
  <!-- 表单验证 -->
  <i class="el-input__icon" v-if="validateState"
    :class="['el-input__validateIcon', validateIcon]">
  </i>
</span>
```

后置内容元素渲染由很多数据状态控制。

```js
getSuffixVisible() {
  return this.$slots.suffix ||       // 传入插槽对象
    this.suffixIcon ||               // 设置尾部图标
    this.showClear ||                // 可清空
    this.showPassword ||             // 密码框
    this.isWordLimitVisible ||       // 输入长度限制
    // 输入时触发表单的校验 并显示校验结果图标
    (this.validateState && this.needStatusIcon);
}
```

#### 可清空

使用`clearable`属性即可得到一个可清空的输入框。 计算属性`showClear`根据组件状态、输入内容等判断功能是否开启。

```js
showClear() {
  return this.clearable &&
    !this.inputDisabled &&       // 非禁用
    !this.readonly &&            // 非只读
    this.nativeInputValue &&     // 值不为空
    (this.focused || this.hovering);  // 获得元素焦点或者鼠标悬停
},
```

图标绑定 click 事件 ，调用方法 `clear` ，更新`v-model`值，触发组件实例的`change`、`clear`等自定义事件。

```js
clear() {
  this.$emit('input', '');  // 用于 v-model 更新
  this.$emit('change', '');
  this.$emit('clear');
},
```

#### 密码框

使用`show-password`属性即可得到一个可切换显示隐藏的密码框。

```js
showPwdVisible() {
  return (
    this.showPassword &&
    !this.inputDisabled &&    // 非禁用
    !this.readonly &&         // 非只读
    (!!this.nativeInputValue || this.focused) // 值不为空 或 获得元素焦点
  );
},
```

图标绑定 click 事件 ，调用方法 `handlePasswordVisible` ，更新内部属性`passwordVisible`值，因为密码显隐是通过渲染不同类型 （`text` 或`password`） 的input控件实现，此时DOM会重新渲染，所以使用`$nextTick`，调用方法`focus`重新获取元素的焦点。

```js
// html
:type="showPassword ? (passwordVisible ? 'text' : 'password') : type"

// methods
handlePasswordVisible() {
  this.passwordVisible = !this.passwordVisible;
  this.$nextTick(() => {
    this.focus();
  });
},
// 获取焦点
focus() {
  this.getInput().focus();
},
```

#### 输入长度限制

通过设置 `show-word-limit` 属性来展示字数统计。只能对类型为 `text` 或 `textarea` 的输入框生效， 使用原生`maxlength`属性限制最大输入长度 。

```js
isWordLimitVisible() {
  return (
    this.showWordLimit &&
    this.$attrs.maxlength &&  // 原生`maxlength`属性 透传 attribute
    (this.type === "text" || this.type === "textarea") && //类型为 `text` 或 `textarea`
    !this.inputDisabled &&    // 非禁用
    !this.readonly &&         // 非禁用
    !this.showPassword        // 非密码框
  );
},
```

使用了计算属性 `textLength`、 `upperLimit` 显示了输入进度。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82cec8baf7b7427d81bb388fde98c303\~tplv-k3u1fbpfcp-watermark.image?)

计算属性`upperLimit` 返回最大输入长度，使用`$attrs`获取原生属性 `maxlength`值。

```js
// 最大输入长度
upperLimit() {
  return this.$attrs.maxlength; // 透传 attributes 
},
```

计算属性`textLength`返回当前输入内容的长度

```js
textLength() {
  if (typeof this.value === "number") {
    return String(this.value).length;
  } 
  return (this.value || "").length;
},
```

计算属性 `inputExceed` 判断是否输入超限，用于生成组件根元素的样式`is-exceed`。

```js
inputExceed() { 
  return this.isWordLimitVisible && this.textLength > this.upperLimit;
},
```

#### 表单验证结果反馈图标

当组件在表单中使用，表单`form`设置属性`status-icon`为输入框添加了表示校验结果的反馈图标。

```js
computed: {
  // 表单域下组件的校验状态 校验中/成功/失败
  validateState() {
    return this.elFormItem ? this.elFormItem.validateState : '';
  },
  // 是否在输入框中显示校验结果反馈图标
  needStatusIcon() {
    return this.elForm ? this.elForm.statusIcon : false;
  },
  // 表单域下组件的校验状态图标
  validateIcon() {
    return {
      validating: 'el-icon-loading',
      success: 'el-icon-circle-check',
      error: 'el-icon-circle-close'
    }[this.validateState];
  }, 
} 
```

组件实现渲染效果如下： ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/228be2cd4f204775abe3f98c1fd56953\~tplv-k3u1fbpfcp-watermark.image?)

### 文本域 textarea

组件通过封装原生 `textarea` 控件实现多行文本输入文本域功能。组件做了封装统一处理，所以`textarea` 控件的属性/事件跟`input`控件是相似的。

```html
<div :class="[type === 'textarea' ? '' : 'el-input']">
  <!-- 单行文本输入框 -->
  <template v-if="type !== 'textarea'">
     <!-- 表单输入控件 -->
     <input>
  </template>
  <!-- 多行文本输入的文本域 -->
  <textarea
    v-else
    :tabindex="tabindex"                           // 元素是否可以聚焦 键盘导航
    class="el-textarea__inner" 
    ref="textarea"
    v-bind="$attrs"                                // 透传 Attributes
    :disabled="inputDisabled"                      // 是否禁用
    :readonly="readonly"                           // 是否只读
    :autocomplete="autoComplete || autocomplete"   // 自动补全
    :style="textareaStyle"                         // 自定义样式
    @compositionstart="handleCompositionStart"     // 输入法编辑器 (IME) 事件
    @compositionupdate="handleCompositionUpdate"
    @compositionend="handleCompositionEnd"
    @input="handleInput"                           // 输入内容
    @focus="handleFocus"                           // 获取焦点
    @blur="handleBlur"                             // 失去焦点
    @change="handleChange"                         // 输入值变化
    :aria-label="label"                            // ARIA 无障碍属性
  >
  </textarea> 
  <!-- 展示字数统计 -->
  <span v-if="isWordLimitVisible && type === 'textarea'" class="el-input__count">
    {{ textLength }}/{{ upperLimit }}
  </span>
</div>
```

### 控件缩放

控件`textarea` 控件通过绑定计算属性`textareaStyle`设置内联样式，实现控件的高度自适应、文本区大小可调整。

计算属性`textareaStyle`中使用了内部属性`textareaCalcStyle`和prop属性 `resize`。

* 属性`textareaCalcStyle`值为文本域的高度属性（`height`、 `min-height`）样式，在方法`resizeTextarea`中由文本域内容和配置项计算生成。
* 属性 `resize` 控制文本区是否可调整大小。
  * `none` 元素不能被用户缩放。
  * `both` 允许用户在水平和垂直方向上调整元素的大小。
  * `horizontal` 允许用户在水平方向上调整元素的大小。
  * `vertical` 允许用户在垂直方向上调整元素的大小。

```js
data() {
  return {
    textareaCalcStyle: {}, 
  };
}, 
computed: { 
  textareaStyle() {
    return merge({}, this.textareaCalcStyle, { resize: this.resize });
  }, 
},
methods: { 
  resizeTextarea() {
    // 计算 textareaCalcStyle
  }, 
}, 
```

### 可自适应文本高度

方法 `resizeTextarea`用来改变文本域的高度大小的。实例挂载时会调用该方法，当实例类型、输入框的值改变时也会多次调用该方法。

通过设置 `autosize` 属性可以使得文本域的高度能够根据文本内容自动进行调整，并且 `autosize` 还可以设定为一个对象，指定最小行数和最大行数。

```js
mounted() { 
  this.resizeTextarea(); 
}, 
watch: {
  value(val) {
    this.$nextTick(this.resizeTextarea); 
  }, 
  type() {
    this.$nextTick(() => {
      this.resizeTextarea(); 
    });
  }
},
methods: { 
  resizeTextarea() {
    // 若服务端渲染，方法中断返回
    if (this.$isServer) return;
    const { autosize, type } = this;
    // 此方法只用于 `textarea` 控件
    if (type !== 'textarea') return; 
    // 属性 autosize 未开启自适应内容高度   只计算控件最小高度
    if (!autosize) {
      this.textareaCalcStyle = {
        minHeight: calcTextareaHeight(this.$refs.textarea).minHeight
      };
      return;
    }
    // autosize 也可传入对象，如 { minRows: 2, maxRows: 6 }
    const minRows = autosize.minRows; // 最少行数 
    const maxRows = autosize.maxRows; // 最大行数 
    // 控件自适应高度样式
    this.textareaCalcStyle = calcTextareaHeight(this.$refs.textarea, minRows, maxRows);
  }, 
}, 
```

### calcTextareaHeight.js

类库导出方法`calcTextareaHeight`，用于动态计算文本域的高度样式。

```js
// packages\input\src\calcTextareaHeight.js

let hiddenTextarea; //  一个临时的文本域元素

// 用于隐藏创建的临时的文本域元素 hiddenTextarea
const HIDDEN_STYLE = `
  height:0 !important; 
  // ...
`;

// 指定实例中文本域元素样式属性列表，获取属性值后用于创建临时的文本域元素
const CONTEXT_STYLE = [  
  'width',
   // ...
];

// 获取指定元素节点的样式
function calculateNodeStyling(targetElement) { 
  // ...
}

// 计算文本域的高度
export default function calcTextareaHeight(
  targetElement,
  minRows = 1,
  maxRows = null
) {
  // ...
};
```

#### calculateNodeStyling()

方法 `calculateNodeStyling` 用于获取实例文本域元素节点的样式，并计算`box-sizing`相关属性值。

* `contextStyle` 复制当前实例元素的样式用于创建隐藏文本域。通过 `window.getComputedStyle()` 获取元素计算后/渲染后的所有 CSS 属性的值，然后获取数组`CONTEXT_STYLE`中指定属性值生成样式对象并转化字符串。
* `boxSizing` 获取元素`box-sizing` 属性值。
* `paddingSize` 获取元素上下内边距和。
* `borderSize` 获取元素上下边框宽度和。

```js
// 指定实例中文本域元素样式属性列表，获取属性值后用于隐藏文本域创建
const CONTEXT_STYLE = [ 
  'line-height',
  'padding-top',
  'padding-bottom', 
  'font-weight',
  'font-size', 
  'width',
   // ...
];

// 获取指定元素节点的样式
function calculateNodeStyling(targetElement) {
  // 获取元素计算后/渲染后的所有 CSS 属性的值
  const style = window.getComputedStyle(targetElement);
  // box-sizing 属性定义了如何计算一个元素的总宽度和总高度。
  const boxSizing = style.getPropertyValue('box-sizing');
  // 只是计算高度 获取上下内边距和
  const paddingSize = (
    parseFloat(style.getPropertyValue('padding-bottom')) +
    parseFloat(style.getPropertyValue('padding-top'))
  );
  // 只是计算高度 获取上下边框宽度和
  const borderSize = (
    parseFloat(style.getPropertyValue('border-bottom-width')) +
    parseFloat(style.getPropertyValue('border-top-width'))
  );
  // 获取元素指定的属性值并生成对象
  const contextStyle = CONTEXT_STYLE
    .map(name => `${name}:${style.getPropertyValue(name)}`)
    .join(';');

  return { contextStyle, paddingSize, borderSize, boxSizing };
}
```

#### calcTextareaHeight()

方法 `calcTextareaHeight`通过创建一个跟实例元素一样的临时文本域，用于计算出自适应高度。

1. 创建临时文本域元素 `hiddenTextarea`，获取实例元素的内容和样式值、内容赋值给临时元素，使其作为实例元素的复制镜像。通过样式 `HIDDEN_STYLE` 用于隐藏临时元素使其不可见。
2. 获取元素内容高度`scrollHeight`，根据不同`box-sizing` 属性计算出元素总高度。
3. 计算出单行文本行高度`singleRowHeight`。
4. 如果设置`minRows`，计算出元素属性`min-height`值。
5. 如果设置`maxRows`，计算出元素最大高度，实际 height 不能超过最大高度。
6. 清除临时文本域元素。
7. 返回计算结果，格式 `{ height:20px }` 或`{ height:20px; minHeight:20px; }`。

```js
// 用于隐藏创建的临时的文本域元素 hiddenTextarea
const HIDDEN_STYLE = `
  height:0 !important;
  visibility:hidden !important;
  overflow:hidden !important;
  position:absolute !important;
  z-index:-1000 !important;
  top:0 !important;
  right:0 !important
`;

// 计算文本域内容高度
export default function calcTextareaHeight(
  targetElement,    // 实例文本域元素
  minRows = 1,      // 最小行数
  maxRows = null    // 最大行数
) {
  // 创建一个临时文本域，下面所有的计算都是在其上模拟的
  if (!hiddenTextarea) {
    hiddenTextarea = document.createElement('textarea');
    document.body.appendChild(hiddenTextarea);
  }
  // 获取当前实例元素样式信息
  let {
    paddingSize,
    borderSize,
    boxSizing,
    contextStyle
  } = calculateNodeStyling(targetElement);

  // contextStyle 让临时文本域元素样式跟实例元素保持一致  
  // HIDDEN_STYLE 用于隐藏临时文本域元素
  hiddenTextarea.setAttribute('style', `${contextStyle};${HIDDEN_STYLE}`);
  // 设置临时文本域内容
  hiddenTextarea.value = targetElement.value || targetElement.placeholder || '';

  // 临时文本域内容高度，包括由于溢出导致的视图中不可见内容。 
  let height = hiddenTextarea.scrollHeight;
  const result = {};
  
  // 计算文本内容高度 因为 scrollHeight 包括元素的 padding，但不包括元素的 border 和 margin。
  if (boxSizing === 'border-box') { 
    height = height + borderSize;    // border-box   加上上下边框宽度和
  } else if (boxSizing === 'content-box') { 
    height = height - paddingSize;   // content-box  减去上下内边距和
  }
  
  hiddenTextarea.value = '';  // 清空内容计算单行高度， 
  let singleRowHeight = hiddenTextarea.scrollHeight - paddingSize;   // 单行文本高度
  
  if (minRows !== null) { 
    let minHeight = singleRowHeight * minRows;          // 最小行数高度和
    if (boxSizing === 'border-box') {
      minHeight = minHeight + paddingSize + borderSize; // border + padding + 内容的高度
    }
    height = Math.max(minHeight, height);
    result.minHeight = `${ minHeight }px`;              // 设置样式 { minHeight:20px; }
  }
  if (maxRows !== null) {
    let maxHeight = singleRowHeight * maxRows;          // 最大行数高度和
    if (boxSizing === 'border-box') {
      maxHeight = maxHeight + paddingSize + borderSize; // border + padding + 内容的高度
    }
    height = Math.min(maxHeight, height);               // 选择最小高度           
  }
  result.height = `${height}px`;                        // 设置样式 { height:20px; }
  // 计算结束，清除临时文本域元素
  hiddenTextarea.parentNode && hiddenTextarea.parentNode.removeChild(hiddenTextarea);
  hiddenTextarea = null; 
  // 返回 { height:20px } 或 { height:20px; minHeight:20px; }
  return result;   
};
```

属性`scrollHeight`是一个元素内容高度，包括由于溢出导致的视图中不可见内容。包括元素的 padding，但不包括元素的 border 和 margin。

属性 `box-sizing` 定义了如何计算一个元素的总宽度和总高度。

* `content-box` 默认值，标准盒子模型。`width`与 `height` 只包括内容的宽和高，不包括边框（border），内边距（padding），外边距（margin）。
  * `width` = 内容的宽度
  * `height` = 内容的高度
* `border-box` `width`与 `height` 属性包括内容，内边距和边框，但不包括外边距。
  * `width` = border + padding + 内容的宽度
  * `height` = border + padding + 内容的高度

### 样式实现

组件样式源码 `packages\theme-chalk\src\input.scss` 使用混合指令嵌套生成组件样式。

```scss
// packages\theme-chalk\src\input.scss 

// 生成 .el-textarea
@include b(textarea) {
  // ...
  // 生成 .el-textarea__inner
  @include e(inner) {
    // ...
    // 生成 .el-textarea__inner::placeholder 
    &::placeholder { /* ... */ }
    // 生成 .el-textarea__inner:hover
    &:hover { /* ... */ }
    // 生成 .el-textarea__inner:focus
    &:focus { /* ... */ }
  }
  // 生成 .el-textarea .el-input__count
  & .el-input__count { /* ... */ }
  
  @include when(disabled) {
    // 生成 .el-textarea.is-disabled .el-textarea__inner
    .el-textarea__inner {
      // ...
      // 生成  .el-textarea.is-disabled .el-textarea__inner::placeholder 
      &::placeholder { /* ... */ }
    }
  }
  @include when(exceed) {
    // 生成 .el-textarea.is-exceed .el-textarea__inner
    .el-textarea__inner { /* ... */ }
    // 生成 .el-textarea.is-exceed .el-input__count
    .el-input__count { /* ... */ }
  }
}
// 生成 .el-input
@include b(input) {
  // ...
  // @include scroll-bar;
  // 生成 .el-input .el-input__clear
  & .el-input__clear {
    // ...
    // 生成 .el-input .el-input__clear:hover 
    &:hover { /* ... */ }
  }
  // 生成 .el-input .el-input__count 
  & .el-input__count {
    // ...
    // 生成  .el-input .el-input__count .el-input__count-inner  
    .el-input__count-inner { /* ... */ }
  }
  // 生成 .el-input__inner 
  @include e(inner) {
    // ...
    // 生成 .el-input__inner::-ms-reveal 
    &::-ms-reveal { /* ... */ }
    // 生成 .el-input__inner::placeholder 
    &::placeholder { /* ... */ }
    // 生成 .el-input__inner:hover 
    &:hover { /* ... */ }
    // 生成 .el-input__inner:focus  
    &:focus { /* ... */ }
  }
  // 生成 .el-input__suffix 
  @include e(suffix) { /* ... */ }
  // 生成 .el-input__suffix-inner 
  @include e(suffix-inner) { /* ... */ }
  // 生成 .el-input__prefix 
  @include e(prefix) { /* ... */ }
  // 生成 .el-input__icon 
  @include e(icon) {
    // ...
    // 生成 .el-input__icon:after 
    &:after { /* ... */ }
  }
  // 生成 .el-input__validateIcon  
  @include e(validateIcon) { /* ... */ }
  @include when(active) {
    // 生成.el-input.is-active .el-input__inner 
    .el-input__inner { /* ... */ }
  }
  @include when(disabled) {
    // 生成 .el-input.is-disabled .el-input__inner 
    .el-input__inner {
      // ...
      // 生成 .el-input.is-disabled::placeholder
      &::placeholder { /* ... */ }
    }
    // 生成 .el-input.is-disabled .el-input__icon  
    .el-input__icon { /* ... */ }
  }
  @include when(exceed) {
    // 生成 .el-input.is-exceed .el-input__inner 
    .el-input__inner { /* ... */ }
    
    .el-input__suffix {
      // 生成 .el-input.is-exceed .el-input__suffix .el-input__count  
      .el-input__count { /* ... */ }
    }
  }
  @include m(suffix) {
    // 生成 .el-input--suffix .el-input__inner 
    .el-input__inner { /* ... */ }
  }
  @include m(prefix) {
    // 生成 .el-input--prefix .el-input__inner 
    .el-input__inner { /* ... */ }
  }
  // 生成 .el-input--medium 
  @include m(medium) {
    // ...
    // 生成 .el-input--medium .el-input__inner  
    @include e(inner) { /* ... */ }
    // 生成 .el-input--medium .el-input__icon 
    .el-input__icon { /* ... */ }
  } 
  // small/mini ...
  
}
// 生成 .el-input-group 
@include b(input-group) {
  // ...
  // 生成 .el-input-group > .el-input__inner 
  > .el-input__inner { /* ... */ }
  // 生成 .el-input-group__append, .el-input-group__prepend
  @include e((append, prepend)) {
    // ...
    // 生成 .el-input-group__append:focus,.el-input-group__prepend:focus 
    &:focus { /* ... */ }
    
    /* 生成
    .el-input-group__append .el-button,
    .el-input-group__append .el-select,
    .el-input-group__prepend .el-button,
    .el-input-group__prepend .el-select */
    .el-select,.el-button { /* ... */ }
    
    /* 生成
    .el-input-group__append button.el-button,
    .el-input-group__append div.el-select .el-input__inner,
    .el-input-group__append div.el-select:hover .el-input__inner,
    .el-input-group__prepend button.el-button,
    .el-input-group__prepend div.el-select .el-input__inner,
    .el-input-group__prepend div.el-select:hover .el-input__inner */
    button.el-button,
    div.el-select .el-input__inner,
    div.el-select:hover .el-input__inner {
      // ...
    }
    
    /* 生成
    .el-input-group__append .el-button,
    .el-input-group__append .el-select,
    .el-input-group__prepend .el-button,
    .el-input-group__prepend .el-select */
    .el-button,.el-input { /* ... */ }
  }
  // 生成 .el-input-group__prepend 
  @include e(prepend) { /* ... */ }
  // 生成 .el-input-group__append 
  @include e(append) { /* ... */ }
  // 生成
  @include m(prepend) {
    // 生成 .el-input-group--prepend .el-input__inner 
    .el-input__inner { /* ... */ }
    // 生成 .el-input-group--prepend .el-select .el-input.is-focus .el-input__inner
    .el-select .el-input.is-focus .el-input__inner { /* ... */ }
  }

  @include m(append) {
    // 生成 .el-input-group--append .el-input__inner 
    .el-input__inner { /* ... */ }
    // 生成 .el-input-group--append .el-select .el-input.is-focus .el-input__inner 
    .el-select .el-input.is-focus .el-input__inner { /* ... */ }
  }
}

/** disalbe default clear on IE */
.el-input__inner::-ms-clear { /* ... */ }
```

混合指令`scroll-bar`定义如下：

```scss

/* Scrollbar
 -------------------------- */
@mixin scroll-bar {
  $--scrollbar-thumb-background: #b4bccc;
  $--scrollbar-track-background: #fff;

  &::-webkit-scrollbar {
    z-index: 11;
    width: 6px;

    &:horizontal {
      height: 6px;
    }

    &-thumb {
      border-radius: 5px;
      width: 6px;
      background: $--scrollbar-thumb-background;
    }

    &-corner {
      background: $--scrollbar-track-background;
    }

    &-track {
      background: $--scrollbar-track-background;

      &-piece {
        background: $--scrollbar-track-background;
        width: 6px;
      }
    }
  }
} 
```

### 📚参考&关联阅读

["getComputedStyle",MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle)\
["CSS/box-sizing",MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-sizing)\
["Element/scrollHeight",MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollHeight)\
["表单输入绑定",vuejs](https://v2.cn.vuejs.org/v2/guide/forms.html)\
["input\_event",MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/input\_event)\
["change\_event",MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/change\_event)\
["Input/text",MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input/text)\
["String",MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/String)
