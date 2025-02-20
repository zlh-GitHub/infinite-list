<template>
  <!-- 可视区域 -->
  <div class="infinite-list-container" :style="{ height: `${screenHeight}px` }" @scroll="onILCScroll">
    <!-- 总列表高度，用于生成滚动条 -->
    <div class="infinite-list-phantom" :style="{ height: `${totalHeight}px` }"></div>
    <!-- 可视区域列表 -->
    <div class="infinite-list" :style="{ height: `${showTotalHeight}px`, transform: `translateY(${startOffset}px)` }">
      <div v-for="item in showList" :key="item" class="infinite-list-item" :style="{ height: `${itemHeight}px` }">
        {{ item }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed, ref } from 'vue';
const itemSize = 10000;
const itemHeight = 30; // 每个列表项高度
const screenHeight = 500; // 可视区域高度
const showCount = Math.ceil(screenHeight / itemHeight); // 可视区域显示的列表项个数
const list = reactive(new Array(itemSize).fill(0).map((_, i) => i)); // 总列表
const totalHeight = computed(() => itemSize * itemHeight); // 总高度
const showTotalHeight = computed(() => showCount * itemHeight); // 展示列表的总高度
const startIndex = ref(0); // 展示列表的起始索引
const endIndex = ref(showCount); // 展示列表的结束索引
const showList = computed(() => list.slice(startIndex.value, endIndex.value)); // 展示的列表
const startOffset = ref(0); // 可视区域偏移量

const onILCScroll = e => {
  const { target: { scrollTop } } = e;
  startOffset.value = scrollTop - (scrollTop % itemHeight); // ‼️关键代码 为什么要减去余数？因为 scrollTop 是相对于可视区域的，而我们需要的是相对于列表的偏移量
  startIndex.value = Math.floor(scrollTop / itemHeight);
  endIndex.value = Math.min(startIndex.value + showCount, itemSize);
}
</script>

<style scoped>
.infinite-list-container {
  width: 300px;
  overflow-y: auto;
  border: 1px solid #000;
  position: relative;

  .infinite-list {
    width: 300px;
    position: absolute;
    left: 0;
    top: 0;

    .infinite-list-item {
      width: 100%;
      border-bottom: 1px solid #000;
    }
  }
}
</style>