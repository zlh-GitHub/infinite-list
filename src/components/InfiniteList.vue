<template>
  <!-- 可视区域 -->
  <div class="infinite-list-container" @scroll="onILCScroll">
    <!-- 总列表高度，用于生成滚动条 -->
    <div class="infinite-list-phantom" :style="{ height: `${totalHeight}px` }"></div>
    <!-- 可视区域列表 -->
    <div class="infinite-list" :style="{ transform: `translateY(${startOffset}px)` }">
      <div v-for="item in showList" :key="item" class="infinite-list-item" ref="itemRefs">
        <slot :item="item" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, defineProps, reactive, watch, useTemplateRef } from 'vue';
const { listData: list, itemHeight, radianCount, height: screenHeight, estimatedItemHeight } = defineProps({
  listData: { // 总数据
    type: Array,
    required: true,
    default: [],
  },
  height: {
    type: Number, // 可视区域高度
    required: true,
    default: 500,
  },
  itemHeight: { // 每个列表项高度
    type: Number,
    required: false,
    validator(_, props) {
      if (!props.itemHeight && !props.estimatedItemHeight) {
        console.error('itemHeight and estimatedItemHeight must have at least one');
        return false;
      }
      return true;
    }
  },
  estimatedItemHeight: { // 估计的列表项高度
    type: Number,
    required: false,
    validator(_, props) {
      if (!props.itemHeight && !props.estimatedItemHeight) {
        console.error('itemHeight and estimatedItemHeight must have at least one');
        return false;
      }
      return true;
    }
  },
  radianCount: { // 预加载的列表项个数(前后缓冲区数量)
    type: Number,
    required: true,
    default: 5,
  },
});
const itemCount = computed(() => list.length); // 列表项个数
const showCount = Math.ceil(screenHeight / (itemHeight || estimatedItemHeight)); // 可视区域显示的列表项个数
const renderCount = showCount + radianCount * 2; // 渲染列表项个数
// const showTotalHeight = computed(() => renderCount * itemHeight); // 展示列表的总高度
const startIndex = ref(0); // 展示列表的起始索引
const endIndex = ref(renderCount); // 展示列表的结束索引
const showList = computed(() => list.slice(startIndex.value, endIndex.value)); // 展示的列表
const startOffset = ref(0); // 可视区域偏移量

const itemRefs = useTemplateRef('itemRefs'); // 列表项的 ref
// 预估高度来进行初始化
const itemsPosition = reactive(list.map((_, i) => ({
  index: i,
  height: estimatedItemHeight,
  top: i * estimatedItemHeight,
  bottom: (i + 1) * estimatedItemHeight,
}))); // 每个列表项的大小位置信息

const totalHeight = computed(() => itemsPosition[itemCount.value - 1].bottom); // 总高度

const onDefaultILCScroll = e => {
  const { target: { scrollTop } } = e;
  if (endIndex.value < itemCount.value) { // endIndex.value 是上次渲染的结束索引，确保最后的偏移准确，不会少偏移一个 itemHeight
    // 当渲染完最后 renderCount 个列表项时，不需要更新列表和偏移量
    let offset = scrollTop - (scrollTop % itemHeight); // 计算偏移量
    offset = offset - radianCount * itemHeight; // 当有缓冲区时，减去缓冲区的高度，才能实现缓冲区生效，即顶部可视区域外始终有 raidanCount 个元素
    offset = Math.max(0, offset); // 保证 offset 不会小于 0
    startOffset.value = offset;
  }
  let sIndex = Math.floor(scrollTop / itemHeight); // 计算开始索引
  sIndex = Math.floor(scrollTop / itemHeight) - radianCount; // 当缓冲区内容还在可视区域时，不更新开始索引
  sIndex = Math.max(0, sIndex); // 保证开始索引不会小于 0
  sIndex = Math.min(sIndex, itemCount.value - renderCount); // 保证开始索引不会超过最大值
  startIndex.value = sIndex; // 更新开始索引

  endIndex.value = Math.min(startIndex.value + renderCount, itemCount.value); // 更新结束索引
}

const onEstimatedILCScroll = e => {
  const { target: { scrollTop } } = e;
  let sIndex = itemsPosition.findIndex(i => i.bottom >= scrollTop); // 计算开始索引，找到第一个 bottom 值大于 scrollTop 的
  const firstItem = itemsPosition[sIndex] || {};
  if (endIndex.value < itemCount.value) { // endIndex.value 是上次渲染的结束索引，确保最后的偏移准确，不会少偏移一个 itemHeight
    // 当渲染完最后 renderCount 个列表项时，不需要更新列表和偏移量
    let offset = firstItem.top; // 计算偏移量
    // 当有缓冲区时，减去缓冲区的高度，才能实现缓冲区生效，即顶部可视区域外始终有 raidanCount 个元素
    // 倒序遍历减去前 radianCount 的高度和
    for(let i = 0; i < radianCount; i++) {
      const prevItem = itemsPosition[sIndex - i - 1];
      if (prevItem) {
        offset -= prevItem.height;
      }
    }
    offset = Math.max(0, offset); // 保证 offset 不会小于 0

    startOffset.value = offset;
  }
  
  sIndex -= radianCount; // 当缓冲区内容还在可视区域时，不更新开始索引
  sIndex = Math.max(0, sIndex); // 保证开始索引不会小于 0
  sIndex = Math.min(sIndex, itemCount.value - renderCount); // 保证开始索引不会超过最大值
  startIndex.value = sIndex; // 更新开始索引
  endIndex.value = Math.min(sIndex + renderCount, itemCount.value); // 更新结束索引
}

// 更新缓存的列表项位置信息
const updateItemsPosition = () => {
  for(let i = startIndex.value; i < itemCount.value; i++) {
    const index = i - startIndex.value;
    const itemPosition = itemsPosition[i];
    const { bottom: prevItemBottom = 0 } = itemsPosition[i - 1] || {};
    const itemDom = itemRefs.value[index];
    if (itemDom) {
      itemPosition.height = itemDom.getBoundingClientRect().height;
      itemPosition.top = prevItemBottom;
      itemPosition.bottom = itemPosition.top + itemPosition.height;
    } else {
      itemPosition.height = estimatedItemHeight;
      itemPosition.top = prevItemBottom;
      itemPosition.bottom = prevItemBottom + estimatedItemHeight;
    }
  }
}

watch(startIndex, () => {
  requestAnimationFrame(() => {
    // 更新高度
    updateItemsPosition();
  });
}, { immediate: true })

const onILCScroll = e => {
  if (itemHeight && !estimatedItemHeight) {
    onDefaultILCScroll(e);
    return;
  } else if (!itemHeight && estimatedItemHeight) {
    onEstimatedILCScroll(e);
  }
}
</script>

<style scoped>
.infinite-list-container {
  width: 100%;
  height: 100%;
  overflow-y: auto;
  position: relative;

  .infinite-list {
    width: 100%;
    position: absolute;
    left: 0;
    top: 0;

    .infinite-list-item {
      height: 100%;
    }
  }
}
</style>