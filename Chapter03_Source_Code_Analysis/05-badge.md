# 3.5 Badge 标记

### 0x00 简介

组件 `Badge` 一般和其它组件配合使用，用于进数字或状态标记提醒效果。

本文将深入分析组件 `Badge` 源码，剖析其实现原理，耐心读完，相信会对您有所帮助。 [组件文档 Badge](https://element.eleme.cn/#/zh-CN/component/badge)

`packages/badge/src/main.vue` 文件是组件源码实现。 [github 源码 main.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/badge/src/main.vue)

### 0x01 template 模板

从模板内容看出，组件封装一个 `<div>` 元素，包含两项内容：

1. 提供了  `slot` ，没有设置后备内容（默认值）；
2. 使用了内置过渡动画的`<sup>`元素,用于定义上标文本。

```js
<template>
  <div class="el-badge">
    <slot></slot>
    <transition name="el-zoom-in-center">
      <sup
        v-show="!hidden && (content || content === 0 || isDot)"
        v-text="content"
        class="el-badge__content"
        :class="[
          'el-badge__content--' + type,
          {
            'is-fixed': $slots.default,
            'is-dot': isDot
          }
        ]">
      </sup>
    </transition>
  </div>
</template>
```

组件渲染后生成一个 class 值为 `el-badge` 的 `<div>` 元素作为容器。`slot` 没有设置后备内容， 只有在组件包裹内容时才会进行渲染。生成一个 class 值为 `el-badge__content` 的`<sup>` 元素定义上标文本，用于实现数字或状态标记效果，使用内置组件  `transition`  实现缩放`zoom-in-center`效果。

`<sup>` 的可见性由组件 prop `hidden`、`isDot` 和 计算属性`content` 进行控制。文本内容为`content`属性值。根据插槽传值`$slots.default`、`type`、`isDot`等属性值动态添加 class，实现不同场景的形式展示。

### 0x02 attributes 属性

组件提供了 5 个 `prop`。

```js
<script>
export default {
  name: 'ElBadge',

  props: {
    value: [String, Number], // 显示值
    max: Number, // 最大值，超过最大值会显示 '{max}+'
    isDot: Boolean, // 是否显示小圆点
    hidden: Boolean, // 是否隐藏 badge
    // 类型
    type: {
      type: String,
      validator(val) {
        return ['primary', 'success', 'warning', 'info', 'danger'].indexOf(val) > -1;
      }
    }
  },

  computed: {
    // 计算属性 上标显示内容  根据 isDot、value、max进行判断计算。
    content() {
      // ...
    }
  }
};
</script>
```

#### 属性描述

各属性功能描述如下：

| 参数     | 说明                                         | 类型             | 可选值                                         | 默认值   |
| ------ | ------------------------------------------ | -------------- | ------------------------------------------- | ----- |
| value  | 显示值                                        | string, number | —                                           | —     |
| max    | 最大值，超过最大值会显示 '{max}+'，要求 value 是 Number 类型 | number         | —                                           | —     |
| is-dot | 小圆点                                        | boolean        | —                                           | false |
| hidden | 隐藏 badge                                   | boolean        | —                                           | false |
| type   | 类型                                         | string         | primary / success / warning / danger / info | —     |

`type` 提供了 `primary / success / warning / danger / info`可选值，提供了自定义验证函数。

```js
// 自定义验证函数
type: {
  validator(val) {
    // 这个值必须匹配下列字符串中的一个
    return ['primary', 'success', 'warning', 'info', 'danger'].indexOf(val) > -1;
  }
}
```

当 prop 验证失败的时候，(开发环境构建版本的) Vue 将会产生一个控制台的警告。 此时 `type` 值为`undefined` 。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f9998a83a0c4a26ac8cf7592575be09\~tplv-k3u1fbpfcp-watermark.image)

#### 计算属性

计算属性`content`值是上标显示内容。 根据 `isDot`、`value`、`max` 等 prop 进行判断计算。

```js
  computed: {
    content() {
      // 小红点形式
      if (this.isDot) return;

      const value = this.value;
      const max = this.max;

      // 数值超过最大值处理
      if (typeof value === 'number' && typeof max === 'number') {
        return max < value ? `${max}+` : value;
      }

      return value;
    }
  }
```

若使用小圆点形式， `isDot` 值为 `true`， `if (this.isDot) return;` ，`content`值为 `undefined`。

若不使用小圆点形式时，`value`是 `Number` 类型且设置了 `max`,判断是否超过最大值，超过会显示 `{max}+`，其余情况 `content`值等于 `value`值。

#### `<sup>`显隐逻辑

`v-show` 指令条件 `!hidden && (content || content === 0 || isDot)` 短路表达式是可知，若 `hidden` 值为`true`隐藏不显示，后续条件不执行。

若显示上标`hidden` 值为`false`，则继续根据显示内容 `content`是否存在内容进行判断， `content === 0` 逻辑是为了防止 当 `value` 为 0 时，表达式 `(content)` 返回 `false`。 使用圆点形式时，由上文可知`content`值 `undefined`，`isDot` 值为`true`，此时表达式返回 `true`。

#### 动态 class

根据 `type` 会动态的设置 `el-badge__content--[type]`，用于不同类型的上标颜色显示 `el-badge__content--[primary/success/warning/danger/info]`。当 `type` 未设置或者不是指定值时,class 值为`el-badge__content--undefined` 。

当组件和其它组件配合使用，包裹内容时提供给插槽值 `$slots.default` 为 `[Vnode]`,设置 `is-fixed`固定上标位置-右上角。

若使用小圆点形式时，设置`is-dot`控制上标的样式为红点。

### 0x03 组件样式

#### src/badge.scss

组件样式源码 `packages\theme-chalk\src\badge.scss` 使用 `scss` 的混合指令 `b`、 `e`、 `m`、 `when` 嵌套生成组件样式。

```

// 生成 el-badge
@include b(badge) {
  // ...

  // 生成 el-badge__content
  @include e(content) {
    // ...

    // 生成 el-badge__content.is-fixed
    @include when(fixed) {
      // ...

      // 生成 el-badge__content.is-fixed.is-dot
      @include when(dot) {
        // ...
      }
    }

    // 生成 el-badge__content.is-fixed.is-dot
    @include when(dot) {
      // ...
    }

    @each $type in (primary, success, warning, info, danger) {
      // 生成 .el-badge__content--[primary/success/warning/info/danger]
      @include m($type) {
        // ...
      }
    }
  }
}
```

#### lib/badge.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\badge.scss`，内容格式如下。

```css
.el-badge {
  //...
}
.el-badge__content {
  //...
}
.el-badge__content.is-fixed {
  // ...
}
.el-badge__content.is-fixed.is-dot {
  // ...
}
.el-badge__content.is-dot {
  // ...
}
.el-badge__content--primary {
  background-color: #409eff;
}
.el-badge__content--success {
  background-color: #67c23a;
}
.el-badge__content--warning {
  background-color: #e6a23c;
}
.el-badge__content--info {
  background-color: #909399;
}
.el-badge__content--danger {
  background-color: #f56c6c;
}
```

### 0x04 📚 参考

["sass 在线编辑器",sassmeister](https://www.sassmeister.com/)
