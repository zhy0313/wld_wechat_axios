<template>
  <div class="personalAuthentication">
    <header-top linkUs="true" headTitle="获取额度" ></header-top>
    <group>
      <cell title="工作城市" :value="city" is-link @click.native="toSelectCity"></cell>

      <popup-picker  
        title="教育程度" 
        :data="edus" 
        v-model="edu"
        ref="edu"
        show-name 
        :columns="1"
        @on-change="onEduChange" 
        placeholder="请选择教育程度">
      </popup-picker>

      <popup-picker  
        title="婚姻状况" 
        :data="marries" 
        v-model="marry" 
        :columns="1"
        show-name 
        ref="marry"
        @on-change="onMarryChange" 
        placeholder="请选择婚姻状况">
      </popup-picker>

      <popup-picker  
        title="职业类别" 
        :data="jobs" 
        v-model="job" 
        :columns="1"
        ref="job"
        show-name 
        @on-change="onJobChange" 
        placeholder="请选择职业">
      </popup-picker>

      <x-input 
        title="月收入" 
        placeholder="请输入月收入工资" 
        :show-clear="true" 
        required 
        type="number" 
        :max="5"
        text-align="right" 
        v-model="cardSalary" >
        <span slot="right" style="padding-left:10px;">元</span>
      </x-input>
    </group>
    <div class="uploadImg">
      <p>请拍摄身份证照片</p>
      <div class="upload">
        <div  class='imgs' >
          <img :src="faceImg" v-if="faceImg"/>
          <img src="../../../images/idCardFace.png" v-else/>
          <input @change='addFaceImg'  type="file" accept="image/*" capture="camera" />
        </div>
        <p :class="[faceImg  ? 'success' : '']">{{faceImg ? '上传成功' : '请拍摄身份证正面'}}</p>
      </div>
      <div class="upload">
        <div  class='imgs' >
          <img :src="backImg" v-if="backImg"/>
          <img src="../../../images/idCardBack.png" v-else/>
          <input @change='addBackImg'  type="file" accept="image/*" capture="camera" />
        </div>
        <p :class="[backImg  ? 'success' : '']">{{backImg ? '上传成功' : '请拍摄身份证反面'}}</p>
      </div>
      <div class="upload">
        <div  class='imgs' >
          <img :src="handImg" v-if="handImg"/>
          <img src="../../../images/idCardHand.png" v-else/>
          <input @change='addHandImg'  type="file" accept="image/*" capture="camera" />
        </div>
        <p :class="[handImg  ? 'success' : '']">{{handImg ? '上传成功' : '请拍摄手持身份证照'}}</p>
      </div>
    </div>
<!-- 图片上传成功或显示 -->
    <div v-if="faceImg">
      <group>
        <cell  title="姓名"  is-link>
          <input slot="value" v-model="name"  class="idCardName" placeholder="请输入姓名"/>
        </cell>
        <cell  title="证件号" :value="cardNo"></cell>
      </group>
    </div>
    <x-button  @click.native="onSubmit" :disabled="name === '' || city === '' || edu===[] || marry===[] || job===[] || faceImg==='' || backImg==='' || handImg==='' || cardSalary===''">下一步</x-button>
    <div>
      <confirm v-model="showComfirm"
        title="请核对您的基本信息"
        @on-confirm="onConfirm"
        cancel-text="修改">
          <div class="info"><span>姓名</span><span>{{name}}</span></div>
          <div class="info"><span>身份证号</span><span>{{cardNo}}</span></div>
          <div class="info"><span>工作城市</span><span>{{city}}</span></div>
          <div class="info"><span>教育程度</span><span>{{$refs.edu&&$refs.edu.getNameValues()}}</span></div>
          <div class="info"><span>婚姻状况</span><span>{{$refs.marry&&$refs.marry.getNameValues()}}</span></div>
          <div class="info"><span>职业类别</span><span>{{$refs.job&&$refs.job.getNameValues()}}</span></div>
          <div class="info"><span>进卡工资</span><span>{{cardSalary}}</span></div>
      </confirm>
    </div>
  </div>
</template>

<script>
  import{ XButton,Group,PopupPicker,Cell,XInput,XHeader,Confirm } from 'vux';
  import {getQuery,customToast,getStorage,compressImg } from '../../../config/mUtils'
  import { getDictionaries,uploadFacePic,uploadBackPic,uploadHandPic,getIdCardInfo,getOrder,saveOrderInfo } from '../../../service/getData'
  
  import headerTop from '../../../components/header/head';
  export default {
    data() {
      return{
        city:'',
        edu:[],
        edus:[], 
        marry: [],
        marries:[],
        job:[],
        jobs:[],
        cardSalary:'',
        orderId:'',
        showComfirm:false,
        name: '',
        cardNo: '',
        isShow:false,
        faceImg:'',
        backImg:'',
        handImg:'',
        
      }
    },
    created() {    
      //获取数据字典
      getDictionaries().then((data) =>{           
        this.edus = data.data.education;
        this.marries = data.data.marriage;
        this.jobs = data.data.occupationalClass;
      })
      this.city = getQuery('city') || '请选择城市';
      if(getQuery('city')) {  //如果是从城市选择页回来，则读取localStorage中的数据
        const personalDraft = JSON.parse(getStorage('personalDraft')); 
        this.faceImg = personalDraft.faceImg;
        this.backImg = personalDraft.backImg;
        this.handImg = personalDraft.handImg;
        this.name = personalDraft.name;
        this.cardNo = personalDraft.cardNo;
        return;
      }
      const accountId = window.localStorage.getItem('accountId');
      getOrder({  //根据accountId获取orderId
        accountId:accountId
      }).then((data) =>{
        if(data.error.error){
          customToast(data)
          return ;
        }
        this.city = data.data.order.workCity;   //获取上次填写的城市名
        
        this.edu[0] = data.data.order.education;  //获取上次填写的教育程度
        console.log(this.edu)
        this.marry[0] = data.data.order.marriage;   //获取上次填写的婚姻状况
        this.job[0] = data.data.order.profession;   //获取上次填写的工作情况
        this.cardSalary = data.data.order.salary;   //获取上次填写的薪水
        // console.log(this.edu)
        this.orderId = data.data.order.orderId;
        window.localStorage.setItem('orderId',this.orderId);
        getIdCardInfo({ //获取身份证信息
          orderId:this.orderId
        }).then(data =>{
          this.faceImg = data.data.idcardImageObverseBase64;
          this.backImg = data.data.idcardImageReverseBase64;
          this.handImg = data.data.holdingIdcardBase64;
          this.name = data.data.name;
          this.cardNo = data.data.idcardNo;
        })
      }) 
      
    },

    components:{
      XButton,Group,PopupPicker,Cell,XInput,XHeader,Confirm,headerTop
    },

    computed:{
       
    },

    methods:{
      toSelectCity() {
        const draft = {
          faceImg:this.faceImg,
          backImg:this.backImg,
          handImg:this.handImg,
          name:this.name,
          cardNo:this.cardNo,
        }
        localStorage.setItem('personalDraft',JSON.stringify(draft));
        this.$router.push('/cityPicker?from=auth');
      },
      
      addFaceImg(event){ //上传身份证正面图片
        var imgFile=event.target.files[0];
        //判断类型是不是图片 
        if(!/image\/\w+/.test(imgFile.type)){     
          alert("请确保文件为图像类型");   
          return false;   
        } 
        var reader =new FileReader();
        reader.readAsDataURL(imgFile);
        var that=this;
        reader.onloadend=function(){  //文件读取结束的时候上传到服务器
         	const orderId = window.localStorage.getItem("orderId");
          that.$vux.loading.show({
            text: '上传中'
          })
          var img = new Image();
          img.src = reader.result;
          let width = document.querySelector(".upload img").offsetWidth;
          let height = document.querySelector(".upload img").offsetHeight;
          var result = compressImg(img,width,height)
          alert("压缩前"+reader.result.length+",压缩后"+result.length)
          uploadFacePic({
             idcardImageObverseBase64:reader.result.split(",")[1],  //只需要，后面的信息
            //  idcardImageObverseBase64:result.split(",")[1],  //只需要，后面的信息
             orderId:orderId
           }).then((data) =>{
            that.$vux.loading.hide();
             //上传成功回调，响应身份证号，姓名
            if(data.error.error){
              customToast(data)
              return ;
            }
            that.cardNo = data.data.content.cardNo;
            that.faceImg = reader.result;
           })
        }   
      },
      addBackImg(event){ //上传身份证反面图片
        var imgFile=event.target.files[0];
        //判断类型是不是图片 
        if(!/image\/\w+/.test(imgFile.type)){     
          alert("请确保文件为图像类型");   
          return false;   
        } 
        var reader =new FileReader();
        reader.readAsDataURL(imgFile);
        var that=this;
        reader.onloadend=function(){  //文件读取结束的时候上传到服务器
          const orderId = window.localStorage.getItem("orderId");
         	that.$vux.loading.show({
            text: '上传中'
          })
          var img = new Image();
          img.src = reader.result;
          let width = document.querySelector(".upload img").offsetWidth;
          let height = document.querySelector(".upload img").offsetHeight;
          var result = compressImg(img,width,height)
          alert("压缩前"+reader.result.length+",压缩后"+result.length)
          uploadBackPic({
            idcardImageReverseBase64:reader.result.split(",")[1],
            orderId:orderId
          }).then((data) =>{
            //成功回调
            that.$vux.loading.hide();
            if(data.error.error){
              customToast(data)
              return ;
            }
            
            that.backImg = reader.result;
          })
        }   
      },
      addHandImg(event){ //上传图片
        var imgFile=event.target.files[0];
        //判断类型是不是图片 
        if(!/image\/\w+/.test(imgFile.type)){     
          alert("请确保文件为图像类型");   
          return false;   
        } 
        var reader =new FileReader();
        reader.readAsDataURL(imgFile);
        var that=this;
        reader.onloadend=function(){  //文件读取结束的时候上传到服务器
          const orderId = window.localStorage.getItem("orderId");
         	that.$vux.loading.show({
            text: '上传中'
          });
          var img = new Image();
          img.src = reader.result;
          let width = document.querySelector(".upload img").offsetWidth;
          let height = document.querySelector(".upload img").offsetHeight;
          var result = compressImg(img,width,height)
          alert("压缩前"+reader.result.length+",压缩后"+result.length) 
          uploadHandPic({
            facebase64:reader.result.split(",")[1],  
            orderId:orderId
          }).then((data) =>{
            that.$vux.loading.hide()
            if(data.error.error){
              customToast(data)
              return ;
            }
            that.handImg = reader.result;
          })
          
        }   
      },
      onSubmit() {  //提交表单
        this.showComfirm = true;
      },
      onConfirm() { //确认信息
        const accountId = window.localStorage.getItem('accountId');
        const orderId = window.localStorage.getItem('orderId');
        saveOrderInfo({
          accountId:accountId,
          orderId: orderId,
          city:this.city,
          education:this.edu[0],
          marriage:this.marry[0],
          profession:this.job[0],
          salary:this.cardSalary,
          customer_name:this.name
        }).then((data) =>{
          console.log(data)
          if(data.error.error) {
            customToast(data)
            return ;
          }
          this.showComfirm = false;
          this.$router.push('/getLimit/savingCard')
        })
        
      },   
     
      onEduChange(val) {
        this.edu[0] = val;
      },
      onMarryChange(val) {
        this.marry[0] =val;
      },
      onJobChange(val) {
        this.job[0] = val;
      },
     
    },
  }

</script>

<style lang="scss">
 
  .personalAuthentication {
    .vux-cell-primary{
      white-space:nowrap ; 
    }
    .vux-label{
      text-align:left !important;
    }

    .weui-cells{
      margin-top:0;
      font-size:14px;
    }
    .weui-label{
      text-align:left !important;
    }
    
    .idCardName{
      text-align: right;
    }

    /*图片上传*/
    .uploadImg{
      display:flex;
      margin-top:5px;
      flex-flow:row wrap;
      justify-content:space-around;
      background-color: #fff;
      p{
        width: 96%;
        padding-left:4%;
        font-family: PingFangSC-Regular;
        font-size: 14px;
        color: #666666;
        line-height: 14px;
        margin: 0.2rem 0;
      }
      
      .upload{
        width: 40%;
        height: auto;
        padding-bottom: 0.24rem;
        p{
          font-family: PingFangSC-Regular;
          font-size: 12px;
          color: #999999;
          line-height: 12px;
          width: 100%;
          text-align: center;
        }
        .success{
          color:#5FCC28;
        }
      }
      input{
        position: absolute;
        top: 0px;
        left: 0px;
        width: 100%;
        height: 100%;
        z-index: 11; 
        opacity: 0;   
      }
      .imgs{
        width: 100%;
        height:98px ;
        margin-right: 0.3%;
        position: relative;
        overflow: hidden;
        img{
          width: 100%;
          position: absolute;
          top:50%;
          left:-50%;
          transform:translate(50%,-50%);
        }
        span{
          position: absolute;
          width: 20px;
          height: 20px;
          right: 5px;
          /*top:3px;*/
        }
      }
    }
    button{
      width: 56% !important;
      margin:0 22%;
      color:#fff;
      background: linear-gradient(to right, #00E6E2 0%, #00CDD4  100%);
      border-radius:0.6rem;
      height:1.2rem;
      margin-top:3.6%;
    }
    button[disabled]{
      background: #CFCFCF;
    }
    /*确认框样式*/
    .weui-dialog{
      top:50px;
      transform: translate(-50%,0)
    }
    .weui-dialog__btn_primary {
      color: #00BBCC;
      font-size: 14px;
    }
    .weui-dialog__btn_default{
      color:#999;
      font-size:14px;
    }
    .weui-dialog__hd{
      padding: 0.83rem 0;
    }
    .info{
      width: 100%;
      display: flex;
      justify-content: space-between;
      margin-bottom:0.55rem;
      .weui-cell{
        border:none;
        margin:0;
        padding:0;
        color:#333;
      }
    }
  }
  
 
</style>
