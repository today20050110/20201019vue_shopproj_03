<template>
  <div>
      <!-- 麵包屑導航區 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>權限管理</el-breadcrumb-item>
      <el-breadcrumb-item>權限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片視圖 -->
    <el-card>
      <el-table :data="rightsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="權限名稱" prop="authName"></el-table-column>
        <el-table-column label="路徑" prop="path"></el-table-column>
        <el-table-column label="權限等級" prop="level">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'">一級</el-tag>
            <el-tag v-else-if="scope.row.level === '1'" type="success">二級</el-tag>
            <el-tag v-else type="warning">三級</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 權限列表
      rightsList: []
    }
  },
  created() {
    // 獲取所有的權限
    this.getRightsList()
  },
  methods: {
    // 獲取權限列表
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('獲取權限列表失敗!')
      }
      this.rightsList = res.data
      // console.log(this.rightsList)
    }
  }
}
</script>

<style lang="less" scoped>

</style>
