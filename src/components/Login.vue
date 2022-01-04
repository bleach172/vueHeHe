<template>
  <div class="login_container">
    <div class="login_box">
      <h1 class="login_title">vue</h1>
      <el-form
        :model="loginForm"
        :rules="loginFormRules"
        ref="ruleForm"
        label-width="auto"
        class="login_form"
        hide-required-asterisk
      >
        <el-form-item label="账号" prop="username">
          <el-input
            v-model="loginForm.username"
            prefix-icon="iconfont icon-peoplefill"
          ></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="iconfont icon-mima"
            type="password"
          ></el-input>
        </el-form-item>
        <el-row class="btn">
          <el-button type="primary" @click="login" round>登录</el-button>
          <el-button type="info" @click="resetForm">重置</el-button>
        </el-row>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          { required: true, message: '请输入账号', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetForm () {
      this.$refs.ruleForm.resetFields()
    },
    login () {
      this.$refs.ruleForm.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background: #f9f9f9;
  height: 100%;
  width: 100%;
}
.login_title {
  text-align: center;
  margin-top: 40px;
}
.login_box {
  width: 540px;
  height: 400px;
  background: white;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.login_form {
  bottom: 40px;
  position: absolute;
  width: 100%;
  padding: 0 40px;
  box-sizing: border-box;
}
.btn {
  padding-top: 40px;
  display: flex;
  justify-content: space-between;
  // width: 360px;
  margin: 0 auto;
}
.btn .el-button:first-child {
  width: 70%;
}
.btn .el-button:last-child {
  width: 20%;
  background: transparent;
  border: none;
  color: #409eff;
}
</style>
