<template>
  <div class="login-container">
    <el-row>
      <el-col :span="24">
        <el-form
          :model="ruleForm"
          :rules="rules"
          ref="ruleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item prop="password">
            <el-input
              prefix-icon="el-icon-lock"
              :clearable="true"
              placeholder="请输入密码"
              v-model="ruleForm.password"
            ></el-input>
          </el-form-item>
          <el-form-item>
            <el-button
              type="primary"
              :loading="btnLoading"
              @click="submitForm('ruleForm')"
            >{{ btnText }}</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "login",
  created () {
    sessionStorage.removeItem('auth')
  },
  data() {
    return {
      ruleForm: {
        password: ""
      },
      rules: {
        password: [{ required: true, message: "请输入密码", trigger: "blur" }]
      },
      btnLoading: false,
      btnText: "登录"
    };
  },
  methods: {
    submitForm(formName) {
      this.btnText = "登录中";
      this.btnLoading = true;
      this.$refs[formName].validate(valid => {
        if (valid) {
          axios
            .get("https://apiv1.wdnm.icu/av/key.php", {
              params: {
                key: this.ruleForm.password
              }
            })
            .then(res => {
              if (res.data.code == "200") {
                this.$notify({
                  title: "登录成功",
                  message: "密码正确",
                  type: "success",
                  position: "bottom-left",
                  duration: 2000
                });
                let auth = {
                  pwd: this.ruleForm.password,
                  key: res.data.key
                }
                sessionStorage.setItem('auth', JSON.stringify(auth))

                this.$router.push("/home");
              } else {
                this.btnText = "登录";
                this.btnLoading = false;
                this.$notify.error({
                  title: "密码错误",
                  message: "请检查密码",
                  position: "bottom-left",
                  duration: 2000
                });
                this.ruleForm.password = "";
              }
            });
        } else {
          this.btnText = "登录";
          this.btnLoading = false;
          return false;
        }
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.login-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  /deep/ .el-form-item__content {
    margin-left: 0px !important;
  }
}
</style>