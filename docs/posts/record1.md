---
date: 2025-09-10
category:
  - AI
  - 微信小程序
tag:
  - AI
  - 微信小程序
highlight: a11y-dark
---

# 微信小程序AI开发

> 本文记录了第一次在微信小程序中集成AI助手功能的完整开发过程，包含了大量踩坑经验和解决方案。如果你有更好的经验，请告诉我～；如果你也在第一次做类似项目，希望这篇文章能帮你少走弯路。

## 一、开发过程中的血泪踩坑

### 1、原生微信小程序不支持EventSource API - 第一个大坑

**问题描述：**
刚开始开发时，我天真地以为可以直接使用浏览器原生的 `EventSource` API 来实现 Server-Sent Events (SSE) 连接。结果发现微信小程序环境完全不支持这个API，而且后续使用`for...await`解析流式数据也将成为泡沫，这让我一度怀疑人生（别人的小程序到底怎么做的？？？？）。

**踩坑过程：**

*   第一次尝试：直接写 `new EventSource(url)`，结果报错 `EventSource is not defined`
*   第二次尝试：查找各种polyfill库，发现大部分都不兼容小程序环境
*   第三次尝试：使用原生小程序搭建项目，并且使用`wx.request`，但是没有成熟的`markdown`渲染库，一开始使用的是小程序的插件叫`html2wxml`，只能将代码拷贝到项目中，但是体积太大又过不了关
*   最终发现：使用Mpx安装兼容版本的`markdown-it库`和 `wx.request` 配合 `enableChunked` 参数，但文档极其简陋

**技术难点详解：**

*   `运行环境限制`：微信小程序运行在阉割版的 JSCore 环境中，缺少大量浏览器原生API。你以为的"标准Web API"在这里都是奢望
*   `API设计理念冲突`：微信小程序的设计理念是"轻量级、安全可控"，所以只开放了有限的Web API。EventSource这种"高级"网络API根本不在考虑范围内
*   `数据解析噩梦`：使用 `wx.request` 的 `onChunkReceived` 接收到的数据是 ArrayBuffer 格式，需要手动转换为字符串，然后还要处理SSE特有的 `data:` 前缀格式
*   `错误处理复杂`：流式数据解析过程中，任何一步出错都可能导致整个连接中断，调试起来极其痛苦

**解决方案的曲折过程：**

```javascript
// 第一次尝试 - 直接失败
const eventSource = new EventSource(url); // ❌ 报错

// 第二次尝试 - 使用wx.request但解析失败
wx.request({
  enableChunked: true,
  onChunkReceived: (res) => {
    console.log(res.data); // 输出的是ArrayBuffer，完全看不懂，使用自带的textcoder也无济于事，不稳定
  }
});

// 最终方案 - 经过无数次调试才搞定的完整代码
fetchTask.onChunkReceived(async (res: { data: ArrayBuffer }) => {
  const u8a = new Uint8Array(res.data);
  const buffer2string = new TextDecoder('utf-8').decode(u8a);
  // 这里还要处理SSE的特殊格式...
});
```

### 2、第三方库引入限制 - 第二个大坑

**问题描述：**
习惯了现代前端开发的npm生态，结果发现微信小程序对第三方库的引入有极其严格的限制。很多你以为"理所当然"能用的库，在这里都是奢望。

**踩坑过程：**

*   第一次尝试：`npm install markdown-it`，结果发现无法直接使用
*   第二次尝试：在微信开发者工具中开启npm支持，但构建失败
*   第三次尝试：手动复制库源码到项目中，但发现依赖了浏览器API
*   最终解决：找到兼容的库版本，并手动处理依赖关系

**技术难点详解：**

*   `构建环境限制`：小程序不支持标准的npm包管理，需要特殊的构建流程
*   `API兼容性问题`：大部分npm包都依赖浏览器API，在小程序环境中无法运行
*   `包大小限制`：小程序有严格的包大小限制，大型库无法直接使用
*   `版本兼容性`：即使找到了兼容的库，版本更新也可能破坏兼容性

**解决方案的探索过程：**

```javascript
// 第一次尝试 - 直接引入失败
import MarkdownIt from 'markdown-it'; // ❌ 报错

// 第二次尝试 - 手动复制源码
// 把整个markdown-it库的源码复制到项目中
// 但发现依赖了document、window等浏览器API

// 最终方案 - 使用Mpx吧
```

### 3、数据流解析的噩梦 - 第三个大坑

**问题描述：**
即使搞定了网络请求，数据解析又是一个巨大的挑战。SSE协议的数据格式、编码问题、错误处理，每一个细节都可能让你调试一整天。

**踩坑过程：**

*   第一次尝试：直接使用 `TextDecoder`，结果发现编码问题
*   第二次尝试：使用第三方编码库，但发现兼容性问题
*   第三次尝试：手动处理编码，但发现数据不完整
*   最终解决：找到稳定的编码方案，并完善错误处理

**技术难点详解：**

*   `编码问题`：微信小程序的 `TextDecoder` 实现不稳定，需要寻找替代方案
*   `数据完整性`：流式数据可能被分割，需要正确处理数据拼接
*   `错误恢复`：网络中断、数据格式错误等情况需要优雅处理
*   `性能优化`：大量数据流处理需要考虑内存和性能问题

## 二、技术方案设计 - 从绝望到希望

在经历了上述种种困难后，我决定重新思考技术方案。原本想用原生小程序开发，但发现限制太多。最终选择了 [Mpx](https://mpxjs.cn/guide/basic/start.html) 这个跨平台框架。

**为什么选择Mpx：**

*   支持更复杂的逻辑处理和数据绑定
*   提供更完善的组件化方案
*   更好的开发体验，支持热重载和TypeScript
*   更容易集成第三方库（虽然还是有限制）

**但即使使用Mpx，仍然需要面对小程序的各种限制：**

*   网络请求仍然需要使用 `wx.request`
*   第三方库仍然需要特殊处理，选择小程序兼容版本
*   调试工具仍然不够完善

## 三、最终的技术实现

### 1、核心代码实现

**SSE连接处理（经过无数次调试的最终版本）：**

```javascript
import { TextDecoder } from 'text-encoding-shim'

let response = ''
const currentAPI = {
  url: 'https://api.openai.com/v1/chat/completions', // 示例API
  token: 'sk-xxxxxxxxxx' // 请替换为你的API Key
}

const paramServer = {
   model: currentAPI.model,
   messages: [{ role: 'user', content: 'hello' }],
   stream: true,
   temperature: 0.7
}

// 这个实现经过了无数次调试和优化
const fetchTask = wx.request({
  url: currentAPI.url,
  method: 'POST',
  header: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${currentAPI.token}`
  },
  data: JSON.stringify(paramServer),
  enableChunked: true,  // 关键参数，启用分块传输
  responseType: 'text',
  complete: () => {
    console.log('请求完成')
  },
  fail: (error) => {
    console.error('请求失败:', error)
  }
})

// 数据解析的核心逻辑 - 这里花了我最多时间
fetchTask.onChunkReceived(async (res: { data: ArrayBuffer }) => {
  // 如下图是res的格式
  console.log('res>>>>>>>>>>', res)
  // 1. 将ArrayBuffer转换为Uint8Array
  const u8a = new Uint8Array(res.data)
  
  // 2. 使用稳定的TextDecoder进行解码
  // 注意：这里使用第三方库而不是原生的TextDecoder
  const buffer2string = new TextDecoder('utf-8').decode(u8a)
  
  // 3. 处理SSE特有的data:前缀格式
  const currentChunkArr = buffer2string.split('data: ').filter((chunk: string) => chunk)
  
  // 4. 逐个处理数据块
  for (const chunk of currentChunkArr) {
    if (chunk.includes('[DONE]') || !chunk) {
      break
    }
    
    try {
      // 5. 解析JSON数据
      const dealChunk = {
        choices: [{
          delta: {
            content: ''
          }
        }]
      }
      const chunkToobj = JSON.parse(chunk) || dealChunk
      
      // 6. 提取内容并拼接
      if (chunkToobj.choices) {
        response += chunkToobj.choices[0].delta.content || ''
        // 这里可以触发UI更新
      }
    } catch (err) {
      console.error('数据解析错误:', err)
      // 错误处理很重要，避免整个连接中断
    }
  }
})
```

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/a33cbb7890a446d5aee05df6b3296069~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2l56KX55uQ54SX5pif55CD:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMzMxNjU1MTk4ODU1MzcwOSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1758091924&x-orig-sign=e%2BBn9g0UOrnK0y98rlczwHftTRk%3D)

### 2、rich-text渲染

```html
<!-- htmlContent=md.render(response) -->
<template>
  <view class="resp-panel">
    <rich-text
        class="resp-panel_content"
        nodes="{{ htmlContent }}"
        user-select="true"
    ></rich-text>
  </view>
</template>
```

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/a7493ad77c08428e84d24a31c0342ac8~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5p2l56KX55uQ54SX5pif55CD:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMzMxNjU1MTk4ODU1MzcwOSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1758091924&x-orig-sign=JA1DFIF2sdjeoRxChmt%2FSRxJhAg%3D)

TODO：感觉`Markdown`的样式调整（特别是行间距）也还是一个问题😑，放在`rich-text`的nodes里。。。

## 四、开发过程中的其他坑

### 1、调试工具的限制

*   微信开发者工具的调试功能有限，只能打印看到数据结构
*   网络请求的调试信息不够详细
*   错误堆栈信息经常不准确

### 2、性能优化问题

*   大量文本渲染可能导致页面卡顿
*   内存使用需要严格控制
*   网络请求的并发限制

### 3、用户体验问题

*   流式数据的显示效果需要优化
*   错误状态的用户提示
*   加载状态的展示

## 五、总结与建议

### 开发建议：

1.  **充分了解平台限制**：在开始开发前，一定要详细了解微信小程序的API限制
2.  **选择合适的框架**：考虑使用Mpx等跨平台框架，但不要期望完全解决所有问题
3.  **做好技术调研**：对于关键功能，一定要提前验证可行性
4.  **重视错误处理**：在小程序环境中，错误处理比在Web环境中更重要
5.  **性能优化**：注意内存使用和渲染性能

### 技术选型建议：

*   网络请求：使用 `wx.request` + `enableChunked`
*   数据解析：使用稳定的编码库，如 `text-encoding-shim`
*   框架选择：考虑Mpx等跨平台框架
*   第三方库：优先选择轻量级、无浏览器依赖的库

### 踩坑经验：

1.  不要假设Web API在小程序中可用
2.  第三方库的兼容性需要充分测试
3.  流式数据处理需要特别注意编码和解析
4.  错误处理要更加完善
5.  性能优化要从小处做起

这个项目虽然最终完成了，但开发过程中的困难和挑战远超预期。希望这篇文章能帮助其他开发者少走弯路，也希望微信小程序平台能够提供更好的开发体验。好了，调style和交互去了～。
