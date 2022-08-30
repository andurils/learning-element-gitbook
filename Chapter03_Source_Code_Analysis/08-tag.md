# 3.8 Tag 标签

### 0x00 简介

组件`Tag` 多用于标记和分类。 本文将深入分析组件源码，剖析其实现原理，耐心读完，相信会对您有所帮助。`packages/tag/src/tag.vue` 文件是组件源码实现。 🔗 [组件文档 Tag](https://element.eleme.cn/#/zh-CN/component/tag) 🔗 [github 源码 tag.vue](https://github1s.com/ElemeFE/element/blob/dev/packages/tag/src/tag.vue)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 0x01 组件源码

使用 **渲染函数**  创建组件，代码结构如下 👇。

```js
<script>
  export default {
    name: 'ElTag',
    // 组件 prop
    props: {
      // ...
    },
    methods: {
      // 关闭 Tag 时触发的事件
      handleClose(event) {
        // ...
      },
      // 点击 Tag 时触发的事件
      handleClick(event) {
        // ...
      }
    },
    // 计算属性
    computed: {
      // 标签尺寸
      tagSize() {
        // ...
      }
    },
    // 渲染 标签 虚拟DOM
    render(h) {
      // ...
    }
  };
</script>
```

#### attributes 属性

组件定义了 8 个 `prop`。

{% code lineNumbers="true" %}
```js
props: {
  text: String,  // 代码中没有被使用
  closable: Boolean, // 是否可关闭
  type: String, // 类型
  hit: Boolean, // 是否有边框描边
  disableTransitions: Boolean, // 是否禁用渐变动画
  color: String, // 背景色
  size: String, // 尺寸
  // 主题
  effect: {
    type: String,
    default: 'light',
    validator(val) {
      return ['dark', 'light', 'plain'].indexOf(val) !== -1;
    }
  }
},
```
{% endcode %}

`prop`详细描述如下(只有 7 个)：

| 参数                  | 说明       | 类型      | 可选值                         | 默认值   |
| ------------------- | -------- | ------- | --------------------------- | ----- |
| type                | 类型       | string  | success/info/warning/danger | —     |
| closable            | 是否可关闭    | boolean | —                           | false |
| disable-transitions | 是否禁用渐变动画 | boolean | —                           | false |
| hit                 | 是否有边框描边  | boolean | —                           | false |
| color               | 背景色      | string  | —                           | —     |
| size                | 尺寸       | string  | medium / small / mini       | —     |
| effect              | 主题       | string  | dark / light / plain        | light |

**prop** 中 `text` 虽然定义了，但是在代码中没有被使用。 `type`、`size`、`effect` 的值若没有匹配指定的字符串，根据值生成无效的 class(样式规则未定义)。

#### 计算属性

`tagSize` 根据`size`、`$ELEMENT`动态计算标签的尺寸。

组件定义了 prop `size` ， 则`tagSize` 值为 `size` 定义值。若 `size` 值为 `undefined`、`''`, `tagSize` 值为由全局配置 `$ELEMENT`决定。

若 `$ELEMENT` 对象不为空且包含 `size` 属性， `tagSize` 值为对象的属性值 `$ELEMENT.size`；否则 `tagSize` 值为 `undefined`,

```js
// 标签尺寸
tagSize() {
    return this.size || (this.$ELEMENT || {}).size;
}
```

前文可知，组件入口文件中 `install` 方法中定义了对象 `$ELEMENT` 全局配置。其中组件的默认尺寸 `size`值为`''` 。

```js
// src/index.js  组件入口文件
const install = function (Vue, opts = {}) {
  // ...
  // 全局配置   组件的默认尺寸size  弹框的初始z-index
  Vue.prototype.$ELEMENT = {
    size: opts.size || "",
    zIndex: opts.zIndex || 2000,
  };
  // ...
};
```

完整引入组件时，使用 `vue.use()`注册组件,会执行  `install`  方法，声明了全局属性 `$ELEMENT`。

```js
import Element from "element-ui";
Vue.use(Element);

// 手动设定 组件默认size
Vue.use(Element, {
  size: "small",
});
```

按需引入时, 需要手动配置 `Vue.prototype.$ELEMENT = { size: 'small', zIndex: 3000 };`，若不配置 `$ELEMENT` 未定义值为 `undefined`,若此调用 `$ELEMENT.size` 则会抛出异常`Uncaught ReferenceError: $ELEMENT is not defined`。

```js
import { Tag } from "element-ui";

// 全局配置
Vue.prototype.$ELEMENT = { size: "small", zIndex: 3000 };

// 引入组件
Vue.use(Tag);
```

> 表达式 `(this.$ELEMENT || {})` 防止`$ELEMENT` 未定义赋予空对象防止异常调用。

#### events 事件

点击 Tag 时触发 `click` 事件。

```js
// 点击 Tag 时触发的事件
handleClick(event) {
  // 触发当前实例上的事件
  this.$emit('click', event);
}
```

关闭 Tag 时触发 `close` 事件。

```js
// 关闭 Tag 时触发的事件
handleClose(event) {
  // 阻止捕获和冒泡阶段中当前事件的进一步传播。
  event.stopPropagation();
  // 触发当前实例上的事件
  this.$emit('close', event);
},
```

#### render() 渲染函数

组件将创建一个 `<span>` 元素 VNode，动态添加 class，使用内联样式设置背景色，定义了组件点击事件。 `<span>` 元素包含 2 个子节点

1. 默认插槽；
2. 名为`el-icon-close`的`Icon`图标，定义了关闭 Tag 时 click 事件。当 `closable` 值为`true`时才会渲染。

当`disableTransitions`值为`true`时，使用内置组件  `transition`  包裹 `<span>` 元素实现缩放`zoom-in-center`效果。

```js
// 渲染 标签 虚拟DOM
render(h) {
  const { type, tagSize, hit, effect } = this;
  // 动态添加class
  const classes = [
    'el-tag',
    type ? `el-tag--${type}` : '',
    tagSize ? `el-tag--${tagSize}` : '',
    effect ? `el-tag--${effect}` : '',
    hit && 'is-hit'
  ];
  // tag 元素
  const tagEl = (
    <span
      class={ classes }
      style={{ backgroundColor: this.color }}
      on-click={ this.handleClick }>
      { this.$slots.default }
      {
        this.closable && <i class="el-tag__close el-icon-close" on-click={ this.handleClose }></i>
      }
    </span>
  );
  // 组件VNode
  return this.disableTransitions ? tagEl : <transition name="el-zoom-in-center">{ tagEl }</transition>;
}
```

根据组件 prop 动态添加 `class`。

* `'el-tag'` 组件默认样式。
* `type ? 'el-tag--${type}' : ''` 设置组件不同类型颜色。若 `type` 值不是以下`success/info/warning/danger`其中一个，设置无效(生成无效的 class)。
* `tagSize ? 'el-tag--${tagSize}' : ''` 设置组件不同尺寸。若 `tagSize` 值不是以下`medium/small/mini`其中一个，设置无效(生成无效的 class)。
* `effect ? 'el-tag--${effect}' : ''` 设置组件不同主题。若 `effect` 值不是以下`dark/light/plain`其中一个，设置无效(生成无效的 class)。 `effect` 默认值为 `light`，对应的 `el-tag--light`是无效的样式，未定义。
* `hit && 'is-hit'` 边框描边效果 。

`color` 属性定义组件内联样式 `backgroundColor`，权重较高会覆盖其他样式的颜色。

### 0x02 组件样式

#### src/tag.scss

组件样式源码 `packages\theme-chalk\src\tag.scss` 使用混合指令 `b`、 `m`、`genTheme` 嵌套生成组件样式。

**genTheme() 组件主题**

混合指令 `genTheme` 用于生成组件的主题样式。

```scss
@mixin genTheme(
  $backgroundColorWeight,
  $borderColorWeight,
  $fontColorWeight,
  $hoverColorWeight
) {
  background-color: mix(
    $--tag-primary-color,
    $--color-white,
    $backgroundColorWeight
  );
  // ...

  // 生成 &.is-hit
  @include when(hit) {
    // ...
  }

  .el-tag__close {
    // ...
    &:hover {
      // ...
    }
  }
  // info / success / warning / danger 结构相同
  &.el-tag--info {
    // ...

    // 生成 &.is-hit
    @include when(hit) {
      // ...
    }

    .el-tag__close {
      // ...
      &:hover {
        // ...
      }
    }
  }
  // success
  // warning
  // danger
}
```

`Mix` 函数是将两种颜色根据一定的比例混合在一起，生成另一种颜色。 前两个参数是想混合的颜色（可以使用颜色变量、十六进制、RGBA、RGB、HSL 或者 HSLA 颜色值），第三个参数是第一种颜色的比例值。

```scss
mix($color1, $color2, $weight: 50%) //=> color
```

组件主题默认样式直接在 `el-tag` 下生成 。

```scss
// 生成 .el-tag
@include b(tag) {
  // 主题规则  默认light
  @include genTheme(10%, 20%, 100%, 100%);
  // ...

  // 生成 .el-tag--dark
  @include m(dark) {
    // 主题规则
    @include genTheme(100%, 100%, 0, 80%);
  }

  // 生成 .el-tag--plain
  @include m(plain) {
    // 主题规则
    @include genTheme(0, 40%, 100%, 100%);
  }
}
```

结合上述规则生成的主题样式如下

```scss
// el-tag 可以替换为 el-tag--dark   el-tag--plain
.el-tag {
  // ...
}
.el-tag.is-hit {
  // ...
}
.el-tag .el-tag__close {
  // ...
}
.el-tag .el-tag__close:hover {
  // ...
}
// info / success / warning / danger 结构相同
.el-tag.el-tag--info {
  // ...
}
.el-tag.el-tag--info.is-hit {
  // ...
}
.el-tag.el-tag--info .el-tag__close {
  // ...
}
.el-tag.el-tag--info .el-tag__close:hover {
  // ...
}
// success ...
// warning ...
// danger ...
```

**组件样式**

组件样式逻辑如下。

```scss
// 生成 .el-tag
@include b(tag) {
  // 默认light样式
  // ...

  // 生成 .el-tag .el-icon-close
  .el-icon-close {
    // ...
    // 生成 .el-tag .el-icon-close::before
    &::before {
      // ...
    }
  }

  // dark样式 ...
  // plaink样式 ...

  // 生成 .el-tag--medium
  @include m(medium) {
    // ...
    // 生成 .el-tag--medium .el-icon-close
    .el-icon-close {
      // ...
    }
  }

  // 生成 .el-tag--small
  @include m(small) {
    // ...
    // 生成 .el-tag--small .el-icon-close
    .el-icon-close {
      // ...
    }
  }
  // 生成 .el-tag--mini
  @include m(mini) {
    // ...
    // 生成 .el-tag--mini .el-icon-close
    .el-icon-close {
      // ...
    }
  }
}
```

#### lib/tag.scss

前文可知使用 `gulpfile.js`编译 `scss` 文件转换为`CSS`,经过浏览器兼容、格式压缩，最后生成 `packages\theme-chalk\lib\tag.scss`，内容格式如下。

```css
.el-tag {
  // ...
}
.el-tag .el-tag__close {
  // ...
}
.el-tag .el-tag__close:hover {
  // ...
}

/* theme start --  el-tag / el-tag--dark / el-tag--plain
-------------------------- */
.el-tag.is-hit {
  // ...
}
.el-tag .el-icon-close {
  // ...
}
.el-tag .el-icon-close::before {
  // ...
}
/* type --  info / success / warning / danger
-------------------------- */
.el-tag.el-tag--info {
  // ...
}
.el-tag.el-tag--info.is-hit {
  // ...
}
.el-tag.el-tag--info .el-tag__close {
  // ...
}
.el-tag.el-tag--info .el-tag__close:hover {
  // ...
}
// success ...
// warning ...
// danger ...

/* el-tag--dark
-------------------------- */
// ...

/* el-tag--plain
-------------------------- */
// ...

/* theme end --  el-tag / el-tag--dark / el-tag--plain
-------------------------- */

/* size --  medium / small / mini
-------------------------- */
.el-tag--medium {
  // ...
}
.el-tag--medium .el-icon-close {
  // ...
}
```

### 0x03 📚 参考

[“stopPropagation”，MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/stopPropagation)\
[“mix funciton color”，sass](https://sass-lang.com/documentation/modules/color#mix)
