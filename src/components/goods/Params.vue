/* eslint-disable camelcase */
<template>
  <div>
    <!-- 麵包屑導航區 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>參數列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片視圖區域 -->
    <el-card>
      <!-- 頭部警告區域 -->
      <el-alert
        title="注意: 只允許為第三級分類設置相關參數!!"
        type="warning"
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 選擇商品分類區域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>選擇商品分類: </span>
          <!-- 選擇商品分類的級聯選擇框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="{
              expandTrigger: 'hover',
              value: 'cat_id',
              label: 'cat_name',
              children: 'children'
            }"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab頁籤區域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!--添加動態參數的面板 -->
        <el-tab-pane label="動態參數" name="many">
          <el-button type="primary"
          size="mini"
          :disabled="isBtnDisabled"
          @click="addDialogVisible=true">添加動態參數</el-button>
          <!-- 動態參數表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展開行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循環渲染tag標籤 -->
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)">{{item}}</el-tag>
                <!-- 輸入的文本框 -->
                <el-input
                class="input-new-tag"
                v-if="scope.row.inputVisible"
                v-model="scope.row.inputValue"
                ref="saveTagInput"
                size="small"
                @keyup.enter.native="handleInputConfirm(scope.row)"
                @blur="handleInputConfirm(scope.row)"
              >
              </el-input>
              <!-- 添加按鈕 -->
              <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="參數名稱" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">編輯</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">刪除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!--添加靜態屬性的面板 -->
        <el-tab-pane label="靜態屬性" name="only">
          <el-button type="primary"
          size="mini"
          :disabled="isBtnDisabled"
          @click="addDialogVisible=true"
          >添加靜態屬性</el-button>
          <!-- 靜態屬性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展開行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循環渲染tag標籤 -->
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)">{{item}}</el-tag>
                <!-- 輸入的文本框 -->
                <el-input
                class="input-new-tag"
                v-if="scope.row.inputVisible"
                v-model="scope.row.inputValue"
                ref="saveTagInput"
                size="small"
                @keyup.enter.native="handleInputConfirm(scope.row)"
                @blur="handleInputConfirm(scope.row)"
              >
              </el-input>
              <!-- 添加按鈕 -->
              <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="屬性名稱" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">編輯</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">刪除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加參數的對話框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @closed="addDialogClosed">
      <!-- 添加參數的對話框 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">確 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改參數的對話框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @closed="editDialogClosed">
      <!-- 添加參數的對話框 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">確 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分類列表
      catelist: [],
      // 級聯選擇框雙向綁定到的數組
      selectedCateKeys: [],
      // 被激活的頁籤名稱
      activeName: 'many',
      // 動態參數的數據
      manyTableData: [],
      // 靜態屬性的數據
      onlyTableData: [],
      // 控制添加對話框的顯示與隱藏
      addDialogVisible: false,
      // 添加參數的表單數據對象
      addForm: {
        attr_name: ''
      },
      // 添加表單的驗證規則對象
      addFormRules: {
        attr_name: [
          { required: true, message: '請輸入參數名稱', trigger: 'blur' }
        ]
      },
      // 控制修該對話框的顯示與隱藏
      editDialogVisible: false,
      // 修改的表單數據對象
      editForm: {},
      // 修改表單的驗證規則對象
      editFormRules: {
        attr_name: [
          { required: true, message: '請輸入參數名稱', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('獲取商品分類失敗')
      }
      this.$message.success('獲取商品分類成功')
      this.catelist = res.data
      // console.log(this.catelist)
    },
    // 級聯選擇框選中項變化,會觸發這個函數
    handleChange() {
      // 20-day04-19的視頻有介紹,好像不需要做
      this.getParamsData()
    },
    // tab 頁籤點擊事件的處理函數
    handleTabClick() {
      this.getParamsData()
      // console.log(this.activeName)
    },
    // 獲取參數的列表數據
    async getParamsData () {
      // 證明選中的不是三級分類
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // console.log(this.selectedCateKeys)
      // 根據所選的分類ID,和當前所屬的面板,獲取對應的參數
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('獲取參數列表失敗')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的顯示與隱藏
        item.inputVisible = false
        // 文本框中輸入的值
        item.inputValue = ''
      })
      // console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 監聽添加對話框的關閉事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 點擊按鈕,添加參數
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加參數失敗!')
        }
        this.$message.success('添加參數成功!')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 點擊按鈕,展示修改的畫框
    // eslint-disable-next-line camelcase
    async showEditDialog (attr_id) {
      // 查詢當前參數的信息
      // eslint-disable-next-line camelcase
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, {
        params: { attr_sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('獲取參數信息失敗!')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 重製修改的表單
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 點擊按鈕,修改參數信息
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          })
        if (res.meta.status !== 200) {
          return this.$message.error('修改參數失敗!')
        }
        this.$message.success('修改參數成功!')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 根據id刪除對應的操作項
    // eslint-disable-next-line camelcase
    async removeParams (attr_id) {
      const confirmResult = await this.$confirm('此操作將永久删除該參數, 是否繼續?', '提示', {
        confirmButtonText: '確定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 用戶取消了刪除操作
      if (confirmResult !== 'confirm') {
        return this.$message.info('以取消刪除')
      }
      // 刪除的業務邏輯
      // eslint-disable-next-line camelcase
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('刪除參數失敗')
      }
      this.$message.success('刪除參數成功')
      this.getParamsData()
    },
    // 文本框失去焦點或按下了enter鍵都會觸發
    async handleInputConfirm (row) {
      // console.log('ok')
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        // eslint-disable-next-line no-useless-return
        return
      }
      // 如果沒有return,則證明輸入的內容,需要做後續處理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 需要發起請求,儲存這次操作
      this.saveAttrVals(row)
    },
    // 將對attr_vals的操作,保存到數據庫
    async saveAttrVals (row) {
      // 需要發起請求,儲存這次操作
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改參數項失敗')
      }
      this.$message.success('修改參數項成功')
    },
    // 點擊按鈕展示文本輸入框
    showInput (row) {
      row.inputVisible = true
      // 讓文本框自動獲得焦點
      // $nextTick方法的作用,就是當頁面上元素被重新渲染之後,才會指定回掉函數中的代碼
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 刪除對應的參數可選項
    handleClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      } else {
        return false
      }
    },
    // 當前選中的三級分類Id
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      } else {
        return null
      }
    },
    // 動態計算標題的文本
    titleText () {
      if (this.activeName === 'many') {
        return '動態參數'
      } else {
        return '靜態屬性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
