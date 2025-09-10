---
title: 模块化
tags:
  - 模块化
  - CommonJS
  - ES Module
createTime: 2025/09/10 20:29:06
permalink: /article/rg79e9r8/
---

`模块化指的是系统组件可以分离和重新组合的程度，也指软件包的逻辑单元划分。模块化系统的优势在于可以独立地推理各个部分。 -- MDN`

## 1、CommonJS

CommonJS 是 Node.js 的模块系统，采用同步加载的方式。

### 基本语法

```javascript
// 导出模块
module.exports = {
  name: 'CommonJS',
  version: '1.0.0'
};

// 或者
exports.name = 'CommonJS';
exports.version = '1.0.0';

// 导入模块
const module = require('./module');
const { name, version } = require('./module');
```

### 特点

- **同步加载**：模块在运行时同步加载
- **运行时确定**：模块的依赖关系在代码运行时确定
- **动态导入**：支持条件导入和动态路径
- **缓存机制**：模块只加载一次，后续引用返回缓存

### 使用场景

- Node.js 服务端开发
- 传统的前端构建工具（如 Webpack 早期版本）

### 限制

- **同步加载限制**：在浏览器环境中，同步加载会阻塞页面渲染
- **不支持 Tree Shaking**：无法在编译时消除未使用的代码
- **循环依赖问题**：处理循环依赖时可能出现意外行为
- **静态分析困难**：无法在编译时确定所有依赖关系
- **浏览器兼容性**：需要构建工具转换才能在浏览器中使用

## 2、ES Module

ES Module 是 ECMAScript 2015 (ES6) 引入的官方模块系统，采用静态分析的方式。

### 基本语法

```javascript
// 导出模块
export const name = 'ES Module';
export const version = '1.0.0';

// 默认导出
export default function() {
  return 'Hello ES Module';
}

// 导入模块
import { name, version } from './module.js';
import defaultExport from './module.js';
import * as module from './module.js';
```

### 特点

- **静态分析**：模块依赖在编译时确定
- **异步加载**：支持异步模块加载
- **Tree Shaking**：支持死代码消除
- **循环依赖处理**：更好的循环依赖处理机制

### 使用场景

- 现代前端开发
- 浏览器原生支持
- 现代构建工具（Vite、Webpack 5+）

### 限制

- **静态导入限制**：`import` 声明必须在模块的顶层作用域，不能嵌套在条件语句或函数中。但可以使用动态 `import()` 来实现条件或按需加载。
- **文件扩展名要求**：在浏览器中原生使用 ES Module 时，必须明确指定文件扩展名（如 `.js`）。在 Node.js 或打包工具中，此限制可能被放宽或通过配置解决。
- **Node.js 兼容性**：在 Node.js 中，要使用 ES Module 语法，需将文件扩展名设为 `.mjs`，或在 `package.json` 中设置 `"type": "module"`。
- **动态导入复杂性**：动态导入 `import()` 返回 Promise，需要异步处理，这增加了代码的复杂性。
- **循环依赖处理**：虽然支持更好，但复杂的循环依赖仍可能导致逻辑错误，应尽量避免。

## 3、CommonJS vs ES Module

| 特性 | CommonJS | ES Module |
|------|----------|-----------|
| 加载方式 | 同步加载，适用于服务器端文件系统 | 异步加载，更适合网络环境的浏览器 |
| 分析时机 | 运行时 | 编译时 |
| 导入语法 | `require()` | `import` |
| 导出语法 | `module.exports` | `export` |
| 动态导入 | 支持 | 支持（`import()`） |
| 循环依赖 | 部分支持 | 更好支持 |
| Tree Shaking | 不支持 | 支持 |
| 浏览器支持 | 需要打包 | 原生支持 |

### 互操作性

在 Node.js 中，可以通过以下方式实现互操作：

```javascript
// CommonJS 中使用 ES Module
import('./es-module.js').then(module => {
  console.log(module.default);
});

// ES Module 中使用 CommonJS
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const commonjsModule = require('./commonjs-module.js');
```

## 4、参考文档

- [CommonJS - 阮一峰](https://javascript.ruanyifeng.com/nodejs/module.html)
- [ES6 Module - 阮一峰](https://es6.ruanyifeng.com/#docs/module)
- [Webpack模块化原理](https://webpack.js.org/concepts/modules/)
- [Vite模块化实践](https://vitejs.dev/guide/features.html#es-modules)