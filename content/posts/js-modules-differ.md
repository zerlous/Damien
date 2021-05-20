---
title: "javascript 模块化异同"
date: 2021-05-20T10:05:13+08:00
draft: true
---

### CommonJS

`CommonJS`为**同步加载**模式，主要应用场景为服务端，通常应用于`Node.js`的模块化输出与引用。核心的模块化环境变量有`module`、`exports`、`require`、`global`。

- `require`命令用于提供模块引用。
- `module.exports`命令用以提供规范模块的输出。（创建后被引用，会创建引用缓存）

```Javascript
// a.js
const text = 'hello world';
const printText = (name) => {
    console.log(text);
    console.log('my name is ', name);
};

module.exports = {
    printText
}
```

```Javascript
// b.js
const { printText } = require('./a.js');

printText('zerlous');
// hello world
// my name is zerlous
```

- `CommonJS`主要采用**同步加载模块**，并且应用于服务端，大部分加载的文件资源都在本地服务器，所以执行效率很高。
- 不适用于客户端（浏览器端）。

### AMD

`AMD` (_Asynchronous Module Definition_)为**异步加载**模式，模块的加载不会产生阻塞，所有依赖该模块的的依赖语法都会在一个回调函数`callback`中，只有等依赖模块加载完成后，才会执行该回调函数。（\* 核心`RequireJS`）。

核心的模块功能命令有`define`、`require`、`return`、`define.amd`。

- `define`为全局环境变量下的全局函数，用于定义模块。eg:`define(id?, dependencies?, factory)`。
- `require`命令用于提供模块引用。
- `return`命令用以提供规范模块的输出。
- `define.amd`属性是一个对象，此属性的存在来表明函数遵循[**AMD 规范**](<https://en.wikipedia.org/wiki/Asynchronous_module_definition#:~:text=Asynchronous%20module%20definition%20(AMD)%20is,loads%20them%20asynchronously%20if%20desired.>)。

```Javascript
// a.js
define(function() {
  const text = 'hello world';
  const printText = (name) => {
    console.log(text);
     console.log('my name is ', name);
  };
  return {
    printText
  };
});
```

```Javascript
// app.js
define(function () {
  const a_model = require('./a.js');
  a_model.printText('zerlous');
  // hello world
  // my name is zerlous
});
```

```Html
<script src="https://cdn.bootcss.com/require.js/2.3.6/require.min.js"></script>
<script>
    requirejs(['app']);
</script>
```

### CMD

`CMD`(*Common Module Definition*)为**通用模块定义**，同时适用于服务端及客户端。（\* 核心`Sea.js`）。
`CMD`模块加载模式与`AMD`类似，但是有差异。

- `AMD`依赖前置、提前执行。
  > `AMD`在加载模块完成后，**立即会执行该模块**，所有模块加载执行完成后，执行`callback`。主流程一定是在所有依赖加载完成后执行的，但是依赖模块的**执行顺序是依赖网络，并非书写顺序**。
- `CMD`依赖就近、延迟执行
  > `CMD`在加载完模块后，**并不会立即执行**，所有模块加载执行完成后，执行`callback`。在`callback`中，遇到`require`再执行对应的模块，**执行顺序与书写顺序保持一致**。

```Javascript
// a.js
define(function(require, exports, module) {
  const text = 'hello world';
  const printText = (name) => {
    console.log(text);
     console.log('my name is ', name);
  };
  exports.printText = printText;
});
```

```Javascript
// b.js
define(function(require, exports, module) {
  const printDate = () => {
     console.log('print datastamp is: ', (new Date())*1));
  };
  exports.printDate = printDate;
});
```

```Javascript
// app.js
define(function (require, exports, module) {
  const a_model = require('./a.js');
  a_model.printText('zerlous');
  // hello world
  // my name is zerlous
  const b_model = require('./b.js');
  b_model.printDate();
  // print datastamp is: 1621111111111
});
```

```Html
<script src="https://cdn.bootcss.com/seajs/3.0.3/sea.js"></script>
<script>
    seajs.use('./app.js')
</script>
```


### UMD

`UMD`(*Universal Module Definition *)为**通用模块定义**，主要解决`CommonJS`和`AMD`两种模式下代码不通用的问题，同时支持老式的全局变量规范。

- ⭐️⭐️⭐️`AMD`规范。 `define`为函数类型，且存在`define.amd`对象.
- ⭐️⭐️⭐️`CommonJS`规范。`module`为对象类型，且存在`module.exports`.

```Javascript
// app.js
(function (global, factory) {
    typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
    typeof define === 'function' && define.amd ? define(factory) :
    (global = global || self, global.app = factory()); // 原始代码
}(this, (function () { 'use strict';
    const app = () => {
        return 'hello world';
    };
    return app;
})));
```

```Html
<!-- index.html -->
<script src="app.js"></script>
<script>
  console.log(app());
  // hello world
</script>
```

### ES Modules
`ESM`(*ECMA Script Modules*)为javascript官方的**标准化模块系统**。 同时适用于服务端和客户端。是当前主流使用的多浏览器兼容的模块管理系统。

- 通过`import`和`export`来确定模块的导入导出。且与`CommonJS`模块可混合使用。
- `ESM`输出的是值的引用，输出接口动态绑定。而`CommonJS`输出的是值的拷贝。
- `ESM`编译时执行。 而`CommonJS`模块总是在运行时加载。

#### ESM引用输出与CommonJS值拷贝输出的区别？

- `ESM`值引用输出, 引用模块内的作用域是维持的。引用方法是会重复执行的。
- `CommonJS`值拷贝缓存，重复引用不会重复执行，获取的是模块缓存。

##### ESM引用输出
```Javascript
// a.js
import { age } from './b.js';

console.log(age);
setTimeout(() => {
    console.log(age);
    import('./b.js').then(({ age }) => {
        console.log(age);
    })
}, 100);

// b.js
export let age = 1;

setTimeout(() => {
    age = 2;
}, 10);
// 打开 index.html
// 执行结果：
// 1
// 2
// 2
```

##### CommonJS值拷贝输出
```Javascript
// a.js
const b = require('./b');
console.log(b.age);
setTimeout(() => {
  console.log(b.age);
  console.log(require('./b').age);
}, 100);

// b.js
let age = 1;
setTimeout(() => {
  age = 18;
}, 10);
module.exports = {
  age
}
// 执行：node a.js
// 执行结果：
// 1
// 1
// 1
```


##### 动态加载和静态编译的区别?

- 动态加载，只有当模块运行时，才知道导出的模块key。
- 静态编译，在编译阶段就知道导出的模块key。

```Javascript
// 动态加载
var test = 'hello'
module.exports = {
  [test + '1']: 'world'
}

// ----------------------
// 静态编译
export function hello() {return 'world'}
```

##### ESM的特性

- `import`命令会被 JavaScript 引擎静态分析，优先于模块内的其他内容执行。
- `export`命令会有变量声明提前的效果。


### 总结
- `CommonJS`同步加载
- `AMD`异步加载
- `UMD` = `CommonJS` + `AMD`
- `ESM`是标准规范, 取代`UMD`

### 参考
https://mp.weixin.qq.com/s/uZASCbdLrL8tWBa31CgTjA
https://github.com/rollup/rollup/issues/3011#issuecomment-516084596
https://github.com/rollup/plugins/tree/master/packages/commonjs
https://www.zhihu.com/question/63240671
https://www.yuque.com/baichuan/notes/emputh
https://github.com/indutny/webpack-common-shake#limitations
http://xbhong.top/2018/03/12/commonjs/
https://www.douban.com/note/283566440/
https://blog.fundebug.com/2018/08/15/reduce-js-payload-with-tree-shaking/
http://huangxuan.me/js-module-7day/#/13
https://www.jianshu.com/p/6c26fb7541f1

### References
- [1] [CommonJS规范](http://wiki.commonjs.org/wiki/CommonJS)
- [2] [AMD规范](https://github.com/amdjs/amdjs-api/wiki/AMD)
- [3] [CMD规范](https://github.com/cmdjs/specification/blob/master/draft/module.md)
- [4] [UMD文档](https://github.com/umdjs/umd): 
- [5] [ES Modules 文档](http://es6.ruanyifeng.com/#docs/module-loader)
- [6] [深入理解 ES6 模块机制](https://juejin.im/entry/5a879e28f265da4e82635152)