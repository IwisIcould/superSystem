<template>
    <div class="login">
        <!-- 登录表单容器 -->
        <div class="login-wrapper">
            <!-- 登录标题 -->
            <h1 class="title">
                <i class="el-icon-menu"></i>
                华联超市管理系统-登录
            </h1>

            <!-- 登录表单 -->
            <el-form :model="loginForm" status-icon :rules="rules" ref="loginForm" label-width="100px" class="demo-ruleForm">
                <!-- 账号 -->
                <el-form-item label="账号" prop="username">
                    <el-input type="text" v-model="loginForm.username" autocomplete="off"></el-input>
                </el-form-item>

                <!-- 密码 -->
                <el-form-item label="密码" prop="password">
                    <el-input type="text" v-model="loginForm.password" autocomplete="off"></el-input>
                </el-form-item>

                <!-- 确认密码 -->
                <el-form-item label="确认密码" prop="checkPwd">
                    <el-input type="text" v-model="loginForm.checkPwd" autocomplete="off" @keyup.enter.native="submitForm('loginForm')"></el-input>
                </el-form-item>

                <!-- 登录按钮&重置按钮 -->
                <el-form-item>
                    <el-button type="primary" @click="submitForm('loginForm')">登录</el-button>
                    <el-button @click="resetForm('loginForm')">重置</el-button>
                </el-form-item>

            </el-form>
        </div>


    </div>
</template>>

<script>
// 引入qs库 处理axios 的post方式的参数
import qs from 'qs';
    export default{
        data() {
            // 包含特殊字符的函数
            const checkSpecificKey = str => {
                var specialKey =
                    "[`~!#$^&*()=|{}':;',\\[\\].<>/?~！#￥……&*（）——|{}【】‘；：”“'。，、？]‘'";
                for (var i = 0; i < str.length; i++) {
                    if (specialKey.indexOf(str.substr(i, 1)) != -1) {
                        return false;
                    }
                }
                return true;
            };

            // 验证密码的函数
            const pass = (rule, value, callback) => {
                if (value === "") {
                    callback(new Error("请输入密码"));
                } else if (value.length < 3 || value.length > 6) {
                    callback(new Error("密码长度 3 - 6 位"));
                } else if (!checkSpecificKey(value)) {
                    callback(new Error("密码不能包含特殊字符"));
                } else {
                    if (this.loginForm.checkPwd !== "") {
                        // 如果确认密码不为空
                        this.$refs.loginForm.validateField("checkPwd"); // 调用确认密码的验证（一致性验证）
                    }
                    // 成功的回调
                    callback();
                }
            };

            // 确认密码的验证函数
            const checkPass = (rule, value, callback) => {
                // rule是传入的验证规则  value是用户输入的值 callback是一个回调函数
                if (value === "") {
                    // 如果等于空
                    callback(new Error("请再次输入密码")); // 输出提示
                } else if (value !== this.loginForm.password) {
                    // 如果确认密码 和 密码不同
                    callback(new Error("两次输入密码不一致"));
                }
                // 如果直接调用 不传入任何错误信息 就是成功 绿色的勾勾
                callback();
            };

            return {
                // 登录表单的数据
                loginForm: {
                    username: "",
                    password: "",
                    checkPwd: ""
                },
                // 验证的规则（一份数据）
                rules: {
                    // 验证用户名
                    username: [
                        // 非空验证
                        { required: true, message: "请输入账号", trigger: "blur" },
                        { min: 3, max: 5, message: "账号长度在 3 - 5 位", trigger: "blur" }
                    ],
                    // 验证密码
                    password: [
                        // 非空验证
                        { required: true, validator: pass, trigger: "blur" }
                    ],
                    // 验证确认密码
                    checkPwd: [
                        // 自定义验证函数
                        { required: true, validator: checkPass, trigger: "blur" }
                    ]
                    /*
                    验证规则字段说明：
                        { required: true/false 必填,   message: "错误的提示信息", trigger: "触发验证的方式" }
                        { min: 最小长度, max: 最大长度, message: "错误的提示信息", trigger: "触发验证的方式" }
                        { validator： 自定义验证规则函数名，  trigger: 'blur'}
                    */
                }
            };
        },
        methods: {
            // 点击登录按钮 触发这个函数
            submitForm(formName) {
                // 获取表单组件 调用验证方法
                this.$refs[formName].validate(valid => {
                    // 如果所有验证通过 valid就是true
                    if (valid) {
                        // alert("前端验证通过 可以提交给后端！");
                        // 后续就可以把收集的账号和密码 一起发送给后端 验证用户名和密码的正确性。
                        // 收集账号和密码
                        let params = {
                            username: this.loginForm.username,
                            password: this.loginForm.password
                        };

                        // 允许携带cookie
                        this.axios.defaults.withCredentials=true;//---2

                        this.axios.post('http://127.0.0.1:3000/users/checklogin',
                        qs.stringify(params), //这是解决参数问题哦
                        { headers: {'Content-Type':'application/x-www-form-urlencoded'} })  //post  设置请求头
                        .then(response => {
              // 接收后端响应的数据 判断
              if (response.data.rstCode === 1) {
                // 成功 弹出登录成功的提示
                this.$message({
                  type: 'success',
                  message: response.data.msg
                });

                // 跳转到首页
                setTimeout(() => {
                  this.$router.push("/");
                }, 500)
              } else {
                // 失败 弹出失败的提示
                this.$message.error(response.data.msg)
              }
            })
                    } else {
                        // 否则就是false
                        alert("前端验证失败 不能提交给后端！");
                        return false;
                    }
                });
            },
            // 点击重置按钮 触发这个函数
            resetForm(formName) {
                // this.$refs.loginForm.resetFields() 获取整个表单组件 调用重置方法
                this.$refs[formName].resetFields();
            }
        }
    }

</script>


<style lang="less">
    .login {
        height: 100%;
        width: 100%;
        background-color: #2d3a4b;
        .login-wrapper {
            width: 500px;
            height: 350px;
            border-radius: 8px;
            background-color: rgba(0, 0, 0, 0.2);
            color: #ffffff;

            // 垂直水平居中
            position: fixed;
            top: 0;
            right: 0;
            bottom: 0;
            left: 0;

            margin: auto;
            padding: 30px 50px 0 10px;
            .title {
                font-size: 20px;
                margin-left: 60px;
                margin-bottom: 20px;
            }
            .el-form {
                .el-form-item {
                    .el-form-item__label {
                        color: #ffffff;
                    }
                }
            }
        }
    }


</style>
