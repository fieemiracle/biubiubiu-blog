<script setup>
import { useBlogCategory } from '@vuepress/plugin-blog/client'
import ParentLayout from '@vuepress/theme-default/layouts/Layout.vue'
import { RouteLink, useRoutePath } from 'vuepress/client'
import ArticleList from '../components/ArticleList.vue'
import { computed } from 'vue'

const tagMap = useBlogCategory('tag')
const routePath = useRoutePath()

// 为每个 tag 生成基于名称的稳定随机颜色
const getTagColor = (tagName) => {
  // 使用 tag 名称作为种子生成稳定的随机数
  let hash = 0
  for (let i = 0; i < tagName.length; i++) {
    const char = tagName.charCodeAt(i)
    hash = ((hash << 5) - hash) + char
    hash = hash & hash // 转换为32位整数
  }
  
  // 使用哈希值生成 RGB 颜色
  const r = Math.abs(hash) % 151 + 50
  const g = Math.abs(hash >> 8) % 151 + 50
  const b = Math.abs(hash >> 16) % 151 + 50
  
  return `rgb(${r}, ${g}, ${b})`
}

</script>

<template>
  <ParentLayout>
    <template #page>
      <main class="page">
        <div class="tag-wrapper">
          <RouteLink
            v-for="({ items, path }, name) in tagMap.map"
            :key="name"
            :to="path"
            :active="routePath === path"
            :style="{ backgroundColor: getTagColor(name) }"
            class="tag"
          >
            {{ name }}
            <span class="tag-num">
              {{ items.length }}
            </span>
          </RouteLink>
        </div>

        <ArticleList :items="tagMap.currentItems ?? []" />
      </main>
    </template>
  </ParentLayout>
</template>

<style lang="scss">
@use '@vuepress/theme-default/styles/mixins';

.tag-wrapper {
  @include mixins.content-wrapper;

  margin-top: calc(var(--navbar-height) + 1rem) !important;
  padding-bottom: 0 !important;
  font-size: 0.875em;
  width: fit-content;
  height: fit-content;

  .route-link {
    color: inherit;
  }

  .tag {
    display: inline-block;
    vertical-align: middle;

    overflow: hidden;

    margin: 0.3rem 0.6rem 0.8rem;
    padding: 0.4rem 0.8rem;
    border-radius: 0.25rem;

    cursor: pointer;

    transition:
      background 0.3s,
      color 0.3s;

    @media (max-width: 419px) {
      font-size: 0.9rem;
    }

    .tag-num {
      display: inline-block;

      min-width: 1rem;
      height: 1.2rem;
      margin-inline-start: 0.2em;
      padding: 0 0.1rem;
      border-radius: 0.6rem;

      font-size: 0.7rem;
      line-height: 1.2rem;
      text-align: center;
    }

    &.route-link-active {
      background: var(--vp-c-accent-bg);
      color: var(--vp-c-accent-text);

      .tag-num {
        color: var(--vp-c-accent-text);
      }
    }
  }
}
</style>
