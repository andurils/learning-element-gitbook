# 3.26 Notification 通知

### 简介

通知组件`Notification` 常用于全局展示通知提醒信息。本文将分析其源码实现，耐心读完，相信会对您有所帮助。🔗 [组件文档 Notification](https://element.eleme.cn/#/zh-CN/component/notification) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/blob/dev/packages/notification/)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。。

### Notification vs Message

`Notification` 在功能配置以及源码实现上与 `Message` 非常类似，所以部分重复内容本文将不做详尽解释。为了更好理解，请先阅读 [Message 组件实现](https://juejin.cn/post/7130117232138387492)、[Message 服务实现](https://juejin.cn/post/7130485322315464718)。

`Message` 常用于主动操作后的反馈提示，

* 可提供成功、警告和错误等反馈信息。
* 顶部居中显示并自动消失，是一种不打断用户操作的轻量级提示方式。

`Notification` 常用于显示全局的通知提醒消息。

* 较为复杂的通知内容。
* 系统主动推送。
* 悬浮出现在页面角落。

***

### 使用方式

跟`Message`组件一样，`Notification`以服务的方式调用。调用方法为 `Notification(options)`,组件为每个 type 定义了各自的方法，如 `Notification.success(options)`，并且可以调用 `Notification.closeAll()` 手动关闭所有实例。

`options` 参数配置项，在此不做详尽解释,详见 [组件文档 Notification#options](https://element.eleme.cn/#/zh-CN/component/notification#options)。

当组件库完整引入，直接使用`this.$notify(options)`。

```js
// packages\notification\index.js
import Notification from './src/main.js';
export default Notification;

// src/index.js
const install = function(Vue, opts = {}) { 
 Vue.prototype.$notify = Notification;
};

// 完整引入
this.$notify(options);

// 单独引用
import { Notification } from 'element-ui'; 
Notification(options);
```

### 组件源码

#### DOM结构

组件 `Notification` 的 DOM 层次结构跟 `Alert`非常类似。

```html
<transition name="el-notification-fade">
  <!-- 组件根节点 -->
  <div class="el-notification">
    <!-- icon 图标 -->
    <i class="el-notification__icon"></i>
    <!-- 文字内容区域 -->
    <div class="el-notification__group">
      <!-- 标题 -->
      <h2 class="el-notification__title" v-text="title"></h2>
      <!-- 说明文字 -->
      <div class="el-notification__content" v-show="message">
        <slot>
          <p v-if="!dangerouslyUseHTMLString">{{ message }}</p>
          <p v-else v-html="message"></p>
        </slot>
      </div>
      <!-- 关闭按钮 -->
      <div class="el-notification__closeBtn el-icon-close"></div>
    </div>
  </div>
</transition>
```

类名为`el-notification`的`<div>` 元素根节点 `1️⃣`，使用 `transition` 实现过渡效果，包含两个子节点：

1. `2️⃣`左侧的图标 。
2. `3️⃣`右侧的文字内容区域 。
   * `4️⃣` 标题
   * `5️⃣` 说明文字
   * `6️⃣` 关闭按钮（定位使用绝对布局）

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/804a67be20094c6f9cbec539d076e530\~tplv-k3u1fbpfcp-watermark.image?)

下图为组件`Alert`的DOM层次结构，非常相似。具体可以阅读前文 [源码剖析之Alert](https://juejin.cn/post/6998892221562880008) 。 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5add985891b244bc85ab05fc29f7642f\~tplv-k3u1fbpfcp-watermark.image)

#### 组件功能

事件监听器。它可以是 `transitionend` 或 `animationend`

`notification` 功能实现跟 `message`类似，接下来主要说明下不同之处，相似代码将省略。

```html
// packages\notification\src\main.vue
<template>
  <transition name="el-notification-fade">
    <div
      :class="['el-notification', customClass, horizontalClass]"
      v-show="visible"
      :style="positionStyle"
      @mouseenter="clearTimer()"
      @mouseleave="startTimer()"
      @click="click" 
    >
      // 省略...
    </div>
  </transition>
</template>
<script type="text/babel">
  export default {
    data() {
      return {
        // ... 
        title: '', // 标题
        position: 'top-right' // 自定义弹出位置
      };
    },
    computed: {
      // typeClass() 图标类名 
      // horizontalClass() 根据position 判定水平方向属性 
      // verticalProperty() 根据position 判定垂直方向属性 
      // positionStyle()  
    },
    watch: {
      // closed()
    },
    methods: {
      // destroyElement()
      // click() 点击回调事件 
      // close()  
      // clearTimer()  
      // startTimer()  
      // keydown 
    },
    mounted() {
      // startTimer
      // add keydown Listener
    },
    beforeDestroy() {
      // remove keydown Listener 
    }
  };
</script> 
```

#### 生命周期 & 事件

跟message组件一样，当被挂载之后调用方法`startTimer`启用定时器，实现实例的自动关闭。挂载之后添加`keydown`事件监听。实例销毁之前,会移除`keydown`事件监听。

根节点不仅绑定 `mouseenter`、`mouseleave` 事件，也绑定了 `click` 事件，用于点击实例时调用传入的回调函数。

```js
click() {
  if (typeof this.onClick === 'function') {
    this.onClick();
  }
},
```

组件 `transition` 没有绑定`after-leave`钩子函数，而是在侦听器中添加了 `transitionend` 事件监听，调用方法 `destroyElement` 用于组件关闭后的销毁工作。

```js
// watch  侦听
closed(newVal) {
  if (newVal) {
    this.visible = false;
    this.$el.addEventListener('transitionend', this.destroyElement); // 添加过渡效果事件监听
  }
}

// methods
destroyElement() {
  this.$el.removeEventListener('transitionend', this.destroyElement); //移除过渡效果事件监听
  // vm destroy  &&  remove el 
},
```

方法 `keydown`不仅实现按`ESC`键关闭消息组件，同时支持按`backspace` `detele`键取清除定时器，按其他键恢复计时器。

```js
keydown(e) {
  // 8 backspace 46 detele 清除定时器
  if (e.keyCode === 46 || e.keyCode === 8) {
    this.clearTimer(); 
  } else if (e.keyCode === 27) { // esc关闭消息
    if (!this.closed) {
      this.close();
    }
  } else {
    this.startTimer(); // 恢复计时器
  }
}
```

#### 位置偏移

属性`position`定义实例的弹出位置，支持四个选项：`top-right`、`top-left`、`bottom-right`、`bottom-left`，默认为`top-right`。

组件使用绝对定位，根据属性`position`值判定上下、左右边界偏移属性。

计算属性`verticalProperty` 用于判定上下偏移属性使用top或bottom；计算属性`positionStyle`基于 `verticalProperty` 值生成内联样式 `top/bottom:20px`。

```js
verticalProperty() {
  return /^top-/.test(this.position) ? 'top' : 'bottom';
}, 
positionStyle() {
  return {
    [this.verticalProperty]: `${ this.verticalOffset }px`
  };
}
```

左右偏移使用计算属性`horizontalClass` 生成类名`right` 或`left`。

```js
horizontalClass() {
  return this.position.indexOf('right') > -1 ? 'right' : 'left';
},
```

相关样式定义：

```css
.el-notification.right {
    right: 16px
} 
.el-notification.left {
    left: 16px
}
```

### 服务实现

`Notification` 在源码实现上与 `Message` 非常类似，具体的功能流程讲解请阅读前文[Message 服务实现](https://juejin.cn/post/7130485322315464718)，此处不在过多赘述。

源码精简后结构如下，代码创建了`function`类型的对象`Notification`，同时给对象添加属性方法 `close`、`closeAll`、 `warning`、`info`、 `error`，导出对象 `Notification`。

```js
// packages\notification\src\main.js
const NotificationConstructor = Vue.extend(Main); // 组件构造器  
let instance; // 组件实例 
let instances = []; // 存储所有组件实例数组
let seed = 1; // 用于递增计数 

const Notification = function(options) {
  // 逻辑 ...
};

['success', 'warning', 'info', 'error'].forEach(type => {
  // 逻辑 ...
});

Notification.close = function(id, userOnClose) {
  // 逻辑 ...
};

Notification.closeAll = function() {
  // 逻辑 ...
};

export default Notification; 
```

`Notification`支持自定义弹出位置，可以从屏幕四角中的任意一角弹出，但是所有实例都在保存在同一数组中，在样式计算的过程中，新增了逻辑区分位置相同的元素。

函数`Notification`中，根据属性`position`过滤元素个数，进行偏移量计算。即使未设置`offset`值，组件默认偏移量为 `16` 。

```js
const Notification = function(options) {
  // ...
  const position = options.position || 'top-right';  
  // ...
  let verticalOffset = options.offset || 0;
  instances.filter(item => item.position === position).forEach(item => {
    verticalOffset += item.$el.offsetHeight + 16;
  });
  verticalOffset += 16; 
  instance.verticalOffset = verticalOffset;
  // ...
};
```

函数`close`中，当删除实例后，重新计算只需要调整索引值大于当前实例index的偏移量，根据属性`position`过滤元素，同时根据计算属性`verticalProperty`更新DOM元素样式。

```js
Notification.close = function(id, userOnClose) {
  // ...
  
  // 数据更新后 偏移量计算
  const position = instance.position;
  const removedHeight = instance.dom.offsetHeight;
  for (let i = index; i < len - 1; i++) {
    if (instances[i].position === position) {
      instances[i].dom.style[instance.verticalProperty] =
        parseInt(instances[i].dom.style[instance.verticalProperty], 10) - removedHeight - 16 + 'px';
    }
  }
};

// 计算属性
verticalProperty() {
  return /^top-/.test(this.position) ? 'top' : 'bottom';
},
```

### 样式实现

组件样式源码 `packages\theme-chalk\src\notification.scss` 使用混合指令 `b`、`when`、`m`、`e` 嵌套生成组件样式。

```scss
// 生成 .el-notification
@include b(notification) {
  // ...
  
  // 生成 .el-notification.right
  &.right {
    // ...
  }
  // 生成 .el-notification.left
  &.left {
    // ...
  }
  // 生成 .el-notification__group
  @include e(group) {
    // ...
  }
  // 生成 .el-notification__title
  @include e(title) {
    // ...
  }
  // 生成 .el-notification__content
  @include e(content) {
    // ...
    
    // 生成 .el-notification__content p
    p {
      // ...
    }
  }
  // 生成 .el-notification__icon
  @include e(icon) {
    // ...
  }
  // 生成 .el-notification__closeBtn
  @include e(closeBtn) {
    // ...
    
    // 生成 .el-notification__closeBtn:hover
    &:hover {
      // ...
    }
  }
  // 生成 .el-notification .el-icon-success/error/info/warning
  .el-icon-success {
    // color ...
  }
  // error/info/warning
}

.el-notification-fade-enter {
  // 生成 .el-notification-fade-enter.right
  &.right {
    // ...
  }
  // 生成 .el-notification-fade-enter.left
  &.left {
    // ...
  }
}

// 生成 .el-notification-fade-leave-active 
.el-notification-fade-leave-active {
  // ...
}
```
