<template>
    <div class="module">
        <!--        <pre>{{JSON.stringify(model, null, 4)}}</pre>-->

        <div class="login" v-show="loginShow">

            <div class="row">
                <div><span>登陆账号</span></div>
                <div><input spellcheck="false" v-model="user.userName" placeholder="请输入登陆账号"
                            @change="checkUserExist(1)"/></div>
            </div>
            <div class="row">
                <div><span>密码</span></div>
                <div><input spellcheck="false" v-model="user.passWord" type="password" placeholder="请输入密码"/></div>
            </div>

            <div class="row animated fadeInRight" v-show="showRegist">
                <div><span>确认密码</span></div>
                <div><input spellcheck="false" v-model="user.passWord2" type="password" placeholder="请再次输入密码"/></div>
            </div>

            <div class="row animated fadeInRight" v-show="showRegist">
                <div><span>邮箱</span></div>
                <div><input spellcheck="false" v-model="user.email" placeholder="请输入邮箱" @change="checkUserExist(2)"/>
                </div>
            </div>

            <div class="row verifyCode animated fadeInRight" v-show="showRegist">
                <div><span>验证码</span></div>
                <div>
                    <input spellcheck="false" v-model="user.verifyCode" placeholder="请输入验证码"/>
                </div>
                <div>
                    <button @click="sendEmail()">发 送</button>
                </div>
            </div>

            <div class="btn">
                <button @click="login()" v-show="!showRegist">登 陆</button>
                <button style="background-color: orange;" @click="registUser()">注 册</button>
            </div>


        </div>

        <div v-show="!loginShow && !modifyUserPwd && !updateUser && !deleteUser" class="info">
            <div class="user-info">
                <span>{{openUserInfo.user.userName}} 你好</span>
                <!--                <img src="../assets/img/logout.svg" title="退出登陆" @click="clearLocalStorage()"/>-->
            </div>
            <div class="user-info-detail">
                <div>
                    <label>注册时间 : </label><span>{{openUserInfo.user.createDate}}</span>
                </div>
                <div class="operation">
                    <button @click="clearLocalStorage()">登出</button>
                    <button @click="showMdPwd()">修改密码</button>
                    <button @click="showMdUserInfo()">修改用户信息</button>
                    <button @click="showDeleteUser()">注销用户</button>
                </div>
            </div>
        </div>

        <div class="login" v-show="modifyUserPwd || updateUser || deleteUser">

            <div class="row animated fadeInRight">
                <div><span>邮箱</span></div>
                <div><input spellcheck="false" v-model="user.email" placeholder="请输入邮箱" readonly/>
                </div>
            </div>

            <div class="row verifyCode animated fadeInRight">
                <div><span>验证码</span></div>
                <div>
                    <input spellcheck="false" v-model="user.verifyCode" placeholder="请输入验证码"/>
                </div>
                <div>
                    <button @click="sendEmail()">发 送</button>
                </div>
            </div>

            <div class="row animated fadeInRight" v-show="modifyUserPwd">
                <div><span>密码</span></div>
                <div><input spellcheck="false" v-model="user.passWord" type="password" placeholder="请输入密码"/></div>
            </div>

            <div class="row animated fadeInRight" v-show="modifyUserPwd">
                <div><span>确认密码</span></div>
                <div><input spellcheck="false" v-model="user.passWord2" type="password" placeholder="请再次输入密码"/>
                </div>
            </div>

            <div class="row" v-show="updateUser">
                <div><span>登陆账号</span></div>
                <div><input spellcheck="false" v-model="user.userName" placeholder="请输入登陆账号"/></div>
            </div>

            <div class="btn">
                <button style="" @click="submitAction()">
                    {{modifyUserPwd ? '修改密码' : updateUser ? '修改用户信息' : deleteUser ? '注销' : ''}}
                </button>
            </div>
        </div>


        <div class="tips animated fadeInUp">
            <span>提示：</span>
            <span>1.如遇部分问题可以点击 <a style="color: #0f88eb;" href="javascript:void(0);"
                                  @click="clearLocalStorage()">强制清理缓存</a></span>
            <span>2.没有账号可以先点击注册</span>
            <span>3.登陆账号后，可以同步你的自定义数据哦</span>
            <span>4.邮箱也可以用来登陆</span>
            <span>5.<a style="color: #0f88eb;" href="https://support.qq.com/products/170200">关于&反馈</a></span>

        </div>

    </div>
</template>

<script>
    export default {
        name: "User",
        components: {},
        data() {
            return {
                loginShow: true,
                user: {
                    userName: '',
                    passWord: '',
                    passWord2: '',
                    email: '',
                    verifyCode: '',
                    birthday: '',
                    nickName: '',
                },
                showUserInfo: {},
                showRegist: false,
                modifyUserPwd: false,
                updateUser: false,
                deleteUser: false
            }
        },
        computed: {
            openUserInfo() {
                return JSON.parse(JSON.stringify(this.$store.getters.openUserInfo));
            }
        },
        watch: {
            openUserInfo: {
                handler() {
                    this.init();
                },
                deep: true
            },
        },
        mounted() {
            this.init();
        },
        methods: {
            submitAction() {
                if (this.modifyUserPwd) {
                    this.modifyUserPwdMethod()
                } else if (this.updateUser) {
                    this.updateUserMethod();
                } else if (this.deleteUser) {
                    this.deleteUserMethod();
                }
            },
            deleteUserMethod() {
                if (this.user.email == '' || this.user.verifyCode == '') {
                    this.$toast('请先填写信息后再提交');
                    return;
                }

                let result = window.confirm('确认注销吗？')
                if (result) {
                    let url = this.Utils.basicUrl() + '/user/v1/deleteUser';
                    let param = {
                        "userCode": this.openUserInfo.user.userCode,
                        "verifyCode": this.user.verifyCode
                    };
                    this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {
                        if (!response || response.code !== '0') {
                            this.$toast(response.message);
                            return;
                        }
                        this.$toast("注销成功");
                        setTimeout(() => {
                            this.Utils.clearLocalStorage();
                        }, 1000)

                    });
                }
            },
            showDeleteUser() {
                this.deleteUser = true;
                this.modifyUserPwd = false;
                this.updateUser = false;

                this.loginShow = false;
                this.showRegist = false;
                this.user.email = this.openUserInfo.user.email;
            },
            updateUserMethod() {
                if (this.user.userName == '' || this.user.email == '' || this.user.verifyCode == '') {
                    // this.$toast('请先填写信息后在提交');
                    this.$toast('请先填写信息后再提交');
                    return;
                }

                if (this.user.passWord !== this.user.passWord2) {
                    this.$toast('两次填写的密码不一致');
                    return;
                }

                let url = this.Utils.basicUrl() + '/user/v1/updateUser';
                let param = {
                    "userCode": this.openUserInfo.user.userCode,
                    "userName": this.user.userName,
                    "verifyCode": this.user.verifyCode
                };
                this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {
                    if (!response || response.code !== '0') {
                        this.$toast(response.message);
                        return;
                    }
                    this.$toast("修改成功，请重新登陆");
                    setTimeout(() => {
                        this.Utils.clearLocalStorage();
                    }, 1000)

                });
            },
            showMdUserInfo() {
                this.updateUser = true;
                this.modifyUserPwd = false;
                this.deleteUser = false;

                this.loginShow = false;
                this.showRegist = false;
                this.user.email = this.openUserInfo.user.email;
                this.user.userName = '';
            },
            modifyUserPwdMethod() {
                if (this.user.passWord == '' || this.user.email == '' || this.user.verifyCode == '') {
                    // this.$toast('请先填写信息后在提交');
                    this.$toast('请先填写信息后再提交');
                    return;
                }

                if (this.user.passWord !== this.user.passWord2) {
                    this.$toast('两次填写的密码不一致');
                    return;
                }

                let url = this.Utils.basicUrl() + '/user/v1/modifyUserPwd';
                let param = {
                    "userCode": this.openUserInfo.user.userCode,
                    "passWord": this.user.passWord,
                    "verifyCode": this.user.verifyCode
                };
                this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {
                    if (!response || response.code !== '0') {
                        this.$toast(response.message);
                        return;
                    }
                    this.$toast("修改成功，请重新登陆");
                    setTimeout(() => {
                        this.Utils.clearLocalStorage();
                    }, 500)

                });
            },
            showMdPwd() {
                this.modifyUserPwd = true;
                this.updateUser = false;
                this.deleteUser = false;

                this.loginShow = false;
                this.showRegist = false;
                this.user.email = this.openUserInfo.user.email;
                this.user.passWord = '';
                this.user.passWord2 = '';
            },
            registUser() {
                if (this.showRegist) {
                    if (this.user.userName == '' || this.user.passWord == '' || this.user.email == '' || this.user.verifyCode == '') {
                        // this.$toast('请先填写信息后在提交');
                        this.$toast('请先填写信息后再提交');
                        return;
                    }

                    if (this.user.passWord !== this.user.passWord2) {
                        this.$toast('两次填写的密码不一致');
                        return;
                    }

                    let url = this.Utils.basicUrl() + '/user/v1/registUser';
                    let param = {
                        "userName": this.user.userName,
                        "passWord": this.user.passWord,
                        "email": this.user.email,
                        "verifyCode": this.user.verifyCode
                    };

                    this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {
                        if (!response || response.code !== '0') {
                            this.$toast(response.message);
                            return;
                        }
                        this.$toast("注册成功，请登陆");
                        this.showRegist = false;
                    });
                } else {
                    this.showRegist = true;
                }
            },
            checkUserExist(type) {
                if (!this.showRegist) {
                    return;
                }
                let userName = '';
                if (type === 1) {
                    userName = this.user.userName;
                }
                if (type === 2) {
                    userName = this.user.email;
                }

                let url = this.Utils.basicUrl() + '/user/v1/checkUserExist';
                let param = {
                    userName: userName
                };

                this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {
                    if (!response || response.code !== '0') {
                        this.$toast(response.message);
                        return;
                    }
                });
            },
            sendEmail() {
                let email = this.user.email;
                if (!email || email === '') {
                    this.$toast('邮箱地址不能为空');
                    return;
                }

                let url = this.Utils.basicUrl() + '/user/v1/sendEMail';
                let param = {
                    email: email
                };

                this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {
                    if (!response || response.code !== '0') {
                        this.$toast('邮件发送失败！');
                        return;
                    }

                    this.$toast('邮件发送成功，请查收邮件！');
                });

            },
            login() {

                if (this.user.userName === '' || this.user.passWord === '') {
                    this.$toast('请输入账号和密码');
                    return;
                }

                let url = this.Utils.basicUrl() + '/user/v1/loginUser';
                let param = {
                    userName: this.user.userName,
                    passWord: this.user.passWord
                };

                this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {
                    if (!response || response.code !== '0') {
                        this.$toast('登陆失败, 请核对账号密码');
                        return;
                    }
                    // this.$toast('登陆成功');

                    this.loginShow = false;

                    let user = response.data;


                    // 获取 服务器端配置
                    url = this.Utils.basicUrl() + '/user/v1/getUserExtInfo';
                    let param = {
                        userCode: user.userCode
                    };
                    this.Utils.postJson(url, this.Utils.getCommonReq(param)).then(response => {

                        let ext = this.openUserInfo.ext;
                        if (response.data != null && response.data.userSet != null && response.data.userSet != '' && response.data.userSet != {}
                            && JSON.parse(response.data.userSet).searchEngineList && JSON.parse(response.data.userSet).searchEngineList.length > 0) {
                            ext = JSON.parse(response.data.userSet);
                        }

                        this.openUserInfo.user = user;
                        this.openUserInfo.ext = ext;
                        this.$store.commit('uOpenUserInfo', this.openUserInfo);

                        if (!response || response.code !== '0') {
                            this.$toast('获取用户设置失败');
                            return;
                        }


                    });
                });
            },
            clearLocalStorage() {
                this.Utils.clearLocalStorage();
            },
            save(type) {
                let tmp = JSON.parse(this.showUserInfo);
                tmp.user = this.openUserInfo.user;
                this.$store.commit('uOpenUserInfo', tmp);

                if (type === 1) {
                    this.$toast('保存到本地成功');
                } else if (type === 2) {
                    this.Utils.saveUserInfoExt();
                } else if (type === 3) {
                    let passwd = window.prompt('输入当前登陆用户的密码，清空本地和服务器个性化设置信息');
                    if (!passwd || passwd === '') {
                        this.$toast('密码不能为空');
                        return;
                    }

                    // 删除 服务器端配置
                    let url = '/common-server/user/api/v1/delUserInfoExt';
                    let data = {
                        param: {
                            userId: this.openUserInfo.user.id,
                            passwd,
                        },
                    };
                    this.Utils.postJson(url, data).then(response => {
                        // window.console.log(response);
                        if (!response || response.code !== '0') {
                            this.$toast(response.message);
                            return;
                        }
                        this.$toast('删除成功，即将刷新页面...');

                        let that = this;
                        setTimeout(function () {
                            that.clearLocalStorage();
                        }, 1000);

                    });

                    // let re = window.confirm('清空本地和服务器个性化设置信息');
                    // if (re) {
                    //     // 清空 todo..
                    //     alert('清空 待完成');
                    // }
                }

            },
            init() {
                let user = this.openUserInfo.user;
                if (user.userCode !== '-1') {
                    this.loginShow = false;
                }
                let tmp = JSON.parse(JSON.stringify(this.openUserInfo));
                delete tmp.user;
                this.showUserInfo = JSON.stringify(tmp, null, 4);
            },
            submit() {
                alert(JSON.stringify(this.model));
            },
            reset() {
                this.$refs.JsonEditor.reset();
            },
        },
    }
</script>

<style scoped>

    .module {
        /*padding: 10px 20px 0;*/
        display: grid;
        grid-template-columns: 1fr;

        grid-row-gap: 20px;
        /*justify-content: center;*/
        /*justify-items: center;*/
    }

    .login {
        /*height: 300px;*/

        display: grid;
        grid-template-columns: 1fr;
        grid-template-rows: repeat(auto-fit, 40px);

        grid-row-gap: 10px;
    }

    .row {
        display: grid;
        grid-template-columns: 80px 1fr 0;
        grid-column-gap: 10px;


        border: 1px solid lightgrey;
        border-radius: 3px;

        align-items: center;
        align-content: center;
        line-height: 40px;
    }

    .row > div {
        width: 100%;
        height: 100%;
        text-align: center;

        display: grid;
        grid-template-columns: 1fr;
    }

    .row > div:first-child {
        border-right: 1px solid lightgrey;
    }

    .row span {
        font-size: 13px;
    }


    input {
        outline: none;
        border: none;
        background: transparent;
        font-size: 15px;
        font-weight: bold;
    }

    .info {

        display: grid;
        grid-template-columns: 1fr;
        /*grid-template-rows: repeat(auto-fit, auto);*/

        grid-row-gap: 10px;

        height: 240px;

        align-content: start;

    }

    textarea {
        justify-self: center;
        height: 300px;
        width: 100%;
        font-weight: bold;

        /*去除选中状态边框*/
        outline: none;

        font-size: 15px;
        border: 2px dashed darkgray;
        /*width: calc(100% - 20px);*/
    }


    .operation {
        margin-top: 10px;
        display: grid;
        grid-row-gap: 10px;
    }


    .user-info {
        display: grid;
        grid-template-columns: repeat(3, auto);
        justify-content: start;
        align-items: center;
        grid-column-gap: 10px;
    }

    .user-info img {
        width: 25px;
        height: 25px;
        cursor: pointer;
    }

    .user-info span:first-child {
        font-style: oblique;
        padding: 0 5px;
    }

    .user-info span:last-child {
        background-color: cornsilk;
        border-radius: 12px;
        padding: 0 5px;
    }

    button {
        height: 40px;
        outline: none;
        border: none;
        font-size: 16px;
        cursor: pointer;
        color: white;
        background-color: #409EFF;
        /*text-shadow: 0 1px 0 #fff;*/

        /*word-spacing: 2px;*/

        letter-spacing: 5px;

        border-radius: 5px;
    }


    .operation > button:nth-child(2) {
        background-color: orange;
    }

    .operation > button:last-child {
        background-color: orangered;
    }


    button:hover {
        background: #0d79d1;
        border-color: #0f88eb;
    }

    .operation > button:nth-child(2):hover {
        background: darkorange;
    }

    .operation > button:last-child:hover {
        background: red;
    }


    .btn {
        display: grid;
        grid-row-gap: 10px;
    }


    .row:last-child > span {
        grid-column: 1/3;
        padding: 0 10px;
        /*background-color: cornsilk;*/
        text-align: start;
        line-height: 25px;
    }

    /*.verifyCode {*/
    /*    grid-template-columns: 80px 1fr 50px;*/
    /*}*/


    .verifyCode > div:last-child > button {
        position: relative;
        right: 50px;
        width: 50px;
        letter-spacing: 0;
    }

    .user-info-detail > div:first-child {
        background-color: cornsilk;
        border-radius: 12px;
        padding: 0 5px;
    }

    @media screen and (max-width: 700px) {

        span, textarea {
            font-size: 13px;
        }

        .user-info {
            grid-template-columns: repeat(2, auto);
        }

        .user-info > span:last-child {
            grid-column: 1/3;
        }

        /*.verifyCode {*/
        /*    grid-template-columns: 80px 160px auto;*/
        /*}*/
        /*.verifyCode > div:last-child {*/
        /*    justify-self: right;*/
        /*}*/


    }

</style>