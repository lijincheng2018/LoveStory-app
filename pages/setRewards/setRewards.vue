<template>
	<view class="bg">
		<view class="backIcon" @click="toheartValue()">
			<image src="../../static/back.png" style="height: 16px; width: 9px;"></image>
		</view>
		<view class="headTitle" style="display: flex;">Ta的<image src="../../static/heartValue.png"
				style="height: 36px; width: 36px; margin: 0 10px;"></image>可用积分：{{ heartValue }}</view>

		<view class="shopBox">
			<view class="title" style="display: flex;">您获得的每一次心跳，都是Ta的每一次<image src="../../static/full_heart.png"
					style="height: 14px; width: 14px; margin: 0 5px;"></image>动！</view>
			<scroll-view scroll-y style="height: 65vh;">
				<view class="itemBox" v-for="(item, index) in shopList" :key="index" @tap="editReward(item)" @longtap="onLongPress(item)">

					<view class="cost">{{ item.cost }}分</view>
					<view class="title">{{ item.title }}</view>
					<view class="getButtom">
						<view :class="item.status == 1 ? 'ExchangedButton' : item.cost <= heartValue ? 'ActiveButton' : 'DisabledButton'">
							{{ item.status == 1 ? '已兑换' : item.cost <= heartValue ? '可兑换' : '积分不足' }}</view>
					</view>
				</view>
			</scroll-view>

			<view class="submitButton" @click="isAddData = true, rewardData.title = '', rewardData.cost = ''">添加奖励</view>
		</view>
		<!-- 模态框 -->
		<view class="mask" v-show="isAddData === true" @click="isAddData = false"></view>
		<view class="modelBox" v-show="isAddData === true">
			<view>
				<span>奖励名称：</span>
				<input class="boxInput" v-model="rewardData.title" placeholder="请输入奖励名称" placeholder-style="font-size: 16px;"/>
			</view>
			<view style="margin-top: 20px;">
				<span>需要心跳值：</span>
				<input class="boxInput" v-model="rewardData.cost" placeholder="请输入心跳值" placeholder-style="font-size: 16px;"/>
			</view>
			
			<view class="submitButton" style="top: 5px; width: 90%;" @click="isEditData === false ? Submit() : SubmitEdit()">{{ isEditData === false ? '确认添加' : '确认修改' }}</view>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				heartValue: 0,
				shopList: [],
				rewardData: {
					id: '',
					title: '',
					cost: ''
				},
				
				isAddData: false,
				isEditData: false
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
					url: 'https://lovestory.ljcljc.cn/api/queryTaRewards',
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
			},
			// 新建奖励
			Submit() {
				let that = this
				uni.request({
					url: "https://lovestory.ljcljc.cn/api/addRewards",
					method:'POST',
					header: {
					    'token': uni.getStorageSync('ljc_token')
					},
					data: that.rewardData,
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
						}
						else if (res.data.msg != null && res.data.code != 200) {
							uni.showModal({
							    title: '提示',
							    content: res.data.msg,
								showCancel: false
							})
						}
						else if (res.data.code === 200){
							that.shopList.push(res.data.data)
							that.isAddData = false
							uni.showToast({
								icon: 'success',
								title: '添加成功',
								duration: 1500
							});
						}
						else {
							uni.showToast({
								icon: 'error',
								title: '添加失败',
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
			},
			// 修改奖励
			SubmitEdit() {
				let that = this
				uni.request({
					url: "https://lovestory.ljcljc.cn/api/updateRewards",
					method:'POST',
					header: {
					    'token': uni.getStorageSync('ljc_token')
					},
					data: that.rewardData,
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
						}
						else if (res.data.msg != null && res.data.code != 200) {
							uni.showModal({
							    title: '提示',
							    content: res.data.msg,
								showCancel: false
							})
						}
						else if (res.data.code === 200){
							that.isAddData = false
							that.isEditData = false
							for(let i=0;i<that.shopList.length;i++) {
							    if(that.shopList[i].id == that.rewardData.id) {
							        that.shopList[i] = JSON.parse(JSON.stringify(that.rewardData))
							        break;
							    }
							}
							uni.showToast({
								icon: 'success',
								title: '修改成功',
								duration: 1500
							});
						}
						else {
							uni.showToast({
								icon: 'error',
								title: '添加失败',
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
			},
			// 删除奖励
			onLongPress(item) {
				let that = this
			    uni.showModal({    // 弹框询问是否进行下一步事件
			        title: '提示',
			        content: '是否删除'+ item.title +'奖励',
			        success(res) {
			            if (res.confirm) {
			                uni.request({
			                	url: 'https://lovestory.ljcljc.cn/api/delReward?id=' + item.id,
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
			                			uni.showToast({
			                				icon: 'success',
			                				title: '删除成功',
			                				duration: 1500
			                			});
										for(let i=0;i<that.shopList.length;i++) {
											if(that.shopList[i].id == item.id) {
											    that.shopList.splice(i, 1);
											    break;
											}
										}
										
			                		} else {
			                			uni.showToast({
			                				icon: 'error',
			                				title: '加载失败',
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
			        }
			    });
			},
			// 编辑奖励
			editReward(item) {
				this.isEditData = true
				this.isAddData = true
				this.rewardData = JSON.parse(JSON.stringify(item)) 
			}
			
		}
	}
</script>

<style>
	.bg {
		background: linear-gradient(197.99deg, #2C9AFF -14.52%, #9C74F3 16.96%, #EC81B4 54.17%, #F1BC97 74.34%, #F4E283 100%);
		background-size: 100% 100%;
		min-height: 90vh;
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
		height: 90vh;

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
	.boxInput {
		border-bottom: 2px solid rgba(100, 100, 100, 0.8);
		border-top: 0px;
		border-left: 0px;
		border-right: 0px;
		font-family: 'Roboto';
		font-style: normal;
		font-weight: 400;
		font-size: 20px;
		line-height: 20px;
		letter-spacing: 0.2px;
		color: #000;
		height: 20px;
		width: 100%;
		line-height: 30px;
	}
	.inputGroup {
		display: flex;
	}
	.modelBox {
		box-sizing: border-box;
		z-index: 999;
		margin: 50% auto;
		position: absolute;
		padding: 30px;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		height: 300px;
		width: 90%;
		background-color: rgba(255, 255, 255);
		border-radius: 10px;
		font-size: 20px;
		text-align: center;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	
	.mask {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.5);
		z-index: 998;
	}
	
</style>