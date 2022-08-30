# 3.7 Llink 文字链接

### 0x00 简介

本文将深入分析组件 `Link` 源码，剖析其实现原理，耐心读完，相信会对您有所帮助。 [组件文档 Link](https://element.eleme.cn/#/zh-CN/component/link)

`packages/link/src/main.vue` 文件是组件源码实现。 [github源码 main.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/link/src/main.vue)

### 0x01 组件源码

#### template 模板内容

从模板内容看出，组件封装一个 `<a>` 元素，包含3个子节点：

1. 图标节点,定义 `icon`，渲染此节点 ；
2. 提供了 `slot` 的 `<span>` 元素，`class` 名为`el-link--inner`（**此样式规则未定义，无效样式**），没有设置后备内容（默认值）；
3. 具名插槽 `icon`。

3个子节点都设置 `v-if`判断是否渲染节点元素。

```js
<template>
  <a
    :class="[
      'el-link',
      type ? `el-link--${type}` : '',
      disabled && 'is-disabled',
      underline && !disabled && 'is-underline'
    ]"
    :href="disabled ? null : href"
    v-bind="$attrs"
    @click="handleClick"
  >
    <!-- 图标 -->
    <i :class="icon" v-if="icon"></i>
    <!-- 不带 name 的 <slot> 出口会带有隐含的名字“default”。 -->
    <span v-if="$slots.default" class="el-link--inner">
      <slot></slot>
    </span>
    <!-- 具名插槽 icon -->
    <template v-if="$slots.icon"><slot v-if="$slots.icon" name="icon"></slot></template>  
  </a>
</template>
```

根据组件prop 动态添加 `class`。

* `'el-link'` 组件默认样式。
* `type ? 'el-link--${type}' : ''` 设置组件文字不同类型颜色。若 `type` 值不是以下`default/primary / success / warning / danger / info`其中一个，设置无效(生成无效的class)。
* `disabled && 'is-disabled'` 禁用状态下样式。
* `underline && !disabled && 'is-underline'` 文字链接下划线样式，**禁用状态下无效**。

> **逻辑与(&&)** 看左边的值是真还是假。如果值是真,返回的是右边的值,如果值是假；返回的是左边的值(**只有false 、0、NaN、null、undefined、空字符串为假, 其余都是真**)

`:href="disabled ? null : href"` 禁用状态下值为 `null`,设置无效。

`@click="handleClick"` 监听`click`事件提供处理方法。

**接收额外属性**

`$attrs` 包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (`class` 和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件。

**具名插槽**

组件提供了两个插槽，一个在默认插槽，一个具名插槽 `icon`。

```html
<template>
    ...
    <!-- 具名插槽 icon --> 
    <template v-if="$slots.icon">
        <slot v-if="$slots.icon" name="icon"></slot>
    </template>
    ...
</template>
```

具名插槽 `icon` 使用了多层 `<template>` 标签嵌套,`<template>` 不会渲染成元素，用 `div`的话会被渲染成元素。把 `v-if`、`v-show`、`v-for` 等抽取出来放在`<template>`上面，把绑定的事件放在`<template>`里面的元素上，可以使结构更加清晰，还可以改善元素嵌套过深。

具名插槽的使用示例，使用 `<template slot="name">`指定名称。

```html
<template>
  <div style="display: flex;flex-direction: column;">
    <!-- prop underline  没有值，意味着 `true`。-->
    <el-link underline>
      查看 <i class="el-icon-view el-icon--right"></i>
      <template slot="icon">
        <h3>Link</h3>
      </template>
    </el-link>
    <el-link underline>
      <template slot="default">
        查看 <i class="el-icon-view el-icon--right"></i>
      </template>
      <template slot="icon">
        <h3>Link</h3>
      </template>
    </el-link>
  </div>
</template>
```

两种使用方式渲染内容相同。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0caad7ad8c7046a0815b340b5d95abaf\~tplv-k3u1fbpfcp-watermark.image)

#### attributes 属性

组件提供了5个 `prop`。

```js
props: {
    type: {
      type: String,
      default: 'default'
    },
    underline: {
      type: Boolean,
      default: true
    },
    disabled: Boolean,
    href: String,
    icon: String
},
```

`prop`详细描述如下：

| 参数        | 说明         | 类型      | 可选值                                         | 默认值     |
| --------- | ---------- | ------- | ------------------------------------------- | ------- |
| type      | 类型         | string  | primary / success / warning / danger / info | default |
| underline | 是否下划线      | boolean | —                                           | true    |
| disabled  | 是否禁用状态     | boolean | —                                           | false   |
| href      | 原生 href 属性 | string  | —                                           | -       |
| icon      | 图标类名       | string  | —                                           | -       |

#### events 事件

组件提供了 `click`事件 。

```js
handleClick(event) {
  // 非禁用状态
  if (!this.disabled) {
    // href未定义值
    if (!this.href) {
      // 触发当前实例上的事件
      this.$emit('click', event);
    }
  }
}
```

`click`事件只有非禁用状态且`href`未定义值的状态下生效，用于需要代码处理页面跳转的场景。

```html
<template>
  <div>
    <el-link @click="goToPage">默认链接</el-link>
  </div>
</template>
<script>
export default {
  methods: {
    goToPage(e) {
      // url redirect
    },
  },
};
</script> 
```

### 0x02 组件样式

#### src/link.scss

组件样式源码 `packages\theme-chalk\src\link.scss` 使用 `scss` 的混合指令 `b`、 `when` 嵌套生成组件样式。

```scss

// Maps 可视为键值对的集合
$typeMap: (
  primary: $--link-primary-font-color,
  danger: $--link-danger-font-color,
  success: $--link-success-font-color,
  warning: $--link-warning-font-color,
  info: $--link-info-font-color,
);

// 生成 .el-link
@include b(link) {
  // ...

  // 生成.el-link.is-underline:hover:after
  @include when(underline) {
    &:hover:after {
      // ...
    }
  }
  // 生成 .el-link.is-disabled
  @include when(disabled) {
    // ...
  }

  // 生成 .el-link [class*=el-icon-] + span
  & [class*="el-icon-"] {
    & + span {
      // ...
    }
  }
  // 生成 .el-link.el-link--default
  &.el-link--default {
    // ...

    // .el-link.el-link--default:hover
    &:hover {
      // ...
    }

    // 生成  .el-link.el-link--default:after
    &:after {
      // ...
    }
    // 生成 .el-link.el-link--default.is-disabled
    @include when(disabled) {
      // ...
    }
  }

  @each $type, $primaryColor in $typeMap {
    // 生成 .el-link.el-link--[type]
    &.el-link--#{$type} {
      // ...
      
      // 生成 .el-link.el-link--[type]:hover
      &:hover {
        // ...
      }
      // 生成 .el-link.el-link--[type]:after
      &:after {
        // ...
      }
      // 生成 .el-link.el-link--[type].is-disabled
      @include when(disabled) {
        // ...
      }
      // 生成 .el-link.el-link--[type].is-disabled:hover:after
      @include when(underline) {
        &:hover:after {
          // ...
        }
      }
    }
  }
}
```

#### lib/link.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\link.scss`，内容格式如下。

```css
.el-link {
  //...
}
.el-link.is-underline:hover:after {
   //...
}
.el-link.is-disabled {
   //...
}
.el-link [class*="el-icon-"] + span {
   //...
} 

.el-link.el-link--default:after,
.el-link.el-link--primary.is-underline:hover:after,
.el-link.el-link--primary:after {
   //...
} 
/* ------------default------------- */
.el-link.el-link--default {
   //...
}
.el-link.el-link--default:hover {
   //...
}
.el-link.el-link--default.is-disabled {
   //...
}
/* ------------primary------------- */
.el-link.el-link--primary {
  //...
}
.el-link.el-link--primary:hover {
  //...
}
.el-link.el-link--primary.is-disabled {
  //...
}  

/* type danger
-------------------------- */
.el-link.el-link--danger.is-underline:hover:after,
.el-link.el-link--danger:after {
  border-color: #f56c6c;
}
.el-link.el-link--danger {
  color: #f56c6c;
}
.el-link.el-link--danger:hover {
  color: #f78989;
}
.el-link.el-link--danger.is-disabled {
  color: #fab6b6;
}
/* type success
-------------------------- */
// ...

/* type warning
-------------------------- */
// ... 

/* type info
-------------------------- */
// ... 
 
```

### 0x03 📚参考

["vm-attrs",vuejs.org](https://cn.vuejs.org/v2/api/#vm-attrs) [“JavaScript 逻辑运算规则 ”, cnblogs](https://www.cnblogs.com/rencoo/p/9384710.html)
