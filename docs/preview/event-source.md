---
title: EventSource
tags:
  - eventsource
createTime: 2025/11/06 17:50:50
permalink: /article/fqt7t18c/
---

## 一、背景知识
```javascript
const onSubmit = () => {
  const params = new URLSearchParams({xxx})
  const eventSource = new EventSource(`xxxx?${params.toString()}`)
  // sse请求成功回调
  eventSource.onmessage = (event) => {
    try {
      const response = JSON.parse(event.data)
      if (response.data.status === 'done') { // 判断sse是否结束
         eventSource.close()
      }
      // 数据解析
    } catch (err) {
      // 错误处理
    }
  }
  // sse请求失败回调
  eventSource.onerror = (error) => {
    // eventSource.close()
  }
}
```
如上是简易的基本的`EventSource`使用方式，当开始创建`sse`之后，正常情况最终会走到`response.data.status === 'done'`，表示`sse`流正常输出并且结束，结束之后需要手动关闭。

当sse长时间未向前端输出内容的时候，`sse`会触发自身的重试机制，但是在debug过程只能看到一次sse的调用请求，这样就很难跟踪sse的状态了，这会导致前端的`sse`实际上已经重新请求多次（但是真正手动发起的请求只有第一次是），但是实际上可能存在后端在接收到的第一次`sse`请求目前还在继续输出内容，只不过前端没有接收到，但是现在有收到前端的第二次、第三次等等的sse请求。这样的话，前端已经做完第一次sse请求的处理，并不知道sse内部重试机制，导致前端继续接收第二次，第三次等等`sse`请求流，最终显示的内容是多次请求拼接的流内容。

这么说些许难以理解了，简单来说就是：当调用`onSubmit`请求时，创建`SSE`，前端在接收`SSE`流的过程中，由于服务端请求超时或其他问题，导致服务端并没有吐流内容或其他任何响应内容给前端，导致`SSE`超时，会触发`SSE`自身的重试机制。但是`SSE`自身的重试机制前端无法感知，无法跟踪，而前端在接收到的流内容是拼接了重试机制返回的内容，导致内容比实际冗长、重复、错乱、乱码等问题。这就需要摒弃SSE自身的重试机制。

**SSE的默认重连行为**：
（1）当 SSE 连接意外中断（如网络超时、服务器无响应），浏览器会自动尝试重连
（2）默认重连间隔为 3秒左右，并可能逐渐增加（指数退避），直到连接成功或达到上限
难道不能修改SSE本身重试的时间吗？比如10分钟之后让它重试？
EventSource的规范中没有提供设置超时或重连间隔的API。因此这种想法无法直接实现。EventSource的功能不像XMLHttpRequest那样设置超时。

## 二、超时重试机制

要摒弃`SSE`自身的超时重试机制，就需要在当`SSE`走`onerror`回调时，手动关闭`SSE`，重新调用`onSubmit`函数，当然为避免数据内容错误，还需要考虑到原本接收到的内容。但是这样导致服务端还在输出信息，而客户端已经关闭连接，导致信息接收不全，所以会想着加上定时器延长关闭时间是否可行。答案是不行的，这是有很严重的缺陷：

假设在 `onerror` 回调中设置一个 3分钟后关闭连接 的逻辑（`setTimeout(..., 3*60*1000)`），但在此期间，`EventSource `已经触发了多次自动重连，服务器超时后未正确关闭连接，导致浏览器持续尝试恢复。

`EventSource`的自动重连机制、代码逻辑中的多次调用，或者服务器响应都可能触发重连。

`EventSource`在连接断开时会自动尝试重新连接，默认的重试时间可能由浏览器控制，或者服务器通过`Retry-After`头指定。如果服务器没有正确关闭连接，或者在错误处理中没有正确关闭`eventSource`，可能会导致多次重试。

### 1、心跳检测

当服务端长时间没有内容响应给前端时，可以手动控制每间隔2～3s给前端吐空字符串，前端加上等待的过程，这样可以增加体验。但这个不是长久之计，真遇到超时太久了，也是治标不治本。

### 2、数据拉齐

要实现超时重试，需要服务端配合，不然直接`onerror`就关闭`SSE`，再重新调用`onSubmit`，无异于中断请求，重新尝试。
因为`SSE`返回的数据服务端是可以控制的，每一条流消息都可以自定义一些辅助字段，比如给每一条流消息增加一个序号seq。在重试过程，可以给后端传递seq告诉服务端，上次sse是在哪断的，后端再根据seq去返回响应的内容，而不是全部返回（或者不是全部重新返回一遍，可以把seq之前的内容一次性返回，这样就不用等待从0～seq这些已经接收过的流内容了）。
另外重试可以限制次数，总不能一直重试，再安排一个每次重试呈指数退避。
总结：
显式关闭EventSource：在onerror或onmessage中检测到错误或完成时，立即关闭eventSource，而不是等待超时。不要试图在onerror中加一个x分钟的延迟关闭，这可能导致在此期间浏览器多次尝试重连。
禁用自动重连：虽然EventSource没有直接提供禁用重连的选项，但可以通过在错误处理中关闭eventSource，并在需要时手动重新连接。
检查服务器端的关闭逻辑：确保服务器在完成响应或超时后正确关闭SSE流，发送适当的事件或HTTP状态码，以通知客户端不要重连。同时，服务端在请求策略接口过程，可以每间隔2～3s给客户端返回""
添加重连次数限制：在客户端代码中添加计数器，限制重连次数，避免无限重试。

## 三、断线重连

断线重连：聚焦 “断开的网络连接”，核心是在网络中断、连接超时后，重新建立客户端与服务端的通信链路，是实现续传的前提。
断线重连场景：
网络突然中断（如 Wi-Fi 断开、4G/5G 切换），此时客户端与服务端的 SSE 连接直接断开。
应用后台运行过久，系统回收网络资源导致连接失效。
服务端短暂宕机后恢复，客户端需要重新建立连接。
客户端检测到连接断开（如心跳超时、网络错误），发起重新连接请求，直到连接成功或达到重试上限。

## 四、断线续传机制
断线续传：聚焦 “未完成的业务内容”，核心是让中断的数据流（如 SSE 未输出完的文本）继续往下输出，不遗漏、不重复。

断线续传的场景：
（1）当SSE正在输出，网络切断、退出应用到后台，后台清除等，再次进入时，需要继续上次未输出完毕的流内容
（2）当SSE正在输出（比如ChatGPT），切换对话，来回切换，比如对话A正在输出，现在切换到对话B，如果对话B也有上次未输出完的sse，那么就连上对话B的sse，让对话B继续输出；如果对话B没有上次未输出完毕的SSE，当切换回对话A，此时流内容应该要输出完毕或未输出完毕正在继续输出，而不是被中断
关于对话最后一条内容是否未输出完毕，需要服务端配合加一个标识，比如status: 'avaliable' 、'pending'等，根据status是否为pending决定是否要重试连接SSE进行输出
连接重建后，客户端带着 “上次接收的最后一个数据位置”“对话 ID”“服务端 pending 标识”，向服务端请求 “继续输出剩余内容”，服务端从断点处继续推送数据。
断线重连关注的是status='pending'来判断是否需要断线重连，如果不需要也不会继续出发点断线续传，如果需要，肯定需要出发断线续传。手动中断停止服务端输出，这个需要告诉服务端，你终止了，所以服务端也应该终止。除了这种情况，其余服务端都不会终止（除非服务端输出完毕或手动终止，但是sse是单向通信，不会影响到服务端）
断线续传关注的是上次输出的seq序号，以便传给服务端拿内容。

## 五、代码

### 1、readme.md

::: tip 基本使用

#### SSE 封装使用指南

##### 概述

本项目提供了两个SSE封装类：
- `SSEManager`: 基础SSE连接管理器
- `ChatSSEManager`: 专门用于聊天功能的SSE管理器

###### 文件结构

```
src/classes/eventsource/
├── sseManager.js          # 基础SSE管理器
├── chatSSEManager.js      # 聊天SSE管理器
└── README.md             # 使用说明
```

#### 基础SSE管理器 (SSEManager)

##### 功能特性

- 自动重连机制（指数退避）
- 事件监听器管理
- 连接状态管理
- 错误处理

##### 基本使用

```javascript
import SSEManager from '@/classes/eventsource/sseManager';

const sseManager = new SSEManager();

// 连接SSE
await sseManager.connect('https://api.example.com/sse', {
  headers: {
    'Authorization': 'Bearer token'
  },
  onMessage: (data, event) => {
    console.log('收到消息:', data);
  },
  onError: (error, retryCount) => {
    console.error('连接错误:', error);
  },
  onOpen: (event) => {
    console.log('连接已建立');
  },
  onClose: (event) => {
    console.log('连接已关闭');
  }
});

// 关闭连接
sseManager.close();

// 检查连接状态
if (sseManager.isConnected()) {
  console.log('连接正常');
}
```

#### 聊天SSE管理器 (ChatSSEManager)

##### 功能特性

- 继承基础SSE管理器
- 专门处理聊天流数据
- 自动处理消息状态更新
- 支持思考状态管理
- 提供回调函数接口

##### 基本使用

```javascript
import ChatSSEManager from '@/classes/eventsource/chatSSEManager';

const chatSSE = new ChatSSEManager();

// 开始聊天流
await chatSSE.startChatStream(requestParams, messageIndex, {
  onStreamUpdate: (data) => {
    // 处理流更新
    const { messageIndex, streamContent, status, thinkEnd, hasStreamContent } = data;
    console.log('流更新:', streamContent);
  },
  onFinish: (meta) => {
    // 处理完成
    console.log('流完成:', meta);
  },
  onError: (error) => {
    // 处理错误
    console.error('流错误:', error);
  },
  onScroll: (data) => {
    // 处理滚动
    const { mode, isFinish } = data;
    if (isFinish) {
      // 滚动到底部
    }
  }
});
```

#### 在聊天组件中使用

##### 1. 替换原有的 fetchReplyMsg 方法

```javascript
// 原来的方法
async fetchReplyMsg(requestParams, index, retryCount = 0) {
  // 大量的SSE处理逻辑...
}

// 替换为
async fetchReplyMsg(requestParams, index, retryCount = 0) {
  try {
    // 设置SSE管理器的消息数据
    const finalIndex = index || this.currentImMessageList.length - 1;
    const userMessage = this.currentImMessageList[finalIndex - 1];
    const assistantMessage = this.currentImMessageList[finalIndex];
    
    this.chatSSE.setMessageData(userMessage, assistantMessage);
    
    // 开始SSE流
    await this.chatSSE.startChatStream(requestParams, finalIndex, {
      onStreamUpdate: this.handleStreamUpdate.bind(this),
      onFinish: this.handleStreamFinish.bind(this),
      onError: this.handleStreamError.bind(this),
      onScroll: this.handleStreamScroll.bind(this),
    });
  } catch (error) {
    console.error('SSE连接失败:', error);
    this.handleStreamError('连接失败');
  }
}
```

##### 2. 实现回调方法

```javascript
// 处理流更新
handleStreamUpdate(data) {
  const { messageIndex, streamContent, status, thinkEnd, hasStreamContent } = data;
  
  // 更新状态
  this.thinkEnd = thinkEnd;
  this.hasStreamContent = hasStreamContent;
  
  // 更新消息列表
  this.updateMessageList(messageIndex, streamContent, status);
  
  // 处理滚动
  this.handleScrollLogic();
}

// 处理流完成
handleStreamFinish(meta) {
  const lastIndex = this.currentImMessageList.length - 1;
  
  // 更新最终状态
  setTimeout(() => {
    this.currentImMessageList[lastIndex].meta = {
      ...(this.currentImMessageList?.[lastIndex]?.meta || {}),
      status: meta?.status || AgentStatus.AVAILABLE,
    };
    this.setChatMessage(cloneDeep(this.currentImMessageList));
  });
  
  // 清理状态
  this.cleanupStreamState();
  
  // 更新历史会话
  this.fetchSessionList();
}

// 处理流错误
handleStreamError(error) {
  console.error('SSE流错误:', error);
  
  // 显示错误提示
  this.showToast({
    title: error || '网络异常',
  });
  
  // 更新消息状态为失败
  this.updateMessageStatusToFail();
  
  // 清理状态
  this.cleanupStreamState();
}

// 处理流滚动
handleStreamScroll(data) {
  const { mode, isFinish } = data;
  
  if (isFinish) {
    this.handleFinishScroll();
  }
}
```

##### 3. 在组件中初始化

```javascript
import ChatSSEManager from '@/utils/chatSSEManager';

export default {
  data() {
    return {
      chatSSE: null,
      // 其他数据...
    };
  },
  
  mounted() {
    // 初始化SSE管理器
    this.chatSSE = new ChatSSEManager();
  },
  
  destroy() {
    // 清理SSE连接
    if (this.chatSSE) {
      this.chatSSE.close();
    }
  }
};
```

#### 优势

##### 1. 代码复用
- 将SSE逻辑从组件中抽离
- 可以在多个组件中复用

##### 2. 更好的维护性
- 逻辑清晰，职责分离
- 易于测试和调试

##### 3. 更强的扩展性
- 可以轻松添加新功能
- 支持不同的SSE场景

##### 4. 更好的错误处理
- 统一的错误处理机制
- 自动重连功能

#### 注意事项

1. **状态管理**: 确保在组件销毁时正确清理SSE连接
2. **错误处理**: 实现适当的错误处理逻辑
3. **性能优化**: 避免在回调中执行过重的操作
4. **内存管理**: 及时清理不需要的引用


:::

### 2、sseManager.js

```javascript
import SSEManager from './sseManager';
import { AgentMode, AgentStatus, ThinkType } from '@/common/enum/index';
import { cloneDeep } from 'lodash-es';
import { trackEvent, trackError } from '@/common/track';
import { getPublick } from '@/common/publicParams';
import { serialize } from '@/common/sig';
import { IM_BASE_URL } from '@/api/urls';
import {
  LOAD_FAIL_CONTENT,
  SSE_ESTABLISH_CONTENT,
  SSE_DISCONNECT_CONTENT,
} from '@/common/constant';

/**
 * 聊天SSE管理器
 * @extends SSEManager
 * @property {number} currentMessageIndex - 当前消息索引
 * @property {Object} userMessage - 用户消息
 * @property {Object} assistantMessage - 助手消息
 * @property {Array} chatStream - 聊天流
 * @property {number} offsetId - 流消息ID
 * @property {boolean} thinkEnd - 思考结束
 * @property {boolean} hasStreamContent - 是否有流内容
 * @property {Object} messageCallbacks - 消息回调
 * @property {Object} sseOptions - 上次SSE连接选项
 * @function startChatStream - 开始聊天流
 * @function handleChatMessage - 处理聊天消息
 * @function handleStreamContent - 处理流内容
 * @function handleError - 处理错误
 * @function handleFinish - 处理完成
 * @function resetChatState - 重置聊天状态
 * @function setMessageData - 设置消息数据
 * @function getCurrentStream - 获取当前流内容
 * @function getCurrentOffset - 获取当前流式消息ID
 * @function getSseUrl - 获取SSE URL
 * @function isThinking - 检查是否正在思考
 * @function hasContent - 检查是否有流内容
 * @function handleChatError - 处理聊天错误
 * @function handleChatOpen - 处理聊天连接打开
 * @function handleChatClose - 处理聊天连接关闭
 */
const _instance = new WeakMap();
class ChatSSEManager extends SSEManager {
  constructor() {
    if (_instance.has(ChatSSEManager)) {
      throw new Error('单例已存在，使用 getInstance() 方法获取');
    }
    super();
    this.currentMessageIndex = -1;
    this.userMessage = null;
    this.assistantMessage = null;
    this.chatStream = [];
    this.offsetId = 0;
    this.thinkEnd = false;
    this.hasStreamContent = false;
    this.messageCallbacks = {
      onStreamUpdate: null,
      onFinish: null,
      onError: null,
      onScroll: null,
    };
    this.url = '';
  }
  static getInstance() {
    if (!_instance.has(ChatSSEManager)) {
      _instance.set(ChatSSEManager, new ChatSSEManager());
    }
    return _instance.get(ChatSSEManager);
  }

  /**
   * 开始聊天流
   * @param {Object} SSE_PASH - 地址
   * @param {Object} requestParams - 请求参数
   * @param {number} messageIndex - 消息索引
   * @param {Object} sseHeaders - 请求头
   * @param {Object} callbacks - 回调函数
   * @param {Function} callbacks.onStreamUpdate - 流更新回调
   * @param {Function} callbacks.onFinish - 完成回调
   * @param {Function} callbacks.onError - 错误回调
   * @param {Function} callbacks.onScroll - 滚动回调
   */
  async startChatStream(SSE_PASH, requestParams, messageIndex, sseHeaders, callbacks = {}) {
    // 重置状态
    this.resetChatState();

    // 设置回调
    this.messageCallbacks = { ...this.messageCallbacks, ...callbacks };

    // 设置消息索引
    this.currentMessageIndex = messageIndex;
    this.url = SSE_PASH;
    this.requestParams = requestParams;

    // 构建SSE URL
    const publicParams = getPublick(requestParams, SSE_PASH);
    const sseUrl = `${IM_BASE_URL}${SSE_PASH}?${serialize(publicParams)}`;
    const sseOptions = {
      headers: {
        'didi-header-hint-content': JSON.stringify(sseHeaders || {}),
      },
      onMessage: this.handleChatMessage.bind(this),
      onError: this.handleChatError.bind(this),
      onOpen: this.handleChatOpen.bind(this),
      onClose: this.handleChatClose.bind(this),
    };
    this.sseOptions = sseOptions;

    // 连接SSE
    try {
      await this.connect(sseUrl, sseOptions, false);
    } catch (error) {
      console.log('SSE连接失败:', error);
      const errorMessage = error?.message || String(error) || error?.toString();
      trackError('ChatSSEManager', {
        source: 'startChatStream',
        error: `===SSE连接失败===${errorMessage}`,
      });
      //这里再throw，谁来承接 需要throw吗
      throw new Error(error);
    }
  }

  /**
   * 处理聊天消息
   * @param {Object} response - 响应数据
   * @param {Event} event - 原始事件
   */
  handleChatMessage(response, event) {
    try {
      const { mode, message_id, finish_reason, data, meta } = response;
      const safeData = !Array.isArray(data) ? [] : data.map((dataItem) => ({ ...dataItem, mode }));
      // console.log('mode===', mode, 'safeData===', safeData);
      let replyStatus = AgentStatus.GENERATING;

      // 更新思考状态
      this.updateThinkState(safeData);

      // 根据mode处理数据
      switch (mode) {
        case AgentMode.APPEND:
        case AgentMode.UPDATE:
        case AgentMode.OVERWRITE:
          this.handleStreamContent(safeData, mode);
          break;
        case AgentMode.ERROR:
          this.handleError(finish_reason);
          replyStatus = AgentStatus.FINISH;
          break;
        case AgentMode.FINISH:
          this.handleFinish(meta);
          replyStatus = AgentStatus.FINISH;
          break;
      }

      // 更新消息状态
      this.updateMessageStatus(message_id, mode, finish_reason, replyStatus, meta);

      const { onStreamUpdate, onScroll } = this.messageCallbacks;
      // 触发流更新回调
      if (onStreamUpdate && typeof onStreamUpdate === 'function') {
        onStreamUpdate({
          message_id,
          messageIndex: this.currentMessageIndex,
          streamContent: cloneDeep(this.chatStream),
          status: replyStatus,
          thinkEnd: this.thinkEnd,
          hasStreamContent: this.hasStreamContent,
        });
      }

      // 处理滚动
      if (onScroll && typeof onScroll === 'function') {
        onScroll({
          isFinish: mode === AgentMode.ERROR || mode === AgentMode.FINISH,
        });
      }

      // 处理完成状态
      if (mode === AgentMode.FINISH || mode === AgentMode.ERROR) {
        this.handleFinalStatus(meta);
      }
    } catch (error) {
      console.log('处理聊天消息错误:', error);
      const errorMessage = error?.message || String(error) || error?.toString();
      trackError('ChatSSEManager', {
        source: 'handleChatMessage',
        error: `===处理聊天消息错误===${errorMessage}`,
      });
    }
  }

  /**
   * 处理流内容
   * @param {Array} eventData - 流内容数据
   * @param {string} mode - 模式
   */
  handleStreamContent(eventData, mode) {
    if (mode === AgentMode.UPDATE) {
      trackEvent('', { updateContent: JSON.stringify(eventData) }, '==更新数据==');
    }
    const hasValidData = eventData?.length;
    if (hasValidData) {
      const lastData = eventData[eventData.length - 1];
      const { id } = lastData || {};
      if (id) {
        this.offsetId = id;
      }
    }
    const preChatStream = cloneDeep(this.chatStream);
    this.chatStream = [...preChatStream, ...eventData];
  }

  /**
   * 处理错误
   * @param {string} finishReason - 完成原因
   */
  handleError(finishReason) {
    const { onError } = this.messageCallbacks;
    if (onError && typeof onError === 'function') {
      onError(finishReason || LOAD_FAIL_CONTENT);
    }
  }

  /**
   * 处理完成
   * @param {Object} meta - meta信息
   */
  handleFinish(meta) {
    this.userMessage = null;
    this.assistantMessage = null;
    const { onFinish } = this.messageCallbacks;
    if (onFinish && typeof onFinish === 'function') {
      onFinish(meta);
    }
  }

  /**
   * 更新思考状态
   * @param {string} type - 内容类型
   * @param {string} content - 内容
   */
  updateThinkState(eventData) {
    const isNonThinkType = eventData.some((item) => item.type != ThinkType.THINK);
    if (!this.thinkEnd && isNonThinkType) {
      // console.log('===updateThinkState===', this.thinkEnd, eventData);
      this.thinkEnd = true;
    }
    const isThinkType = eventData.some((item) => item.type == ThinkType.THINK);
    if (!this.hasStreamContent && isThinkType) {
      this.hasStreamContent = true;
    }
  }

  /**
   * 更新消息状态
   * @param {string} messageId - 消息ID
   * @param {string} mode - 模式
   * @param {string} finishReason - 完成原因
   * @param {string} replyStatus - 回复状态
   * @param {Object} meta - meta信息
   */
  updateMessageStatus(messageId, mode, finishReason, replyStatus, meta) {
    // TODO: 可以根据需要更新消息状态实现消息管理逻辑
  }

  /**
   * 处理最终状态
   * @param {Object} meta - meta信息
   */
  handleFinalStatus(meta) {
    setTimeout(() => {
      const { onFinish } = this.messageCallbacks;
      if (onFinish && typeof onFinish === 'function') {
        onFinish({
          ...meta,
          status: meta?.status || AgentStatus.AVAILABLE,
        });
      }
    });
  }

  /**
   * 处理聊天错误
   * @param {Error} error - 错误对象
   */
  //todo: yuhua 这块代码 check下 看着有点怪怪的
  async handleChatError(error) {
    const currentRetryCount = this.getRetryCount();
    const { onError } = this.messageCallbacks;
    // console.log('SSE错误:', this.offsetId, currentRetryCount);

    if (currentRetryCount < this.maxRetry) {
      try {
        console.log(`尝试第${currentRetryCount + 1}次重连...`);
        const sseUrl = this.getSseUrl(this.offsetId);
        await this.retry(sseUrl, this.sseOptions);
      } catch (retryError) {
        console.log('重试失败:', retryError);
        if (onError && typeof onError === 'function') {
          onError(retryError || new Error(LOAD_FAIL_CONTENT));
        }
      }
    } else {
      console.log('重试次数已达上限:', currentRetryCount);
      if (onError && typeof onError === 'function') {
        onError(error || new Error(LOAD_FAIL_CONTENT));
      }
    }
  }

  /**
   * 处理聊天连接打开
   * @param {Event} event - 事件对象
   */
  handleChatOpen(event) {
    console.log(SSE_ESTABLISH_CONTENT);
  }

  /**
   * 处理聊天连接关闭
   * @param {Event} event - 事件对象
   */
  handleChatClose(event) {
    console.log(SSE_DISCONNECT_CONTENT);
  }

  /**
   * 重置聊天状态
   */
  resetChatState() {
    this.currentMessageIndex = -1;
    this.chatStream = [];
    this.offsetId = 0;
    this.thinkEnd = false;
    this.hasStreamContent = false;
  }

  /**
   * 设置消息数据
   * @param {Object} userMessage - 用户消息
   * @param {Object} assistantMessage - 助手消息
   */
  setMessageData(userMessage, assistantMessage) {
    this.userMessage = userMessage;
    this.assistantMessage = assistantMessage;
    this.chatStream = assistantMessage.contents ? cloneDeep(assistantMessage.contents) : [];
  }

  /**
   * 获取当前流内容
   * @returns {Array}
   */
  getCurrentStream() {
    return cloneDeep(this.chatStream);
  }

  /**
   * 获取当前流式消息ID
   * @returns {number}
   */
  getCurrentOffset() {
    return this.offsetId;
  }

  /**
   * 获取SSE URL
   * @param {number} offset - 上次中断的流式消息ID
   * @returns {string}
   */
  getSseUrl(offset) {
    const { chat_id } = this.assistantMessage?.meta || {};
    const paramsServer = {
      ...this.requestParams,
      chat_id,
      offset,
    };
    const publicParams = getPublick(paramsServer, this.url);
    const sseUrl = `${IM_BASE_URL}${this.url}?${serialize(publicParams)}`;
    return sseUrl;
  }

  /**
   * 检查是否正在思考
   * @returns {boolean}
   */
  isThinking() {
    return !this.thinkEnd;
  }

  /**
   * 检查是否有流内容
   * @returns {boolean}
   */
  hasContent() {
    return this.hasStreamContent;
  }
}

export default ChatSSEManager;
```

### 3、chatSSEManager.js

```javascript
import SSEManager from './sseManager';
import { AgentMode, AgentStatus, ThinkType } from '@/common/enum/index';
import { cloneDeep } from 'lodash-es';
import { trackEvent, trackError } from '@/common/track';
import { getPublick } from '@/common/publicParams';
import { serialize } from '@/common/sig';
import { IM_BASE_URL } from '@/api/urls';
import {
  LOAD_FAIL_CONTENT,
  SSE_ESTABLISH_CONTENT,
  SSE_DISCONNECT_CONTENT,
} from '@/common/constant';

/**
 * 聊天SSE管理器
 * @extends SSEManager
 * @property {number} currentMessageIndex - 当前消息索引
 * @property {Object} userMessage - 用户消息
 * @property {Object} assistantMessage - 助手消息
 * @property {Array} chatStream - 聊天流
 * @property {number} offsetId - 流消息ID
 * @property {boolean} thinkEnd - 思考结束
 * @property {boolean} hasStreamContent - 是否有流内容
 * @property {Object} messageCallbacks - 消息回调
 * @property {Object} sseOptions - 上次SSE连接选项
 * @function startChatStream - 开始聊天流
 * @function handleChatMessage - 处理聊天消息
 * @function handleStreamContent - 处理流内容
 * @function handleError - 处理错误
 * @function handleFinish - 处理完成
 * @function resetChatState - 重置聊天状态
 * @function setMessageData - 设置消息数据
 * @function getCurrentStream - 获取当前流内容
 * @function getCurrentOffset - 获取当前流式消息ID
 * @function getSseUrl - 获取SSE URL
 * @function isThinking - 检查是否正在思考
 * @function hasContent - 检查是否有流内容
 * @function handleChatError - 处理聊天错误
 * @function handleChatOpen - 处理聊天连接打开
 * @function handleChatClose - 处理聊天连接关闭
 */
const _instance = new WeakMap();
class ChatSSEManager extends SSEManager {
  constructor() {
    if (_instance.has(ChatSSEManager)) {
      throw new Error('单例已存在，使用 getInstance() 方法获取');
    }
    super();
    this.currentMessageIndex = -1;
    this.userMessage = null;
    this.assistantMessage = null;
    this.chatStream = [];
    this.offsetId = 0;
    this.thinkEnd = false;
    this.hasStreamContent = false;
    this.messageCallbacks = {
      onStreamUpdate: null,
      onFinish: null,
      onError: null,
      onScroll: null,
    };
    this.url = '';
  }
  static getInstance() {
    if (!_instance.has(ChatSSEManager)) {
      _instance.set(ChatSSEManager, new ChatSSEManager());
    }
    return _instance.get(ChatSSEManager);
  }

  /**
   * 开始聊天流
   * @param {Object} SSE_PASH - 地址
   * @param {Object} requestParams - 请求参数
   * @param {number} messageIndex - 消息索引
   * @param {Object} sseHeaders - 请求头
   * @param {Object} callbacks - 回调函数
   * @param {Function} callbacks.onStreamUpdate - 流更新回调
   * @param {Function} callbacks.onFinish - 完成回调
   * @param {Function} callbacks.onError - 错误回调
   * @param {Function} callbacks.onScroll - 滚动回调
   */
  async startChatStream(SSE_PASH, requestParams, messageIndex, sseHeaders, callbacks = {}) {
    // 重置状态
    this.resetChatState();

    // 设置回调
    this.messageCallbacks = { ...this.messageCallbacks, ...callbacks };

    // 设置消息索引
    this.currentMessageIndex = messageIndex;
    this.url = SSE_PASH;
    this.requestParams = requestParams;

    // 构建SSE URL
    const publicParams = getPublick(requestParams, SSE_PASH);
    const sseUrl = `${IM_BASE_URL}${SSE_PASH}?${serialize(publicParams)}`;
    const sseOptions = {
      headers: {
        'didi-header-hint-content': JSON.stringify(sseHeaders || {}),
      },
      onMessage: this.handleChatMessage.bind(this),
      onError: this.handleChatError.bind(this),
      onOpen: this.handleChatOpen.bind(this),
      onClose: this.handleChatClose.bind(this),
    };
    this.sseOptions = sseOptions;

    // 连接SSE
    try {
      await this.connect(sseUrl, sseOptions, false);
    } catch (error) {
      console.log('SSE连接失败:', error);
      const errorMessage = error?.message || String(error) || error?.toString();
      trackError('ChatSSEManager', {
        source: 'startChatStream',
        error: `===SSE连接失败===${errorMessage}`,
      });
      //这里再throw，谁来承接 需要throw吗
      throw new Error(error);
    }
  }

  /**
   * 处理聊天消息
   * @param {Object} response - 响应数据
   * @param {Event} event - 原始事件
   */
  handleChatMessage(response, event) {
    try {
      const { mode, message_id, finish_reason, data, meta } = response;
      const safeData = !Array.isArray(data) ? [] : data.map((dataItem) => ({ ...dataItem, mode }));
      // console.log('mode===', mode, 'safeData===', safeData);
      let replyStatus = AgentStatus.GENERATING;

      // 更新思考状态
      this.updateThinkState(safeData);

      // 根据mode处理数据
      switch (mode) {
        case AgentMode.APPEND:
        case AgentMode.UPDATE:
        case AgentMode.OVERWRITE:
          this.handleStreamContent(safeData, mode);
          break;
        case AgentMode.ERROR:
          this.handleError(finish_reason);
          replyStatus = AgentStatus.FINISH;
          break;
        case AgentMode.FINISH:
          this.handleFinish(meta);
          replyStatus = AgentStatus.FINISH;
          break;
      }

      // 更新消息状态
      this.updateMessageStatus(message_id, mode, finish_reason, replyStatus, meta);

      const { onStreamUpdate, onScroll } = this.messageCallbacks;
      // 触发流更新回调
      if (onStreamUpdate && typeof onStreamUpdate === 'function') {
        onStreamUpdate({
          message_id,
          messageIndex: this.currentMessageIndex,
          streamContent: cloneDeep(this.chatStream),
          status: replyStatus,
          thinkEnd: this.thinkEnd,
          hasStreamContent: this.hasStreamContent,
        });
      }

      // 处理滚动
      if (onScroll && typeof onScroll === 'function') {
        onScroll({
          isFinish: mode === AgentMode.ERROR || mode === AgentMode.FINISH,
        });
      }

      // 处理完成状态
      if (mode === AgentMode.FINISH || mode === AgentMode.ERROR) {
        this.handleFinalStatus(meta);
      }
    } catch (error) {
      console.log('处理聊天消息错误:', error);
      const errorMessage = error?.message || String(error) || error?.toString();
      trackError('ChatSSEManager', {
        source: 'handleChatMessage',
        error: `===处理聊天消息错误===${errorMessage}`,
      });
    }
  }

  /**
   * 处理流内容
   * @param {Array} eventData - 流内容数据
   * @param {string} mode - 模式
   */
  handleStreamContent(eventData, mode) {
    if (mode === AgentMode.UPDATE) {
      trackEvent('', { updateContent: JSON.stringify(eventData) }, '==更新数据==');
    }
    const hasValidData = eventData?.length;
    if (hasValidData) {
      const lastData = eventData[eventData.length - 1];
      const { id } = lastData || {};
      if (id) {
        this.offsetId = id;
      }
    }
    const preChatStream = cloneDeep(this.chatStream);
    this.chatStream = [...preChatStream, ...eventData];
  }

  /**
   * 处理错误
   * @param {string} finishReason - 完成原因
   */
  handleError(finishReason) {
    const { onError } = this.messageCallbacks;
    if (onError && typeof onError === 'function') {
      onError(finishReason || LOAD_FAIL_CONTENT);
    }
  }

  /**
   * 处理完成
   * @param {Object} meta - meta信息
   */
  handleFinish(meta) {
    this.userMessage = null;
    this.assistantMessage = null;
    const { onFinish } = this.messageCallbacks;
    if (onFinish && typeof onFinish === 'function') {
      onFinish(meta);
    }
  }

  /**
   * 更新思考状态
   * @param {string} type - 内容类型
   * @param {string} content - 内容
   */
  updateThinkState(eventData) {
    const isNonThinkType = eventData.some((item) => item.type != ThinkType.THINK);
    if (!this.thinkEnd && isNonThinkType) {
      // console.log('===updateThinkState===', this.thinkEnd, eventData);
      this.thinkEnd = true;
    }
    const isThinkType = eventData.some((item) => item.type == ThinkType.THINK);
    if (!this.hasStreamContent && isThinkType) {
      this.hasStreamContent = true;
    }
  }

  /**
   * 更新消息状态
   * @param {string} messageId - 消息ID
   * @param {string} mode - 模式
   * @param {string} finishReason - 完成原因
   * @param {string} replyStatus - 回复状态
   * @param {Object} meta - meta信息
   */
  updateMessageStatus(messageId, mode, finishReason, replyStatus, meta) {
    // TODO: 可以根据需要更新消息状态实现消息管理逻辑
  }

  /**
   * 处理最终状态
   * @param {Object} meta - meta信息
   */
  handleFinalStatus(meta) {
    setTimeout(() => {
      const { onFinish } = this.messageCallbacks;
      if (onFinish && typeof onFinish === 'function') {
        onFinish({
          ...meta,
          status: meta?.status || AgentStatus.AVAILABLE,
        });
      }
    });
  }

  /**
   * 处理聊天错误
   * @param {Error} error - 错误对象
   */
  //todo: yuhua 这块代码 check下 看着有点怪怪的
  async handleChatError(error) {
    const currentRetryCount = this.getRetryCount();
    const { onError } = this.messageCallbacks;
    // console.log('SSE错误:', this.offsetId, currentRetryCount);

    if (currentRetryCount < this.maxRetry) {
      try {
        console.log(`尝试第${currentRetryCount + 1}次重连...`);
        const sseUrl = this.getSseUrl(this.offsetId);
        await this.retry(sseUrl, this.sseOptions);
      } catch (retryError) {
        console.log('重试失败:', retryError);
        if (onError && typeof onError === 'function') {
          onError(retryError || new Error(LOAD_FAIL_CONTENT));
        }
      }
    } else {
      console.log('重试次数已达上限:', currentRetryCount);
      if (onError && typeof onError === 'function') {
        onError(error || new Error(LOAD_FAIL_CONTENT));
      }
    }
  }

  /**
   * 处理聊天连接打开
   * @param {Event} event - 事件对象
   */
  handleChatOpen(event) {
    console.log(SSE_ESTABLISH_CONTENT);
  }

  /**
   * 处理聊天连接关闭
   * @param {Event} event - 事件对象
   */
  handleChatClose(event) {
    console.log(SSE_DISCONNECT_CONTENT);
  }

  /**
   * 重置聊天状态
   */
  resetChatState() {
    this.currentMessageIndex = -1;
    this.chatStream = [];
    this.offsetId = 0;
    this.thinkEnd = false;
    this.hasStreamContent = false;
  }

  /**
   * 设置消息数据
   * @param {Object} userMessage - 用户消息
   * @param {Object} assistantMessage - 助手消息
   */
  setMessageData(userMessage, assistantMessage) {
    this.userMessage = userMessage;
    this.assistantMessage = assistantMessage;
    this.chatStream = assistantMessage.contents ? cloneDeep(assistantMessage.contents) : [];
  }

  /**
   * 获取当前流内容
   * @returns {Array}
   */
  getCurrentStream() {
    return cloneDeep(this.chatStream);
  }

  /**
   * 获取当前流式消息ID
   * @returns {number}
   */
  getCurrentOffset() {
    return this.offsetId;
  }

  /**
   * 获取SSE URL
   * @param {number} offset - 上次中断的流式消息ID
   * @returns {string}
   */
  getSseUrl(offset) {
    const { chat_id } = this.assistantMessage?.meta || {};
    const paramsServer = {
      ...this.requestParams,
      chat_id,
      offset,
    };
    const publicParams = getPublick(paramsServer, this.url);
    const sseUrl = `${IM_BASE_URL}${this.url}?${serialize(publicParams)}`;
    return sseUrl;
  }

  /**
   * 检查是否正在思考
   * @returns {boolean}
   */
  isThinking() {
    return !this.thinkEnd;
  }

  /**
   * 检查是否有流内容
   * @returns {boolean}
   */
  hasContent() {
    return this.hasStreamContent;
  }
}

export default ChatSSEManager;
```

SSE需要根据业务封装

## 六、温馨提示

### 1、请求头
如果对于SSE的请求要求需要请求头，那么EventSource就不满足，因为SSE无法设置请求头，那么就可以使用event-source-polyfill这个库，支持设置请求头

### 2、单例模式
如果SSE需要封装，可以写成单例模式，这样不会导致内存泄露；如果不封装，可以全局设置一个eventSource，必要时机设置为null，以防内存泄漏