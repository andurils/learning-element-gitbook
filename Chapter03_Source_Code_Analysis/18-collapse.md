# 3.18 Collapse 折叠面板

### 简介

组件 `Collapse` 可以折叠/展开的内容区域。 本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。

`Collapse` 折叠面板组件有两部分 `<el-collapse>` 、 `<el-collapse-item>`。组件源码详见`packages/collapse/src/`  文件夹下  `collapse.vue`、 `collapse-item.vue` 等组件。在项目工程化机制下，每个组件对应各自的文件夹 `component-name`， 定义导出组件并为其扩展 `install` 方法，以  `commonjs2`  规范对每个组件单独打包构建，支持按需引入。🔗 [组件文档 Collapse](https://element.eleme.cn/#/zh-CN/component/collapse) 🔗 [gitee 源码](https://gitee.com/ElemeFE/element/blob/dev/packages/collapse/src/)

本专栏的 `gitbook` 版本地址已经发布 [anduril.gitbook.io/learning-el…](https://link.juejin.cn/?target=https%3A%2F%2Fanduril.gitbook.io%2Flearning-element%2F) ，内容同步更新中！

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### collapse 组件

`collapse.vue` 是折叠面板的顶层组件。

#### template 模板内容

模板创建一个 class 名为`.el-collapse`的`<div>`元素作为包裹容器，提供了匿名插槽，将提供`collapse-item`组件引用内容。

```html
<template>
  <div class="el-collapse" role="tablist" aria-multiselectable="true">
    <slot></slot>
  </div>
</template>
<script>
  export default {
    name: "ElCollapse",
    componentName: "ElCollapse",
  };
</script>
```

#### ARIA 无障碍访问

`role` 表示当前元素的类型。`aria-multiselectable`表示当前元素是否可多选。

#### attributes 属性

组件定义了 2 个 prop : `accordion` 、`value` 。

* `accordion` 用于是否开启手风琴模式。
* `value` 用于当前激活的面板，默认值为 `[]`,如果是手风琴模式，绑定值类型需要为 string，否则为 array。

```js
props: {
  accordion: Boolean,
  value: {
    type: [Array, String, Number],
    default() {
      return [];
    }
  }
},
data() {
  return {
    activeNames: [].concat(this.value),
  };
},
watch: {
  value(value) {
    this.activeNames = [].concat(value);
  }
},
```

组件的 `activeNames` 属性表示激活的面板数组（可同时展开多个面板），将`value` 转化为数组（因为`value` 会传入 `String`, `Number`值 ）。 `value` 添加了监听器用于更新 `activeNames` 。

#### provide/inject 依赖注入

使用 **依赖注入**，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在其上下游关系成立的时间里始终生效。

将 `collapse` 组件实例提供给后代组件。

```js
// collapse\src\collapse.vue
provide() {
  return {
    collapse: this
  };
},
```

在任何后代组件里，使用  `inject`  选项来指定接收添加在这个实例上的 property。`collapse-item`组件访问父级组件`collapse`的实例。

```js
// collapse\src\collapse-item.vue
inject: ['collapse'],
```

#### created()

组件实例创建后，注册 `item-click`事件监听，回调函数 `handleItemClick`更新激活的面板。

```js
created() {
  this.$on("item-click", this.handleItemClick);
},
```

子组件中调用 `dispatch`触发`collapse` 组件的`item-click`事件。

```js
// collapse\src\collapse-item.vue
this.dispatch("ElCollapse", "item-click", this);
```

#### methods & events

`handleItemClick`方法用于`item-click`事件回调。根据是否手风琴模式根据点击选项更新最`activeNames` 值。

```js
handleItemClick(item) {
  // 手风琴模式
  if (this.accordion) {
    this.setActiveNames(
      // 手风琴模式下 activeNames 长度为 1，元素类型为 `String`  `Number`
      // 判断当前是否存在激活面板 也就是 activeNames 存在元素
      // '' 表示面板关闭   item.nam 激活指定面板
      (this.activeNames[0] || this.activeNames[0] === 0) &&
      this.activeNames[0] === item.name
        ? '' : item.name
    );
  } else {
    // 同时展开多个面板
    let activeNames = this.activeNames.slice(0);
    // 获取当前tab在activeNames
    let index = activeNames.indexOf(item.name);
    if (index > -1) {
      // itemname 在 activeNames,说明已经打开，点击后关闭
      activeNames.splice(index, 1);
    } else {
      // itemname 不在 activeNames,点击后打开
      activeNames.push(item.name);
    }
    this.setActiveNames(activeNames);
  }
}
```

`setActiveNames` 方法用于更新 `activeNames`值,调用 `$emit`触发当前组件实例上的`change`事件 。

```js
// 更新activeNames 激活的面板
setActiveNames(activeNames) {
  activeNames = [].concat(activeNames);
  // activeNames: array / string
  // 如果是手风琴模式，参数 activeNames 类型为string，否则为array
  let value = this.accordion ? activeNames[0] : activeNames;
  this.activeNames = activeNames;
  this.$emit('input', value);  // 更新v-model
  this.$emit('change', value);
},
```

### Collapse-item 组件

`Collapse-item.vue`用于渲染面板内容。

#### template 模板内容

模板创建一个 class 名为 `el-collapse-item` 的 `div` 元素作为标签容器,包含 2 个子节点  `tab` 面板标题、`tabpanel` 面板内容。

```html
<template>
  <div
    class="el-collapse-item"
    :class="{'is-active': isActive, 'is-disabled': disabled }"
  >
    <!-- `tab` 面板标题 -->
    <div>...</div>
    <el-collapse-transition>
      <!-- `tabpanel` 面板内容 -->
      <div>...</div>
    </el-collapse-transition>
  </div>
</template>
```

#### 根节点

根据传入 prop `disabled` 动态添加禁用样式 `is-disabled`。

根据计算属性 `isActive` 动态添加激活样式 `is-active`。

计算属性`isActive`通过注入获取顶层组件中属性`activeNames`，判断当前面板是否激活。

```js
computed: {
  isActive() {
    return this.collapse.activeNames.indexOf(this.name) > -1;
  }
},
```

#### `tab` 面板标题

该节点是一个设置了`role`,`aria-expanded`,`aria-controls`,`aria-describedby`等 ARIA 属性的 `div`包裹容器，里面嵌套了一个 class 名为 `el-collapse-item__header` 的 `div` 元素，用于展示面板标题。

面板标题节点元素内部包含了：

* 一个名为`title`具名插槽设置，使用 prop 的 `title`值作为其的后备；
* 一个图标用于展示面板的激活状态。

```html
<div
  role="tab"
  :aria-expanded="isActive"
  :aria-controls="`el-collapse-content-${id}`"
  :aria-describedby="`el-collapse-content-${id}`"
>
  <div
    class="el-collapse-item__header"
    @click="handleHeaderClick"
    role="button"
    :id="`el-collapse-head-${id}`"
    :tabindex="disabled ? undefined : 0"
    @keyup.space.enter.stop="handleEnterClick"
    :class="{
      'focusing': focusing,
      'is-active': isActive
    }"
    @focus="handleFocus"
    @blur="focusing = false"
  >
    <slot name="title">{{title}}</slot>
    <i
      class="el-collapse-item__arrow el-icon-arrow-right"
      :class="{'is-active': isActive}"
    >
    </i>
  </div>
</div>
```

当面板被禁用， 元素不可聚焦(`tabindex="0"` 表示元素是可聚焦的)。

```bash
:tabindex="disabled ? undefined : 0"
```

按键修饰符 `keyup` `space``enter`用于监听键盘事件。事件修饰符`stop`阻止单击事件继续传播。

方法`handleEnterClick` 用于触发顶层组件的 `item-click` 事件 。

```bash
// @keyup.space.enter.stop="handleEnterClick"

handleEnterClick() {
  this.dispatch('ElCollapse', 'item-click', this);
}
```

添加了 `click` `focus` `blur` 等事件监听。

```bash
@click="handleHeaderClick"
@focus="handleFocus"
@blur="focusing = false"

handleFocus() {
  setTimeout(() => {
    if (!this.isClick) {
      this.focusing = true;  // 聚焦状态
    } else {
      this.isClick = false;  // 点击状态
    }
  }, 50);
},
// 点击面板标题后，更新 focusing isClick，触发事件 item-click
handleHeaderClick() {
  if (this.disabled) return;
  this.dispatch('ElCollapse', 'item-click', this);
  this.focusing = false;
  this.isClick = true;
},
```

折叠状态切换时，图标动效使用 `rotate` 旋转实现。

```css
.el-collapse-item__arrow.is-active {
  transform:rotate(90deg); 
}
```

#### `tabpanel` 面板内容

该节点一个 class 名为 `el-collapse-item__wrap` 的 `div` 元素作为内容容器,包含一个子节点 class 名为 `el-collapse-item__content` 的 `div` 元素，里面提供了匿名插槽，用于面板内容展示。

该节点使用了 `collapse-transition` 组件，用于实现折叠展开过渡效果。根据面板激活状态(计算属性`isActive`)进行显隐 `v-show="isActive"`

```html
<el-collapse-transition>
  <div
    class="el-collapse-item__wrap"
    v-show="isActive"
    role="tabpanel"
    :aria-hidden="!isActive"
    :aria-labelledby="`el-collapse-head-${id}`"
    :id="`el-collapse-content-${id}`"
  >
    <div class="el-collapse-item__content">
      <slot></slot>
    </div>
  </div>
</el-collapse-transition>
```

### 组件样式

组件样式源码 `packages\theme-chalk\src\collapse.scss`使用混合指令嵌套生成组件样式。

```scss
// packages\theme-chalk\src\collapse.scss

// 生成 .el-collapse 
@include b(collapse) { /* ... */ }

@include b(collapse-item) { 
  @include when(disabled) {
    // 生成 .el-collapse-item.is-disabled .el-collapse-item__header 
    .el-collapse-item__header { /* ... */ }
  }
  // 生成 .el-collapse-item__header 
  @include e(header) {
    // ...
    
    // 生成 .el-collapse-item__arrow  
    @include e(arrow) {
      // ...
      // 生成  .el-collapse-item__arrow.is-active 
      @include when(active) { /* ... */ }
    }
    // 生成 .el-collapse-item__header.focusing:focus:not(:hover) 
    &.focusing:focus:not(:hover){ /* ... */ }
    // 生成 .el-collapse-item__header.is-active 
    @include when(active) { /* ... */ }
  }
  // 生成 .el-collapse-item__wrap 
  @include e(wrap) { /* ... */ }
  // 生成 .el-collapse-item__content 
  @include e(content) { /* ... */ }
  // 生成 .el-collapse-item:last-child 
  &:last-child { /* ... */ }
}
```

### 📚参考&关联阅读

["依赖注入",vuejs.org](https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)\
[“组件注入”，vue-router](https://router.vuejs.org/zh/api/#%E6%B3%A8%E5%85%A5%E7%9A%84%E5%B1%9E%E6%80%A7)\
["ARIA(Accessible Rich Internet Applications)",MDN](https://developer.mozilla.org/zh-CN/docs/Web/Accessibility/ARIA)
