# 3.1 组件概述

上一章节中整体介绍了项目的工程化流程。接下来将开始深入分析组件源码系列学习，抽丝剥茧，学习各组件的逻辑实现。

首先看下组件源码涉及的目录结构,分析了解其机制。

## 目录结构

目录 `packages` 和 `src` 存放了组件源码、入口文件、各种公共辅助工具等。

* 📁`packages`：组件源码（包含 主题样式 `theme-chalk`）等；
* 📁`src`：存放入口文件、自定义指令、国际化、混入方法、过渡效果、工具方法等。

```js
├─packages                  # 存放组件源码 及主题（样式）
|  ├─component-name         # 组件源码
|  ├─theme-chalk            # 主题（样式）
├─src                       # 存放入口文件以及各种公共辅助工具
|  ├─directives             # 自定义指令（滚轮事件优化、鼠标点击优化）
|  ├─locale                 # 国际化
|  ├─mixins                 # mixin混入
|  ├─transition             # 过渡效果
|  ├─utils                  # 工具方法
|  ├─index.js               # 组件库入口文件
```

接下来将对其逐一分析，耐心读完，相信会对您有所帮助！

## 📁 packages/component-name

组件功能的逻辑实现存放在 `packages` 目录下对应的同名目录下，组件名 `component-name` (`kebab-case` 风格) 。

执行`make`命令 `make new <component-name> [中文名]`,调用脚本`build/bin/new.js` 自动生成组件基本代码,目录结构如下：

```js
├─component-name    # 组件目录，kebab-case 风格
| ├─index.js        # 导出组件，同时定义其 install 方法
| ├─src             # 组件源码目录
| | ├─main.vue      # 组件功能逻辑实现
```

接下来以组件`Avatar`示例，看下其 `packages\avatar\index.js` 代码实现：

```javascript
// 引入组件
import Avatar from "./src/main.vue";

// 定义组件Avatar的 install 方法
Avatar.install = function (Vue) {
  // 注册全局组件Avatar
  Vue.component(Avatar.name, Avatar);
};
// 导出组件
export default Avatar;
```

`index.js`主要是为组件扩展 `install` 方法。当在项目中按需引入 `alert` 功能组件时，可以使用`vue.use(alert)` 注册全局组件 `Avatar`。

```javascript
// 按需引入
import Vue from "vue";
import { Avatar } from "element-ui";

Vue.use(Avatar);
```

还有一些组件 `Loading`、`MessageBox`、`Notification`、`Message` 使用方式与上述有所不同，将在后续文章中详细讨论。

## 📁 packages/theme-chalk 主题

`packages/theme-chalk` 目录存放各组件对应的 `scss` 文件、`scss` 相关变量、字体文件、`mixin` 及公共样式设置。 详情请阅读前文 [工程化系列(五)](https://juejin.cn/post/6971434343516340238#heading-1) 中“主题构建”章节，在此不再赘述。

## 📃 src/index.js

`src/index.js`是组件库入口文件，引入所有组件、定义完整引入时注册组件的 install 方法，并导出组件库版本信息、国际化配置、install 和 各个组件（用于按需引入）。

执行脚本  `build/bin/build-entry.js`，基于组件清单文件`components.json`结合字符串模版库`json-templater/string`自动生成。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d44c86962f324bbfb87ae71c50cd92e4\~tplv-k3u1fbpfcp-watermark.image)

### 完整引入

在引入 `Element` 时，传入一个对象用于语言设置、自定义 i18n 的处理方法、 组件的默认尺寸 size、弹框的初始 z-index 。

```js
import Vue from "vue";
import Element from "element-ui";
import lang from "element-ui/lib/locale/lang/en";

Vue.use(Element, {
  locale: lang,
  size: "small",
  zIndex: 3000,
  i18n: function (path, options) {
    // ...
  },
});

new Vue({
  el: "#app",
  render: (h) => h(App),
});
```

### 按需引入

引入部分组件比如 `Button` 和 `` Select，可以使用两种方式进行注册：插件注册 `Vue.use()` 或 组件全局注册`Vue.component()`；引入 `locale` 方法进行语言设置;将全局配置添加到`Vue.prototype`上。

```js
import Vue from "vue";
import { Button, Select } from "element-ui";
import lang from "element-ui/lib/locale/lang/en";
import locale from "element-ui/lib/locale";

// 设置语言
locale.use(lang);

// 全局配置
Vue.prototype.$ELEMENT = { size: "small", zIndex: 3000 };

// 引入组件
Vue.use(Button);
Vue.component(Select.name, Select);

new Vue({
  el: "#app",
  render: (h) => h(App),
});
```

## 📁 src/directives 自定义指令

### 📃 mousewheel.js

滚轮事件优化，解决不同浏览器、不同平台的兼容性问题。 用于 `packages/table/src/table.vue` 组件中的 `v-mousewheel`指令。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9e1d8e32e8348c6b102c20f859d0344\~tplv-k3u1fbpfcp-watermark.image)

### 📃 repeat-click.js

鼠标点击优化，当用户鼠标左键一直按住不松手，只会触发一次触发 mousedown 的回调。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdeb0dd306b04be8829a533e0f8f6f8a\~tplv-k3u1fbpfcp-watermark.image)

用于 `packages\input-number\src\input-number.vue`组件中的 `v-repeat-click`指令。 在`InputNumber` 计数器点击 ➕、➖ 时会触发该指令。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23facd7cbf554327b93a1be5d431941d\~tplv-k3u1fbpfcp-watermark.image)

用于 `packages\date-picker\src\basic\time-spinner.vue` 组件中的 `v-repeat-click`指令。在`TimePicker` 时间选择器点击 🔺、🔻 时会触发该指令。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6034137bad4e4d80b54d989b935fa514\~tplv-k3u1fbpfcp-watermark.image)

## 📁 src/mixins 混入方法

混入 (mixin) 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。

### 📃 emitter.js

`broadcast`用于上层组件通知下层组件的事件广播，遍历当前组件的所有子组件，找到名称为 componentName 的子组件，然后调用其 $emit() 事件。

```js
/**
 * @param componentName 组件名称
 * @param eventName 事件名称
 * @param params  传递的参数;
 */
function broadcast(componentName, eventName, params) {
  // 遍历所有子组件
  this.$children.forEach((child) => {
    var name = child.$options.componentName;
    // 找到组件名为componentName的子组件，并调用该子组件的$emit方法；
    // 否则，继续递归
    if (name === componentName) {
      child.$emit.apply(child, [eventName].concat(params));
    } else {
      broadcast.apply(child, [componentName, eventName].concat([params]));
    }
  });
}
```

`dispatch`用于子组件发送消息给上层组件的组件通信，查找所有父级，直到找到名称为 componentName 的父组件， 然后调用其 $emit() 事件。

```js
/**
 * @param componentName 组件名称
 * @param eventName 事件名称
 * @param params  传递的参数;
 */
dispatch(componentName, eventName, params) {
  // 当前父组件
  var parent = this.$parent || this.$root;
  // 当前父组件的组件名
  var name = parent.$options.componentName;

  // 通过$parent，一直向上找，直到组件名等于componentName
  while (parent && (!name || name !== componentName)) {
    parent = parent.$parent;

    if (parent) {
      name = parent.$options.componentName;
    }
  }
  if (parent) {
    // 如果找到目标组件，那么调用目标组件的$emit方法
    parent.$emit.apply(parent, [eventName].concat(params));
  }
}
```

### 📃 focus.js

指定组件获取焦点。

```js
export default function (ref) {
  return {
    methods: {
      focus() {
        this.$refs[ref].focus(); // 获取焦点;
      },
    },
  };
}
```

### 📃 locale.js

提供了 `t` 方法用于`i18n`处理。

```js
// t:i18n处理方法
import { t } from "element-ui/src/locale";

export default {
  methods: {
    t(...args) {
      return t.apply(this, args);
    },
  },
};
```

在`Select`组件源码中可以看到 `t` 函数是根据传入的 path 路径(`el.select.loading`)，从语言包中找到对应的文案。

```js
import Locale from "element-ui/src/mixins/locale";

export default {
  mixins: [Locale],
  name: "ElSelect",
  componentName: "ElSelect",
  computed: {
    emptyText() {
      if (this.loading) {
        return this.loadingText || this.t("el.select.loading");
      }
      // ...
    },
  },
  // ...
};
```

`src\locale\lang\zh-CN.js` 可以查看到该组件的简体中文语言配置内容。`el.select.loading`等同于 `加载中`。

```js
export default {
  el: {
    // ...
    select: {
      loading: "加载中",
      noMatch: "无匹配数据",
      noData: "无数据",
      placeholder: "请选择",
    },
    // ...
  },
};
```

### 📃 migrating.js

对组件发生改动的 props 或 eventName 给予提醒。

```js
// 驼峰形式转成短横线连接的形式
import { kebabCase } from "element-ui/src/utils/util";

export default {
  mounted() {
    // 只在非生产环境下进行
    if (process.env.NODE_ENV === "production") return;
    // VNode 虚拟 DOM。Vue 通过建立一个虚拟 DOM 对真实 DOM 发生的变化保持追踪
    if (!this.$vnode) return;
    // 获取组件变更的Attributes Events 的名称和提醒文本
    const { props = {}, events = {} } = this.getMigratingConfig();
    // 获取当前页面组件使用的 Attributes 和 Events
    const { data, componentOptions } = this.$vnode;
    const definedProps = data.attrs || {};
    const definedEvents = componentOptions.listeners || {};

    // 判断当前使用Attributes Events，若在变更列表中，给予提醒。

    for (let propName in definedProps) {
      propName = kebabCase(propName); // compatible with camel case
      if (props[propName]) {
        console.warn(
          `[Element Migrating][${this.$options.name}][Attribute]: ${props[propName]}`
        );
      }
    }

    for (let eventName in definedEvents) {
      eventName = kebabCase(eventName); // compatible with camel case
      if (events[eventName]) {
        console.warn(
          `[Element Migrating][${this.$options.name}][Event]: ${events[eventName]}`
        );
      }
    }
  },
  methods: {
    getMigratingConfig() {
      return {
        props: {},
        events: {},
      };
    },
  },
};
```

`Input` 组件中的引用。

```js
import Migrating from "element-ui/src/mixins/migrating";

export default {
  name: "ElInput",
  mixins: [Migrating],
  methods: {
    getMigratingConfig() {
      return {
        props: {
          icon: "icon is removed, use suffix-icon / prefix-icon instead.",
          "on-icon-click": "on-icon-click is removed.",
        },
        events: {
          click: "click is removed.",
        },
      };
    },
  },
};
```

页面组件使用 `click` 事件，加载后会有警告提醒 `click is removed`。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d0069ae0da34f1386f392ff7ac9ada7\~tplv-k3u1fbpfcp-watermark.image)

## 📁 src/transition 过渡效果

`collapse-transition.js` 导出一个函数式组件，用于实现折叠展开过渡效果。在 `Tree 树形控件`、`Collapse 折叠面板`、`NavMenu 导航菜单`组件中用于折叠展开效果。

函数式组件只处理状态和行为，将内容和行为分离，实现代码解耦，让其更容易复用。

```js
// addClass:移除class类名方法  removeClass:移除class类名方法
import { addClass, removeClass } from "element-ui/src/utils/dom";

/**
 * 定义组件 Transition  进入中、离开时的事件函数
 * @class Transition
 */
class Transition {
  // --------
  // 进入中
  // --------
  beforeEnter(el) {
    // ...
  }

  enter(el) {
    // ...
  }

  afterEnter(el) {
    // ...
  }

  // --------
  // 离开时
  // --------
  beforeLeave(el) {
    // ...
  }

  leave(el) {
    // ...
  }

  afterLeave(el) {
    // ...
  }
}
// 导出函数式组件
export default {
  name: "ElCollapseTransition",
  functional: true,
  /**
   * 渲染函数
   * @param {Function} h createElement的别名
   * @param {String | Array} { children }   children:VNode 子节点的数组
   * @return {VNode} 创建虚拟节点 (virtual node)
   *
   */
  render(h, { children }) {
    // 定义 on 事件修饰符
    const data = {
      on: new Transition(),
    };
    // 创建虚拟节点
    return h("transition", data, children);
  },
};
```

`createElement`  函数参数

```js
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // 一个 HTML 标签名、组件选项对象，或者
  // resolve 了上述任何一种的一个 async 函数。必填项。
  "div",

  // {Object}
  // 一个与模板中 attribute 对应的数据对象。可选。
  {
    // ...
  },

  // {String | Array}
  // 子级虚拟节点 (VNodes)，由 `createElement()` 构建而成，
  // 也可以使用字符串来生成“文本虚拟节点”。可选。
  [
    "先写一些文字",
    createElement("h1", "一则头条"),
    createElement(MyComponent, {
      props: {
        someProp: "foobar",
      },
    }),
  ]
);
```

`h('transition', data, children)` 功能等同于 `createElement` 函数使用 `transition` 内置组件向子元素或子组件传递事件，创建新的虚拟节点。

```js
createElement(
  "transition",
  // on 事件修饰符
  {
    on: {
      beforeEnter(el) {
        // ...
      }

      enter(el) {
        // ...
      }

      afterEnter(el) {
        // ...
      }

      beforeLeave(el) {
        // ...
      }

      leave(el) {
        // ...
      }

      afterLeave(el) {
        // ...
      }
    }
  },
  [VNode]
);
```
