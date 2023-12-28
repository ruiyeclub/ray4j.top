---
title: Node.js模块化选择Commonjs和ES6
date: 2023-12-28 18:03:40
tags:
---

> Node.js 使用的模块系统是基于 CommonJS 的，但随着时间的推移，ES6 模块也得到了越来越多的支持。以下是关于 CommonJS 和 ES6 模块的简要概述：

1.**CommonJS**：

- 最初为 Node.js 设计，所以又称 Node.js 模块系统。
- 使用 `require()` 函数来导入模块，使用 `module.exports` 或 `exports` 对象来导出模块。

2.**ES6 模块**：

- 是 ECMAScript 6 (ES2015) 中引入的新模块系统。
- 使用 `import` 语句来导入模块，使用 `export` 语句来导出模块。
- 有静态和动态两种导入方式，提供更强的类型检查和代码编译时优化。

> 请注意，随着Node.js的发展，许多库已经迁移到ES模块，因此确保你使用的库与你当前的Node.js版本兼容是很重要的。



### 一、搭建ES6模块系统

1.把需要更换的`require`全部换成`import`

2.在src下创建一个.babelrc文件

```js
{
  "presets":["es2015"]
}
```

3.在package.json中的一段加上`--exec babel-node`

```js
{
  "name": "api",
  "version": "1.0.0",
  "description": "",
  "type": "module", // 添加type类型
  "main": "index.js",
  "scripts": {
    "dev": "nodemon ./src/main.js --exec babel-node",
    "test": "echo \"Error: no test specified\" && exit 1 --exec babel-node"
  },
```

4.安装插件 `npm install babel-preset-es2015`



### 二、关于两者的引用区别

1.Commonjs：

```js
// moduleA.js  
const message = 'Hello from CommonJS';  
module.exports = message;
```

```js
// main.js  
const moduleA = require('./moduleA');  
console.log(moduleA); // 输出 'Hello from CommonJS'
```

**需要注意的是，CommonJS 的 `require` 函数返回的是一个对象，该对象的属性对应于模块导出的内容。在上面的例子中，`moduleA` 是一个包含 `message` 属性的对象。因此，我们可以使用 `moduleA.message` 来访问导出的内容。**

2.ES6：

```js
// moduleA.js  
export const message = 'Hello from ES6';
```

```js
// main.js  
import { message } from './moduleA';  
console.log(message); // 输出 'Hello from ES6'
```

**需要注意的是，ES6 的 `import` 语句返回的是导出的内容本身，而不是一个对象。因此，我们可以直接使用导入的内容，而不需要通过属性来访问。**

