<template>
	<view class="bg">
		<view class="headTitle">Love Story</view>
		<view class="myForm">
			<view class="itemGroup">
				<label class="itemLabel">姓名</label>
				<input v-model="name" type="text" class="itemInput"/>
			</view>
			<view class="itemGroup">
				<label class="itemLabel">手机号</label>
				<input v-model="tel" type="text" class="itemInput" placeholder="手机号将作为账号" placeholder-class="placeholderClass"/>
			</view>
			<view class="itemGroup">
				<label class="itemLabel">设置密码</label>
				<input v-model="passwd" type="password" class="itemInput"/>
			</view>
			<view class="footer">
				<view class="submitButton" @click="Register()">注册</view>
				<view style="margin-top: 10px; color: #ffffff;" @click="toLogin()">已经有账号了？点击登录</view>
			</view>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				name: '',
				tel: '',
				passwd: ''
			}
		},
		methods: {
			Register() {
				// 使用正则表达式判断手机号是否合法函数
				function checkChinesePhoneNumber(number) {
				  const regex = /^1[3456789]\d{9}$/;
				  return regex.test(number);
				}
				if(this.name === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入姓名',
						duration: 2000
					});
				}
				else if(this.tel === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入手机号',
						duration: 2000
					});
				}
				else if(!checkChinesePhoneNumber(this.tel)) {
					uni.showToast({
						icon: 'error',
						title: '手机号不合法',
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
						title: '请稍后'
					});
					uni.request({
						url:"https://lovestory.ljcljc.cn/api/register",
						method:'POST',
						header:{
							'Content-Type': 'application/x-www-form-urlencoded'
						},
						data:{
							name: this.name,
							username: this.tel,
							password: this.passwd
						},
						success(res) {
							if(res.data.code === 200) {
								uni.hideLoading()
								uni.showToast({
									icon: 'success',
									title: '注册成功',
									duration: 1500,
									complete: () => {
										setTimeout(() => {
											uni.navigateTo({
												url: '/pages/login/login'
											})
										}, 1500)
									}
								})
							}
							else if(res.data.code === 100) {
								uni.hideLoading();
								uni.showToast({
									icon: 'error',
									title: '用户已存在',
									duration: 1500
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
			// 前往登录页
			toLogin() {
				uni.navigateTo({
					url: '/pages/login/login'
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
		padding-top: 130px;
		width: 100%;
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
		padding: 0 30px;
	}
	.itemGroup {
		margin-bottom: 50px;
	}
	.itemInput {
		border-bottom: 1px solid rgba(255, 255, 255, 0.8);
		border-top:0px;
		border-left:0px;
		border-right:0px;
		font-family: 'Roboto';
		font-style: normal;
		font-weight: 400;
		font-size: 28px;
		line-height: 28px;
		letter-spacing: 0.2px;
		color: #FFFFFF;
		height: 70px;
		width: 100%;
		line-height:30px;
	}
	.itemLabel {
		width: 80%;
		height: 18px;
		
		font-family: 'PingFang SC';
		font-style: normal;
		font-weight: 500;
		font-size: 18px;
		line-height: 18px;
		
		color: #FFFFFF;
		opacity: 0.8;
	}
	.footer {
		margin-top: 110px;
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
	.placeholderClass {
		color: #ebebeb;
		opacity: .3;
		font-size: 20px;
	}
</style>
