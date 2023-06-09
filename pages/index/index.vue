<template>
	<view class="bg">
		<view class="headTitle">Love Story</view>
		<view class="headSubTitle">输入口令，恋接心动的Ta ~</view>
		<view class="myCode">
			我的口令：{{ my_code }}
			<view class="copyButton" @click="copyCode">复制</view>
		</view>
		<input v-model="inputKey" type="text" class="myInput" placeholder="Ta一直在等你..."
			placeholder-class="myInputPlaceholder" />

		<view class="footer">
			<image src="../../static/heart.png" style="height: 199px; width: 257.92px" @click="Check()"></image><br>
			<image src="../../static/login_img.png" mode="widthFix"></image>
		</view>
	</view>
</template>

<script>
	let pollingTimer = null;
	export default {
		data() {
			return {
				inputKey: '',
				my_code: uni.getStorageSync('my_code')
			}
		},
		mounted() {
			// 组件挂载后启动轮询函数
			this.startPolling();
		},
		beforeDestroy() {
			// 组件销毁前停止轮询函数
			this.stopPolling();
		},
		methods: {
			// 绑定处理函数
			Check() {
				if (this.inputKey === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入口令',
						duration: 2000
					});
				} else {
					uni.showLoading({
						title: '绑定中...'
					});
					uni.request({
						url: "https://lovestory.ljcljc.cn/api/bind",
						method: 'POST',
						header: {
							'Content-Type': 'application/x-www-form-urlencoded',
							'token': uni.getStorageSync('ljc_token')
						},
						data: {
							code: this.inputKey
						},
						success(res) {
							uni.hideLoading()
							if (res.data.code != null && res.data.code === 400) {
								console.log(res.data.code)
								uni.showToast({
									icon: 'error',
									title: '登录状态过期',
									duration: 1000
								});
								uni.clearStorageSync('ljc_token')
								setTimeout(() => {
									uni.redirectTo({
										url: '/pages/login/login'
									})
								}, 1500)
							} else if (res.data.msg != null && res.data.code != 200) {
								uni.showModal({
									title: '提示',
									content: res.data.msg,
									showCancel: false
								})
							} else if (res.data.code === 200) {

								uni.showToast({
									icon: 'success',
									title: '绑定成功',
									duration: 1500
								});
								setTimeout(() => {
									uni.redirectTo({
										url: '/pages/heartValue/heartValue'
									})
								}, 1500)
							} else {
								uni.showToast({
									icon: 'error',
									title: '绑定失败',
									duration: 1500
								});
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
			},
			// 复制绑定码
			copyCode() {
				uni.setClipboardData({
					data: this.my_code,
					success() {
						uni.showToast({
							icon: 'success',
							title: '复制成功'
						})
					}
				});
			},
			startPolling() {
				// 每3秒执行一次轮询函数
				pollingTimer = setInterval(() => {
					this.checkBindStatus();
				}, 3000);
			},
			stopPolling() {
				// 停止轮询函数
				clearInterval(pollingTimer);
			},
			async checkBindStatus() {
				try {
					// 发起API请求
					const res = await uni.request({
						url: 'https://lovestory.ljcljc.cn/api/checkbind',
						header: {
						    'token': uni.getStorageSync('ljc_token')
						},
						method: 'GET',
					});
					if (res.data.code != null && res.data.code === 400) {
						console.log(res.data.code)
						uni.showToast({
							icon: 'error',
							title: '登录状态过期',
							duration: 1000
						});
						uni.clearStorageSync('ljc_token')
						setTimeout(() => {
							uni.redirectTo({
								url: '/pages/login/login'
							})
						}, 1500)
					}
					// 若isBind为true，则提示绑定成功并跳转到目标页面
					else if (res.data.data.isBind) {
						this.stopPolling();
						uni.showToast({
							title: '绑定成功',
							icon: 'success',
						});
						uni.navigateTo({
							url: '/pages/heartValue/heartValue',
						});
					}
				} catch (err) {
					console.error(err);
				}
			}
		}
	}
</script>

<style>
	.bg {
		background: linear-gradient(197.99deg, #2C9AFF -14.52%, #9C74F3 16.96%, #EC81B4 54.17%, #F1BC97 74.34%, #F4E283 100%);
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

	.headSubTitle {
		height: 25px;
		margin: 30px 10px;

		font-style: normal;
		font-weight: 500;
		font-size: 18px;
		line-height: 25px;
		text-align: left;

		color: #FFFFFF;
	}

	.myCode {
		height: 20px;
		margin: -20px 20px 20px 10px;

		font-style: normal;
		font-weight: 500;
		font-size: 18px;
		line-height: 25px;
		text-align: left;

		color: #FFFFFF;

		display: flex;
	}

	.copyButton {
		height: 23px;
		width: 40px;
		background-color: coral;
		border-radius: 6px;
		font-size: 14px;
		line-height: 23px;
		text-align: center;
	}

	.myInput {
		width: 50%;
		height: 46px;
		left: 24px;
		top: 206px;

		background: rgb(255, 255, 255, 0.6);

		border-radius: 25px;

		padding-left: 22px;
		font-weight: 400;
		font-size: 18px;
		line-height: 18px;
		color: #63479A;
		opacity: 0.6;
	}

	.myInputPlaceholder {
		font-weight: 500;
		font-size: 18px;
		line-height: 25px;
		text-align: left;

		color: #FFFFFF;

	}

	.footer {
		margin-top: 17px;
		text-align: center;
	}
</style>