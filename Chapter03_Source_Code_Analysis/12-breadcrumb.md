# 3.12 Breadcrumb 面包屑

### 0x00 简介

组件 `Breadcrumb` 用于显示当前页面的路径，快速返回之前的任意页面。 本文将深入分析源码，剖析其实现原理，耐心读完，相信会对您有所帮助。源码实现详见`packages/breadcrumb/src/` 文件夹下  `breadcrumb.vue`、 `breadcrumb-item.vue` 等组件实现。 🔗 [组件文档 Breadcrumb](https://element.eleme.cn/#/zh-CN/component/breadcrumb) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/blob/dev/packages/breadcrumb/src/)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

面包屑组件有两部分 `<breadcrumb>` 、 `<breadcrumb-item>`，组件源码都在 `packages/breadcrumb/src/` 文件夹下。在项目工程化机制下，每个组件对应各自的文件夹 `component-name`， 定义导出组件并为其扩展 `install` 方法，以 `commonjs2` 规范对每个组件单独打包构建，支持按需引入。

### 0x01 breadcrumb 组件

`breadcrumb.vue` 组件是面包屑组件的容器。

#### template 模板内容

模板创建一个 class 名为`.el-breadcrumb`的`<div>`元素作为包裹容器，提供了匿名插槽，将提供`breadcrumb-item`组件引用内容。

```html
<template>
  <div class="el-breadcrumb" aria-label="Breadcrumb" role="navigation">
    <slot></slot>
  </div>
</template>
```

#### ARIA 无障碍访问

组件添加了 `role` 和 `aria-label` 属性。 `role` 表示当前元素的类型。`aria-label`属性用来给当前元素加上的标签描述。

> `Accessible Rich Internet Applications`  **(ARIA)**  是能够让残障人士更加便利的访问 Web 内容和使用 Web 应用的一套机制。 更多内容详见 ["ARIA",MDN](https://developer.mozilla.org/zh-CN/docs/Web/Accessibility/ARIA)

#### attributes 属性

组件定义了 2 个 prop : `separator` 、`separatorClass` 。 这两个prop用于在`<el-breadcrumb>`标签中设置分隔符的形式。

```js
props: {
  separator: {
    type: String,
    default: '/'
  },
  separatorClass: {
    type: String,
    default: ''
  }
},
```

默认使用`separator`属性，只能是字符串，默认为斜杠`/`。也可通过设置 `separatorClass` 使用相应的 `iconfont` 作为分隔符，格式内容为`el-icon-[icon-name]`，这将使 `separator` 设置失效。具体逻辑实现详见下文章节 **breadcrumb-item 组件#分隔符** 。

#### provide/inject 依赖注入

使用 **依赖注入**，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在其上下游关系成立的时间里始终生效。

`provide` 选项允许指定想要提供给后代组件的数据/方法。这里将提供组件 `breadcrumb` 的 vue 实例 。

```js
// breadcrumb\src\breadcrumb.vue
provide() {
  return {
    elBreadcrumb: this
  };
},
```

在任何后代组件里，使用 `inject` 选项来指定接收添加在这个实例上的 property。`breadcrumb-item`组件访问父级组件`breadcrumb`的实例。

```js
// breadcrumb\src\breadcrumb-item.vue
inject: ['elBreadcrumb'],
```

#### mounted()

实例被挂载后，即 `mounted` 方法中，通过 `$el.querySelectorAll('.el-breadcrumb__item')` 在面包屑组件的DOM元素中查找`<breadcrumb-item>`子节点。为最后一个 `<breadcrumb-item>` 节点添加属性`aria-current="page"`。

```js
// 实例被挂载后
mounted() {
  // $el Vue 实例使用的根 DOM 元素。 
  const items = this.$el.querySelectorAll('.el-breadcrumb__item');
  if (items.length) {
    items[items.length - 1].setAttribute('aria-current', 'page');
  }
}
```

组件DOM渲染如下，指明`<breadcrumb-item>`集合中的最后一个代表当前页面。

```html
<div aria-label="Breadcrumb" role="navigation" class="el-breadcrumb">
    <span class="el-breadcrumb__item">...</span>
    <span class="el-breadcrumb__item">...</span>
    <span class="el-breadcrumb__item" aria-current="page">...</span>
</div>
```

### 0x02 breadcrumb-item 组件

`breadcrumb-item.vue`组件用于实现每一个`el-breadcrumb-item` 子标签。

#### template 模板内容

模板创建一个 class 名为`el-breadcrumb__item`的`<span>`元素作为标签容器,包含 2 个子节点 `标签inerr`、`分隔符`。

```html
<template>
  <span class="el-breadcrumb__item">
    <span
      :class="['el-breadcrumb__inner', to ? 'is-link' : '']"
      ref="link"
      role="link">
      <slot></slot>
    </span>
    <i v-if="separatorClass" class="el-breadcrumb__separator" :class="separatorClass"></i>
    <span v-else class="el-breadcrumb__separator" role="presentation">{{separator}}</span>
  </span>
</template>
```

#### 标签 inner

该节点渲染一个 class 名为`el-breadcrumb__inner`的`<span>`元素, 根据 prop 传入的`to`值动态添加链接样式。当prop属性 `to` 是 `truthy` 时,添加`is-link`样式。 同时提供了匿名插槽自定义标签内容。

`ref` 声明用来给元素注册引用信息。`role="link"` 声明用于识别创建与应用或外部资源的超链接的元素。

**attributes 属性**

组件定义了 2 个 prop `to` 、`replace`用于路由设置。

* `to` 路由跳转对象，同 `vue-router` 的 `to` 。
* `replace` 在使用 to 进行路由跳转时，启用 replace 将不会向 history 添加新记录。

```js
props: {
  to: {},
  replace: Boolean
},
```

**mounted()**

实例被挂载后，根据 `to` 、`replace`的传入值动态配置路由设置。

1. 使用`$refs`获取 `标签 inner`节点标签元素信息。
2. 为节点元素添加`click`监听事件

```js
mounted() { 
  // $refs 只会在组件渲染完成之后生效，并且它们不是响应式的。
  const link = this.$refs.link;
  // span 已声明 role="link" 重复设置
  link.setAttribute('role', 'link1');
  link.addEventListener('click', _ => {
    // 通过在 Vue 根实例的 router 配置传入 router 实例
    const { to, $router } = this;
    // router 实例必须注册 和 路由跳转对象必须设置
    if (!to || !$router) return;
    // $router.replace方法不会向 history 添加新记录- 替换掉当前的 history 记录
    // $router.push方法会向 history 栈添加一个新的记录
    this.replace ? $router.replace(to) : $router.push(to);
  });
}
```

标签监听事件主要用于，点击标签时，判断是否 注册 router 实例 和 设置路由跳转对象（prop 的 `to`）,若都满足，则根据 `replace` 在使用 to 进行路由跳转时使用不同编程式的导航 。

| 声明式                               | 编程式                   | 描述                                               |
| --------------------------------- | --------------------- | ------------------------------------------------ |
| `<router-link :to="...">`         | `router.push(...)`    | 向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL。 |
| `<router-link :to="..." replace>` | `router.replace(...)` | 不会向 history 添加新记录，替换掉当前的 history 记录。             |

#### 分隔符

当属性 `separatorClass` 是 `truthy` 时，使用相应的 `iconfont` 作为分隔符。

否则使用`separator`属性，默认为斜杠`/`，此时渲染的 `<span>` 元素， 其`role="presentation"` 属性声明用于从元素及其子元素中删除语义含义。

```js
<i v-if="separatorClass" class="el-breadcrumb__separator" :class="separatorClass"></i>
<span v-else class="el-breadcrumb__separator" role="presentation">{{separator}}</span>
```

`separator`、`separatorClass`属性值通过依赖注入从父组件获取实例，在 `mounted` 方法中获取父组件的prop中的属性值进行赋值。

```js
data() {
  return {
    separator: '',
    separatorClass: ''
  };
},
// 接收父组件提供的实例
inject: ['elBreadcrumb'],
// 实例被挂载后
mounted() {
  // 分割线的设置，获取父级组件的prop属性值
  this.separator = this.elBreadcrumb.separator;
  this.separatorClass = this.elBreadcrumb.separatorClass; 
}
```

***

### 0x03 组件样式

#### src/breadcrumb.scss

组件样式源码 `packages\theme-chalk\src\breadcrumb.scss` 使用混合指令 `b`、`e`、`utils-clearfix`嵌套生成组件样式。

混合指令`utils-clearfix` 用于清除浮动。

```scss
@mixin utils-clearfix {
  $selector: &;

  @at-root {
    #{$selector}::before,
    #{$selector}::after {
      display: table;
      content: "";
    }
    #{$selector}::after {
      clear: both
    }
  }
}
```

`breadcrumb.scss` 生成逻辑如下。

```scss
// 生成 .el-breadcrumb
@include b(breadcrumb) {
  //...
  
  // 生成 .el-breadcrumb::before,.el-breadcrumb::after { /*...*/ } 
  // 生成 .el-breadcrumb::after { /*...*/ }
  @include utils-clearfix;
  
  // 生成 .el-breadcrumb__separator
  @include e(separator) {
    //...
    
    // 生成 .el-breadcrumb__separator[class*=icon]
    &[class*=icon] {
      //...
    }
  }
  // 生成 .el-breadcrumb__item
  @include e(item) {
    //...
    
    // 生成 .el-breadcrumb__inner
    @include e(inner) {
      //...
      
      // 生成  .el-breadcrumb__inner.is-link, .el-breadcrumb__inner a
      &.is-link, & a {
        //...
        
        // 生成  .el-breadcrumb__inner.is-link:hover, .el-breadcrumb__inner a:hover
        &:hover {
          //...
        }
      }
    }
    

    /*  生成
    .el-breadcrumb__item:last-child .el-breadcrumb__inner, 
    .el-breadcrumb__item:last-child .el-breadcrumb__inner a, 
    .el-breadcrumb__item:last-child .el-breadcrumb__inner a:hover, 
    .el-breadcrumb__item:last-child .el-breadcrumb__inner:hover */
    &:last-child {
      .el-breadcrumb__inner,
      .el-breadcrumb__inner a {
        &, &:hover {
          //...
        }
      }
      
      // 生成  .el-breadcrumb__item:last-child .el-breadcrumb__separator { display: none; }
      .el-breadcrumb__separator {
        display: none;
      }
    }
  }
}
```

> 虽然定义了 `breadcrumb-item.scss`样式文件，但是所有的样式逻辑都在`breadcrumb.scss`中实现。

#### lib/breadcrumb.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\breadcrumb.scss`，内容格式如下。

```css
.el-breadcrumb { /*...*/ }
.el-breadcrumb::after,.el-breadcrumb::before { /*...*/ }
.el-breadcrumb::after { /*...*/ }
.el-breadcrumb__separator { /*...*/ }
.el-breadcrumb__separator[class*="icon"] { /*...*/ }
.el-breadcrumb__item { /*...*/ }
.el-breadcrumb__inner { /*...*/ }
.el-breadcrumb__inner a,.el-breadcrumb__inner.is-link { /*...*/ }
.el-breadcrumb__inner a:hover,.el-breadcrumb__inner.is-link:hover { /*...*/ }

.el-breadcrumb__item:last-child .el-breadcrumb__inner,
.el-breadcrumb__item:last-child .el-breadcrumb__inner a,
.el-breadcrumb__item:last-child .el-breadcrumb__inner a:hover,
.el-breadcrumb__item:last-child .el-breadcrumb__inner:hover {
  /*...*/
}
/* 最后标签的分隔符不显示 */ 
.el-breadcrumb__item:last-child .el-breadcrumb__separator { display: none; }
```

### 0x04 📚参考

["依赖注入",vuejs.org](https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)\
[“组件注入”，vue-router](https://router.vuejs.org/zh/api/#%E6%B3%A8%E5%85%A5%E7%9A%84%E5%B1%9E%E6%80%A7)\
[“Using ARIA: Roles, states, and properties”，MDN](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA\_Techniques)\
[“Breadcrumb Example”，w3.org](https://www.w3.org/TR/wai-aria-practices-1.1/examples/breadcrumb/index.html)\
["ref",vuejs.org](https://cn.vuejs.org/v2/api/#ref)
