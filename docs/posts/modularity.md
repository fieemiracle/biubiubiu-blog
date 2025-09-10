---
date: 2025-09-10
category:
  - 模块化
tag:
  - CommonJS
  - ES Module
  - AMD
  - UMD
  - 前端工程化
---

# JavaScript 模块化发展历程

## 一、什么是模块化

模块化指的是系统组件可以分离和重新组合的程度，也指软件包的逻辑单元划分。模块化系统的优势在于可以独立地推理各个部分，提高代码的可维护性、可复用性和可测试性。
核心概念：
- **封装性**：模块内部实现细节对外部隐藏
- **独立性**：模块之间相互独立，减少耦合
- **复用性**：模块可以在不同项目中重复使用
- **可维护性**：模块化使代码结构清晰，便于维护

## 二、早期JavaScript的问题

在模块化规范出现之前，JavaScript开发存在以下严重问题：

### 1. 全局污染问题

```javascript
// example1.js
var globalVar = "I'm global";
function funcA() {
    console.log("funcA from example1");
}

// example2.js  
var globalVar = "I'm also global"; // 覆盖了example1的变量
function funcA() {
    console.log("funcA from example2"); // 覆盖了example1的函数
}

// example3.js
function funcA() {
    console.log("funcA from example3"); // 再次覆盖
}
```

```html
<body>
  <script src="./example1.js"></script>
  <script src="./example2.js"></script>
  <script src="./example3.js"></script>
</body>
```

**问题分析：**
- 所有变量和函数都挂载到全局对象（window）上
- 同名变量和函数会被覆盖
- 无法确定变量和函数的来源
- 难以追踪依赖关系

### 2. 依赖管理混乱

```javascript
// 依赖关系不明确
// example1.js 需要 example2.js 的某些功能
// 但无法明确表达这种依赖关系
```
JavaScript加载顺序是从上到下的，如果example2.js和example3.js都依赖example1.js的功能，那么：
- example2.js和example3.js能调用example1.js的方法
- example1.js无法调用example2.js和example3.js的方法
- 如果example2.js依赖example3.js，就会出现问题

## 三、模块化解决方案的发展

### 1. 立即执行函数表达式（IIFE）

早期的模块化尝试：

```javascript
// 使用IIFE创建模块
var MyModule = (function() {
    var privateVar = "private";
    
    function privateFunction() {
        return "private function";
    }
    
    return {
        publicVar: "public",
        publicFunction: function() {
            return privateFunction();
        }
    };
})();

// 使用
console.log(MyModule.publicVar); // "public"
console.log(MyModule.publicFunction()); // "private function"
```

**优点：**
- 避免了全局污染
- 实现了基本的封装

**缺点：**
- 仍然需要手动管理依赖
- 没有统一的规范
- 无法处理循环依赖

### 2. AMD（Asynchronous Module Definition）

AMD是RequireJS推广的模块化规范，专门为浏览器环境设计。

#### AMD语法

```javascript
// 定义模块
define(['dependency1', 'dependency2'], function(dep1, dep2) {
    // 模块代码
    return {
        // 导出的内容
    };
});

// 使用模块
require(['module1', 'module2'], function(mod1, mod2) {
    // 使用模块
});
```

#### 实际例子

```javascript
// math.js - 定义模块
define([], function() {
    return {
        add: function(a, b) {
            return a + b;
        },
        multiply: function(a, b) {
            return a * b;
        }
    };
});

// calculator.js - 依赖math模块
define(['math'], function(math) {
    return {
        calculate: function(x, y) {
            return math.add(math.multiply(x, 2), y);
        }
    };
});

// main.js - 入口文件
require(['calculator'], function(calculator) {
    console.log(calculator.calculate(3, 4)); // 10
});
```

**优点：**
- 异步加载，适合浏览器环境
- 依赖关系明确
- 支持动态加载

**缺点：**
- 语法复杂
- 不是JavaScript原生支持
- 需要额外的加载器

### 3. CommonJS

CommonJS是Node.js采用的模块化规范，主要特点是同步加载。

#### CommonJS语法

```javascript
// 导出模块
module.exports = {
    // 导出的内容
};

// 或者
exports.functionName = function() {
    // 函数实现
};

// 导入模块
const module = require('./module');
```

#### 实际例子

```javascript
// math.js
function add(a, b) {
    return a + b;
}

function multiply(a, b) {
    return a * b;
}

module.exports = {
    add,
    multiply
};

// calculator.js
const math = require('./math');

function calculate(x, y) {
    return math.add(math.multiply(x, 2), y);
}

module.exports = {
    calculate
};

// main.js
const calculator = require('./calculator');
console.log(calculator.calculate(3, 4)); // 10
```

#### CommonJS的特点

**优点：**
- 语法简单直观
- 同步加载，适合服务器环境
- 支持循环依赖（有限制）
- 每个文件都是一个模块

**缺点：**
- 同步加载不适合浏览器环境
- 不是JavaScript原生支持
- 无法进行静态分析

#### 静态分析

静态分析是指在代码运行之前，通过分析源代码的语法和结构来理解代码的行为，而不需要实际执行代码。

##### 什么是静态分析

```javascript
// 静态分析可以分析的内容：
// 1. 变量声明和使用
const name = 'John';
console.log(name); // 静态分析器可以知道这里使用了 name 变量

// 2. 函数调用关系
function greet(name) {
    return `Hello, ${name}!`;
}
greet('Alice'); // 静态分析器可以追踪函数调用

// 3. 模块依赖关系
import { add } from './math.js'; // 静态分析器可以确定依赖关系
```

##### CommonJS 无法静态分析的原因

**1. 动态 require**

```javascript
//  CommonJS - 无法静态分析
const moduleName = getUserInput(); // 运行时才能确定
const module = require(moduleName); // 依赖关系在运行时确定

// 或者
const modules = ['module1', 'module2', 'module3'];
modules.forEach(name => {
    const module = require(`./${name}`); // 动态路径
});

// 或者
if (condition) {
    const module = require('./module1');
} else {
    const module = require('./module2');
}
```

**2. 运行时确定依赖**

```javascript
//  CommonJS - 依赖在运行时确定
function loadModule(type) {
    switch(type) {
        case 'math':
            return require('./math');
        case 'string':
            return require('./string');
        default:
            return require('./default');
    }
}

// 静态分析器无法知道会加载哪些模块
const module = loadModule('math');
```

**3. 条件依赖**

```javascript
//  CommonJS - 条件依赖
let math;
if (process.env.NODE_ENV === 'development') {
    math = require('./math-dev');
} else {
    math = require('./math-prod');
}
```

##### ES Module 支持静态分析的原因

**1. 静态导入**

```javascript
// ✅ ES Module - 可以静态分析
import { add, multiply } from './math.js'; // 依赖关系在编译时确定
import utils from './utils.js';

// 静态分析器可以：
// - 确定依赖关系
// - 进行 Tree Shaking
// - 优化打包
// - 检测未使用的代码
```

**2. 明确的依赖路径**

```javascript
//  ES Module - 路径在编译时确定
import { API_URL } from './config.js';
import { validateEmail } from '../utils/validation.js';

// 静态分析器可以构建完整的依赖图
```

**3. 动态导入的明确性**

```javascript
//  ES Module - 动态导入也是可分析的
async function loadModule(name) {
    const module = await import(`./modules/${name}.js`);
    return module;
}

// 静态分析器可以分析可能的导入路径
```

##### 静态分析的实际应用

**1. Tree Shaking（死代码消除）**

```javascript
// math.js
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) { return a / b; }

// main.js
import { add } from './math.js'; // 只导入 add

// 静态分析器可以确定：
// - multiply 和 divide 没有被使用
// - 可以在打包时移除这些未使用的代码
```

**2. 依赖图构建**

```javascript
// 静态分析器可以构建这样的依赖图：
// main.js
//   ├── math.js
//   │   └── constants.js
//   ├── utils.js
//   │   └── helpers.js
//   └── components/
//       ├── Button.js
//       └── Input.js
```

**3. 循环依赖检测**

```javascript
// a.js
import { funcB } from './b.js';
export function funcA() { return funcB(); }

// b.js
import { funcA } from './a.js'; // 静态分析器可以检测到循环依赖
export function funcB() { return funcA(); }
```

##### 构建工具中的静态分析

**Webpack 的模块分析**

```javascript
// webpack 可以静态分析 ES Module
import { add } from './math.js';

// 但无法完全分析 CommonJS
const math = require('./math'); // 需要运行时分析
```

**Rollup 的 Tree Shaking**

```javascript
// rollup.config.js
export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    // Rollup 可以静态分析 ES Module 进行 Tree Shaking
};
```

##### 实际对比示例

**CommonJS 示例**

```javascript
// math.js (CommonJS)
function add(a, b) { return a + b; }
function multiply(a, b) { return a * b; }
function divide(a, b) { return a / b; }

module.exports = { add, multiply, divide };

// main.js (CommonJS)
const math = require('./math');
console.log(math.add(1, 2));

// 问题：
// 1. 静态分析器无法确定 math 对象包含哪些方法
// 2. 无法进行 Tree Shaking，所有方法都会被打包
// 3. 无法在编译时检测未使用的代码
```

**ES Module 示例**

```javascript
// math.js (ES Module)
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) { return a / b; }

// main.js (ES Module)
import { add } from './math.js';
console.log(add(1, 2));

// 优势：
// 1. 静态分析器可以确定只使用了 add 函数
// 2. multiply 和 divide 可以在打包时被移除
// 3. 依赖关系在编译时确定
```

##### 静态分析的限制

即使是 ES Module，某些情况下也无法完全静态分析：

```javascript
// 动态导入路径
const moduleName = getModuleName(); // 运行时确定
const module = await import(`./${moduleName}.js`);

// 条件导入
if (condition) {
    await import('./module1.js');
} else {
    await import('./module2.js');
}
```

##### 总结

静态分析是现代前端工程化的重要基础：

- **CommonJS**：依赖关系在运行时确定，无法进行静态分析
- **ES Module**：依赖关系在编译时确定，支持静态分析
- **静态分析的好处**：Tree Shaking、依赖优化、循环依赖检测、更好的打包优化
- **实际影响**：影响构建工具的性能优化能力，影响最终打包文件的大小

#### CommonJS的模块系统

```javascript
// 模块对象结构
console.log(module);
/*
{
    id: '/path/to/file.js',
    exports: {},
    parent: null,
    filename: '/path/to/file.js',
    loaded: false,
    children: [],
    paths: ['/path/to/node_modules', ...]
}
*/

// require函数的工作原理
function require(id) {
    // 1. 解析模块路径
    // 2. 检查模块缓存
    // 3. 创建新模块
    // 4. 执行模块代码
    // 5. 返回module.exports
}
```

#### module.exports vs exports

在 CommonJS 中，`module.exports` 和 `exports` 是两个容易混淆的概念，理解它们的区别对于正确使用 CommonJS 模块系统至关重要。

##### 基本概念

```javascript
// 每个模块都有一个 module 对象
console.log(module);
// module.exports 是模块的真正导出对象
// exports 是 module.exports 的一个引用

console.log(exports === module.exports); // true
```

##### 使用方式对比

**1. 使用 module.exports**

```javascript
// math.js
function add(a, b) {
    return a + b;
}

function multiply(a, b) {
    return a * b;
}

// 方式1：直接赋值
module.exports = {
    add,
    multiply
};

// 方式2：逐个添加属性
module.exports.add = add;
module.exports.multiply = multiply;

// 方式3：导出单个函数
module.exports = add;
```

**2. 使用 exports**

```javascript
// math.js
function add(a, b) {
    return a + b;
}

function multiply(a, b) {
    return a * b;
}

// 方式1：逐个添加属性
exports.add = add;
exports.multiply = multiply;

// 方式2：使用对象展开
Object.assign(exports, {
    add,
    multiply
});
```

##### 关键区别和陷阱

**1. 引用关系**

```javascript
// exports 是 module.exports 的引用
console.log(exports === module.exports); // true

// 但是，如果重新赋值 exports，引用关系会断开
exports = { add: function() {} }; // ❌ 错误！不会影响 module.exports

// 正确的做法是修改 module.exports
module.exports = { add: function() {} }; // ✅ 正确
```

**2. 常见错误示例**

```javascript
// ❌ 错误示例1：重新赋值 exports
exports = {
    add: function(a, b) { return a + b; }
};
// 这样写不会导出任何内容，因为 exports 的引用被改变了

// ❌ 错误示例2：混合使用
exports.add = function(a, b) { return a + b; };
module.exports = { multiply: function(a, b) { return a * b; } };
// 最终只会导出 multiply，add 被覆盖了

// ✅ 正确示例1：统一使用 module.exports
module.exports = {
    add: function(a, b) { return a + b; },
    multiply: function(a, b) { return a * b; }
};

//  正确示例2：统一使用 exports
exports.add = function(a, b) { return a + b; };
exports.multiply = function(a, b) { return a * b; };
```

**3. 实际测试**

```javascript
// test-exports.js
console.log('初始状态:');
console.log('exports === module.exports:', exports === module.exports);
console.log('module.exports:', module.exports);
console.log('exports:', exports);

// 使用 exports 添加属性
exports.name = 'test';
exports.getValue = function() { return 'value'; };

console.log('\n使用 exports 添加属性后:');
console.log('exports === module.exports:', exports === module.exports);
console.log('module.exports:', module.exports);
console.log('exports:', exports);

// 重新赋值 exports
exports = { newProp: 'new' };

console.log('\n重新赋值 exports 后:');
console.log('exports === module.exports:', exports === module.exports);
console.log('module.exports:', module.exports);
console.log('exports:', exports);

// 重新赋值 module.exports
module.exports = { finalProp: 'final' };

console.log('\n重新赋值 module.exports 后:');
console.log('exports === module.exports:', exports === module.exports);
console.log('module.exports:', module.exports);
console.log('exports:', exports);
```

运行结果：
```shell
初始状态:
exports === module.exports: true
module.exports: {}
exports: {}

使用 exports 添加属性后:
exports === module.exports: true
module.exports: { name: 'test', getValue: [Function] }
exports: { name: 'test', getValue: [Function] }

重新赋值 exports 后:
exports === module.exports: false
module.exports: { name: 'test', getValue: [Function] }
exports: { newProp: 'new' }

重新赋值 module.exports 后:
exports === module.exports: false
module.exports: { finalProp: 'final' }
exports: { newProp: 'new' }
```

##### 最佳实践

**1. 选择一种方式并保持一致**

```javascript
// 推荐：使用 module.exports
module.exports = {
    add: function(a, b) { return a + b; },
    multiply: function(a, b) { return a * b; }
};

// 或者：使用 exports
exports.add = function(a, b) { return a + b; };
exports.multiply = function(a, b) { return a * b; };
```

**2. 避免混合使用**

```javascript
//  避免这样做
exports.add = add;
module.exports.multiply = multiply;

//  选择一种方式
module.exports = { add, multiply };
// 或者
exports.add = add;
exports.multiply = multiply;
```

**3. 导出单个值时使用 module.exports**

```javascript
// 导出单个函数
module.exports = function(a, b) { return a + b; };

// 导出单个类
module.exports = class Calculator {
    add(a, b) { return a + b; }
};

// 导出单个值
module.exports = 'Hello World';
```

##### 与 ES Module 的对比

```javascript
// CommonJS
module.exports = { add, multiply };
const { add, multiply } = require('./math');

// ES Module
export { add, multiply };
import { add, multiply } from './math.js';

// CommonJS 默认导出
module.exports = add;
const add = require('./math');

// ES Module 默认导出
export default add;
import add from './math.js';
```

##### 总结

- `exports` 是 `module.exports` 的引用
- 重新赋值 `exports` 会断开引用关系
- 推荐统一使用 `module.exports` 或 `exports`，避免混合使用
- 导出单个值时使用 `module.exports`
- 理解这些概念有助于避免常见的模块导出错误

### 4. UMD（Universal Module Definition）

UMD试图兼容AMD和CommonJS，让同一份代码可以在不同环境下运行。

```javascript
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD环境
        define(['exports'], factory);
    } else if (typeof exports === 'object' && typeof exports.nodeName !== 'string') {
        // CommonJS环境
        factory(exports);
    } else {
        // 浏览器全局变量
        factory((root.MyModule = {}));
    }
}(typeof self !== 'undefined' ? self : this, function (exports) {
    // 模块代码
    exports.add = function(a, b) {
        return a + b;
    };
}));
```

### 5. ES Module（ES6 Modules）

ES Module是JavaScript的官方模块化标准，从ES6开始引入。

#### ES Module语法

```javascript
// 导出模块
export const name = "ES Module";
export function add(a, b) {
    return a + b;
}

// 默认导出
export default function multiply(a, b) {
    return a * b;
}

// 导入模块
import { name, add } from './module.js';
import multiply from './module.js';

// 或者
import * as math from './module.js';
```

#### 实际例子

```javascript
// math.js
export const PI = 3.14159;

export function add(a, b) {
    return a + b;
}

export function multiply(a, b) {
    return a * b;
}

export default function calculate(x, y) {
    return add(multiply(x, 2), y);
}

// calculator.js
import calculate, { add, multiply, PI } from './math.js';

export function advancedCalculate(x, y) {
    return calculate(x, y) * PI;
}

// main.js
import { advancedCalculate } from './calculator.js';
console.log(advancedCalculate(3, 4)); // 31.4159
```

#### ES Module的特点

**优点：**
- JavaScript原生支持
- 静态分析，编译时确定依赖关系
- 支持循环依赖
- 支持动态导入（import()）
- 更好的Tree Shaking支持

**缺点：**
- 需要现代浏览器支持
- 需要服务器环境或构建工具

#### 动态导入

```javascript
// 动态导入
async function loadModule() {
    const module = await import('./math.js');
    console.log(module.add(1, 2));
}

// 条件导入
if (condition) {
    import('./module1.js');
} else {
    import('./module2.js');
}
```

## 四、模块化规范对比

| 特性         | IIFE   | AMD        | CommonJS     | ES Module  |
| ------------ | ------ | ---------- | ------------ | ---------- |
| 语法复杂度   | 中等   | 高         | 低           | 低         |
| 浏览器支持   | 好     | 需要加载器 | 需要构建工具 | 现代浏览器 |
| 服务器支持   | 好     | 需要加载器 | 原生支持     | 原生支持   |
| 静态分析     | 否     | 否         | 否           | 是         |
| 循环依赖     | 不支持 | 支持       | 有限支持     | 支持       |
| Tree Shaking | 否     | 否         | 否           | 是         |

## 五、CommonJS 与 ES Module 互操作性问题

在现代 JavaScript 开发中，CommonJS 和 ES Module 的混用是一个常见问题，特别是在 Node.js 环境中。

### 1. 互操作性问题

#### 问题1：导入 CommonJS 模块到 ES Module

```javascript
// commonjs-module.js (CommonJS)
module.exports = {
    add: function(a, b) { return a + b; },
    multiply: function(a, b) { return a * b; }
};

// es-module.js (ES Module)
//  这样导入会有问题
import { add, multiply } from './commonjs-module.js'; // 错误！

//  正确的导入方式
import math from './commonjs-module.js';
console.log(math.add(1, 2)); // 3

// 或者使用 default import
import * as math from './commonjs-module.js';
console.log(math.default.add(1, 2)); // 3
```

#### 问题2：导入 ES Module 到 CommonJS

```javascript
// es-module.js (ES Module)
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;
export default function calculate(x, y) {
    return add(multiply(x, 2), y);
}

// commonjs-module.js (CommonJS)
// 不能直接使用 require
const { add, multiply } = require('./es-module.js'); // 错误！

// 需要使用动态 import
async function loadESModule() {
    const esModule = await import('./es-module.js');
    console.log(esModule.add(1, 2)); // 3
    console.log(esModule.default(3, 4)); // 10
}
```

### 2. Node.js 中的解决方案

#### 使用 .mjs 和 .cjs 扩展名

```javascript
// math.mjs (ES Module)
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// calculator.cjs (CommonJS)
const { add, multiply } = await import('./math.mjs');
module.exports = {
    calculate: (x, y) => add(multiply(x, 2), y)
};
```

#### 使用 package.json 配置

```json
{
  "name": "my-package",
  "type": "module",  // 整个包使用 ES Module
  "main": "index.js"
}
```

或者：

```json
{
  "name": "my-package",
  "type": "commonjs",  // 整个包使用 CommonJS
  "main": "index.js"
}
```

#### 混合使用配置

```json
{
  "name": "my-package",
  "main": "index.cjs",
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.cjs"
    }
  }
}
```

### 3. 构建工具中的处理

#### Webpack 配置

```javascript
// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            ['@babel/preset-env', {
                                modules: false // 保持 ES Module 语法
                            }]
                        ]
                    }
                }
            }
        ]
    }
};
```

#### Vite 配置

```javascript
// vite.config.js
export default {
    build: {
        rollupOptions: {
            external: ['fs', 'path'], // 排除 Node.js 内置模块
            output: {
                format: 'es' // 输出 ES Module 格式
            }
        }
    }
};
```

### 4. 实际开发中的最佳实践

#### 统一模块系统

```javascript
// 推荐：在项目中统一使用一种模块系统
// 如果使用 ES Module，所有文件都使用 .mjs 或配置 "type": "module"

// math.mjs
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// calculator.mjs
import { add, multiply } from './math.mjs';
export const calculate = (x, y) => add(multiply(x, 2), y);
```

#### 渐进式迁移

```javascript
// 步骤1：将 CommonJS 模块改为支持两种格式
// math.js
function add(a, b) { return a + b; }
function multiply(a, b) { return a * b; }

// 同时支持 CommonJS 和 ES Module
if (typeof module !== 'undefined' && module.exports) {
    // CommonJS 环境
    module.exports = { add, multiply };
} else {
    // ES Module 环境
    export { add, multiply };
}

// 步骤2：逐步迁移到纯 ES Module
// math.mjs
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;
```

### 5. 常见错误和解决方案

#### 错误1：循环依赖

```javascript
// a.js
const b = require('./b');
module.exports = { name: 'a', b };

// b.js
const a = require('./a'); // 循环依赖
module.exports = { name: 'b', a };

// 解决方案：延迟加载
// a.js
module.exports = { 
    name: 'a',
    getB: () => require('./b')
};
```

#### 错误2：动态 require

```javascript
// ❌ 错误：动态 require 在 ES Module 中不支持
function loadModule(name) {
    return require(`./modules/${name}`);
}

// ✅ 正确：使用动态 import
async function loadModule(name) {
    const module = await import(`./modules/${name}.js`);
    return module.default;
}
```

#### 错误3：__dirname 和 __filename 在 ES Module 中不可用

```javascript
// CommonJS
console.log(__dirname);  // /path/to/directory
console.log(__filename); // /path/to/file.js

// ES Module 中的替代方案
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

console.log(__dirname);  // /path/to/directory
console.log(__filename); // /path/to/file.js
```

## 六、现代前端工程化中的模块化

### 1. 构建工具的作用

现代前端开发中，构建工具（如Webpack、Vite、Rollup）扮演着重要角色：

```javascript
// webpack.config.js
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                use: 'babel-loader'
            }
        ]
    }
};
```

### 2. 模块解析策略

```javascript
// 相对路径
import './utils.js';
import '../components/Button.js';

// 绝对路径
import '/src/utils.js';

// 模块路径（node_modules）
import 'lodash';
import 'react';

// 别名路径
import '@/components/Button';
import '@utils/helpers';
```

### 3. 代码分割

```javascript
// 路由级别的代码分割
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));

// 组件级别的代码分割
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// 条件加载
const loadFeature = async () => {
    if (user.isPremium) {
        const { premiumFeature } = await import('./premium');
        return premiumFeature;
    }
};
```

## 七、最佳实践

### 1. 模块设计原则

```javascript
// 单一职责原则
// math.js - 只负责数学运算
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }

// string.js - 只负责字符串处理
export function capitalize(str) { 
    return str.charAt(0).toUpperCase() + str.slice(1); 
}

// 避免在一个模块中混合不同职责
// ❌ 不好的例子
export function add(a, b) { return a + b; }
export function formatDate(date) { return date.toISOString(); }
export function validateEmail(email) { return /@/.test(email); }
```

### 2. 依赖管理

```javascript
// 明确依赖关系
import { add, multiply } from './math.js';
import { formatDate } from './date.js';
import { validateEmail } from './validation.js';

// 避免深层嵌套
// 不好的例子
import { helper } from '../../../utils/helpers.js';

// 好的例子
import { helper } from '@/utils/helpers.js';
```

### 3. 导出策略

```javascript
// 命名导出 - 适合多个相关功能
export const API_URL = 'https://api.example.com';
export function fetchData() { /* ... */ }
export function postData() { /* ... */ }

// 默认导出 - 适合主要功能
export default class ApiClient {
    // ...
}

// 混合导出
export const API_URL = 'https://api.example.com';
export default class ApiClient {
    // ...
}
```

## 八、写在结尾

JavaScript模块化的发展历程反映了前端工程化的演进：

1. **IIFE时代**：解决了全局污染问题
2. **AMD时代**：为浏览器环境提供了异步模块加载
3. **CommonJS时代**：为Node.js提供了同步模块系统
4. **ES Module时代**：JavaScript原生模块化标准

现代前端开发中，ES Module已经成为主流，配合构建工具和现代浏览器，提供了完整的模块化解决方案。理解这些模块化规范的特点和适用场景，对于编写可维护、可扩展的前端代码至关重要。

## 参考资源

- [CommonJS规范](https://javascript.ruanyifeng.com/nodejs/module.html)
- [ES6 Module详解](https://es6.ruanyifeng.com/#docs/module)
- [Webpack模块化原理](https://webpack.js.org/concepts/modules/)
- [Vite模块化实践](https://vitejs.dev/guide/features.html#es-modules)