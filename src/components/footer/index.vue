<template>
  <a-layout-footer class="footer">{{ appStore.projectName }}</a-layout-footer>
</template>

<script setup>
import { useAppStore } from '@/store'

const appStore = useAppStore()
</script>

<style lang="less" scoped>
  .footer {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 40px;
    color: var(--color-text-2);
    text-align: center;
  }
</style>
