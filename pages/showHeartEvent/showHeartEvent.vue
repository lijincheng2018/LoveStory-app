<template>
	<view class="bg">
		<view class="Heading-title">+{{ eventInfo.changeValue }}分</view>
		<view class="backIcon" style="display: flex;" @click="toheartValue()">
			<image src="../../static/back.png" style="height: 16px; width: 9px;"></image>
		</view>
		<view class="headTitle">时间：</view>
		<view class="subTitle">{{ eventInfo.time }}
			<view class="changeButton" style="top: 5px;" @click="changeDate()" v-if="power === true">修改</view>
		</view>
		<view class="shopBox">
			<view class="labelTitle">标题：</view>
			<view class="labelContent">
				<span v-if="isChangeTitle === false">{{ eventInfo.title }}</span>
				<input v-else-if="isChangeTitle === true" v-model="new_title" class="labelContentInput"/>
				<view class="changeButton" style="top: 60px;" @click="isChangeTitle === false ? changeTitle() : saveTitle()" v-if="power === true">{{ isChangeTitle === false ? '修改' : '保存' }}</view>
			</view>
			<view class="labelTitle">增加心跳值：</view>
			<view class="labelContent">
				<span v-if="isChangeValue === false">{{ eventInfo.changeValue }}</span>
				<input v-else-if="isChangeValue === true" v-model="new_value" class="labelContentInput"/>
				<view class="changeButton" style="top: 140px;" @click="isChangeValue === false ? changeValue() : saveValue()" v-if="power === true">{{ isChangeValue === false ? '修改' : '保存' }}</view>
			</view>
			<view class="labelTitle">描述：<view class="changeButton" style="top: 195px;" @click="isChangeContent === false ? changeContent() : saveContent()" v-if="power === true">{{ isChangeContent === false ? '修改' : '保存' }}</view>
			</view>
			<view class="Wordcanves">
				<view class="content">
					<span v-if="isChangeContent === false">{{ eventInfo.content.replace(/\\n/g,'\n')  }}</span>
					<textarea v-else-if="isChangeContent === true" class="labelContentTextarea" v-model="new_content"></textarea>
				</view>
				
			</view>
			<view class="submitButton" @click="SubmitEvent" v-if="power === true">确认提交</view>
		</view>
		<view class="mask" v-show="isChangeDate === true" @click="isChangeDate = false"></view>
		<view class="modelBox" v-show="isChangeDate === true">
			<view style="display: flex;justify-content: center; align-items: center;">
				<input placeholder="年" v-model="year" class="yearInput" style="width: 20%;" /><span>年</span>
				<input placeholder="月" v-model="month" class="yearInput" style="width: 20%;" /><span>月</span>
				<input placeholder="日" v-model="day" class="yearInput" style="width: 20%;" /><span>日</span>
			</view>
			<view style="display: flex;justify-content: center; align-items: center;margin-top: 10px">
				<span>时间：</span>
				<picker @change="bindPickerChange" :value="index" :range="timeArray" range-key="name">
					<view style="padding: 20rpx;background-color: white;">{{ timeArray[index].name }}</view>
				</picker>
			</view>
			<view class="submitButton" style="top: 5px; width: 100%;" @click="saveDate" >保存</view>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				eventInfo: {
					changeValue: '',
					time: '',
					title: '',
					content: ''
				},
				
				timeArray: [{
						name: '上午'
					},
					{
						name: '中午'
					},
					{
						name: '下午'
					},
					{
						name: '晚上'
					}
				],
				year: '',
				month: '',
				day: '',
				index: 2,
				isChangeDate: false,
				
				new_title: '',
				isChangeTitle: false,
				
				new_content: '',
				isChangeContent: false,
				
				new_value: '',
				isChangeValue: false,
				
				power: false  // 用于确定是不是有编辑权限
			}
		},
		onLoad(option) {
			this.getLoverData(option.id);
		},
		methods: {
			// 返回心跳页面
			toheartValue() {
				uni.redirectTo({
					url: '/pages/heartValue/heartValue'
				})
			},
			// 处理时间选择
			bindPickerChange(e) {
				this.index = e.detail.value;
			},
			// 处理修改日期
			changeDate() {
				this.isChangeDate = true
				const [dateStr, timeStr] = this.eventInfo.time.split(" ");
			
				const [year, month, day] = dateStr.replace(/年|月/g, "-").replace("日", "")
					.split("-")
					.map((item) => item.padStart(2, "0"));
			
				this.year = year
				this.month = month
				this.day = day
			
				if (timeStr == "上午") this.index = 0;
				else if (timeStr == "中午") this.index = 1;
				else if (timeStr == "下午") this.index = 2;
				else if (timeStr == "晚上") this.index = 3;
			},
			// 保存日期
			saveDate() {
				this.eventInfo.time = this.year + '年' + this.month.padStart(2, "0") + '月' + this.day.padStart(2, "0") + '日'
				if(this.index === 0) this.eventInfo.time += ' 上午'
				else if(this.index === 1) this.eventInfo.time += ' 中午'
				else if(this.index === 2) this.eventInfo.time += ' 下午'
				else if(this.index === 3) this.eventInfo.time += ' 晚上'
				this.isChangeDate = false
			},
			// 处理修改标题
			changeTitle() {
				this.isChangeTitle = true
				this.new_title = this.eventInfo.title
			},
			// 保存标题
			saveTitle() {
				this.isChangeTitle = false
				this.eventInfo.title = this.new_title
			},
			// 处理修改描述
			changeContent() {
				this.isChangeContent = true
				this.new_content = this.eventInfo.content.replace(/\\n/g,'\n') 
			},
			// 保存描述
			saveContent() {
				this.isChangeContent = false
				this.eventInfo.content = this.new_content
			},
			// 处理修改心跳值
			changeValue() {
				this.isChangeValue = true
				this.new_value = this.eventInfo.changeValue
			},
			// 保存心跳值
			saveValue() {
				this.isChangeValue = false
				this.eventInfo.changeValue = this.new_value
			},
			// 提交事件
			SubmitEvent() {
				// 检验事件标题是否存在空值
				if(this.eventInfo.title === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入标题',
						duration: 2000
					});
					return false
				} 
				// 检验事件心跳值是否存在空值
				else if(this.eventInfo.changeValue === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入心跳值',
						duration: 2000
					});
					return false
				} 
				// 检验事件描述是否存在空值
				else if(this.eventInfo.content === '') {
					uni.showToast({
						icon: 'error',
						title: '请输入描述',
						duration: 2000
					});
					return false
				} 
				uni.showLoading({
					title: '添加中...'
				});
				let that = this
				
				//发送更新请求
				uni.request({
					url: "https://lovestory.ljcljc.cn/api/updateEvent",
					method:'POST',
					header: {
					    'token': uni.getStorageSync('ljc_token')
					},
					data: that.eventInfo,
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
							
							uni.showToast({
								icon: 'success',
								title: '修改成功',
								duration: 1500
							});
						}
						else {
							uni.showToast({
								icon: 'error',
								title: '修改失败',
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
			// 发送获取详细信息请求
			getLoverData(id) {
				let that = this
				uni.request({
					url: 'https://lovestory.ljcljc.cn/api/queryEventById?id=' + id,
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
							that.eventInfo = res.data.data.data
							that.power = res.data.data.power
							
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
	}
</script>

<style>
	.bg {
		background: #FECACA;
		background-size: 100% 100%;
		min-height: 100vh;
		padding: 0 0;
		padding-top: 130px;
		width: 100%;
	}

	.backIcon {
		position: absolute;
		left: 30px;
		top: 64px;
	}
	.Heading-title {
		position: absolute;
		height: 22px;
		top: 64px;
		width: 100%;
		font-size: 16px;
		line-height: 15px;
		
		text-align: center;
		color: #FFFFFF;
	}

	.headTitle {
		position: absolute;
		height: 36px;
		left: 26px;
		top: 109px;

		font-weight: 500;
		font-size: 36px;
		line-height: 36px;


		color: #FFFFFF;
	}
	
	.subTitle {
		height: 10px;
		position: relative;
		font-weight: 500;
		font-size: 24px;
		margin-top: 30px;
		margin-left: 26px;
		color: #FFFFFF;
	}
	
	.changeButton {
		position: absolute;
		width: 51px;
		height: 24px;
		right: 25px;

		background: #FECACA;
		border-radius: 36px;
		color: #FFFFFF;
		text-align: center;
		line-height: 24px;
		
		font-size: 14px;
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

	.shopBox .labelTitle {
		font-weight: 600;
		font-size: 24px;
		line-height: 24px;
		margin-left: 24px;
		margin-bottom: 12px;
		color: #121B36;
	}
	
	.shopBox .labelContent {
		font-weight: 600;
		font-size: 18px;
		line-height: 18px;
		margin-left: 24px;
		margin-bottom: 30px;
		color: rgba(18, 27, 54, 0.6);
	}
	
	.shopBox .Wordcanves {
		height: 330px;
		width: 330px;
		margin: auto auto;
		background: url('../../static/worldbackground.png') no-repeat;
		background-size: 100% 100%;
		padding: 0;
	}
	
	.shopBox .Wordcanves .content {
		position: relative;
		top: 20px;
		width: 100%;
		font-size: 18px;
		line-height: 30px;
		font-weight: 600;
		text-align: center;
		white-space: pre-wrap; 
		color: rgba(18, 27, 54, 0.6);
	}
	
	.labelContentInput {
		position: relative;
		font-weight: 600;
		font-size: 18px;
		line-height: 18px;
	}
	
	.labelContentTextarea {
		font-size: 18px;
		line-height: 30px;
		font-weight: 600;
		text-align: center;
		padding: 0;
		margin-left: 15px;
		white-space: pre-wrap;
		color: rgba(18, 27, 54, 0.6);
	}

	.submitButton {
		position: relative;
		top: 30px;
		left: 0;
		right: 0;

		width: 90%;
		height: 50px;

		margin: auto auto;

		background: #FECACA;
		border-radius: 36px;
		font-size: 24px;
		line-height: 50px;

		text-align: center;

		color: #FFFFFF;
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
		height: 200px;
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
	
	.yearInput {
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
	
</style>