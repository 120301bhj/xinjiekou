<template>
	<view class="page">
		<view class="uni-margin-wrap">
			<swiper class="swiper" circular :indicator-dots="indicatorDots" :autoplay="autoplay" :interval="interval" :duration="duration">
				<swiper-item v-for="item in detailsArray.pics" :key='item.id'>
					<image class="image" :src="item"></image>
				</swiper-item>
			</swiper>
		</view>
		<view class="content">
			<view class="neirong">
				<rich-text :nodes="detailsArray.content"></rich-text>
			</view>
		</view>
		<view class="fixed-bar">
			<view class="yuliu">
				<button v-if="!phone" type="primary" class="zui-btn" :animation="animationData" open-type="getPhoneNumber" @getphonenumber="TureGetSessionKeyPhone">获取报价</button>
				<button v-else type="primary" class="zui-btn" :animation="animationData" @click="yuliuTel">获取报价</button>
			</view>
			<view class="boda">
			  	<button type="primary" class="mobile-btn" :animation="animationData" @click="callPhone">拨打电话</button>
			</view>
		</view>
		
		<!-- 授权弹框 start -->
		<view class="zan-dialog" v-show="shouquanPhone">
			<view class="zan-dialog__mask" style="z-index: 100;background-color: #1618239e;"/>
				<view class="zan-dialog__container" style="height:20%;top: 50%;width: 90%;left: 50%;transform: translate(-50%,-50%);border-radius: 1rem;z-index: 110;">
				  <view style="background-color: #f2f2f2;border-radius: 1rem;">
					<text @click="phoneDialog" style="color: #d2d2d2;margin-left: 20rpx;">×</text>
					<view class="form-wrap" style="text-align: center;padding: 0rem 1rem 1.11rem;">
						<text class="h3" style="font-weight: 900;">服务通知</text>
						<view class="form_box">
							<view style="padding:1rem">
								为了更好的服务体验，请同意授权
							</view>
							<view class="btn_area" style="padding-top: 1rem;" @click="yunxvClick">
								<button style="background: #3c76ff;" type="primary" class="middle-btn" open-type="getPhoneNumber" @getphonenumber="getPhoneNumber">同意</button>
							</view>
						</view>
					</view>
				  </view>
				</view>
			</view>
		</view>
		<!-- 开盘授权弹框通知 end  -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				detailsArray:{
					title:''
				},
				indicatorDots: false,
				autoplay: true,
				interval: 2000,
				duration: 900,
				animationData:{},
				
				openid:"",
				session_key:'',
				laiyuanwangzhan:'',
				cid:0,
				document_id:0,
				shouquanPhone:false,//授权手机号
				phone:''
			}
		},
		onLoad(option) {
			console.log(option)
			this.document_id = option.document_id?option.document_id:280
			this.laiyuanwangzhan = option.laiyuanwangzhan?option.laiyuanwangzhan:''
			this.phone = uni.getStorageSync("phone")?uni.getStorageSync("phone"):''
			this.detailsData(this.document_id)
			getApp().Login().then(()=>{
				this.openid = uni.getStorageSync("openid")?uni.getStorageSync("openid"):'',
				this.session_key = uni.getStorageSync("session_key")?uni.getStorageSync("session_key"):'',
				this.CheckShouQuan()
			})
		},
		onShow() {
			this.doAnimation()
		},
		methods:{
			yuliuTel(){
				console.log(this.phone)
				uni.showToast({
					title: '预留成功',
					mask: false,
				})
			},
			/**
			 * @param {Object} e
			 * 点击允许
			 */
			yunxvClick(){
			    uni.request({
					url: "https://dzxjk.com/api/xinjiekou_baidu/clickpop",
					data:{
						"openid":this.openid,
						"cid":this.cid,
						'type':'2',
						"laiyuanwangzhan":this.laiyuanwangzhan,
						"document_id":this.document_id,
						"document_title":this.detailsArray.title
					},
					method:'POST',
					success: (res) => {},
					fail: (res) => {},
			    });
			},
			
			/**
			 * @param {Object} e
			 * 弹窗  授权手机号
			 */
			getPhoneNumber(e) {
				console.log(e)
			    if(e.detail.errMsg == "getPhoneNumber:ok"){
			      this.TongjiPhone(1)
			    }else{
			      this.TongjiPhone(2)
			    }
			    if(!e.detail.iv){
			      this.phoneDialog();
			      return false;
			    }
			    this.phoneDialog();
			    uni.request({
			        url: "https://dzxjk.com/api/xinjiekou_baidu/getBaiduPhone",
			        data:{
						'laiyuanwangzhan':this.laiyuanwangzhan,
						'iv':e.detail.iv,
						'encryptedData':e.detail.encryptedData,
						'session_key':this.session_key,
						'cid':this.cid, 
						'openid':this.openid,
						'status':'shangjianaimo',
						"document_id":this.document_id,
						"document_title":this.detailsArray.title
			        },
					method:'POST',
					success: (res) => {
						console.log(res)
						if(res.data.code==1){
							uni.setStorageSync('phone',res.data.data.phone);
							this.phone = res.data.data.phone
							uni.showToast({
								title: '授权成功',
								mask: false,
							})
						}else{
							uni.showToast({
								title: '授权失败，请点击下方【预约电话】'
							})
						}
					},
					fail: (res) => {
						console.log(res)
			        },
			    });
			},
			
			TureGetSessionKeyPhone(e) {
				console.log(e)
			    uni.request({
			        url: "https://dzxjk.com/api/xinjiekou_baidu/TureGetSessionKeyPhone",
			        data:{
						'laiyuanwangzhan':this.laiyuanwangzhan,
						'iv':e.detail.iv,
						'encryptedData':e.detail.encryptedData,
						'session_key':this.session_key,
						'cid':this.cid,
						'openid':this.openid,
						'status':'shangjianaimo',
						"document_id":this.document_id,
						"document_title":this.detailsArray.title
			        },
					method:'POST',
					success: (res) => {
						console.log(res)
						if(res.data.code==1){
							uni.setStorageSync('phone',res.data.data.phone);
							this.phone = res.data.data.phone
							uni.showToast({
								title: '授权成功',
								mask: false,
							})
						}
					},
					fail: (res) => {
						console.log(res)
			        },
			    });
			},
			
			/**
			 * 判断是否已授权手机号
			 */
			CheckShouQuan(){
				uni.request({
					url: "https://dzxjk.com/api/xinjiekou_baidu/CheckShouquan", // 需要开发者访问业务服务端，同时业务服务端通过 code 换取 session_key 等信息，返回自定义登陆态
					method: "POST",
					data: {
						'cid':this.cid,
						'openid': this.openid,
						'laiyuanwangzhan':this.laiyuanwangzhan,
						'status':'shangjianaimo',
						"document_id":this.document_id,
						"document_title":this.detailsArray.title
					},
					success: res => {
					  if(res.data.data==1){
						  this.phoneDialog()
					  }
					},
					fail: err => {
					}
				})
			},
			//上来就弹出 授权手机号
			phoneDialog(){
			    this.shouquanPhone = !this.shouquanPhone
			},
			//统计点击 电话  同意或拒绝
			TongjiPhone(state){
			    wx.request({
			      url: "https://dzxjk.com/api/xinjiekou_baidu/tongjiphone",
			      data:{
			        "openid":this.openid,
			        "cid":this.cid,
			        "type":'百度',
			        "state":state,
			        "laiyuanwangzhan":this.laiyuanwangzhan,
					"document_id":this.document_id,
					"document_title":this.detailsArray.title
			      },
			      method:'POST',
			      success: (res) => {
			      },
			      fail: (res) => {
			      },
			    });
			},
			/**
			 * 拨打电话
			 */
			callPhone() {
				var phoneNumber = this.detailsArray.phone?this.detailsArray.phone+','+this.detailsArray.extension:this.detailsArray.message_phone
			    uni.makePhoneCall({
			      phoneNumber: phoneNumber, // 电话号码
			      success: function () {
			        console.log('拨号成功');
			      },
			      fail: function (e) {
			        console.log(e);
			      }
			    });
			},
			/**
			 *  动画部分
			 */
			doAnimation(){
				var animation = uni.createAnimation({
				  duration: 500,
				  timingFunction: 'ease',
				})
				this.animation = animation
				var next = true;
				//连续动画关键步骤
				setInterval(function () {
					if (next) {
						this.animation.scale(0.95).step()
						next = !next;
					} else {
						this.animation.scale(1).step()
						next = !next;
					}
					this.animationData = animation.export();
				}.bind(this), 500)
			},
			/**
			 * @param {Object} id
			 * 详情内容
			 */
			detailsData(id){
				var that = this
				uni.request({
					url:"https://dzxjk.com/addons/ldcms/api.v1/detail",
					data:{
						id:id
					},
					method:"POST",
					success: (res) => {
						that.detailsArray = res.data.data
						that.cid = res.data.data.cid
						console.log(res.data.data)
					},fail: (red) => {
						
					}
				})
			}
		}
	}
</script>

<style>
	.page{
		height: 100%;
		background-color: #f7f7f7;
		position: relative;
		padding-bottom: 50px;
	}
	.uni-margin-wrap{
		height: 400rpx;
	}
	.uni-margin-wrap .swiper{
		height: 100%;
	}
	.uni-margin-wrap .swiper .image{
		width: 100%;
		height: 100%;
	}
	
	.content{
		padding: 40rpx;
	}
	.content .content-title,.content .content-tag{
		font-size: 38rpx;
		margin-bottom: 32rpx;
		font-weight: bolder;
	}
	.content .neirong{
		font-size: 28rpx;
	}
	.content .biaoqian{
		padding:7rpx 10rpx;
		background:#d0d0d0;
		font-size:20rpx;
		margin-right: 22rpx;
	}
	
	.content .content-tag text:first-child{
		color: red;
	}
	
	.content .content-tag text:nth-child(2){
		color: red;
	}
	
	/**
	 * 底部导航
	 */
	.fixed-bar {
	    position: fixed;
	    bottom: 0;
	    left: 0;
	    right: 0;
	    height: 100rpx; /* 固定栏的高度 */
	    background-color: #fff;
	    color: black;
	    text-align: center;
	    line-height: 100rpx; /* 垂直居中文本 */
	}
	
	.fixed-bar .yuliu{
		width: 40%;
		float: left;
		margin-left: 50rpx;
	}
	.fixed-bar .boda{ 
		width: 40%;
		float: right;
		margin-right: 50rpx;
	}
	
	.fixed-bar .yuliu button {
		width:initial;
		height:80rpx;
		line-height:80rpx;
		background-color: #00aaff;
		font-size:38rpx;
		box-shadow: 5rpx 10rpx 20rpx #0076b1;
	}
	.mobile-btn {
		width:initial;
		height:80rpx;
		line-height:80rpx;
		font-size:38rpx;
		box-shadow: 5rpx 10rpx 20rpx #177d16;
	}
	
	/**
	 * 弹窗
	 */
	.zan-dialog__mask {
	  position: fixed;  
	  top: 0;  
	  left: 0;  
	  right: 0;  
	  bottom: 0;  
	  z-index: 10;  
	  background: #161823c7;
	  display: block;
	}
	.zan-dialog__container {
	  position: fixed;  
	  /* bottom: 0; */
	  top: 300rpx;
	  left: 0;
	  width: 750rpx;  
	  height: 750rpx;  
	  background: white;  
	  transform: translateY(150%);  
	  transition: all 0.4s ease;  
	  z-index: 12;
	}
	.zan-dialog--show .zan-dialog__container {
	  transform: translateY(0);
	}
</style>