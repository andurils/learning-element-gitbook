# 3.25 Message 消息提示

### 简介

消息提示组件 `Message` 常用于主动操作后的反馈提示，顶部居中显示并自动消失，是一种不打断用户操作的轻量级提示方式。本文将分析其源码实现，耐心读完，相信会对您有所帮助。🔗 [组件文档 Message](https://element.eleme.cn/#/zh-CN/component/message) 🔗 [gitee源码](https://gitee.com/ElemeFE/element/blob/dev/packages/message/src)

更多组件剖析详见 👉 [**📚 Element 2 源码剖析组件总览**](https://juejin.cn/post/6994721241194037255) 。

### 使用方式

组件`Message`以服务的方式调用。`Message` 组件入口文件中没有，没有插件声明，只是导出了方法 `Message`；在组件库入口文件中，将方法 `Message`添加至`Vue.prototype`。

```js
// `Message` 组件入口文件
// packages\message\index.js
import Message from './src/main.js';
export default Message; 
```

在组件库入口文件也是一样的处理。

```js
// 组件库入口 
// src/index.js
import Message from '../packages/message/index.js';
//... 
const install = function(Vue, opts = {}) {
  //...
  Vue.prototype.$message = Message; 

}; 
export default {
  //...
  Message,
}; 
```

调用方法为 `Message(options)`。组件也为每个 type 定义了各自的方法，如 `Message.success(options)`。调用 `Message.closeAll()` 手动关闭所有实例。组件库完整引入，直接使用`this.$message(options)`。

```js
// 完整引入
this.$message(options);

// 单独引用
import { Message } from 'element-ui';
// ...
Message(options);
```

其中 `options` 参数为 `Message` 的配置项，在此不做详尽解释,详见 [组件文档 Message #options](https://element.eleme.cn/#/zh-CN/component/message#options)。

### 组件源码

#### HTML

消息提示组件页面元素结构比较简单。

根节点下元素按照功区分，主要有三部分：

* Icon 图标
* 消息文字
* 关闭按钮

使用 `transition` 组件，在组件根节点的条件展示 (`v-show`)中添加过渡效果,定义了钩子函数`after-leave` 用于设置过渡离开完成之后的组件状态。

```html
// packages\message\src\main.vue
<template>
  <!-- transition过渡组件，绑定after-leave钩子 -->
  <transition name="el-message-fade" @after-leave="handleAfterLeave">
    <!-- 组件根节点 -->
    <div
      :class="[
        'el-message',
        type && !iconClass ? `el-message--${ type }` : '',
        center ? 'is-center' : '',
        showClose ? 'is-closable' : '',
        customClass
      ]"
      :style="positionStyle"
      v-show="visible" >
      <!-- 主题/自定义图标 -->
      <i :class="iconClass" v-if="iconClass"></i>
      <i :class="typeClass" v-else></i>
      <!-- 默认插槽 -->
      <slot>
        <p v-if="!dangerouslyUseHTMLString" class="el-message__content">{{ message }}</p>
        <p v-else v-html="message" class="el-message__content"></p>
      </slot>
      <!-- 关闭图标 -->
      <i v-if="showClose" class="el-message__closeBtn el-icon-close" @click="close"></i>
    </div>
  </transition>
</template> 
<script>  
  // 主题类型图标映射
  const typeMap = {
    success: 'success',
    info: 'info',
    warning: 'warning',
    error: 'error'
  };
  
  export default {
    data() {
      return {
        visible: false, // 组件显示状态
        message: '', // 消息文字
        duration: 3000, // 显示时间, 毫秒
        type: 'info', // 状态主题
        iconClass: '', // 自定义图标的类名
        customClass: '', // 自定义类名
        onClose: null, // 关闭时的回调函数
        showClose: false, // 是否显示关闭按钮
        closed: false, // 组件关闭状态
        verticalOffset: 20, // 距离顶部的偏移 top: 20px
        timer: null, // 定时器，控制组件自动关闭
        dangerouslyUseHTMLString: false, // 是否将 message 属性作为 HTML 片段处理
        center: false // 文字是否居中
      };
    }, 
     computed: {
      // 不同主题type的图标
      typeClass() {
        return this.type && !this.iconClass
          ? `el-message__icon el-icon-${ typeMap[this.type] }`
          : '';
      },
      // 设置top
      positionStyle() {
        return {
          'top': `${ this.verticalOffset }px`
        };
      }
    },
    // ...
  };
</script> 
```

**top 偏移量**

元素根节点是一个类名`el-message`的div元素，使用绝对布局。`fixed`表示脱离文档流，通过指定元素相对于屏幕视口（viewport）的位置来指定元素位置，水平方向居中，垂直方向居上。

```css
.el-message { 
  position: fixed;
  left: 50%;
  top: 20px; 
}
```

使用计算属性 `positionStyle` 基于设置的`verticalOffset`属性值动态计算组件距离顶部的偏移量。

```js
// top: 20px
positionStyle() {
  return {
    'top': `${ this.verticalOffset }px`
  };
}
```

页面中可以存在多个`Message` 实例，新 Message 消息会在旧的下面展示，此时实例根据在数组中所处索引值，计算出实例的距离顶部的偏移量 `verticalOffset`。组件实例随着自动/人工关闭销毁，数组内容变量，其索引值会变化，`verticalOffset`值也会重新计算，下文“服务实现”一节中会详细介绍该逻辑。

**状态主题**

状态主题属性`type`默认值 `info`, 组件支持`success/warning/info/error`共四种可选值。

根节点中基于`type`值生成不同主题样式`el-message--[success/warning/info/error]`。但当传入属性`iconClass`值用于自定义图标的类名，就不会生成主题样式，此时 `type`设置无效。

```js
type && !iconClass ? `el-message--${ type }` : '',
```

**子元素内容布局**

`message` 组件内部使用flex布局。属性 `center`用于生成类名`is-center`设置图标和消息文字居中。

```css
.el-message { 
  display: flex; 
  align-items: center;
} 
.el-message.is-center { 
  justify-content: center;
}
```

关闭图标使用绝对布局，垂直居中水平居右。

```html
<!-- 关闭图标 -->
<i v-if="showClose" class="el-message__closeBtn el-icon-close" @click="close"></i>
      
.el-message__closeBtn {
  position: absolute;
  top: 50%;
  right: 15px; 
}
```

当时显示关闭图标时，会生成类名`is-closable` ，防止消息文字跟关闭按钮由重叠。

```css
.el-message.is-closable .el-message__content {
  padding-right: 16px;
}
```

Icon图标优先显示自定义。主题图标的类名使用计算属性`typeClass`。

```js
<!-- 主题/自定义图标 -->
<i :class="iconClass" v-if="iconClass"></i>
<i :class="typeClass" v-else></i>
```

**消息文字插槽**

属性 `message` 支持传入 HTML 片段，但是需要显示打开此功能（将属性`dangerouslyUseHTMLString` 设置 true）。

```js
<!-- 默认插槽 -->
<slot>
  <p v-if="!dangerouslyUseHTMLString" class="el-message__content">{{ message }}</p>
  <p v-else v-html="message" class="el-message__content"></p>
</slot> 
```

当属性 `message` 传入值类型为`VNode`时，会使用插槽功能，下文“服务实现”一节会详细介绍。

```js
// packages\message\src\main.js
if (isVNode(instance.message)) {
    instance.$slots.default = [instance.message];
    instance.message = null;
  }
```

> 在网站上动态渲染任意 HTML 是非常危险的，因为容易导致 [XSS 攻击](https://en.wikipedia.org/wiki/Cross-site\_scripting)。因此在 `dangerouslyUseHTMLString` 打开的情况下，请确保 `message` 的内容是可信的，**永远不要**将用户提交的内容赋值给 `message` 属性。

#### 生命周期 & 事件

组件被挂载之后调用方法`startTimer`启用定时器，实现 message 实例的自动关闭。在方法`startTimer`中当属性`duration`值大于0时（`if (this.duration > 0)`），才会创建定时器用于自动关闭；若组件不需要自动关闭，将属性`duration`值设置为 `0` 即可。

挂载之后添加`keydown`事件监听。实例销毁之前,会移除`keydown`事件监听。方法 `keydown`用于实现按`ESC`键关闭消息组件。如果页面存在多个实例，会全部关闭。

使用自定义侦听器，当属性`closed`值变化且为 true 时，将属性`visible`值置为false(组件隐藏)。

```js
export default {
  // ... 
  // 挂载时
  mounted() {
    // 启动定时器
    this.startTimer();
    // 监听keydown事件
    document.addEventListener('keydown', this.keydown);
  },
  beforeDestroy() {
    // 取消keydown监听
    document.removeEventListener('keydown', this.keydown);
  },
  watch: {
    closed(newVal) {
      if (newVal) {
        this.visible = false;
      }
    }
  }, 
  methods: { 
    // 组件关闭事件
    close() {
      // ...
    },
    // 过渡`after-leave`钩子函数
    handleAfterLeave() {
      // ...
    },
    // 清除定时器，当mouseenter时调用
    clearTimer() {
      clearTimeout(this.timer);
    },
    // 启动定时器，duration默认是3s，到时间自动隐藏
    startTimer() {
      if (this.duration > 0) {
        this.timer = setTimeout(() => {
          if (!this.closed) {
            this.close();
          }
        }, this.duration);
      }
    },
    // 监听键盘按键事件
    keydown(e) {
      if (e.keyCode === 27) { // esc关闭消息
        if (!this.closed) {
          this.close();
        }
      }
    }
  },
};
```

根节点绑定 `mouseenter`、`mouseleave` 事件，当鼠标移动到 message 实例上，会清除其定时器`clearTimer`，该实例就不会自动关闭。当鼠标移出后，会重新创建定时器`startTimer`，实现自动关闭。

```html
<div @mouseenter="clearTimer"  @mouseleave="startTimer" >
  // ...
</div>
```

方法`close`用于关闭组件，如果用户设置了属性`onClose`值，关闭时会执行该回调函数, 参数为被关闭的 message 实例。

```js
close() {
  this.closed = true;
  if (typeof this.onClose === 'function') {
    this.onClose(this);
  }
},
```

当组件关闭后，会触发`transition` 组件绑定`after-leave`钩子函数，执行方法 `handleAfterLeave`。过渡离开完成之后，执行方法`vm.$destroy()`，完全销毁该实例，触发 `beforeDestroy` 的钩子；同时将实例 DOM 元素从页面移除

```js
handleAfterLeave() {
  // 完全销毁一个实例。清理它与其它实例的连接，解绑它的全部指令及事件监听器
  // 触发 beforeDestroy 和 destroyed 的钩子
  this.$destroy(true);
  this.$el.parentNode.removeChild(this.$el);
},
```

### 服务方式源码

组件服务方式实现的源码文件为 `packages\message\src\main.js`。

源码精简后结构如下，代码创建了`function`类型的对象`Message`，同时给对象添加属性方法 `close`、`closeAll`、 `warning`、`info`、 `error`，导出对象 `Message`。

```js
// packages\message\src\main.js 

let MessageConstructor = Vue.extend(Main);  // message组件构造器
let instance; // message 组件实例 
let instances = []; // 存储所有message实例数组 
let seed = 1; // 用于递增计数

const Message = function(options) {
  // 逻辑 ...
  return instance;
};  

// 定义了各状态的便捷方法 Message.success(options)
['success', 'warning', 'info', 'error'].forEach(type => {
  // 逻辑 ...
});

// 关闭指定id 的Message实例
Message.close = function(id, userOnClose) {
  // 逻辑 ...
};
// 关闭所有Message实例
Message.closeAll = function() {
  // 逻辑 ...
}; 

export default Message; 
```

#### Message()

使用函数表达式（函数字面量）方式，将函数赋值给了变量 `Message`。 该函数用于初始化配置创建组件，并返回该实例。

函数实现功能主要有以下步骤：

1. 参数 `options` 初始化。
2. 使用`Vue.extend`、`vm.$mount()`创建渲染挂载实例，默认将其添加至body元素节点下。
3. 根据数组`instances`中实例数量，计算并设置该实例顶部的垂直偏移。
4. 将实例设置显示可见 `visible = true`。
5. 更新数组`instances`，将该实例添加至其中。
6. 返回`message` 实例。

每个实例生成唯一ID，格式为`message_xx`，用于实例关闭操作，稍后会详尽解释。

```js
// 此处代码未作详尽解释
let MessageConstructor = Vue.extend(Main);  // message组件构造器
let instance; // message 组件实例 
let instances = []; // 存储所有message实例数组 
let seed = 1; // 用于递增计数

const Message = function(options) { 
  
  // options 初始化...
  
  // 实例创建渲染
  let id = 'message_' + seed++; // 组件实例 id
  // 创建组件实例
  instance = new MessageConstructor({
    data: options
  });
  instance.id = id;  
  instance.$mount(); // 渲染为文档之外的的元素 
  document.body.appendChild(instance.$el); // 挂载实例 添加至body元素节点下
  
  // 计算并设置顶部的垂直偏移 ...
  
  instance.visible = true; // 组件显示可见
  instance.$el.style.zIndex = PopupManager.nextZIndex(); // 实例元素zIndex  全局统一管理
  instances.push(instance); // 添加至数组中
  
  return instance;
};
```

**options 类型格式化**

当`options`参数传入不是string类型时，例如 `this.$message('消息文字');`，定义对象并传入的参数值赋值给属性 `message`,等同于 `this.$message({ message:'消息文字'});`。

```js
if (typeof options === 'string') {
  options = {
    message: options
  };
}
```

**VNode支持**

当属性`message`值传入一个 VNode 时，将其赋值给匿名插槽，此时插槽的后备内容不会被渲染。

```js
// 逻辑实现
// packages\message\src\main.js 
if (isVNode(instance.message)) {
  instance.$slots.default = [instance.message];
  instance.message = null;
}

// 调用方式
const h = this.$createElement;
this.$message({
  message: h("p", null, [
    h("span", null, "内容可以是 "),
    h("i", { style: "color: teal" }, "VNode"),
  ]),
});
```

方法`isVNode`使用“鸭式辨型法”判断参数值是否为`VNode`类型。 `VNode`类型更多内容请查看 [VNode class declaration](https://github.com/vuejs/vue/blob/dev/src/core/vdom/vnode.js) 。

```js
export function isVNode(node) {
  return node !== null && typeof node === 'object' && hasOwn(node, 'componentOptions');
};
```

**距离窗口顶部偏移量计算**

页面中可以存在多个`Message` 实例，新 Message 消息会在旧的下面展示，也就是按照创建时间由早到晚，实例从上到下依次展示。

* 首个显示（最上面）的实例的偏移量由属性`offset`值控制。
* `16`用于设置多个实例显示时，实例元素之间的间距。
* `offsetHeight` 返回实例元素的像素高度，高度包含该元素的垂直内边距和边框，且是一个整数。
* 新创建的实例在计算完后偏移量后才会将最添加至数组中。

```js
// 计算距离窗口顶部偏移量计算 
let verticalOffset = options.offset || 20; // 默认是20px
// 新的Message弹框在旧的Message弹框下面展示  垂直偏移要加上当前已有的Message弹框的距离
instances.forEach(item => {
  verticalOffset += item.$el.offsetHeight + 16; 
});
instance.verticalOffset = verticalOffset; // 更新偏移量

// ...

// 添加至数组中
instances.push(instance); 
```

新创建的实例距离窗口顶部偏移量`verticalOffset`计算公式如下：

```js
verticalOffset = offset/20 + ( 实例元素高度(offsetHeight) + 16 ) *显示实例个数(instances.length)
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a09d437fe794a2481aa2849c9c23d7d\~tplv-k3u1fbpfcp-watermark.image?)

数组`instances` 用于存放页面可见（未关闭销毁）的实例。当实例关闭后，数组更新操作会在随后详细讲解。

#### Message.close()

属性方法 `close`由两个参数：组件id（创建实例时生成的，格式为`message_xx`）、用户传入的关闭时回调函数，用于控制整个页面实例数组以及偏移量计算，执行用户传入的关闭时回调函数。

```js
Message.close = function(id, userOnClose) {
  let len = instances.length;
  let index = -1;
  let removedHeight;
  for (let i = 0; i < len; i++) {
    if (id === instances[i].id) {
      removedHeight = instances[i].$el.offsetHeight;
      index = i;
      if (typeof userOnClose === 'function') { 
        userOnClose(instances[i]); // 调用用户传入的关闭时回调函数
      } 
      instances.splice(i, 1); // 从数组instances中去掉移除该实例
      break;
    }
  }
  // 未找到该实例 或者 该实例之后没有元素 退出代码
  if (len <= 1 || index === -1 || index > instances.length - 1) return;
  // 只需要调整index 大于当前Message的实例偏移量
  for (let i = index; i < len - 1 ; i++) {
    let dom = instances[i].$el;
    dom.style['top'] =
      parseInt(dom.style['top'], 10) - removedHeight - 16 + 'px';
  }
};
```

方法实现功能主要有以下步骤：

1. 根据组件id从数组中查找实例的索引index。
2. 未找到对应实例，匹配条件 `index === -1` ，退出方法。
3. 若找到对应实例，记录实例索引index。
   * 获取实例元素 offsetHeight。
   * 调用用户传入的关闭时回调函数。
   * 从数组instances中去掉移除该实例。
   * 判断该索引后是否还有其他元素，没有的话，退出方法；有的话执行下一步。
   * 只调整 index 大于当前Message的实例的高度，也就是实例之后的实例元素。
   * 根据移除实例元素 offsetHeight 和 间距16，重新计算偏移量。

下图展现了关闭页面第二个实例后，随后的二个实例的偏移量需要重新计算。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de3fc6c6972949efb703a6dcd3bf93b9\~tplv-k3u1fbpfcp-watermark.image?)

#### 组件的关闭流程

现在将各功能点串起来，解释下组件关闭时，发生了什么？

对象`Message`定义中，传入组件的关闭回调函数，不是用户传入的原始值，是做了一层包装。通过闭包将id和onClose回调函数作为参数，调用 `Message.close()` 方法。

即使用户没有传入关闭时回调函数，组件实例创建时也会有方法传入，用于组件关闭销毁后更新整个页面实例数组更新剩余实例偏移量。

```js
// packages\message\src\main.js
const Message = function(options) { 
  // ...
  let userOnClose = options.onClose; // 用户传入的关闭时的回调函数 
  let id = 'message_' + seed++; // 组件实例 id
  // 关闭时 回调函数执行逻辑 Message.close
  options.onClose = function() {
    Message.close(id, userOnClose);
  }; 
  // ...
};
```

当实例由关闭图标点击、定时器、ESC按键等方式触发关闭`close`方法时，必然会执行回调函数，相当于`Message.close(id, userOnClose)`。 此时组件也会调用方法 `handleAfterLeave` 销毁实例移除DOM元素。

```js
// packages\message\src\main.vue
// template
<transition name="el-message-fade" @after-leave="handleAfterLeave"> 
  <div v-show="visible">
    // ...
  </div>
</transition>
 
// methods 
handleAfterLeave() { 
  this.$destroy(true);
  this.$el.parentNode.removeChild(this.$el);
},

close() {
  this.closed = true;
  if (typeof this.onClose === 'function') {
    this.onClose(this);
  }
},
```

> 调用 `Message` 或 `this.$message` 会返回当前 Message 的实例。如果需要手动关闭实例，可以调用它的 `close` 方法。

#### Message.closeAll()

属性方法 `closeAll`用于关闭所有 `message` 实例。

遍历 `instances`，逐个调用实例的`close()`方法。相当于按`ESC`键关闭效果。

```js
Message.closeAll = function() {
  for (let i = instances.length - 1; i >= 0; i--) {
    instances[i].close();
  }
};
```

> 此处 `close()` 方法时组件内部定义的，不是 `Message.close()` 。

#### 快捷方法

定义了各状态的便捷属性方法，例如 `Message.success(options)`。通过格式化参数，指定了`options.type`属性值。

```js
// 定义了各状态的便捷方法 Message.success(options)
['success', 'warning', 'info', 'error'].forEach(type => {
  Message[type] = options => {
    if (typeof options === 'string') {
      options = {
        message: options
      };
    }
    options.type = type;
    return Message(options);
  };
});
```

### 样式实现

组件样式源码 `packages\theme-chalk\src\message.scss` 使用混合指令 `b`、`when`、`m`、`e` 嵌套生成组件样式。

```scss
// 生成 .el-message
@include b(message) {
  // ...
  
  // 生成 .el-message.is-center
  @include when(center) {
    // ...
  }
  @include when(closable) {
    // 生成 .el-message.is-closable .el-message__content
    .el-message__content {
      // ...
    }
  }
  // 生成 .el-message p
  p {
    // ...
  }
  
  @include m(info) {
    // 生成 .el-message--info .el-message__content
    .el-message__content {
      // ...
    }
  }
  // 生成 .el-message--success/warning/error
  @include m(success) {
    // ...
    
    // 生成 .el-message--success/warning/error .el-message__content
    .el-message__content {
      // ...
    }
  } 
  // warning/error 省略...
    
  // 生成 .el-message__icon
  @include e(icon) {
    // ...
  }
  // 生成 .el-message__content
  @include e(content) {
    // ...
    
    // 生成 .el-message__content:focus
    &:focus {
      // ...
    }
  }
  // 生成 .el-message__closeBtn
  @include e(closeBtn) {
    // ...
    
    // 生成 .el-message__closeBtn:focus
    &:focus {
      // ...
    }
    // 生成 .el-message__closeBtn:hover
    &:hover {
      // ...
    }
  }
  // 生成  .el-icon-success/error/info/warning
  & .el-icon-success {
    // ...
  }
  // error/info/warning 省略...
}
// 生成 .el-message-fade-enter,.el-message-fade-leave-active
.el-message-fade-enter,
.el-message-fade-leave-active {
  // ...
}
```

### 📚参考&关联阅读

['api/Vue-extend',vuejs](https://cn.vuejs.org/v2/api/#Vue-extend)\
['transitions#JavaScript 钩子',vuejs](https://cn.vuejs.org/v2/guide/transitions.html#JavaScript-%E9%92%A9%E5%AD%90)\
['CSS/position',MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position)\
[自定义指令,vuejs](https://cn.vuejs.org/v2/guide/custom-directive.html)\
['HTMLElement/offsetHeight',MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/offsetHeight)\
['api/Vue-extend',vuejs](https://cn.vuejs.org/v2/api/#Vue-extend)\
['transitions#JavaScript 钩子',vuejs](https://cn.vuejs.org/v2/guide/transitions.html#JavaScript-%E9%92%A9%E5%AD%90)\
['CSS/position',MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position)\
[VNode class declaration](https://github.com/vuejs/vue/blob/dev/src/core/vdom/vnode.js)
