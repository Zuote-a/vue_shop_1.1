<template>
  <div class="login_container">
    <div class="login_back">
      <div class="login_box">
        <div class="login_blog">
          <h2>Shopping</h2>
        </div>
        <!--      用户名-->
        <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
          <el-form-item  prop="username">
            <el-input v-model="loginForm.username"
              placeholder="登录"
              prefix-icon="iconfont icon-user">
            </el-input>
          </el-form-item>

          <!--      密码-->

          <el-form-item prop="password">
            <el-input v-model="loginForm.password"
              placeholder="登录"
              prefix-icon="iconfont icon-lock_fill"
              type="password"
            ></el-input>
          </el-form-item>

          <!--      按钮-->

          <el-form-item class="btns">
            <el-button type="primary" @click="login">登录</el-button>
            <el-button type="primary" @click="resetLoginForm">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  methods: {
    login () {
      this.$refs.loginFormRef.validate(async (valid) => {
        if (valid) {
          const { data: res } = await this.$http.post('login', this.loginForm)
          if (res.meta.status !== 200) return this.$message.error('登录失败账号或密码错误')
          this.$message({
            message: '登陆成功',
            type: 'success'
          })
          // ① 通过 axios 调用登录验证接口
          // ② 登录成功之后保持用户 token 信息
          // ③ 跳转到项目主页

          // Promise {<pending>}
          //   [[Prototype]]: Promise
          //   [[PromiseState]]: "fulfilled"
          //   [[PromiseResult]]: Object
          //  使用 await简化Promise对象,await必须在async中使用
          // eslint-disable-next-line no-irregular-whitespace
          // {data: {…}, status: 200, statusText: 'OK', headers: {…}, config: {…}, …}
          // 其中只有data是服务器返回值
          // 可以这样接收const { data: res }也可以用拦截器
          // ###6.登录成功之后的操作
          // A.登录成功之后，需要将后台返回的token保存到sessionStorage中
          // 操作完毕之后，需要跳转到/home
          //   ```
          window.sessionStorage.setItem('token', res.data.token)
          await this.$router.push('/home')
        }
      })
    },
    resetLoginForm () {
      // $refs: {loginFormRef: VueComponent}
      // console.log(this)
      this.$refs.loginFormRef.resetFields()
    }
  },
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 表单验证
      loginFormRules: {
      //  验证用户名
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 15, message: '长度必须在 3 到 15 个字符', trigger: 'blur' }
        ],
        //  验证密码
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度必须在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  }
}
</script>

<style scoped>
.login_container{
  width: 100%;
  height: 100%;
  background-color: #0086b3;
}
.login_back{
  width: 100%;
  height: 600px;
  position: absolute;
  top: 110px;
  background-image: url("../assets/matter/matter03.jpg");
}
.login_box{
  width: 450px;
  height: 300px;
  background-color: #eeeeee;
  border-radius: 13px;
  border: 1px solid green;
  position: absolute;
  top: 50%;
  left: 0;
  transform: translate(50%,-50%);
}
.login_blog{
  position: absolute;
  left: 50%;
  top: 0;
  transform: translate(-50%,-100%);
  color: #f40;
}
.login_form{
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
  position: absolute;
  bottom: 50%;
  transform: translate(0,50%);
}
.btns{

}
</style>
