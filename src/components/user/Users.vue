<template>
  <div>
  <!-- 麵包屑導航區 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用戶管理</el-breadcrumb-item>
      <el-breadcrumb-item>用戶列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片視圖區域 -->
    <el-card>
      <!-- 搜索與添加區域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="請输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用戶</el-button>
        </el-col>
      </el-row>
      <!-- 用戶列表區域 -->
      <el-table :data="userlist" border>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="郵箱" prop="email"></el-table-column>
        <el-table-column label="電話" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="狀態">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!-- 修改按鈕 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <!-- 刪除按鈕 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
            <!-- 修改角色按鈕 -->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分頁區域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用戶的對話框 -->
    <el-dialog
      title="添加用戶"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 內容主體區域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用戶名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密碼" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="郵箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手機" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部區域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用戶的對話框 -->
    <el-dialog
      title="修改用戶"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用戶名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="郵箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手機號" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    // 驗證郵箱的規則
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        // 合法的郵箱
        return cb()
      }
      cb(new Error('請輸入合法的郵箱'))
    }
    // 驗證手機號的規則
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        // 合法的手機號
        return cb()
      }
      cb(new Error('請輸入合法的手機號'))
    }
    return {
      // 獲取用戶列表的參數對象
      queryInfo: {
        query: '',
        // 當前的頁數
        pagenum: 1,
        // 當前每頁顯示多少條數據
        pagesize: 2
      },
      userlist: [],
      total: 0,
      // 控制添加用戶對話框的顯示與隱藏
      addDialogVisible: false,
      // 添加用戶的表單數據
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表單的驗證規則對象
      addFormRules: {
        username: [
          { required: true, message: '請輸入用戶名', trigger: 'blur' },
          { min: 3, max: 10, message: '用戶名的長度在3~10個字符之間', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '請輸入用戶密碼', trigger: 'blur' },
          { min: 6, max: 15, message: '用戶名的長度在6~15個字符之間', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '請輸入用戶郵箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '請輸入用戶手機', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制修改用戶對話框的顯示與隱藏
      editDialogVisible: false,
      // 查詢到的用戶信息對象
      editForm: {},
      // 修改表單的驗證規則對象
      editFormRules: {
        email: [
          { required: true, message: '請輸入用戶郵箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '請輸入用戶手機', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('獲取用戶列表失敗! ')
      }
      this.userlist = res.data.users
      this.total = res.data.total
    },
    // 監聽pagesize改變的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 監聽頁碼值改變的事件
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 監聽switch開關狀態的改變
    async userStateChanged (userInfo) {
      // console.log(userInfo)
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用戶狀態失敗')
      }
      this.$message.success('更新用戶狀態成功!')
    },
    // 監聽添加用戶對話框的關閉事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 點擊按鈕添加新用戶
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 可以發起添加用戶的網路請求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用戶失敗!')
        }
        this.$message.success('添加用戶成功!')
        // 隱藏添加的對話框
        this.addDialogVisible = false
        // 重新獲取用戶列表數據
        this.getUserList()
      })
    },
    // 展示編輯用戶的對話框
    async showEditDialog (id) {
      // console.log(id)
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查詢用戶信息失敗!')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 監聽修改用戶對話框的關閉事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用戶信息並提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 發起修改用戶信息的數據請求
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        if (res.meta.status !== 200) {
          this.$message.error('更新用戶信息失敗!')
        }
        // 關閉對話框
        this.editDialogVisible = false
        // 刷新數據列表
        this.getUserList()
        // 提示修改成功
        this.$message.success('更新用戶信息成功!')
      })
    },
    // 根據id刪除對應的用戶信息
    async removeUserById (id) {
      // 彈窗提問用戶是否刪除數據
      const confirmResult = await this.$confirm('此操作將永久删除该用戶, 是否繼續?', '提示', {
        confirmButtonText: '確定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(confirmResult)
      // confirm確認刪除cancel取消刪除
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消刪除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('用戶刪除失敗!')
      }
      this.$message.success('用戶成功刪除!')
      this.getUserList()
    }
  }
}
</script>

<style lang="less" scoped></style>
