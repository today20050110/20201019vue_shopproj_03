<template>
  <div>
  <!-- 麵包屑導航區 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分類</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片視圖區域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分類</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
      :data="catelist"
      :columns="columns"
      border show-index
      index-text="#"
      :selection-type="false"
      :expand-type="false"
      :show-row-hover="false"
      class="treetable">
      <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一級</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二級</el-tag>
          <el-tag type="warning" size="mini" v-else>三級</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCateDialog(scope.row.cat_id)">編輯</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)">刪除</el-button>
        </template>
      </tree-table>
      <!-- 分頁區域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分類的對話框 -->
    <el-dialog
      title="添加分類"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed">
      <!-- 添加分類的表單 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分類名稱:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父級分類:">
          <!-- options用來選擇數據源 -->
          <!-- props用來指定配置對象 (不一樣) -->
          <!-- checkStrictly可以單獨選中父級 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{ checkStrictly: true, expandTrigger: 'hover', value: 'cat_id', label: 'cat_name', children: 'children' }"
            @change="parentCateChanged"
            clearable
            ref="elcascaderRefs">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">確 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改分類的對話框 -->
    <el-dialog
      title="修改分類"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editCateDialogClosed">
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="70px">
        <el-form-item label="分類名" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCateInfo">確 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查詢條件
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分類的數據列表,默認為空
      catelist: [],
      // 總數據條數
      total: 0,
      // 為table指定列的定義
      columns: [
        {
          label: '分類名稱',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 將當前列定義為模板列
          type: 'template',
          // 表示當前這一列使用模板名稱
          template: 'isok'
        },
        {
          label: '排序',
          // 將當前列定義為模板列
          type: 'template',
          // 表示當前這一列使用模板名稱
          template: 'order'
        },
        {
          label: '操作',
          // 將當前列定義為模板列
          type: 'template',
          // 表示當前這一列使用模板名稱
          template: 'opt'
        }
      ],
      // 控制添加分類對話框的顯示與隱藏
      addCateDialogVisible: false,
      // 添加分類的表單數據對象
      addCateForm: {
        // 將要添加的分類的名稱
        cat_name: '',
        // 父級分類的id
        cat_pid: 0,
        // 分類的等級,默認要添加的是一級分類
        cat_level: 0
      },
      // 添加分類表單的驗證規則對象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '請輸入分類名稱', trigger: 'blur' }
        ]
      },
      // 父級分類的列表
      parentCateList: [],
      // 選中的父級分類的id數組
      selectedKeys: [],
      // 查詢到的分類信息對象
      editCateForm: {},
      // 控制修改分類對話框的顯示與隱藏
      editCateDialogVisible: false,
      editCateFormRules: {
        cat_name: [
          { required: true, message: '請輸入分類名稱', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 獲取商品分類數據
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.querInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('獲取商品分類失敗!')
      }
      // console.log(res.data)
      // 把數據列表,賦值給catelist
      this.catelist = res.data.result
      // 為總數據條數賦值
      this.total = res.data.total
    },
    // 監聽pagesize改變
    handleSizeChange (newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    // 監聽pagenum改變
    handleCurrentChange (newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 點擊按鈕,展示添加分類的對話框
    showAddCateDialog () {
      // 先獲取父級分類的數據列表
      this.getParentCateList()
      // 再展示出對話框
      this.addCateDialogVisible = true
    },
    // 獲取父級分類的數據列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('獲取父級分類數據失敗')
      }
      // console.log(res.data)
      this.parentCateList = res.data
    },
    // 選擇項發生變化觸發這個函數
    parentCateChanged () {
      // console.log(this.selectedKeys)
      this.$refs.elcascaderRefs.dropDownVisible = false
      // 如果selectedKeys數組中的length大於0,證明選中的父級分類
      // 反之,就說明沒有選中任何父級分類
      if (this.selectedKeys.length > 0) {
        // 父級分類的id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 為當前分類的等級賦值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父級分類的id
        this.addCateForm.cat_pid = 0
        // 為當前分類的等級賦值
        this.addCateForm.cat_level = 0
      }
    },
    // 點擊按鈕添加分類
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分類失敗')
        }
        this.$message.success('添加分類成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 監聽對話框的關閉事件,重置表單數據
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 展示編輯分類的對話框
    async showEditCateDialog (id) {
      // console.log(id)
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查詢分類信息失敗!')
      }
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    // 監聽修改分類對話框的關閉事件
    editCateDialogClosed () {
      this.$refs.editCateFormRef.resetFields()
    },
    // 修改分類信息並提交
    editCateInfo () {
      this.$refs.editCateFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 發起修改用戶信息的數據請求
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, { cat_name: this.editCateForm.cat_name })
        if (res.meta.status !== 200) {
          return this.$message.error('更新分類信息失敗!')
        }
        // 關閉對話框
        this.editCateDialogVisible = false
        // 刷新數據列表
        this.getCateList()
        // 提示修改成功
        this.$message.success('更新分類信息成功!')
      })
    },
    // 根據id刪除對應的分類信息
    async removeCateById (id) {
      // 彈窗提問用戶是否刪除數據
      const confirmResult = await this.$confirm('此操作將永久删除该分類, 是否繼續?', '提示', {
        confirmButtonText: '確定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(confirmResult)
      // confirm確認刪除cancel取消刪除
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消刪除')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('分類刪除失敗!')
      }
      this.$message.success('分類成功刪除!')
      this.getCateList()
      // console.log(opt)
    }
  }
}
</script>

<style lang="less" scoped>
.treetable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
