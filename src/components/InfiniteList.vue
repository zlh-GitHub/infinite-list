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
const radianCount = 5; // 预加载的列表项个数(前后缓冲区数量)
const renderCount = showCount + radianCount * 2; // 渲染列表项个数
const list = reactive(new Array(itemSize).fill(0).map((_, i) => i)); // 总列表
const totalHeight = computed(() => itemSize * itemHeight); // 总高度
const showTotalHeight = computed(() => renderCount * itemHeight); // 展示列表的总高度
const startIndex = ref(0); // 展示列表的起始索引
const endIndex = ref(renderCount); // 展示列表的结束索引
const showList = computed(() => list.slice(startIndex.value, endIndex.value)); // 展示的列表
const startOffset = ref(0); // 可视区域偏移量

const onILCScroll = e => {
  const { target: { scrollTop } } = e;
  if (endIndex.value < itemSize) { // endIndex.value 是上次渲染的结束索引，确保最后的偏移准确，不会少偏移一个 itemHeight
    // 当渲染完最后 renderCount 个列表项时，不需要更新列表和偏移量
    let offset = scrollTop - (scrollTop % itemHeight); // 计算偏移量
    offset = offset - radianCount * itemHeight; // 当有缓冲区时，减去缓冲区的高度，才能实现缓冲区生效，即顶部可视区域外始终有 raidanCount 个元素
    offset = Math.max(0, offset); // 保证 offset 不会小于 0
    startOffset.value = offset;
  }
  let sIndex = Math.floor(scrollTop / itemHeight); // 计算开始索引
  sIndex = Math.floor(scrollTop / itemHeight) - radianCount; // 当缓冲区内容还在可视区域时，不更新开始索引
  sIndex = Math.max(0, sIndex); // 保证开始索引不会小于 0
  sIndex = Math.min(sIndex, itemSize - renderCount); // 保证开始索引不会超过最大值
  startIndex.value = sIndex; // 更新开始索引

  endIndex.value = Math.min(startIndex.value + renderCount, itemSize); // 更新结束索引
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
      box-sizing: border-box;
    }
  }
}
</style>