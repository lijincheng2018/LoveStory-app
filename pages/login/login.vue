<template>
	<view class="bg">
		<view class="headTitle">Love Story</view>
		<view class="myForm">
			<view class="itemGroup">
				<input v-model="tel" type="text" class="itemInput" placeholder-class="itemInputPlaceholder" placeholder="请输入账号"/>
			</view>
			<view class="itemGroup">
				<input v-model="passwd" type="password" class="itemInput" placeholder-class="itemInputPlaceholder" placeholder="请输入密码"/>
			</view>
		</view>
		<view class="footer">
			<view class="submitButton" @click="Login()">登录</view>
			<view style="margin-top: 10px; color: #ffffff;" @click="toRegister">还没有账号？点击注册</view>
			<view style="text-align: center;">
				<image class="footerImg" src="../../static/login_img.png" mode="widthFix"></image>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				tel: '',
				passwd: ''
			}
		},
		methods: {
			Login() {
				// 使用正则表达式校验账号是否合法函数
				function checkChinesePhoneNumber(number) {
				  const regex = /^1[3456789]\d{9}$/;
				  return regex.test(number);
				}
				if(this.tel === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入账号',
						duration: 2000
					});
				}
				else if(!checkChinesePhoneNumber(this.tel)) {
					uni.showToast({
						icon: 'error',
						title: '账号不合法',
						duration: 2000
					});
				}
				else if(this.passwd === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入密码',
						duration: 2000
					});
				}
				else {
					uni.showLoading({
						title: '登录中...'
					});
					uni.request({
						url:"https://lovestory.ljcljc.cn/api/login",
						method:'POST',
						header:{
							'Content-Type': 'application/x-www-form-urlencoded'
						},
						data:{
							username: this.tel,
							password: this.passwd
						},
						success(res) {
							if(res.data.code === 200 || res.data.code === 201) {
								uni.hideLoading()
								uni.setStorageSync('ljc_token', res.data.data.token)
								uni.setStorageSync('my_code', res.data.data.code)
								uni.showToast({
									icon: 'success',
									title: '登录成功',
									duration: 1500,
									complete: () => {
										setTimeout(() => {
											if(res.data.code === 200) {
												 uni.navigateTo({
												 	url: '/pages/index/index'
												})
											}
											else if(res.data.code === 201) {
												uni.navigateTo({
													url: '/pages/heartValue/heartValue'
												})
											}
											
										}, 1500)
									}
								})
							}
							else {
								uni.hideLoading()
								uni.showToast({
									icon: 'error',
									title: '登录失败',
									duration: 1500,
								})
							}
						},
						fail(err) {
							uni.showToast({
								title: '网络出错，请重试',
								icon: 'none',
								duration: 2000,
							});
							console.error(err);
						}
					})		
				}
			},
			// 前往注册页
			toRegister() {
				uni.navigateTo({
					url: '/pages/register/register'
				})
			}
		}
	}
</script>

<style>

	.bg {
		background: linear-gradient(197.99deg, #2C9AFF -14.52%, #9C74F3 16.96%, #EC81B4 54.17%, #F1BC97 74.34%, #F4E283 100%) ;
		background-size: 100% 100%;
		min-height: 100vh;
		padding: 0 37rpx;
		padding-top: 130px;
	}
	.headTitle {
		position: absolute;
		width: 183px;
		height: 36px;
		left: 24px;
		top: 109px;
		
		font-style: normal;
		font-weight: 500;
		font-size: 36px;
		line-height: 36px;
		
		text-align: center;
		
		color: #FFFFFF;
	}
	.myForm {
		margin-top: 116px;
		width: 100%;
		box-sizing: border-box;
	}
	.itemGroup {
		margin-bottom: 24px;
		padding: 0;
	}
	.itemInput {
		height: 52px;
		
		background: rgb(255, 255, 255, 0.4);
		mix-blend-mode: normal;
		border-radius: 41px;
		padding-left: 22px;
		font-weight: 400;
		font-size: 18px;
		line-height: 18px;
		color: #020216;
		opacity: 0.6;
	}
	.itemInputPlaceholder{
		color: #0A0874;
	}
	.footer {
		margin-top: 73px;
	}
	.submitButton {
		width: 100%;
		height: 52px;
		background: #FFFFFF;
		opacity: 0.7;
		border-radius: 32px;
		text-align: center;
		
		font-style: normal;
		font-weight: 600;
		font-size: 18px;
		line-height: 52px;
		color: #63479A;
	}
	.footerImg {
		position: relative;
		margin-top: 45px;
	}
</style>
