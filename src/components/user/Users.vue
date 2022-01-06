<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator>
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <!-- 添加用户 -->
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userList" style="width: 100%" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!-- 修改 -->
            <el-button type="primary" size="mini" @click="showEditDialog(scope.row.id)">
              <i class="el-icon-edit"></i>
            </el-button>
            <!-- 删除 -->
            <el-button type="danger" size="mini" @click="removeUserById(scope.row.id)">
              <i class="el-icon-delete"></i>
            </el-button>
            <!-- 分配角色 -->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" size="mini">
                <i class="el-icon-setting"></i>
              </el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
      <!-- 添加用户的对话框 -->
      <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
        <!-- 类容主体区域 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">提 交</el-button>
        </span>
      </el-dialog>
      <!-- 修改用户对话框 -->
      <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
          <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    // 定义邮箱验证规则
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
      if (regEmail.test(value)) {
        return cb()
      } else {
        cb(new Error('邮箱格式不正确'))
      }
    }
    // 定义手机号的验证规则
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(?:(?:\+|00)86)?1(?:(?:3[\d])|(?:4[5-79])|(?:5[0-35-9])|(?:6[5-7])|(?:7[0-8])|(?:8[\d])|(?:9[189]))\d{8}$/
      if (regMobile.test(value)) {
        return cb()
      } else {
        cb(new Error('手机号格式不正确'))
      }
    }
    return {
      // 添加用户的表单数据的验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 15, message: '用户名长度必须在 3 到 15 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '密码长度必须在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 修改用户的表单数据的验证规则
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 修改用户的查询数据
      editForm: {},
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      // 控制修改话框的显示与隐藏
      editDialogVisible: false,
      // 获取用户列表参数对象
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 2
      },
      userList: [],
      total: 0
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    // 根据id删除对应的用户
    async removeUserById (id) {
      // console.log(id)
      // 是否确认删除的提示框
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      // 确认删除返回confirm字符串
      // 取消都会报错,用.catch()捕获错误,返回cancel字符串
      const { data: res } = await this.$http.delete(`users/${id}`)
      if (res.meta.status !== 200) {
        this.$message.error('删除用户失败')
      }
      this.$message({
        type: 'success',
        message: '删除成功!'
      })
      this.getUserList()
    },
    // 修改用户的请求提交预验证
    editUser () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        // 可以发起添加用户的提交请求
        const { data: res } = await this.$http.put(`users/${this.editForm.id}`, this.editForm)
        if (res.meta.status !== 200) {
          this.$message.error('修改用户失败')
        }
        this.$message.success('修改用户成功')
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    // 监听修改用户对话框关闭事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 展示编辑用户对话框
    async showEditDialog (id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get(`users/${id}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        this.$message.error('用户信息查询失败')
      }
      this.$message.success('查询成功')
      this.editForm = res.data
    },
    // 添加用户的请求提交预验证
    addUser () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 可以发起添加用户的提交请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    // 监听添加用户对话框关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 请求用户数据
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.console.error('获取用户列表失败')
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 监听 pagesize
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听 swith 状态的改变
    async userStateChanged (userinfo) {
      // console.log(userinfo)
      const res = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.console.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    }
  }
}
</script>

<style>
</style>
