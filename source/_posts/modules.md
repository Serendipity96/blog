---
title: 还傻傻分不清 module.exports 和 export default  吗？
tags:
- 模块化
categories:
- JavaScript
# toc: false
date: 2019-09-29 10:10
banner_img: https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/28/16d7860a4beac7a1~tplv-t2oaga2asx-image.image
banner_img_set: https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/28/16d7860a4beac7a1~tplv-t2oaga2asx-image.image
---

## 前言

在使用 vue、react、node 的时候，常常会看到 module.exports，export default，require，import 等字段，因为我对这些字段的概念非常模糊，所以导致我在写代码的时候，在 node 项目里混用了 export default，在 vue 的项目里写 module.exports。

那么今天就来梳理一下有关模块化的知识。

## ESM 的模块

### 语法

ESM（ECMA Script Modules）模块主要由两个命令构成：export 和 import。

```text
暴露模块：export default {} , export {} , export function(){}

引入模块：import {xxx} from 'path'
```

#### 注意
import 的大括号里面指定要从其他模块导入的变量名，如果 export 命令没有写 default，那么 import 大括号里面的变量名，必须与 export 导出的名称相同。
 
```js
// test.js
export {foo}

// main.js
import { foo } from './test.js'

// 如果想为输入的变量重新取一个名字，要使用as关键字，将输入的变量重命名
import { foo as bar } from './test.js';
```

#### default 的作用

default 为模块指定默认输出，这样在引入时就不必关心模块输出的名字。

```js
// test.js
function foo() {};

export default {foo}

// main.js
import bar from './test.js'
```

本质上，export default 就是输出一个叫做 default 的变量或方法，然后系统允许你为它取任意名字。


有关 ESM 模块的语法，可以阅读 [阮一峰](http://es6.ruanyifeng.com/#docs/module) 的文章，这里不详细写出所有写法。

### 加载机制

我们在使用 import 和 export 的时候，常常看到的是在顶层作用域使用 export 和 import，不会在块级作用域内看到 import 和 export，这是为什么呢？

因为 ESM 模块的设计思想是尽量静态化，**编译时**就能确定模块的依赖关系，以及输入和输出的变量，如果处于块级作用域内，就没法做静态优化了，违背了 ES6 模块的设计初衷。

但是我明明在 vue router 的使用中看到了这样的写法：
```js
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
```
此时 import 的写法和常规写法不一样，是 import(), 并且确实出现在了块级作用域内。

ESM 有一个[提案](https://github.com/tc39/proposal-dynamic-import)，建议引入 import()函数，完成动态加载。现在支持动态加载的比如：vue router、webpack。

#### vue router 动态加载
vue router 的 [路由懒加载](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)

#### webpack 动态导入
webpack [动态导入](https://webpack.docschina.org/guides/code-splitting/#%E5%8A%A8%E6%80%81%E5%AF%BC%E5%85%A5-dynamic-imports-)


## Node.js 的模块

### 语法
```text
暴露模块：module.exports = value 或 exports.xxx = value

引入模块：require(xxx),如果是第三方模块，xxx为模块名；如果是自定义模块，xxx为模块文件路径
```

Node 的模块输出和引入的方式与 ESM 不同，Node 采用的是 CommonJS 模块规范。

CommonJS 规范规定，在每个模块内部，module 变量代表当前模块。这个变量是一个对象，它的 exports 属性（module.exports）是对外的接口。

### 为什么采用 CommonJS 规范呢

Node.js 主要用于服务端项目，CommonJS 规范也主要用于服务端编程，所以 Node 的模块设计采用 CommonJS 规范很合适。

## 模块化发展的历史

### 为什么会有 AMD CMD 规范

服务端模块的加载是同步的，但是浏览器资源是异步加载，同步意味着阻塞，在没有 ESM 模块之前，浏览器想做模块化怎么办呢？

AMD CMD 解决方案。

实际上 AMD (Asynchronous Module Definition) 是 RequireJS 在推广过程中对模块定义规范化的产出。

CMD (Common Module Definition) 是 SeaJS 在推广过程中对模块定义的规范化产出。

SeaJS 和 requireJS 解决的都是模块化问题,只不过在模块定义方式和模块加载（可以说运行、解析）时机上有所不同。

以下例子引用自 [WEB 前端模块化都有什么](https://juejin.im/post/6844903717947310093#heading-2)

**AMD**

```js
// hello.js
define(function() {
    console.log('hello init');
    return {
        getMessage: function() {
            return 'hello';
        }
    };
});
// world.js
define(function() {
    console.log('world init');
});

// main
define(['./hello.js', './world.js'], function(hello) {
    return {
        sayHello: function() {
            console.log(hello.getMessage());
        }
    };
});

// 输出
// hello init
// world init
```

**CMD**
```js
// hello.js
define(function(require, exports) {
    console.log('hello init');
    exports.getMessage = function() {
        return 'hello';
    };
});

// world.js
define(function(require, exports) {
    console.log('world init');
    exports.getMessage = function() {
        return 'world';
    };
});

// main
define(function(require) {
    var message;
    if (true) {
        message = require('./hello').getMessage();
    } else {
        message = require('./world').getMessage();
    }
});

// 输出
// hello init
```

CMD 的输出结果中，没有打印"world init", 并是不 world.js 文件没有加载。

#### AMD CMD 规范加载机制

AMD 与 CMD 都是在页面初始化时加载完成所有模块，区别是 CMD 就近依赖, 是当模块被 require 时才会触发执行, AMD 推崇依赖前置，在定义模块的时候就要声明其依赖的模块。

同样都是异步加载模块，AMD 在加载模块完成后就会执行该模块，所有模块都加载执行完后会进入 require 的回调函数，执行主逻辑，这样的效果就是依赖模块的执行顺序和书写顺序不一定一致，看网络速度，哪个先下载下来，哪个先执行，但是主逻辑一定在所有依赖加载完成后才执行。

CMD 加载完某个依赖模块后并不执行，只是下载而已，在所有依赖模块加载完成后进入主逻辑，遇到 require 语句的时候才执行对应的模块，这样模块的执行顺序和书写顺序是完全一致的。

这也是很多人说 AMD 用户体验好，因为没有延迟，依赖模块提前执行了，CMD 性能好，因为只有用户需要的时候才执行。

### UMD 规范解决了什么问题

AMD & CMD，CommonJS 规范是两类规范，AMD & CMD 用于浏览器模块化，CommonJS  用于服务端模块化，但是大家的期望有一个统一的规范来支持这两种规范。于是，UMD 规范诞生了。

UMD (Universal Module Definition)，它可以通过运行时或者编译时让同一个代码模块在使用 CommonJs、CMD / AMD 的项目中运行，同一个 JavaScript 包在浏览器 / 服务端只需要遵守同一个写法就可以了。

UMD 没有自己专有的规范，是集结了 CommonJs、CMD、AMD 的规范于一身，UMD 先判断是否支持 Node 模块格式（exports 是否存在），存在则使用 Node 模块格式，再判断是否支持 AMD（define 是否存在），存在则使用 AMD 方式加载模块，如果前两个都不存在，则将模块公开到全局（window 或 global）。

### 更早之前

立即执行函数实现模块化（IIFE，Immediately-Invoked Function Expression）

使用立即执行函数，表达式中的变量不能从外部访问。现在项目中已经看不到这样的写法了。

例如：

```js
(function(){
    var count = 0;
    return count;
})();
```



## 模块化方案总结
| | ESM | CommonJS | AMD | CMD |UMD|
| ------ | ------ | ------ | ------ | ------ | ------ |
| 加载机制 | 编译时 | 运行时 | 提前预加载 | 编译时 & 运行时按需加载 | - |
| 同步/异步 | 异步 | 同步 | 异步 | 异步，有延迟执行的情况 | - |
| 适用场合 | 浏览器、服务端 | 服务端 | 浏览器 | 浏览器| 浏览器、服务端 |
| 是否常见 | ☆☆☆ |☆☆☆  | ☆ | ☆ | ☆ |

ESM 在语言标准的层面上，成为浏览器和服务端通用的模块解决方案。

## 工具时代
webpack 在定义模块上，支持上面提到的所有模块声明方式，只需要在 webpack 的 output 中添加 [libraryTarget](https://webpack.js.org/configuration/output/#outputlibrarytarget): 'commonjs/amd/umd'即可。


## 模块化的好处
1. 避免命名冲突，每个模块内的变量仅对自己可见，外部获取依赖模块输出
2. 按需加载
3. 解耦、复用、高可维护性

## 参考资料
[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/module)

[前端模块化详解(完整版)](https://juejin.im/post/6844903744518389768)

[AMD 与 CMD 区别](http://www.bubuko.com/infodetail-1125914.html)

[SeaJS 和 RequireJS 的差异](https://github.com/seajs/seajs/issues/277)
