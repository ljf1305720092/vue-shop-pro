<template>
  <div id="login-container">
    <div class="login-box">
      <div class="avatar-box">
        <img src="../assets/img/logo.png" alt>
      </div>

      <el-form :rules="loginFormRules" ref="loginFormRef" :model="loginFrom">
        <el-form-item prop="username">
          <el-input v-model="loginFrom.username">
            <i slot="prefix" class="iconfont icon-user"></i>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input type="password" v-model="loginFrom.password">
            <i slot="prefix" class="iconfont icon-3702mima"></i>
          </el-input>
        </el-form-item>
        <el-row>
          <el-col :offset="15">
            <el-button type="primary" @click="login">登陆</el-button>
            <el-button type="info" @click="resetForm">重置</el-button>
          </el-col>
        </el-row>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 登陆from表单需要的数据对象
      loginFrom: {
        username: '',
        password: ''
      },
      // 给各个表单域 定义效验规则
      loginFormRules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入用户密码', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    login() {
      this.$refs.loginFormRef.validate(async valid => {
        if (valid === true) {
          // 用户信息真实性效验
          const { data: res } = await this.$http.post('/login', this.loginFrom)
          if (res.meta.status !== 200) {
            return this.$message.error('用户名或密码错误')
          }
          // 记录服务器返回的token信息
          window.sessionStorage.setItem('token', res.data.token)
          // 页面重定向到后台首页(home)
          this.$router.push('/home')
        }
      })
    },
    resetForm() {
      this.$refs.loginFormRef.resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
#login-container {
  background: #2b4b6b;
  height: 100%;
  overflow: hidden;
  .login-box {
    width: 450px;
    height: 304px;
    border-radius: 4px;
    background-color: #fff;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    .el-form {
      position: absolute;
      bottom: 0px;
      width: 100%;
      padding: 20px;
      box-sizing: border-box;
    }
    .avatar-box {
      width: 130px;
      height: 130px;
      border: 1px solid #eee;
      border-radius: 50%;
      padding: 8px;
      background: #fff;
      position: absolute;
      left: 50%;
      transform: translate(-50%, -50%);
      // box-sizing: border-box;
      box-shadow: 0 0 10px #eee;
      img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background: #eee;
      }
    }
  }
}
</style>
