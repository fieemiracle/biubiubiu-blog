---
title: 学习文档直通车
sidebar: false
footer: 众里寻她千百度，蓦然回首，那人却在灯火阑珊处。
---

# 🌐 学习文档直通车

## 📝 框架指南

<div class="link-grid">
  <div class="link-card">
    <div class="link-header">👉 Vue2</div>
    <div class="link-desc">渐进式框架（Vue2已经终止支持且不再维护）</div>
    <a href="https://v2.cn.vuejs.org" target="_blank" class="link-btn">参考文档</a>
  </div>
  <div class="link-card">
    <div class="link-header">📚 Vue3</div>
    <div class="link-desc">渐进式框架</div>
    <a href="https://cn.vuejs.org/guide/introduction" target="_blank" class="link-btn">参考文档</a>
  </div>
  <div class="link-card">
    <div class="link-header">🍎 Svelte</div>
    <div class="link-desc">Svelte 是一个用于构建 web 用户界面的框架。它使用编译器将用 HTML、CSS 和 JavaScript 编写的声明式组件</div>
    <a href="https://svelte.yayujs.com/docs/svelte/overview" target="_blank" class="link-btn">参考文档</a>
  </div>
  <div class="link-card">
    <div class="link-header">📖 JavaScript</div>
    <div class="link-desc">JavaScript 就像是 “根据这个标准制造出来的一辆宝马汽车”。它能跑，但安全配置看厂家心情。</div>
    <a href="https://wangdoc.com/javascript/basic/introduction" target="_blank" class="link-btn">参考文档</a>
  </div>
    <div class="link-card">
      <div class="link-header">🌲 TypeScript</div>
      <div class="link-desc">TypeScript 就像是 “给这辆宝马加装了一套顶级的自动驾驶系统和防撞预警系统”。它还是那辆宝马（最终产物还是 JS），但开发过程更安全、更智能，能在上路前就避免很多事故（类型错误）。</div>
      <a href="https://wangdoc.com/typescript/intro" target="_blank" class="link-btn">参考文档</a>
    </div>
    <div class="link-card">
      <div class="link-header">🚗 ECMAScript 6</div>
      <div class="link-desc">ECMAScript 就像是 “官方发布的汽车制造与安全标准”</div>
      <a href="https://es6.ruanyifeng.com" target="_blank" class="link-btn">参考文档</a>
    </div>
</div>

## 🌊 知识库

<div class="link-grid">
  <div class="link-card">
    <div class="link-header">🍇 CSS Grid布局</div>
    <div class="link-desc">网格布局（Grid）是最强大的 CSS 布局方案。它将网页划分成一个个网格，可以任意组合不同的网格，做出各种各样的布局。以前，只能通过复杂的 CSS 框架达到的效果，现在浏览器内置了。</div>
    <a href="https://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html" target="_blank" class="link-btn">参考文档</a>
  </div>
</div>

<style>
.link-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  background: var(--c-bg-soft);
}

.link-card {
  background: var(--c-bg-soft);
  border: 1px solid var(--c-border);
  border-radius: 12px;
  padding: 1.5rem;
  text-align: center;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  cursor: pointer;
}

.link-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  border-color: var(--c-brand);
}

.link-header {
  font-size: 1.6rem;
  margin-bottom: 1rem;
  display: block;
}

.link-desc {
  margin: 0 0 1.5rem 0;
  color: var(--c-text-light);
  font-size: 0.9rem;
  line-height: 1.4;
  text-align: left;
}

.link-btn {
  display: inline-block;
  background: var(--c-brand);
  color: var(--c-text-light);
  border-radius: 6px;
  text-decoration: none;
  font-weight: 500;
  transition: all 0.3s ease;
  font-size: 0.9rem;
}

.link-btn:hover {
  background: var(--c-brand-light);
  transform: scale(1.05);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .link-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  
  .link-card {
    padding: 1rem;
  }
  
  .link-header {
    font-size: 2rem;
  }
}
</style>
