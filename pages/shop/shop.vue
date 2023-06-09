<template>
	<view class="bg">
		<view class="backIcon" @click="toheartValue()">
			<image src="../../static/back.png" style="height: 16px; width: 9px;"></image>
		</view>
		<view class="headTitle" style="display: flex;">我的<image src="../../static/heartValue.png"
				style="height: 36px; width: 36px; margin: 0 10px;"></image>可用积分：{{ heartValue }}</view>

		<view class="shopBox">
			<view class="title" style="display: flex;">您获得的每一次心跳，都是Ta的每一次<image src="../../static/full_heart.png"
					style="height: 14px; width: 14px; margin: 0 5px;"></image>动！</view>
			<scroll-view scroll-y style="height: 65vh;">
				<view class="itemBox" v-for="(item, index) in shopList" :key="index">

					<view class="cost">{{ item.cost }}分</view>
					<view class="title">{{ item.title }}</view>
					<view class="getButtom">
						<view
							:class="item.status == 1 ? 'ExchangedButton' : item.cost <= heartValue ? 'ActiveButton' : 'DisabledButton'"
							@click="item.status == 1 ? null : item.cost <= heartValue ? exchange(item) : null">
							{{ item.status == 1 ? '已兑换' : item.cost <= heartValue ? '兑换' : '积分不足' }}
						</view>
					</view>
				</view>
			</scroll-view>

			<view class="submitButton" @click="OneKeyDuihuan">一键兑换</view>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				heartValue: 0,
				shopList: []
			}
		},
		onShow() {
			this.getRewards();
		},
		methods: {
			// 返回心跳值页面
			toheartValue() {
				uni.navigateTo({
					url: '/pages/heartValue/heartValue'
				})
			},
			// 获取奖励列表
			getRewards(id) {
				uni.showLoading({
					title: '加载中'
				});
				let that = this
				uni.request({
					url: 'https://lovestory.ljcljc.cn/api/queryMyRewards',
					method: 'GET',
					header: {
						'token': uni.getStorageSync('ljc_token')
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
							that.heartValue = res.data.data.heartValue
							that.shopList = res.data.data.rewardList

						} else {
							uni.showToast({
								icon: 'error',
								title: '加载失败',
								duration: 1500
							});
						}

					}
				})
			},
			// 处理兑换奖励
			exchange(item) {
				let that = this
				uni.showModal({
					title: '提示',
					content: `确定要兑换${item.title}吗`,
					success: (res) => {
						uni.showLoading({
							title: '处理中'
						});
						if (res.confirm) {
							uni.request({
								url: 'https://lovestory.ljcljc.cn/api/exchangeReward?id=' + item.id,
								method: 'GET',
								header: {
									'token': uni.getStorageSync('ljc_token')
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
										// 对可兑换值进行减运算
										that.heartValue = that.heartValue - item.cost
										
										// 寻找兑换的物品，在前端将其设置为以兑换
										for (let i = 0; i < that.shopList.length; i++) {
											if (that.shopList[i].id == item.id) {
												that.shopList[i].status = '1';
												break;
											}
										}
										uni.showToast({
											icon: 'success',
											title: '兑换成功',
											duration: 1500
										});
									} else {
										uni.showToast({
											icon: 'error',
											title: '加载失败',
											duration: 1500
										});
									}

								}
							})
						}
					}

				})
			},
			// 一键兑换
			OneKeyDuihuan() {
				let that = this
				uni.showModal({
					title: '提示',
					content: `确定要一键兑换吗？\n此操作将会从积分最少的开始兑换`,
					success: (res) => {
						if(res.confirm) {
							uni.showLoading({
								title: '处理中'
							});
							let isDuihuan = false
							// 拷贝一次
							let tempArray = JSON.parse(JSON.stringify(this.shopList))
							
							// 按cost从小到大排序
							tempArray.sort((x, y) => {return parseInt(x.cost) - parseInt(y.cost)})
							let duihuanList = []
							
							// 寻找最大可兑换的物品
							for (let i = 0; i < tempArray.length; i++) {
								if (tempArray[i].status == 0 && that.heartValue >= tempArray[i].cost) {
									isDuihuan = true
									
									// 找到了，加入duihuanList中
									duihuanList.push(tempArray[i].id)
									that.heartValue -= parseInt(tempArray[i].cost)
									for(let k=0; k<that.shopList.length; k++) {
										if(tempArray[i].id == that.shopList[k].id) {
											that.shopList[k].status = '1';
										}
									}
								}
							}
							// 若找到了，直接统一发送给后端进行处理
							if(duihuanList.length > 0) {
								let duihuanListString = duihuanList.toString()
								console.log(duihuanList)
								uni.request({
									url: 'https://lovestory.ljcljc.cn/api/exchangeAllReward',
									method: 'POST',
									data: {
										duihuanList: duihuanListString
									},
									header: {
										'Content-Type': 'application/x-www-form-urlencoded',
										'token': uni.getStorageSync('ljc_token')
									},
									success(res) {
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
											uni.hideLoading()
											if(isDuihuan === true) {
												uni.showToast({
													icon: 'success',
													title: '兑换成功',
													duration: 1500
												});
											}
										}
									}
								})
							}
							
							uni.hideLoading()
							if(isDuihuan === false) {
								uni.showToast({
									icon:'none',
									title: '没有可以兑换的啦~',
									duration: 1500
								});
							}
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
			
		}
	}
	
</script>

<style>
	.bg {
		background: linear-gradient(197.99deg, #2C9AFF -14.52%, #9C74F3 16.96%, #EC81B4 54.17%, #F1BC97 74.34%, #F4E283 100%);
		background-size: 100% 100%;
		min-height: 100vh;
		padding: 0 0;
		padding-top: 130px;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	.backIcon {
		position: absolute;
		left: 30px;
		top: 64px;
	}

	.headTitle {
		position: absolute;
		height: 36px;
		left: 26px;
		top: 109px;

		font-weight: 500;
		font-size: 34px;
		line-height: 36px;


		color: #FFFFFF;
	}

	.shopBox {
		position: relative;
		margin-top: 40px;
		width: 100%;
		height: 100vh;

		background: #FFFFFF;
		border-radius: 36px 36px 0px 0px;
		padding-top: 24px;
	}

	.shopBox .title {
		font-weight: 400;
		font-size: 14px;
		line-height: 14px;

		margin-left: 24px;

		color: #121B36;
		opacity: 0.8;
	}

	.shopBox .itemBox {
		width: 90%;
		height: 50px;
		background: #FECDCA;
		border-radius: 36px;
		margin: auto auto;
		margin-top: 20px;
		display: flex;
		flex: 0 0;
		align-items: center;
	}

	.shopBox .itemBox .cost {
		width: 20%;
		font-weight: 600;
		font-size: 24px;
		line-height: 50px;
		margin-left: 20px;

		color: #FFFFFF;
	}

	.shopBox .itemBox .title {
		width: 45%;
		font-weight: 600;
		font-size: 18px;
		line-height: 50px;
		text-align: left;
		color: #FF825A;
	}

	.shopBox .itemBox .getButtom {
		margin-right: 10px;
	}

	.shopBox .itemBox .ActiveButton {
		width: 60px;
		height: 30px;
		background: #FF825A;
		border-radius: 36px;
		font-size: 12px;
		line-height: 30px;
		color: #FFFFFF;
		text-align: center;

	}

	.shopBox .itemBox .DisabledButton {
		width: 60px;
		height: 30px;
		border: 2px solid rgba(255, 255, 255, 0.8);
		border-radius: 36px;
		font-size: 12px;
		line-height: 30px;
		color: #FFFFFF;
		text-align: center;
	}

	.shopBox .itemBox .ExchangedButton {
		width: 60px;
		height: 30px;
		background: #029aff;
		border-radius: 36px;
		font-size: 12px;
		line-height: 30px;
		color: #FFFFFF;
		text-align: center;

	}

	.submitButton {
		position: relative;
		bottom: 50px;
		left: 0;
		right: 0;

		width: 90%;
		height: 50px;

		margin: auto auto;

		background: #FF825A;
		border-radius: 36px;
		font-size: 24px;
		line-height: 50px;

		text-align: center;

		color: #FFFFFF;
	}
</style>