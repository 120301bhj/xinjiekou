<script>
	export default {
		onLaunch: function() {
			this.Login()
		},
		onShow: function() {
		},
		onHide: function() {
		},
		
		methods: {
			Login(){
				return new Promise((resolve,reject)=>{
					var that = this
					uni.getLoginCode({
						success: function (loginRes) {
							console.log(loginRes)
						  that.getOpenid(loginRes.code).then(res=>{
							  uni.setStorageSync("openid",res.openid)
							  uni.setStorageSync("session_key",res.session_key)
							  resolve(res)
						  })
						},
					    fail: (err) => {
							// 处理登录失败的逻辑
							console.error('登录失败', err);
							reject(err);
					    }
					})
				})
			},
			
			getOpenid(code) {
			    return new Promise(function (resolve, reject) {
			        uni.request({
			            url: "https://dzxjk.com/api/xinjiekou_baidu/getSessionOpenid",
						data:{
							code:code
						},
			            success: (res) => {
							console.log(res)
			                let openid = res.data.data;
			                resolve(openid)
			            }
			        })
			    })
			}
		}
	}
</script>

<style>
	/*每个页面公共css */
</style>
