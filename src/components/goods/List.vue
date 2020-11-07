<template>
  <div>
    <!-- 麵包屑導航區 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片視圖區域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="請輸入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table表格區域 -->
      <el-table :data="goodslist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名稱" prop="goods_name"></el-table-column>
        <el-table-column label="商品價格(元)" prop="goods_price" width="95px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="創建時間" prop="add_time" width="140px">
          <template slot-scope="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditGoodsDialog(scope.row.goods_id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分頁區域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background>
      </el-pagination>
    </el-card>
    <!-- 修改分類的對話框 -->
    <el-dialog
      title="修改商品"
      :visible.sync="editGoodsDialogVisible"
      width="50%"
      @close="editGoodsDialogClosed">
      <el-form :model="editGoodsForm" :rules="editGoodsFormRules" ref="editGoodsFormRef" label-width="70px">
        <el-form-item label="商品名" prop="goods_name">
          <el-input v-model="editGoodsForm.goods_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editGoodsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editGoodsInfo">確 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查詢參數對象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表
      goodslist: [],
      // 總數據條數
      total: 0,
      editGoodsDialogVisible: false,
      // 查詢到的商品信息對象
      editGoodsForm: {},
      editGoodsFormRules: {
        goods_name: [
          { required: true, message: '請輸入商品名稱', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('獲取商品列表失敗')
      }
      this.$message.success('獲取商品列表成功')
      // console.log(res.data)
      this.goodslist = res.data.goods
      this.total = res.data.total
    },
    // 監聽pagesize改變的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 監聽頁碼值改變的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    async removeById (id) {
      const confirmResult = await this.$confirm('此操作將永久删除該商品, 是否繼續?', '提示', {
        confirmButtonText: '確定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消刪除')
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('刪除失敗')
      }
      this.$message.success('刪除成功')
      this.getGoodsList()
    },
    goAddpage () {
      this.$router.push('/goods/add')
    },
    // 展示編輯商品的對話框
    async showEditGoodsDialog (id) {
      const { data: res } = await this.$http.get('goods/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查詢商品信息失敗!')
      }
      this.editGoodsForm = res.data
      console.log(res.data)
      this.editGoodsDialogVisible = true
    },
    // 監聽修改商品對話框的關閉事件
    editGoodsDialogClosed () {
      this.$refs.editGoodsFormRef.resetFields()
    },
    // 寫不出來
    // 修改商品信息並提交
    editGoodsInfo () {
      this.$refs.editGoodsFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 發起修改商品信息的數據請求
        const { data: res } = await this.$http.put('goods/' + this.editGoodsForm.goods_id,
          {
            goods_name: this.editGoodsForm.goods_name,
            goods_price: this.editGoodsForm.goods_price,
            goods_number: this.editGoodsForm.goods_number
          })
        if (res.meta.status !== 201) {
          console.log(this.editGoodsForm.goods_id)
          console.log(res.meta.status)
          console.log(res.meta.msg)
          console.log(res.data)
          return this.$message.error('更新商品信息失敗!')
        }
        // 關閉對話框
        this.editGoodsDialogVisible = false
        // 刷新數據列表
        this.getGoodsList()
        // 提示修改成功
        this.$message.success('更新商品信息成功!')
      })
    }
  }
}
</script>

<style lang="less" scoped>

</style>
