
<template>
  <div>
    <!-- 麵包屑導航區 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>權限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片視圖 -->
    <el-card>
      <!-- 添加角色按鈕區域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表區域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展開列 -->'
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一級權限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二級和三級權限 -->
              <el-col :span="19">
                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>
              {{scope.row}}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名稱" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">編輯</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">刪除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配權限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色的對話框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 角色內容主體區域 -->
      <el-form :model="addRoleForm" :rules="addRoleRules" ref="addRoleFormRef" label-width="70px">
        <el-form-item label="角色名稱" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部區域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">確 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色的對話框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <el-form :model="editRoleForm" :rules="editRoleFormRules" ref="editRoleFormRef" label-width="70px">
        <el-form-item label="角色名稱" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRuleInfo">確 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配權限的對話框 -->
    <el-dialog
      title="分配權限"
      :visible.sync="seRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed">
      <!-- 樹形控件 -->
      <el-tree
      :data="rightsList"
      :props="treeProps"
      show-checkbox node-key="id"
      default-expand-all
      :default-checked-keys="defKeys"
      ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="seRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights" >確 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表數據
      roleList: [],
      // 控制添加角色對話框的顯示與隱藏
      addDialogVisible: false,
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色表單的驗證規則對象
      addRoleRules: {
        roleName: [
          { required: true, message: '請輸入角色名稱', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '請輸入角色描述', trigger: 'blur' }
        ]
      },
      // 控制修改角色對話框的顯示與隱藏
      editDialogVisible: false,
      // 查詢到的角色信息對象
      editRoleForm: {},
      editRoleFormRules: {
        roleName: [
          { required: true, message: '請輸入角色名稱', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '請輸入角色描述', trigger: 'blur' }
        ]
      },
      // 控制分配權限對話框的顯示與隱藏
      seRightDialogVisible: false,
      // 所有權限的數據
      rightsList: [],
      // 樹型控件的屬性綁定對象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默認選中的id值數組
      defKeys: [],
      // 當前即將分配權限的角色id
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 獲取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('獲取角色列表失敗')
      }
      this.roleList = res.data
      // console.log(this.roleList)
    },
    // 監聽添加角色對話框的關閉事件
    addDialogClosed () {
      this.$refs.addRoleFormRef.resetFields()
    },
    async addRole () {
      this.$refs.addRoleFormRef.validate(async valid => {
      // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 可以發起添加角色的網路請求
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色失敗!')
        }
        this.$message.success('添加角色成功!')
        // 隱藏添加的對話框
        this.addDialogVisible = false
        // 重新獲取角色列表數據
        this.getRolesList()
      })
    },
    // 展示編輯角色的對話框
    async showEditDialog (id) {
      // console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查詢角色信息失敗!')
      }
      this.editRoleForm = res.data
      this.editDialogVisible = true
    },
    // 監聽修改角色對話框的關閉事件
    editDialogClosed () {
      this.$refs.editRoleFormRef.resetFields()
    },
    // 修改角色信息並提交
    editRuleInfo () {
      this.$refs.editRoleFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 發起修改角色信息的數據請求
        const { data: res } = await this.$http.put('roles/' + this.editRoleForm.roleId, { roleName: this.editRoleForm.roleName, roleDesc: this.editRoleForm.roleDesc })
        if (res.meta.status !== 200) {
          this.$message.error('更新角色信息失敗!')
        }
        // 關閉對話框
        this.editDialogVisible = false
        // 刷新數據列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('更新角色信息成功!')
      })
    },
    // 根據id刪除對應的角色信息
    async removeRoleById (id) {
      // 彈窗提問用戶是否刪除數據
      const confirmResult = await this.$confirm('此操作將永久删除該角色, 是否繼續?', '提示', {
        confirmButtonText: '確定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(confirmResult)
      // confirm確認刪除cancel取消刪除
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消刪除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('角色刪除失敗!')
      }
      this.$message.success('角色成功刪除!')
      this.getRolesList()
    },
    // 根據ID刪除對應的權限
    async removeRightById(role, rightId) {
      // 彈框提示用戶是否要刪除
      const confirmResult = await this.$confirm('此操作將永久删除該文件, 是否繼續?', '提示', {
        confirmButtonText: '確定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了刪除!')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        this.$message.error('刪除權限失敗')
      }
      // // 刷新數據列表
      // this.getRolesList()
      // 防止頁面重新刷新
      role.children = res.data
    },
    // 展示分配權限的對話框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 獲取所有權限得數據
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('獲取權限數據失敗')
      }
      // 把獲取到的權限數據保存到data中
      this.rightsList = res.data
      // console.log(this.rightsList)
      // 遞歸獲取三級節點的id
      this.getLeafKeys(role, this.defKeys)
      this.seRightDialogVisible = true
    },
    // 通過遞歸形式,獲取角色下所有三級權限的id,並保存到defKeys數組中
    getLeafKeys (node, arr) {
      // 如果當前node節點,不包括children屬性,則是三級節點
      if (!node.children) {
        return arr.push(node.id)
      }
      // 遞歸
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 監聽分配權限對話框的關閉事件,避免數據殘留
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 點擊為角色分配權限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      // console.log(idStr)
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配權限失敗!')
      }
      this.$message.success('分配權限成功!')
      this.getRolesList()
      this.seRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom:  1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
