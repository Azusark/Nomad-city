<template>
  <div class="card">
    <slot></slot>
  </div>
</template>

<script setup>
// 卡牌组件逻辑
</script>

<style scoped>
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  margin: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
</style> 