<template>
  <div class="keyBoardBox">
      <p class="keyTips">请输入XXX支付密码</p>
      <div class="keyBoardUl">

          <div class="inputBox key-border-radius" v-for="(item,index) in itemAll" :key="index" :id="item.keyId"   data-keybord=true data-val="" data-holder="">
            <div class="ijiamiKey_wrap"  v-html="item.keyEnter"></div>
          </div>

      </div>
      <div class="keyForget" v-if="payType!='forget'"><span class="keyForgetLink" @click='forgetGoenter'>忘记密码</span></div>
      <transition name="slideup">
      <div class="key-pop" onselectstart="return!1" v-if="keyPull">
        <div class="key-con">
          <div id="key-con-main" class="key-con-main">
            <ul >
              <li class="english_key number_key clear"><div class=""><ul>
                <li class="clear  key-border-l">
                  <div  v-for="(keyOne,index) in keyNumAll" class="span-wrap   key-border-t number_keyList" :class="keyIndex" @click="disKeyenter?inputInto('Y',keyOne):''" :key="index" :data-lgh="keyOne.keyLgh"><span class="ui-border-r" ><i class="keyNumber" :style="keyOne.keyValue"></i></span></div>
                </li>
                <li class="clear">
                  <div class="span-wrap class-change-gray nubBtnBlue" @click="completeEnter">完成</div>
                  <div class="span-wrap key-border-t number_keyList" :data-lgh="keyLastLgh" @click="disKeyenter?inputInto('N',0):''"><span><i class="keyNumber" :style="keyLastValue"></i></span></div>
                  <div class="span-wrap x-span" @click="deleteValue"></div>
                </li>
              </ul>
              </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
      </transition>
  </div>
</template>
<script>
import {RSAKeyPair,encryptedStringKey,setMaxDigits} from '../../assets/js/keyRsa'
export default {
  data () {
    return {
      keyPull:true,
      dataImages:'',
      //模拟input 循环数组对象
      itemAll:[
        {'keyId':'keyInput1','keyNub':false,'keyEnter':''},
        {'keyId':'keyInput2','keyNub':false,'keyEnter':''},
        {'keyId':'keyInput3','keyNub':false,'keyEnter':''},
        {'keyId':'keyInput4','keyNub':false,'keyEnter':''},
        {'keyId':'keyInput5','keyNub':false,'keyEnter':''},
        {'keyId':'keyInput6','keyNub':false,'keyEnter':''},
        ],
       keyIndex:'',
       //模拟数字键盘对象前三行
       keyNumAll:[
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         {'imageUrl':'','keyLgh':'','keyValue':{}},
         ],
      //最后一行中间数字值
      keyLastLgh:'',
      keyLastValue:{},
      modulus:'',
      exponent:'',
      offset:[],//偏移量
      endVal:'',//请求的唯一 token 值
      newKeyArray:[],//第一次密码展示效果
      newKeyNum:0,//第一次密码记录值
      keyArray:[],//密码展示效果
      min:'', // 小写下标偏移量
      max:'', // 大写下标偏移量
      nub:'', // 数字下标偏移量
      minnub:'', // 数字下标偏移量  特殊字符中的数字下标
      mark:'', // 符号和小数字的下标偏移量
      nub:'', // 数字下标偏移量
      math:[ - 3, -2, -1, 0, 1, 2, 3], // 随机下标的偏移量
      token:'',
      encodeVal:'',//输入密码和
      key:'',
      pwdParamter:'',//加密后的键盘密码值
      method:'',
      payType:'',//验密入口类型
      cardId:'',//cardId
      securityCardType:'',//安全卡类型
      payPwdProcFlag:'',//payPwdProcFlag判断是设置支付密码还是验证支付密码
      token:'',//获取token 值
      pictureKey:'',//图片钥匙
      keyPair:'',
      disKeyenter:true,//默认第一次可以操作
      payWx:'',//wxh微信进来
    }
  },
  methods:{
    //初始化偏移量计算
   keyBoardInit(){
     this.getKeyBoardImg();//图片请求地址
        for (var i = 0; i < 5; i++) {
        var id = Math.ceil(Math.random() * 6);
        if (i == 0) {
          this.min = this.math[id];
          this.offset.push(this.min);
        } else if (i == 1) {
          this.max = this.math[id];
          this.offset.push(this.max);
        } else if (i == 2) {
          this.nub = this.math[id];
          this.offset.push(this.nub);
        } else if (i == 3) {
          this.minnub = this.math[id];
          this.offset.push(this.minnub);
        } else if (i == 4) {
          this.mark = this.math[id];
          this.offset.push(this.mark);
        }
      }
     // 数字键盘坐标对应赋值随机
      for(var i=0;i<this.keyNumAll.length;i++) {
        this.keyNumAll[i].keyLgh='2.'+this.nub;
        this.nub+=1;
      }
     this.keyLastLgh='2.'+this.nub;

    },
    //获取键盘的数字的显示图片
    getKeyBoardImg(){
      this.ajax({
       data:{
        "channelId": "APP",
        "data": {
          "memberToken":"111111111111",
          "pattern":"APP"
        },
        "merchantCode": "1999000000001",
        "orgCode": "T0000000",
        "storeCode": "15",
        "sysId": "T0000001",
        "transactionUuid": "e39cbd686ac54b27bd4ae93c5dfd45b8"
       },
       apiId: "XXXXXXXXXXXX"
       }).then(res => {
         if(res){
            var data=res.data
            this.dataImages=data.imageUrl;
            //前三行拼接键盘图标的位置
            var numberx=1;
            for(var i=0;i<this.keyNumAll.length;i++) {
              if(i==1){
                numberx-=37;
              }else if(i==2||i==3||i==5){
                numberx-=38;
              } else if(i==4||i==6||i==7||i==8){
                numberx-=37;
              }
              this.keyNumAll[i].keyValue={
                'background-image':'url('+this.dataImages+')',
                'background-position-y': '3px',
                'background-position-x': numberx+'px',
                'background-repeat': 'no-repeat',
                'background-size':'1800%'
              }
            }
            //固定拼接键盘第四行中间图标的位置
            numberx-=38;
            this.keyLastValue={
              'background-image':'url('+this.dataImages+')',
              'background-position-y': '3px',
              'background-position-x': numberx+'px',
              'background-repeat': 'no-repeat',
              'background-size':'1800%'
            }
            this.modulus = data.modulus;
            this.exponent = data.exponent;
            this.pictureKey= data.pictureKey;//图片钥匙
            this.keyPair=data.keyPair;
         }
       }).catch(err=>{

       })
    },
    /*键盘0-9数字输入值
     *type Y表示1-9之间的数字
     *type N表示0数字
     */
    inputInto(type,keyOne){
      //防止操作
      if(this.disKeyenter){
        //验证支付密码
        if(this.payPwdProcFlag=='VERIFYPWD'){
            this.psdOperate(type,keyOne,this.payPwdProcFlag);
        }
        //设置支付密码
        else if(this.payPwdProcFlag=='SETPWD'){
          //第一次进来密码存储
          if(this.newKeyNum==0){
            if(this.newKeyArray.length>6){
                return false
              }
              if(type=='Y'){
                this.newKeyArray.push(keyOne.keyLgh);
              }
              else if(type=='N'){
                this.newKeyArray.push(this.keyLastLgh);
              }
              //循环push 点击的值
              for(var i=0;i<this.newKeyArray.length;i++) {
                for (var j = 0; j < this.itemAll.length; j++) {
                  if(i==j){
                    this.itemAll[j].keyEnter='<b class="keyBoll"></b>'
                  }
                }
              }
            if(this.newKeyArray.length==6){
              this.newKeyNum++;//记录输入密码次数
              //清空密码点
              for (var j = 0; j < this.itemAll.length; j++) {
                this.itemAll[j].keyEnter=''
              }
            }
          }else{
              this.psdOperate(type,keyOne,this.payPwdProcFlag);
          }
        }       
      }
    },
    //输入密码操作
    psdOperate(type,keyOne,payPwdProcFlag){
       
      if(this.keyArray.length>6){
        return false
      }
      if(type=='Y'){
        this.keyArray.push(keyOne.keyLgh);
      }
      else if(type=='N'){
        this.keyArray.push(this.keyLastLgh);
      }
      //循环push 点击的值
      for(var i=0;i<this.keyArray.length;i++) {
        for (var j = 0; j < this.itemAll.length; j++) {
          if(i==j){
            this.itemAll[j].keyEnter='<b class="keyBoll"></b>'
          }
        }
      }
      //验证密码
      if(this.payPwdProcFlag=='VERIFYPWD'){
        if(this.keyArray.length==6) {
          this.disKeyenter=false;//防止点击操作
          this.submitPwdAll();//键盘提交
        }
      }
      //设置密码
      else if(this.payPwdProcFlag=='SETPWD'){
        if(this.keyArray.length==6) {
          //判别两个密码是否相同
          if(this.keyArray.toString()==this.newKeyArray.toString()) {
              this.disKeyenter=false;//防止点击操作
              this.submitPwdAll();//键盘提交
          }else{
              this.newKeyNum=0;//重置为0重新开始
              this.newKeyArray=[];//清空第一次存储的密码值
              this.keyArray=[];//清空第二次存储的密码值
              //清空密码点
              for (var j = 0; j < this.itemAll.length; j++) {
                this.itemAll[j].keyEnter=''
              }
            this.toast('输入的两次密码不一致')
          }
        }
      }
    },
    //清空密码值
    deleteEmpty(){
       //清空密码点
        for (var j = 0; j < this.itemAll.length; j++) {
          this.itemAll[j].keyEnter=''
        }
        if(this.payPwdProcFlag=='VERIFYPWD'){
          this.keyArray=[];//清空第存储的密码值
          this.offset=[];//清空偏移量
        }
         //设置支付密码
        else if(this.payPwdProcFlag=='SETPWD'){
          this.newKeyNum=0;//重置为0重新开始
          this.newKeyArray=[];//清空第一次存储的密码值
          this.keyArray=[];//清空第存储的密码值
          this.offset=[];//清空偏移量
        } 
    },
    //键盘完成按钮隐藏键盘
    completeEnter(){
     if(this.keyArray.length<6){
       this.toast('请输入6位数字密码')
     }
    },
    //删除密码值
    deleteValue(){
      if(this.payPwdProcFlag=='VERIFYPWD'){
        if(this.keyArray.length<=0){
          return false
        }
        this.keyArray=this.keyArray.slice(0,this.keyArray.length-1);
          for (var j = 0; j < this.itemAll.length; j++) {
            if(this.keyArray.length==j){
              this.itemAll[j].keyEnter=''
          }
        }
      }else if(this.payPwdProcFlag=='SETPWD'){
          if(this.newKeyArray.length<=0){
            return false
          }
          if(this.newKeyArray.length>0){
             this.newKeyArray=this.newKeyArray.slice(0,this.newKeyArray.length-1);
              for (var j = 0; j < this.itemAll.length; j++) {
                if(this.newKeyArray.length==j){
                  this.itemAll[j].keyEnter=''
                }
              }
         }
      }
    },
   //忘记密码跳转链接
    forgetGoenter(){
      if(process.env.NODE_ENV==='sit'){
          var  url='https://XXXXXXXXXXXXXXX?='+this.token;
         location.replace(url);

        }else if(process.env.NODE_ENV==='uat'){
          var  url='https://XXXXXXXXXXXXXXX?token='+this.token;
          location.replace(url);

        }else if(process.env.NODE_ENV==='prd'){
          var  url='https://XXXXXXXXXXXXXXX?token='+this.token;
          location.replace(url);
        }
     },
    //最终按钮提交密码
    submitPwdAll(){
        this.endVal='';//清空密码值
        var dataType='number';
        var dataName='pwd';
      if (this.endVal == undefined || this.endVal == "") {
         this.endVal = (";{\"dataType\":\"" + dataType + "\",\"name\":\"" + dataName + '\",\"value\":\"' + this.keyArray + "\"}");
       } else {
         this.endVal += (";{\"dataType\":\"" + dataType + "\",\"name\":\"" + dataName + '\",\"value\":\"' + this.keyArray + "\"}");
      }
      var encodeVal= this.splitEncoder(this.offset + this.endVal);
     
      this.ajax({
        data:{
        "channelId": "APP",
        "data": {
                "memberToken":"111111111111",
                "pattern":"APP",
                "pwdParamter":encodeVal,
                "pictureKey":this.pictureKey,
                "keyPair":this.keyPair,
              },
        "merchantCode": "1999000000001",
        "orgCode": "T0000000",
        "storeCode": "15",
        "sysId": "T0000001",
        "transactionUuid": "e39cbd686ac54b27bd4ae93c5dfd45b8"
       },
       apiId: "XXXXXXXXXXXXXXXX"
      }).then(res => {
        var res=res.data;
        if(res.code=='1'){
           this.deleteEmpty();//清空密码值
           if(this.payWx=='Y'){
             window.name=JSON.stringify({'md5Pwd':res.md5Pwd});
             window.history.go(-1);
           }else{
            //相应密码操作
            if(this.payType=='security'){
                this.setSecurityCard(res.md5Pwd);
              }
              //解绑操作
              else if(this.payType=='unbind'){
                this.bankOperate(res.md5Pwd);
              }
              //忘记密码流程
              else if(this.payType=='forget'){
                this.forgetFind(res.md5Pwd);
              }
              //鉴权验密操作
              else{
                this.payPwdCheck(res.md5Pwd);
              }
            }
        }else if(res.code=='2'){
           this.toast(res.failReason);
           this.deleteEmpty();//清空密码值
           this.disKeyenter=true;//重置点击操作
           this.keyBoardInit();//重新在绘制键盘
        }
      }).catch(err=>{
          this.deleteEmpty();//清空密码值
          this.disKeyenter=true;//重置点击操作
          this.keyBoardInit();//重新在绘制键盘
      })
    },
    // 加密并返回加密值
    splitEncoder(data){
      var length = data.length / 16;
      this.encodeVal = data.substring(0, 16);
      setMaxDigits(130);
      var key = new RSAKeyPair(this.exponent, "", this.modulus);
      this.encodeVal =encryptedStringKey(key, encodeURIComponent(this.encodeVal));
      for (var i = 1; i < length; i++) {
        this.encodeVal += ":" + encryptedStringKey(key, encodeURIComponent(data.substring(16 * i, 16 * (i + 1))));
      }
      return  this.encodeVal;
    },
  },
  created(){
    //密码框标题限制
     if(this.$route.query.payPwdProcFlag){
        this.payPwdProcFlag=this.$route.query.payPwdProcFlag;// payPwdProcFlag判别参数
        if(this.payPwdProcFlag=='VERIFYPWD'){
          G.setDocumentTitle('验证支付密码')
        }else if(this.payPwdProcFlag=='SETPWD'){ 
          G.setDocumentTitle('设置支付密码')
        }
     }else{
        G.setDocumentTitle('验证支付密码')
     }
  },
  mounted(){
   this.keyBoardInit();//初始化键盘
   if(localStorage.getItem("token")){
     this.token=localStorage.getItem("token");
   }
   else if(this.$route.query.token){
      this.token=this.$route.query.token;
   }
    if(this.$route.query.payType){
      this.payType=this.$route.query.payType;// payType验密入口类型
    }
    if(this.$route.query.securityCardType){
      this.securityCardType=this.$route.query.securityCardType;// securityCardType安全卡类型
    }
    if(this.$route.query.cardId){
      this.cardId=this.$route.query.cardId;// cardId卡id
    }
    if(this.$route.query.payWx){
      this.payWx=this.$route.query.payWx;// payWx支付方式
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .keyBoardBox{
    padding:0 1rem;
    position: absolute;
    top: 0px;
    left: 0px;
    background: #fff;
    width: 100%;
    height: 100%;
    z-index:9;
  }
  .keyTips{
    font-size:0.42rem;
    margin-top:3.03rem ;
    margin-bottom:0.73rem ;
    color:#333;
    text-align: left;
  }
  .keyBoardUl{
    display: flex;
    justify-content: space-between;
  }
  .keyBoardUl inputBox{
      flex: 1;
  }
  .keyForgetLink{
     float: right;
     font-size: 0.42rem;
     color:#118AE2 ;
     padding:0.34rem 0;
  }
  .inputBox{
    height:1.2rem;
    width:1.3rem;
    display: block;
    line-height:1.2rem;
    border-radius:4px;
    text-align:center;
  }
  .keyNumber{
    display: inline-block;
    width:40px;
    margin: 0.3rem auto;
    height:0.9rem;
    line-height: 0.9rem;
  }
</style>