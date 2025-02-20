<script setup>
import VirtualList from './components/InfiniteList.vue';
import { reactive } from 'vue';
const itemSize = 10000;
const itemHeight = 30;
const radianCount = 5;
const list = reactive(new Array(itemSize).fill(0).map((_, i) => i)); // 总列表
</script>

<template>
  <div class="list">
    <VirtualList :listData="list" :itemHeight="itemHeight" :height="600" :radianCount="radianCount" v-slot="slotProps">
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
    height: 30px;
    border-bottom: 1px solid red;
    box-sizing: border-box;
  }
}
</style>
