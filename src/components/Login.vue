<template>
  <div id="login">
    <el-form
      :model="userLoginForm"
      :rules="loginRules"
      status-icon
      ref="userLoginFormRef"
      label-position="left"
      label-width="0px"
      class="demo-ruleForm login-page"
    >
      <h3 class="title">系统登录</h3>
      <el-form-item prop="username">
        <el-input
          type="text"
          @keyup.enter.native="handleSubmit"
          v-model="userLoginForm.username"
          auto-complete="off"
          placeholder="用户名"
          prefix-icon="iconfont el-icon-user"
        ></el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          @keyup.enter.native="handleSubmit"
          type="password"
          v-model="userLoginForm.password"
          auto-complete="off"
          placeholder="密码"
          prefix-icon="el-icon-suitcase-1"
        ></el-input>
      </el-form-item>
    
  
    
      <el-checkbox v-model="checked" class="rememberme">记住密码</el-checkbox>
      <el-form-item style="width:100%;">
        <div style="float:right;">
          <el-button
          type="primary" 
            class="el-icon-mobile-phone"
            @click="handleSubmit"
            :loading="loading"
          >登录</el-button>
          <el-button class="el-icon-refresh" @click="resetForm">重置</el-button>
        </div>
      </el-form-item>
    </el-form>
    <!-- 验证码 -->
      <Vcode
      :imgs="[Img1, Img2,Img3,Img4,Img5,Img6,Img7]"
      :show="isShow"
      @success="success"
      @close="close"
      :canvasWidth="500"
      :canvasHeight="350"
    />

<el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="30%"
  >
  <span>《新冠-物资管理系统》 该项目后端采用SpringBoot，Shiro，通用Mapper开发API接口，
    前端使用当今较为流行的Vue.js。
  </span>
  <p>一个可供初学者接触的前后端分离项目</p>
  <p>项目开源：
    <ul>
    
      <li>后端项目： <el-link href='https://github.com/zykzhangyukang/Xinguan'  type="primary"> 厂库链接地址</el-link></li>
      <br>
      <li>前端项目： <el-link href='https://github.com/zykzhangyukang/Vue-Xinguan' type="primary">    厂库链接地址</el-link></li>
    </ul>
  </p>
  <p>项目不定期更新：因为我比较懒~，觉得不错的话，还望各位大佬给个Star哟~。</p>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">点个赞</el-button>
    <el-button type="primary" @click="dialogVisible = false">去登入</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
import Vcode from "vue-puzzle-vcode";
import Img1 from '../assets/b.jpg';
import Img2 from '../assets/c.jpg';
import Img3 from '../assets/a.jpg';
import Img4 from '../assets/d.jpg';
import Img5 from '../assets/e.jpg';
import Img6 from '../assets/f.jpg';
import Img7 from '../assets/g.jpg';

export default {
  data() {
    return {
      Img1,
      Img2,
      Img3,
      Img4,
      Img5,
      Img6,
      Img7,
      isShow:false,
      dialogVisible:true,
      imgCode: undefined,
      //表单用户登入数据
      loading: false,
      userLoginForm: {
        username: "系统测试人员",
        password: "123456",
      },
      checked: true,

      //验证规则
      loginRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 3, max: 12, message: "长度在 3 到 12 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入用户密码", trigger: "blur" },
          { min: 6, max: 15, message: "长度在 6 到 15 个字符", trigger: "blur" }
        ]
      }
    };
  },
  components:{
      Vcode
  },
  methods: {
    //登入提交
    handleSubmit: function() {
      this.$refs.userLoginFormRef.validate( valid => {
        if (!valid) {
          return;
        } else {
          this.isShow=true;
        }
      });
    },
    //重置表单
    resetForm: function() {
      this.$refs.userLoginFormRef.resetFields();
    },
    //验证成功
    async success(){
        this.loading = true;
          //发起登入请求
          const { data: res } = await this.$http.post(
            "user/login?username=" +
              this.userLoginForm.username +
              "&password=" +
              this.userLoginForm.password
          );
          console.log(res);
          if (res.code == 200) {
            this.$message({
              title: "登入成功",
              message: "欢迎你进入系统",
              type: "success"
            });
            //保存token
            localStorage.setItem("JWT_TOKEN", res.data);
            this.getUserInfo();
          } else {
            this.isShow=false;
            this.$message.error({
              title: "登入失败",
              message: res.msg,
              type: "error"
            });

          }
          this.loading = false;
    },

       /**
      获取用户信息
     */
    async getUserInfo() {
      const { data: res } = await this.$http.get("user/info");
      if (res.code !== 200)
        return this.$message.error("获取用户信息失败:" + res.msg);
      var urls = [];
      res.data.menus.forEach((item, index, array) => {
        //执行代码
        if (item != null && item != "") {
          urls.push(item.url);
        }
      });
      var m = urls.filter(s => {
        return s && s.trim();
      });
      this.userInfo=res.data;
      //保存用户
      this.$store.commit('setUserInfo',res.data);
      //跳转到home
      this.$router.push("/home");
    },
    close(){
        this.isShow = false;
    }
  },
  created() {
  }
};
</script>

<style scoped>
.login-container {
  width: 100%;
  height: 100%;
}

.login-page {
  -webkit-border-radius: 5px;
  border-radius: 5px;
  margin: 180px auto;
  width: 350px;
  padding: 40px 35px 15px;
  background: #fff;
  border: 1px solid #eaeaea;
  box-shadow: 0 0 5px #cac6c6;
}
label.el-checkbox.rememberme {
  margin: 0px 0px 15px;
  text-align: left;
}
</style>


