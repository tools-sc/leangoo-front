<template>
  <div class="user-info">
    <div class="container">
      <div class="info">
        <div class="avatar" v-on:mouseover="onShow" v-on:mouseout="onHide" v-on:click="cardShow">
          <img id="img" class="image" :src="user.userAvatar"/>
          <span id="change" v-show="isShow" class="change">修改</span>
        </div>
        <div class="user-base-info" v-if="isChangeInfo==='no'">
          <h3>{{user.userAccount}}</h3>
          <p v-if="user.userSex===0">男</p>
          <p v-if="user.userSex===1">女</p>
          <p v-if="user.userSex===null">未填写</p>
          <p>{{user.userEmail}}</p>
          <p>{{user.userIntro}}</p>
          <b-button variant="secondary" size="sm" v-on:click="changeInfo">
            <icon name="pencil"></icon>
            编辑个人资料
          </b-button>
        </div>
        <div class="user-base-info" v-if="isChangeInfo==='yes'">
          <b-form @submit="onSubmit">
            <b-form-group id="exampleInputGroup3"
                          label="性别:" label-for="userSex">
              <b-form-select id="userSex"
                             :options="sex" required
                             v-model="form.userSex"
                             size="sm"
              ></b-form-select>
            </b-form-group>
            <b-form-group id="exampleInputGroup1"
                          label="邮箱地址:" label-for="userEmail">
              <b-form-input id="userEmail"
                            type="email" v-model="form.userEmail" required
                            placeholder="请输入邮箱地址..."
                            size="sm"
              ></b-form-input>
            </b-form-group>
            <b-form-group id="textarea1"
                          label="简介:" label-for="userIntro">
              <b-form-textarea id="userIntro" placeholder="请输入简介..."
                               :rows="3"
                               :max-rows="6" v-model="form.userIntro" required
                               size="sm">
              </b-form-textarea>
            </b-form-group>
            <b-button type="submit" variant="success" size="sm">保存</b-button>
            <b-button type="reset" variant="primary" size="sm">重置</b-button>
            <b-button variant="secondary" size="sm" v-on:click="notChangeInfo">取消</b-button>
          </b-form>
        </div>
      </div>
    </div>
    <div>
      <b-card v-if="isCardShow==='yes'" class="text-center b-card">
        <div slot="header" style="text-align: left;">
          <small style="text-align: left">修改头像</small>
          <span v-on:click="cardHide" style="cursor: pointer"><icon name="times" style="float: right;"></icon></span>
        </div>
        <b-form-file accept=".jpg, .png" choose-label="上传图片" @change="changeImage($event)" v-model="avatarInput"></b-form-file>
        <hr>
        <b-button variant="success" size="sm" v-on:click="editAvatar()"><icon name="check"></icon>保存头像</b-button>
      </b-card>
    </div>
    <b-card no-body style="text-align: left;">
      <b-tabs ref="tabs" card>
        <b-tab title="设置">
          <div class="myTab-body">
            <div class="myTab-title">
              <h5>基本信息</h5>
              <hr>
            </div>
            <b-button variant="link" size="sm" class="myTab-link" v-on:click="changePwd"><span>修改密码</span></b-button>
            <div>
              <b-card v-if="changePwdShow==='yes'" class="b-card">
                <div slot="header">
                  <small style="text-align: left">修改密码</small>
                  <span v-on:click="changePwdHide" style="cursor: pointer"><icon name="times" style="float: right;"></icon></span>
                </div>
                <b-form @submit="onPwdSubmit">
                  <b-form-group id="oldPwd"
                                label="原密码:" label-for="oldPwd">
                    <b-form-input id="oldPwd"
                                  type="password" v-model="form2.oldPwd" required
                                  placeholder="请输入原密码..."
                                  size="sm"
                    ></b-form-input>
                  </b-form-group>
                  <b-form-group id="newPwd"
                                label="新密码:" label-for="newPwd">
                    <b-form-input id="newPwd"
                                  type="password" v-model="form2.newPwd" required
                                  placeholder="请输入新密码..."
                                  size="sm"
                    ></b-form-input>
                  </b-form-group>
                  <b-form-group id="newPwdConfirm"
                                label="确认新密码:" label-for="newPwdConfirm">
                    <b-form-input id="newPwdConfirm"
                                  type="password" v-model="form2.newPwdConfirm" required
                                  placeholder="请输入确认新密码..."
                                  size="sm"
                    ></b-form-input>
                  </b-form-group>
                  <b-button type="submit" variant="success" size="sm">确认</b-button>
                  <b-button variant="secondary" size="sm" v-on:click="changePwdHide">关闭</b-button>
                </b-form>
              </b-card>
            </div>
          </div>
          <div class="myTab-body">
            <div class="myTab-title">
              <h5>登录账号</h5>
              <hr>
            </div>
            <b-card header="主邮箱" bg-variant="light" class="text-center" style="width: 40%">
              <p class="card-text">{{user.userEmail}}</p>
            </b-card>
            <b-button variant="secondary" size="sm" style="margin-top: 30px;margin-left: 5px;">
              添加备用邮箱
            </b-button>
          </div>
          <div class="myTab-body">
            <div class="myTab-title">
              <h5>通知</h5>
              <hr>
            </div>
            <b-button variant="link" size="sm" class="myTab-link"><span>开启/关闭邮件提醒</span></b-button>
          </div>
        </b-tab>
        <b-tab title="第三方账号">
          <div class="myTab-body">
            <div class="myTab-title">
              <h5>赞助SixTeam团队</h5>
              <hr>
            </div>
            <div class="myTab-text">
              <img src="../../assets/pay.jpg" class="myTab-img"/>
              <b class="myTab-b">个人收款账号</b>
            </div>
            <div class="myTab-text2">
              <p>谢谢您的支持，我们SixTeam会继续努力，做到最好！</p>
            </div>
          </div>
        </b-tab>
        <b-tab title="通讯录">
          <div class="myTab-body">
            <div v-if="addFriendShow == 'no'" class="myTab-title" style="text-align: right">
              <b-button variant="secondary" size="sm" v-on:click="showAddFriend">
                添加好友
              </b-button>
              <hr>
            </div>
            <div v-if="addFriendShow == 'yes'" style="text-align: right">
              <b-form-input size="sm" type="text" v-model="addFriendMsg.friendAccount" placeholder="输入账号" style="width: 200px;display: inline"></b-form-input>
              <b-button variant="success" size="sm" v-on:click="addFriend">添加</b-button>
              <hr>
            </div>
            <div v-for="friend in friends">
              <div class="myTab-friend">
                <div class="myTab-avatar">
                  <span>
                    <img :src="friend.avatar"/>
                  </span>
                </div>
                <div style="text-align: left">
                  <p class="myTab-avatar-p">{{friend.name}}</p>
                  <p>{{friend.email}}</p>
                </div>
              </div>
              <div style="float: right">
                <b-button variant="danger" size="sm" v-on:click="deleteFriend(friend.name)">
                  删除
                </b-button>
              </div>
              <hr>
            </div>
          </div>
        </b-tab>
      </b-tabs>
    </b-card>
  </div>
</template>
<script>
  import Icon from "../../../node_modules/vue-awesome/components/Icon";
  import Bus from "../../bus"
  export  default {
    components: {Icon},
    methods: {
      onShow: function () {
        this.isShow = true;
      },
      onHide: function () {
        this.isShow = false;
      },
      cardShow: function () {
        this.isCardShow = 'yes';
      },
      cardHide: function () {
        this.isCardShow = 'no';
      },
      changePwd: function () {
        this.changePwdShow = 'yes';
      },
      changePwdHide: function () {
        this.changePwdShow = 'no';
      },
      onSubmit(evt) { //表单提交事件在这里
        evt.preventDefault();
        var form = this.form;
        this.$ajax.post('/User/changeUserInfo',
          JSON.stringify(form),
          {
            headers: {
              "Content-Type": "application/json"
            }
          }
        ).then((res) => {
          if(res.data.errcode === 0){
            this.user = res.data.data;
            form.userSex = res.data.data.userSex;
            form.userEmail = res.data.data.userEmail;
            form.userIntro = res.data.data.userIntro;
            this.$notify.success({title:'提示',message:res.data.info})
        
            this.notChangeInfo();
          }else{
             this.$notify.error({title:'提示',message:res.data.info})
          }
        }).catch(res => {
          console.log(res)
        });
      },
      onPwdSubmit(evt) { //表单提交事件在这里
        evt.preventDefault();
        var form2 = this.form2;
        if(form2.oldPwd != this.user.userPassword){
           this.$notify.error({title:'提示',message:"原密码错误，请重新输入!"})
        }else if(form2.newPwd != form2.newPwdConfirm){
            this.$notify.error({title:'提示',message:"原密码错误，请重新输入!"})
        }else {
          this.$ajax.post('/User/changeUserPassword',
            JSON.stringify(this.form2),
            {
              headers: {
                "Content-Type": "application/json"
              }
            }
          ).then((res) => {
            if(res.data.errcode === 0){
              this.user.userPassword = res.data.data.userPassword;
               this.$notify.info({title:'提示',message:res.data.info})
              this.form2={oldPwd: '', newPwd: '', newPwdConfirm: ''};
              this.changePwdHide();
            }else{
               this.$notify.error({title:'提示',message:res.data.info})
            }
          }).catch(res => {
            console.log(res)
          });
        }
      },
      changeInfo(){
        this.isChangeInfo = 'yes';
      },
      notChangeInfo(){
        this.isChangeInfo = 'no';
      },
      showAddFriend(){
          this.addFriendShow = 'yes';
      },
      addFriend(){
        this.$ajax.post(
          '/UserFriend/addFriend',
          JSON.stringify(this.addFriendMsg), {
            headers: {
              "Content-Type": "application/json"
            }
          }
        ).then((res) => {
          if(res.data.errcode === 0){
            this.friends = res.data.data;
             this.$notify.info({title:'提示',message:res.data.info})
          }else{
             this.$notify.error({title:'提示',message:res.data.info})
          }
        }).catch(res => {
          console.log(res)
        });
      },
      deleteFriend(friendAccount){
        this.deleteFriendMsg.friendAccount = friendAccount;
        this.$ajax.post(
          '/UserFriend/deleteFriend',
          JSON.stringify(this.deleteFriendMsg),
          {
            headers: {
              "Content-Type": "application/json"
            }
          }
        ).then((res) => {
          if(res.data.errcode === 0){
            this.friends = res.data.data;
             this.$notify.success({title:'提示',message:res.data.info})
       
          }else{
             this.$notify.error({title:'提示',message:res.data.info})
          }
        }).catch(res => {
          console.log(res)
        });
      },
      editAvatar() {
        // 修改了头像
        if (this.avatarInput !== null) {
          var image = new FormData();
          image.append('userAvatar', this.avatarInput);
          this.$ajax.post('/User/changeAvatar', image, {
            headers: {
              "Content-Type": "multipart/form-data"
            }
          }).then((res) => {
            if (res.data.errcode === 0) {
              this.user.userAvatar = res.data.data.userAvatar;
               this.$notify.info({title:'提示',message:res.data.info})
              this.avatarInput = null;
              this.cardHide();
            } else {
               this.$notify.error({title:'提示',message:res.data.info})
            }
          }).catch(res => {
            console.log(res)
          });
        }
        },
      changeImage(e) {
        var file = e.target.files[0]
        var reader = new FileReader()
        var that = this
        reader.readAsDataURL(file)
        reader.onload = function(e) {
          that.user.userAvatar = this.result
        }
      }
    },
    data () {
      return {
        user:{},
        avatarInput:null,
        isShow: false,
        isCardShow: 'no',
        isChangeInfo: 'no',
        changePwdShow: 'no',
        addFriendShow:'no',
        addFriendMsg:{friendAccount:''},
        deleteFriendMsg:{friendAccount:''},
        form: {
          userEmail: '',
          userSex: null,
          userIntro: ''
          //secret: 'S3CR3T'
        },
        form2:{
          oldPwd: '',
          newPwd: '',
          newPwdConfirm: ''
        },
        sex: [
          {text: '选择性别', value: null},
          {text: '男', value: 0},
          {text: '女', value: 1}
        ],
        friends:[]
      }
    },
    created() {
      Bus.$emit('login_ok');
      this.$ajax.post('/User/getUserInfoById').then((res) => {
          var form = this.form;
        this.user = res.data.data;
        form.userSex = res.data.data.userSex;
        form.userEmail = res.data.data.userEmail;
        form.userIntro = res.data.data.userIntro;
      }).catch(res => {
        console.log(res)
      });
      this.$ajax.post('/UserFriend/getFriendList').then((res) => {
          this.friends = res.data.data;
      }).catch(res => {
          console.log(res)
      });
    }
  }
</script>
<style>
  * {
    box-sizing: border-box;
  }

  .user-info {
    padding-top: 50px;
    background-color: #edeff0;
    width: 100%;
    height: auto;
    margin-left: auto;
    margin-right: auto;
  }

  .container{
    padding: 30px;
    width: 1170px;
    height: auto;
  }

  .info {
    margin-bottom: 0;
    margin-left: 16%;
  }

  .avatar {
    position: relative;
    display: block;
    float: left;
    background: #ccc !important;
    text-align: center;
    cursor: pointer;
    margin: auto;

  }

  .user-base-info {
    margin-top: -6px;
    margin-left: 100px;
    text-align: left;
    font-size: 14px;
    width: 40%;
    background-color: #EDEFF0;
  }

  .user-base-info > p {
    margin: 0 0 7px;
  }

  .image {
    border-radius: 3px;
    height: 85px;
    width: 85px;
  }

  .change {
    display: block;
    line-height: 30px;
    position: absolute;
    background: rgba(0, 0, 0, .5);
    text-decoration: underline;
    color: #fff;
    height: 30px;
    width: 85px;
    bottom: 0;
    text-align: center;
    border-bottom-left-radius: 3px;
    border-bottom-right-radius: 3px;
  }

  .b-card {
    width: 350px;
    top: 26%;
    left: 345px;
    position: fixed !important;
    z-index: 2000;
    right: 20px;
    -webkit-box-shadow: #666 0px 0px 10px;
    -moz-box-shadow: #666 0px 0px 10px;
    box-shadow: #666 0px 0px 10px;
  }

  .custom-file-control::before {
    background-color: #28A745;
  }

  .custom-file-control[data-v-c68bd5f8]::after {
    float: left;
  }
  .card{
    border: 0px !important;
  }
  .card-header{
    background-color: #EDEFF0 !important;
  }
  .card-header-tabs{
    margin-left: 35% !important;
  }
  .myTab-body{
    margin-left: 15%;
    margin-right: 15%;
    padding: 24px;
  }
  .myTab-link{
    display: block;
    margin: 6px 0 6px 30px;
    color: #222;
    padding: 10px;
    border-radius: 3px;
    cursor: pointer;
  }
  .myTab-text{
    display: inline-block;
  }
  .myTab-img{
    width: 150px;
    padding: 4px;
    line-height: 1.42857143;
    border: 1px solid #ddd;
    border-radius: 4px;
    transition: all .2s ease-in-out;
    display: inline-block;
    max-width: 100%;
    height: auto;
  }
  .myTab-b{
    display: block;
    text-align: center;
  }
  .myTab-text2{
    vertical-align: top;
    margin: 50px 0 0 35px;
    display: inline-block;
  }
  .myTab-friend{
    position: relative;
    min-height: 40px;
    padding-left: 50px;
    display: inline-block;
    margin-bottom: -30px;
  }
  .myTab-avatar{
    position: absolute;
    left: 5px;
    top: 4px;
  }
  .myTab-avatar-p{
    margin-bottom: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    word-break: keep-all;
  }
  .myTab-avatar > span{
    font-size: 12px;
    font-weight: 700;
    line-height: 30px;
    display: block;
    overflow: hidden;
    width: 32px;
    height: 32px;
    text-align: center;
    border-radius: 4px;
    background: #ccc !important;
  }
  .myTab-avatar > span > img{
    width: 32px;
    height: 32px;
    display: inline;
  }
</style>
