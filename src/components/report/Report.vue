<template>
  <div>
    <!-- 麵包屑導航區 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>數據統計</el-breadcrumb-item>
      <el-breadcrumb-item>數據報表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片視圖區域 -->
    <el-card>
      <!--2. 为 ECharts 准备一个具备大小（宽高）的 DOM -->
      <div id="main" style="width: 750px;height:400px;"></div>
    </el-card>
  </div>
</template>

<script>
// 1. 導入echarts
import echarts from 'echarts'
import _ from 'lodash'
export default {
  data() {
    return {
      // 需要合併的數據
      options: {
        title: {
          text: '用户來源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  created() {
  },
  // 此時,頁面上的元素,已經被渲染完畢了
  async mounted() {
    // 3. 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'))
    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) {
      return this.$message.error('獲取折線圖數據失敗')
    }
    // 4. 準備數據和配置項
    const result = _.merge(res.data, this.options)
    // 5. 展示數據
    myChart.setOption(result)
  }
}
</script>

<style lang="less" scoped>

</style>
