<template>
  <!-- 容器 -->
  <div
    class="refresh-view"
    ref="refresh-view"
    @mousewheel="scrollChange"
  >
    <!-- 内容 -->
    <slot></slot>
    <!-- 尾部 -->
    <div
      class="refresh-footer"
      ref="refresh-footer"
    >
      <!-- 可自定义尾部 -->
      <slot
        name="footer"
        :isMoreData="isMoreData"
        :isLoading="isLoading"
        :footerStyle="footerStyle"
        :nodata="nodata"
      >
        <!-- 默认尾部元素 -->
        <div
          class="footer-content"
          :style="footerStyle"
        >
          <!-- 加载中... -->
          <slot name="loading" v-if="isMoreData">加载中...</slot>
          <!-- 没有数据 -->
          <slot name="nodata" v-else>{{ nodata }}</slot>
        </div>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    // 是否有更多数据，没有更多数据则显示 '没有数据' 元素，并不会再调用加载回调
    isMoreData: {
      type: Boolean,
      default: true
    },
    // 加载状态，设置为加载中，则不会重复触发加载事件
    isLoading: {
      type: Boolean,
      default: false
    },
    // 是否在加载完成时，检查当前 footer 元素是否在进入加载范围
    // isLoading 设置状态需要放在 isMoreData 设置状态之后，因为需要优先判断是否有数据
    isLoadingWatch: {
      type: Boolean,
      default: true
    },
    // 是否监听窗口变化，有变化则检查是否进入加载范围
    isOnResize: {
      type: Boolean,
      default: false
    },
    // 没有数据提示
    nodata: {
      type: String,
      default: '没有更多数据了'
    },
    // 尾部显示多少比例，则可以进入加载，最大 1
    footerScale: {
      type: Number,
      default: 0.3
    },
    // 尾部样式
    footerStyle: {
      type: Object,
      default: () => { return new Object() }
    }
  },
  watch: {
    // 监听加载状态
    isLoading: {
      handler (newData) {
      // 是否允许检查当前 footer 在进入加载范围内
      // 注意：如果 refresh-view 元素有设置 padding-bottom 值，进入加载范围需要超出 padding-bottom 值，鼠标审查元素移到元素身上，则可以看到 footer 元素是否已经滚到 padding-bottom 值之上了。
      console.log(this.isLoadingWatch);
      if (this.isLoadingWatch) {
          // 为 false 时则算加载完成
          if (!newData) {
            // 延迟调用
            this.$nextTick(() => {
              // 手动触发滚动
              this.scrollChange()
            })
          }
        }
      }
    }
  },
  mounted () {
    // 获取滚动元素
    const scrollview = this.$refs['refresh-view']
    // 添加拖拽滚动条监听
    scrollview.addEventListener('scroll', this.scrollChange, true)
    // 监听窗口变化
    if (this.isOnResize) { window.addEventListener('resize', this.onResize) }
  },
  // beforeDestroy 与 destroyed 里面移除都行
  beforeDestroy () {
    // 获取指定元素
    const scrollview = this.$refs['refresh-view']
    // 移除监听
    scrollview.removeEventListener('scroll', this.scrollChange, true)
    // 移除窗口监听
    if (this.isOnResize) { window.removeEventListener('resize', this.onResize) }
  },
  methods: {
    // 窗口变化
    onResize () {
      // 手动触发滚动
      this.scrollChange()
    },
    // 滚动监听
    scrollChange () {
      // 有数据 && 没有在加载
      if (this.isMoreData && !this.isLoading) {
        // 获取容器对象
        const refreshView = this.$refs['refresh-view']
        const footerView = this.$refs['refresh-footer']
        // 获取容器高度
        const clientHeight = refreshView.clientHeight
        // 获取容器滚动高度
        const scrollHeight = refreshView.scrollHeight
        // 当前滚动容器顶上坐标位置
        const miny = refreshView.scrollTop
        // 当前滚动容器底部坐标位置
        const maxy = miny + clientHeight
        // 底部高度
        const footerHeight = footerView.clientHeight
        // 底部不可见高度
        const footerHideHeight = (1 - Math.min(1, this.footerScale)) * footerHeight
        // 进入加载的最大坐标位置，多减少个 1px ，便于不相等
        const loadingy = scrollHeight - footerHideHeight - 1
        // 是否进入加载范围
        if (maxy >= loadingy) {
          // 进入加载
          this.$emit('loading')
        }
      }
    }
  }
}
</script>

<style scoped>
.refresh-view {
  width: 100%;
  height: 100%;
  overflow-y: auto;
}
.refresh-footer {
  width: 100%;
}
.footer-content {
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(0,0,0,.65);
  font-size: 14px;
  width: 100%;
  height: 48px;
}
</style>