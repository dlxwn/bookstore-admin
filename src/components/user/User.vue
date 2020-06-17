<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索 添加 -->
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getListUserByFuzzy"></el-button>
          </el-input>
        </el-col>
        <el-col :span="1.5">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
        <el-col :span="1.5">
          <el-button type="primary" @click="exportUser">导出用户<i class="el-icon-download el-icon--right"></i></el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist" border stripe>
        <!-- stripe: 斑马条纹
        border：边框-->
        <el-table-column prop="userId" label="#"></el-table-column>
        <el-table-column prop="nickName" label="昵称"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="phoneNumber" label="电话"></el-table-column>
        <el-table-column prop="sex" label="性别"></el-table-column>
        <el-table-column prop="address" label="地址"></el-table-column>
        <el-table-column prop="userLevel" label="用户等级"></el-table-column>
        <el-table-column prop="birthday" label="生日"></el-table-column>
        <el-table-column prop="avatar" label="头像地址"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              circle
              @click="showEditDialog(scope.row.userId)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              circle
              @click="removeUserById(scope.row.userId)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="totle"
      ></el-pagination>
    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <!-- 内容主体 -->
      <el-form
        :model="addUserForm"
        ref="addUserFormRef"
        :rules="addUserFormRules"
        label-width="100px"
      >
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addUserForm.email"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="userPassword">
          <el-input v-model="addUserForm.userPassword"></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input v-model="addUserForm.name"></el-input>
        </el-form-item>
        <el-form-item label="性别" prop="sex">
          <el-radio-group size="medium">
            <el-radio border v-model="addUserForm.sex"  label="男"></el-radio>
            <el-radio border v-model="addUserForm.sex"  label="女"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="生日" prop="birthday">
          <div class="block">
            <el-date-picker
              v-model="addUserForm.birthday"
              type="date"
              format="yyyy 年 MM 月 dd 日"
              value-format="yyyy-MM-dd">
              placeholder="选择日期">
            </el-date-picker>
          </div>
        </el-form-item>
        <el-form-item label="手机" prop="phoneNumber">
          <el-input v-model="addUserForm.phoneNumber"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="nickName">
          <el-input v-model="addUserForm.nickName"></el-input>
        </el-form-item>
        <el-form-item label="地址" prop="address">
          <el-input v-model="addUserForm.address"></el-input>
        </el-form-item>
        <el-form-item label="头像URL" prop="avatar">
          <el-input v-model="addUserForm.avatar"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户的对话框 -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        :model="editUserForm"
        ref="editUserFormRef"
        :rules="editUserFormRules"
        label-width="70px"
      >
        <el-form-item label="编号">
          <el-input v-model="editUserForm.userId" disabled></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input v-model="editUserForm.name"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="nickName">
          <el-input v-model="editUserForm.nickName"></el-input>
        </el-form-item>
        <el-form-item label="性别">
          <el-select v-model="editUserForm.sex" placeholder="性别">
            <el-option label="男" value="男"></el-option>
            <el-option label="女" value="女"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editUserForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="phoneNumber">
          <el-input v-model="editUserForm.phoneNumber"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="userPassword">
          <el-input v-model="editUserForm.userPassword"></el-input>
        </el-form-item>
        <el-form-item label="地址" prop="address">
          <el-input v-model="editUserForm.address"></el-input>
        </el-form-item>
        <el-form-item label="头像URL" prop="avatar">
          <el-input v-model="editUserForm.avatar"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    // 自定义邮箱规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        // 合法邮箱
        return callback()
      }
      callback(new Error('请输入合法邮箱'))
    }

    // 自定义手机号规则
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return callback()
      }
      // 返回一个错误提示
      callback(new Error('请输入合法的手机号码'))
    }

    return {
      // 获取用户列表查询参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页显示多少数据
        pagesize: 5
      },
      userlist: [],
      totle: 0,
      // 添加用户对话框
      addDialogVisible: false,
      // 用户添加
      addUserForm: {
        name: '',
        userPassword: '',
        email: '',
        phoneNumber: '',
        sex: '',
        nickName: '',
        avatar: '',
        address: '',
        birthday: '',
        userLevel: 1,
        integral: 0
      },
      // 用户添加表单验证规则
      addUserFormRules: {
        name: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '用户名的长度在2～10个字',
            trigger: 'blur'
          }
        ],
        nickName: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '用户名的长度在2～10个字',
            trigger: 'blur'
          }
        ],
        userPassword: [
          { required: true, message: '请输入用户密码', trigger: 'blur' },
          {
            min: 4,
            max: 16,
            message: '用户密码的长度在4～16个字',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        phoneNumber: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 修改用户
      editDialogVisible: false,
      editUserForm: {},
      // 编辑用户表单验证
      editUserFormRules: {
        phoneNumber: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ],
        userPassword: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          {
            min: 4,
            max: 16,
            message: '用户密码的长度在4～16个字',
            trigger: 'blur'
          }
        ],
        nickName: [
          { required: true, message: '昵称不能为空', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '昵称的长度在2～10个字',
            trigger: 'blur'
          }
        ],
        name: [
          { required: true, message: '姓名不能为空', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '姓名的长度在2～10个字',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ]
      },
      exportUserURL: 'http://localhost:8088/bookstore/user/export'
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('/user/findAll/' + this.queryInfo.pagenum + '/' + this.queryInfo.pagesize)
      if (res == null) {
        return this.$message.error('获取用户列表失败！')
      }
      console.log(res)
      this.userlist = res.records
      this.totle = res.total
    },
    async getListUserByFuzzy () {
      const { data: res } = await this.$http.get('/user/findList/' + this.queryInfo.query + '/' + this.queryInfo.pagenum + '/' + this.queryInfo.pagesize)
      if (res == null) {
        return this.$message.error('获取用户列表失败！')
      }
      console.log(res)
      this.userlist = res.records
      this.totle = res.total
    },
    async exportUser () {
      window.location.href = this.exportUserURL
    },
    // 监听 pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagenum = newSize
      this.getUserList()
    },
    // 监听 添加用户对话框的关闭事件
    addDialogClosed () {
      this.$refs.addUserFormRef.resetFields()
    },
    // 添加用户
    addUser () {
      // 提交请求前，表单预验证
      this.$refs.addUserFormRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        console.log(this.addUserForm)
        const { data: res } = await this.$http.post('/user/add', this.addUserForm)
        if (res !== true) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        // 隐藏添加用户对话框
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    // 编辑用户信息
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('/user/find/' + id)
      if (res == null) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editUserForm = res
      this.editDialogVisible = true
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed () {
      this.$refs.editUserFormRef.resetFields()
    },
    // 修改用户信息
    editUser () {
      // 提交请求前，表单预验证
      this.$refs.editUserFormRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        const { data: res } = await this.$http.put(
          'user/edit',
          {
            userId: this.editUserForm.userId,
            email: this.editUserForm.email,
            phoneNumber: this.editUserForm.phoneNumber,
            nickName: this.editUserForm.nickName,
            userPassword: this.editUserForm.userPassword,
            name: this.editUserForm.name,
            address: this.editUserForm.address,
            userLevel: this.editUserForm.userLevel,
            integral: this.editUserForm.integral,
            avatar: this.editUserForm.avatar,
            sex: this.editUserForm.sex
          }
        )
        if (res !== true) {
          this.$message.error('更新用户信息失败！')
        }
        // 隐藏添加用户对话框
        this.editDialogVisible = false
        this.$message.success('更新用户信息成功！')
        this.getUserList()
      })
    },
    // 删除用户
    async removeUserById (id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 点击确定 返回值为：confirm
      // 点击取消 返回值为： cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('/user/del/' + id)
      if (res !== true) return this.$message.error('删除用户失败！')
      this.$message.success('删除用户成功！')
      this.getUserList()
    }
  }
}
</script>

<style lang="less" scoped>
</style>
