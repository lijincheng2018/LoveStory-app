<template>
	<view class="bg">
		<view class="backIcon" style="display: flex;" @click="toheartValue()">
			<image src="../../static/back.png" style="height: 16px; width: 9px;"></image>
		</view>
		<view class="title">设置</view>
		<view class="form-item">
			<view class="label">旧密码</view>
			<input class="input" type="password" placeholder="请输入旧密码" v-model="oldPassword" />
		</view>
		<view class="form-item">
			<view class="label">新密码</view>
			<input class="input" type="password" placeholder="请输入新密码" v-model="newPassword" />
		</view>
		<view class="form-item">
			<view class="label">确认密码</view>
			<input class="input" type="password" placeholder="请再次输入新密码" v-model="confirmPassword" />
		</view>
		<view class="btn-wrapper">
			<button class="btn btn-update" @click="handleChangePassword">修改密码</button>
		</view>
		<view class="btn-wrapper">
			<button class="btn btn-logout" @click="handleLogout">退出登录</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				oldPassword: '',
				newPassword: '',
				confirmPassword: '',
			};
		},
		methods: {
			handleChangePassword() {
				if (this.oldPassword === '') {
					uni.showToast({
						title: '请输入旧密码',
						icon: 'none',
						duration: 2000,
					});
					return;
				} else if (this.newPassword === '') {
					uni.showToast({
						title: '请输入新密码',
						icon: 'none',
						duration: 2000,
					});
					return;
				} else if (this.confirmPassword === '') {
					uni.showToast({
						title: '请输入确认密码',
						icon: 'none',
						duration: 2000,
					});
					return;
				}
				// 判断新密码和确认密码是否相同
				else if (this.newPassword !== this.confirmPassword) {
					uni.showToast({
						title: '新密码和确认密码不一致，请重新输入',
						icon: 'none',
						duration: 2000,
					});
					return;
				}
				uni.showLoading({
					title: '处理中...'
				})

				let that = this
				// 发送请求到后端API
				uni.request({
					url: 'https://lovestory.ljcljc.cn/api/changePassword',
					method: 'POST',
					header: {
						'Content-Type': 'application/x-www-form-urlencoded',
						'token': uni.getStorageSync('ljc_token')
					},
					data: {
						oldPassword: that.oldPassword,
						newPassword: that.newPassword,
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
								title: '修改密码成功',
								icon: 'success',
								duration: 2000,
							});
							// 清空表单
							that.oldPassword = '';
							that.newPassword = '';
							that.confirmPassword = '';
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
				});
			},
			handleLogout() {
				uni.showModal({
					title: '提示',
					content: '确定要退出登录吗',
					confirmColor: '#ff0000',
					success: (res) => {
						if (res.confirm) {
							uni.clearStorageSync('ljc_token')
							uni.clearStorageSync('code')
							uni.showToast({
								title: '退出登录成功',
								icon: 'success',
								duration: 2000,
								complete: () => {
									setTimeout(() => {
										uni.redirectTo({
											url: '/pages/login/login'
										})
									}, 1000)
								}
							});
							
						}
					}

				})
			},
			toheartValue() {
				uni.redirectTo({
					url: '/pages/heartValue/heartValue'
				})
			}
		},
	};
</script>

<style>
	.bg {
		background: linear-gradient(197.99deg,
				#2c9aff -14.52%,
				#9c74f3 16.96%,
				#ec81b4 54.17%,
				#f1bc97 74.34%,
				#f4e283 100%);
		background-size: cover;
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.title {
		font-size: 72rpx;
		text-align: center;
		margin: 0 0 100px;
		color: #fff;
		text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.5);
	}

	.form-item {
		display: flex;
		flex-direction: column;
		margin-bottom: 30rpx;
	}

	.label {
		font-size: 36rpx;
		color: #fff;
		margin-bottom: 10rpx;
		text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.5);
	}

	.input {
		background-color: rgba(255, 255, 255, 0.7);
		border-radius: 60rpx;
		padding: 26rpx 30rpx;
		color: #333;
		font-size: 36rpx;
		box-shadow: 0 0 20rpx rgba(255, 255, 255, 0.3);
		margin-bottom: 10rpx;
	}

	.btn-wrapper {
		display: flex;
		justify-content: center;
		margin-top: 40rpx;
	}

	.btn {
		background-color: rgba(255, 255, 255, 0.7);
		border-radius: 60rpx;
		font-size: 36rpx;
		color: #333;
		box-shadow: 0 10rpx 20rpx rgba(255, 255, 255, 0.3);
		padding: 10rpx 80rpx;
		margin: 0 20rpx;
		transition: transform 0.2s ease-in-out;
	}

	.btn:hover,
	.btn:focus {
		transform: translateY(-5rpx);
		outline: none;
	}

	.btn-update {
		background-color: #ff5a5f;
		color: #fff;
		box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.3);
	}

	.btn-update:hover,
	.btn-update:focus {
		background-color: #d15155;
		color: #fff;
		box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.3);
	}

	.btn-logout {
		background-color: #6c757d;
		color: #fff;
		box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.3);
	}

	.btn-logout:hover,
	.btn-logout:focus {
		background-color: #5a6268;
		color: #fff;
		box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.3);
	}

	.backIcon {
		position: absolute;
		left: 30px;
		top: 64px;
	}
</style>