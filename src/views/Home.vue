<template>
  <div>
    <!-- 滚动容器一 -->
    <Refresh
      class="container-refresh"
      :isMoreData="isMoreData"
      :isLoading="isLoading"
      :isLoadingWatch="isLoadingWatch"
      @loading="onLoading"
    >
      <!-- 显示内容 -->
      <div v-for="item in dataSource" :key="item">{{ item }}</div>
    </Refresh>

    <!-- 滚动容器二 -->
    <Refresh class="container-refresh">
      <!-- 显示内容 -->
      <div v-for="item in dataSource" :key="item">{{ item }}</div>
      <!-- 自定义 footer -->
      <!-- <template slot="footer" slot-scope="props">
        {{ props }}
      </template> -->
      <!-- <template slot="footer" slot-scope="props">
        <div :style="props.footerStyle">自定义底部</div>
      </template> -->
    </Refresh>
  </div>
</template>

<script>
// 导入
import Refresh from '@/components/Refresh'
export default {
  components: {
    Refresh
  },
  data () {
    return {
      // 有数据
      isMoreData: true,
      // 加载状态
      isLoading: false,
      // 是否在加载完成时，检查当前 footer 元素是否在进入加载范围
      // 如果确定数据一定会超过滚动视图高度，则不需要配置。
      isLoadingWatch: true,
      // 测试数据源
      dataSource: []
    }
  },
  mounted () {
    // 初始化获取数据
    this.getData()
  },
  methods: {
    // 加载中
    onLoading () {
      // 获取数据
      this.getData()
    },
    // 获取数据
    getData () {
      // 加载中
      this.isLoading = true
      // 模拟请求
      setTimeout(() => {
        // 模拟数据
        var length = this.dataSource.length + 1
        var nums = []
        for (let index = length; index < (length + 5); index++) { nums.push(index) }
        // 拼接数据
        this.dataSource = this.dataSource.concat(nums)
        // 是否还有更多数据
        this.isMoreData = this.dataSource.length < 20
        // 取消加载
        this.isLoading = false
      }, 1000);
    }
  }
}
</script>

<style lang="less" scoped>
.refresh-view {
  margin-top: 20px;
  height: 200px;
  background-color: greenyellow;
}
</style>