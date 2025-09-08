<script setup>
import { useBlogType } from '@vuepress/plugin-blog/client'
import ParentLayout from '@vuepress/theme-default/layouts/Layout.vue'
import ArticleList from '../components/ArticleList.vue'

const timelines = useBlogType('timeline')
</script>

<template>
  <ParentLayout>
    <template #page>
      <main class="page">
        <ArticleList :items="timelines.items" is-timeline />
      </main>
    </template>
  </ParentLayout>
</template>

<style lang="scss">
.timeline-title {
  padding-top: calc(var(--navbar-height) + 1rem) !important;
  text-align: center;
}
</style>
