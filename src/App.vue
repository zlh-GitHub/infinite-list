<script setup>
import VirtualList from './components/InfiniteList.vue';
import { reactive } from 'vue';
const itemSize = 10000;
const itemHeight = 30;
const radianCount = 5;
const list = reactive(new Array(itemSize).fill(0).map((_, i) => `【${i}】${'InfiniteList'.repeat(Math.floor(Math.random() * 5) + 5)}`)); // 总列表
</script>

<template>
  <div class="list">
    <VirtualList :listData="list" :height="600" :radianCount="radianCount" :estimatedItemHeight="50" v-slot="slotProps">
      <div class="item">{{ slotProps.item }}</div>
    </VirtualList>
  </div>
</template>

<style scoped>
.list {
  width: 300px;
  height: 600px;
  border: 1px solid red;
  overflow: hidden;

  .item {
    width: 100%;
    border-bottom: 1px solid red;
    text-align: left;
    box-sizing: border-box;
    word-break: break-all;
  }
}
</style>
