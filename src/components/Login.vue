<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区 -->
      <div class="avatar_box">
        <img src="http://www.sineava.top/img/logo.47ae5af0.png" alt="avatar" />
      </div>
      <!-- 登录表单 -->
      <div>
        <el-form
          status-icon
          ref="loginFormRef"
          :model="loginForm"
          label-width="60px"
          class="login_form"
        >
        <h3 class="title">网上书店管理系统</h3>
          <el-form-item label="邮箱" prop="email"
          :rules="[
            { required: true, message: '请输入邮箱地址', trigger: 'blur' },
            { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
          ]">
            <el-input v-model="loginForm.email" prefix-icon="iconfont icon-user"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password"
          :rules="[
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 6, max: 18, message: '长度在 6 到 18 个字符', trigger: ['blur', 'change'] }
          ]">
            <el-input
              v-model="loginForm.password"
              type="password"
              prefix-icon="iconfont icon-3702mima"
            ></el-input>
          </el-form-item>
          <el-form-item class="btns">
            <el-button type="primary" @click="login">登录</el-button>
            <el-button type="info" @click="resetLoginForm">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        email: '',
        password: ''
      }
      // 表单验证
      // loginFormRules: {
      //   email: [
      //     { required: true, message: '请输入用户名', trigger: 'blur' },
      //     { min: 2, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }
      //   ],
      //   password: [
      //     { required: true, message: '请输入用户密码', trigger: 'blur' },
      //     { min: 3, max: 18, message: '长度在 6 到 18 个字符', trigger: 'blur' }
      //   ]
      // }
    }
  },
  methods: {
    // 表单重置按钮
    resetLoginForm () {
      // console.log(this)
      // resetFields：element-ui提供的表单方法
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      // 表单预验证
      // valid：bool类型
      this.$refs.loginFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return false
        // 返回值为promise，可加await简化操作 相应的也要加async
        const { data: res } = await this.$http.get('/employee/employeeLogin/' + this.loginForm.email + '/' + this.loginForm.password)
        // console.log(res)
        this.curuse.curdata = res
        console.log(this.curuse.curdata)
        if (res.code === 0) return this.$message.error(res.msg)
        this.$message.success('登录成功')
        // 1、将登陆成功之后的token, 保存到客户端的sessionStorage中; localStorage中是持久化的保存
        //   1.1 项目中出现了登录之外的其他API接口，必须在登陆之后才能访问
        //   1.2 token 只应在当前网站打开期间生效，所以将token保存在sessionStorage中
        window.sessionStorage.setItem('token', res.data.isAdmin)
        // 2、通过编程式导航跳转到后台主页, 路由地址为：/home
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
/* // lang="less" 支持less格式
// scoped vue的指令，只在当前组件生效 */
.login_container {
  background-image: url("https://img01.sogoucdn.com/app/a/100520146/9fbb9927723f155c01b666733e483292");
  align-items: center;
  background-size: cover;
  height: 100%;
}
.title{
  text-align: center;
  color: #707070;
  margin: auto auto;
  padding: 10px 0 10px 0;
}
.login_box {
  width: 450px;
  height: 360px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  -webkit-transform: translate(-50%, -50%);
  background-color: #fff;

  .avatar_box {
    width: 130px;
    height: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}
.login_form {
  position: absolute;
  bottom: 50px;
  width: 100%;
  padding: 0px 20px 0px 10px;
  box-sizing: border-box;
}
.btns {
  display: flex;
  justify-content: center;
}
.info {
  font-size: 13px;
  margin: 10px 15px;
}
</style>
