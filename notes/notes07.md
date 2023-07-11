## 30.
滚动虚拟列表的实现原理演示（vue版本）
```
<template>
  <scroll-view 
    class="n-rel" 
    :scroll-y="true" 
    @scroll="handleScroll" 
    :style="viewportHeightStyle"
  >
    <view class="holder" :style="rowListHeightStyle" />
    <view 
      class="row-item-list n-abs n-w100"
      :style="startOffset"
    >
      <view 
        class="row-item n-flex n-justify-sb" 
        v-for="(rowItem, index) in showRowList" 
        :key="rowItem.id"
      >
        {{rowItem.text}}
      </view>
    </view>
  </scroll-view>
</template>

<script>
  // 上下缓冲区行数
  const amountRowsBuffered = 0;

	export default {
		data() {
			return {
        rowList: [], // 虚拟列表（即原始数据）
        rowHeight: 300, // 单项行高，演示场景下写死，实际场景从页面中获取真实行高
        viewportHeight: 600, // 可视区域的视口高度
        scrollTop: 0, // 滚动距离
      }
		},
		computed: {
      // 虚拟列表的总高度，作用是撑开内容，使容器内可滚动
      rowListHeightStyle() {
        const h = this.rowList.length * this.rowHeight;
        return `height: ${h}px;`;
      },
      // 可视区域的视口高度
      viewportHeightStyle() {
        return `height: ${this.viewportHeight}px;`;
      },
      // 真实渲染列表的偏移位置（从顶边开始计算）
      startOffset() {
        const offset = this.scrollTop - (this.scrollTop % this.rowHeight);
        return `transform: translateY(${offset}px);`
      },
      // 真实渲染列表在虚拟列表中的开始索引
      indexStart() {
        return Math.max(
          Math.floor(this.scrollTop / this.rowHeight) - amountRowsBuffered,
          0
        );
      },
      // 真实渲染列表在虚拟列表中的结束索引
      indexEnd() {
        return Math.min(
          Math.ceil((this.scrollTop + this.viewportHeight) / this.rowHeight - 1) + amountRowsBuffered,
          this.rowList.length - 1
        );
      },
      // 真实渲染列表
      showRowList() {
        console.log(`真实渲染列表区间：${this.indexStart}..${this.indexEnd}`);
        return this.rowList.slice(this.indexStart, this.indexEnd + 1);
      },
		},
    methods: {
      // 侦听页面滚动事件，为了不频繁执行而引起卡顿，必须进行节流
      handleScroll: this.$util.throttle(function(e) {
        console.log('滚动事件触发了', e.detail.scrollTop)
        this.scrollTop = e.detail.scrollTop;
      }, 300),
    },
    // 页面初始挂载时获取屏高作为可视区域的视口高度
    mounted() {
      const { windowHeight } = uni.getSystemInfoSync();
      this.viewportHeight = windowHeight;
    },
	}
</script>

<style lang="scss" scoped>
	.row-item-list {
    top: 0;
		.row-item {
      height: 300px;
		}
	}
</style>
```
