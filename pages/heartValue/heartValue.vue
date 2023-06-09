<template>
	<view class="bg">
		<view class="backIcon" @click="toSetting()">
			<image src="../../static/setting.png" style="height: 20px; width: 20px;"></image>
		</view>
		<view class="headTitle">恋爱值：{{ myHeartValue + taHeartValue }}</view>

		<view class="headSubTitle" style="display: flex;" @click="toShop()" v-show="activeTab === 'my'">
			心跳值&nbsp;❤️&nbsp;兑换奖励
			<image src="../../static/front.png" style="height: 16px; width: 9px;margin-left: 10px;"></image>
		</view>
		<view class="headSubTitle" style="display: flex;" @click="toSetRewards()" v-show="activeTab === 'ta'">
			给Ta设置&nbsp;❤️&nbsp;心跳值奖励
			<image src="../../static/front.png" style="height: 16px; width: 9px;margin-left: 10px;"></image>
		</view>

		<view class="ruleBox">
			<view class="title" @click="showRule()">规则</view>
		</view>

		<view class="selectBox">
			<view class="selectBoxItem">
				<view :class="activeTab === 'my' ? 'left on' : 'left'" @click="activeTab = 'my'">
					我的心跳值：{{ myHeartValue }}</view>
				<view :class="activeTab === 'ta' ? 'right on' : 'right'" @click="toTaPanel()">
					Ta的心跳值：{{ taHeartValue }}</view>
			</view>
		</view>

		<!-- 我的心跳值 -->
		<view class="heartBox" v-show="activeTab === 'my'">
			<uni-ec-canvas class="uni-ec-canvas" id="my-Chart" ref="canvas" canvas-id="uni-ec-canvas" :ec="ec">
			</uni-ec-canvas>
			<view style="display: flex; justify-content: space-between;">
				<view class="heartBoxDesc">Today</view>
				<view class="heartBoxDesc">+ {{ eventList.length }}</view>
			</view>
			<view class="eventBox">
				<view class="eventBoxItem" v-for="(item, index) in eventList" :key="index"
					@click="item.changeMode === 'add' ? toShowHeartEvent(item.id) : toShowBrokenHeartEvent(item.id)">
					<view class="number">{{ index+1 }}</view>
					<view class="event">
						<view class="title">{{ item.title }}</view>
						<view class="time">{{ item.time }}</view>
					</view>
					<view class="changeValue" :style="item.changeMode === 'add' ? 'color: #FA57B6' : 'color: #63A1FE'">
						{{ item.changeMode === 'add' ? '+' : '-' }} &nbsp;
						{{ item.changeValue }}
					</view>
					<view class="valueBox">
						<image
							:src="item.changeMode === 'add' ? '../../static/full_heart.png' : '../../static/broken_heart.png'"
							style="height: 16px; width: 16px;"></image>
						<view class="title">{{ item.currentValue }}</view>
					</view>
					<view class="back">
						<image
							:src="item.changeMode === 'add' ? '../../static/full_heart_back.png' : '../../static/broken_heart_back.png'"
							style="height: 16px; width: 9px;"></image>
					</view>
				</view>
			</view>
		</view>


		<!-- Ta的心跳值 -->
		<view class="heartBox" v-show="activeTab === 'ta'">
			<uni-ec-canvas class="uni-ec-canvas" id="ta-Chart" ref="canvas" canvas-id="uni-ec-canvas"
				:ec="ec"></uni-ec-canvas>

			<view style="display: flex; justify-content: space-between;">
				<view class="heartBoxDesc">Today</view>
				<view class="heartBoxDesc">+ {{ taeventList.length }}</view>
			</view>
			<view class="radius-boarder-box">

				<view class="controller-box">
					<image src="../../static/addButton.png" style="height: 44px; width: 44px;"
						@click="toAddHeartEvent()"></image>
					<image src="../../static/minusButton.png" style="height: 44px; width: 44px; margin-left: 30px;"
						@click="toAddBrokenHeartEvent()"></image>
				</view>
				<view class="eventBox">
					<view class="eventBoxItem" v-for="(item, index) in taeventList" :key="index"
						@click="item.changeMode === 'add' ? toShowHeartEvent(item.id) : toShowBrokenHeartEvent(item.id)"
						@longtap="onLongPress(item.id)">
						<view class="number">{{ index+1 }}</view>
						<view class="event">
							<view class="title">{{ item.title }}</view>
							<view class="time">{{ item.time }}</view>
						</view>
						<view class="changeValue"
							:style="item.changeMode === 'add' ? 'color: #FA57B6' : 'color: #63A1FE'">
							{{ item.changeMode === 'add' ? '+' : '-' }} &nbsp;
							{{ item.changeValue }}
						</view>
						<view class="valueBox">
							<image
								:src="item.changeMode === 'add' ? '../../static/full_heart.png' : '../../static/broken_heart.png'"
								style="height: 16px; width: 16px;"></image>
							<view class="title">{{ item.currentValue }}</view>
						</view>
						<view class="back">
							<image
								:src="item.changeMode === 'add' ? '../../static/full_heart_back.png' : '../../static/broken_heart_back.png'"
								style="height: 16px; width: 9px;"></image>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import uniEcCanvas from '@/components/uni-ec-canvas/uni-ec-canvas.vue'
	import * as echarts from '@/components/uni-ec-canvas/echarts.min'

	export default {
		components: {
			uniEcCanvas
		},
		data() {
			return {
				ec: {
					lazyLoad: true
				},

				myChartData: [],
				taChartData: [],
				myMarkPointData: [],
				taMarkPointData: [],

				myHeartValue: 0,
				taHeartValue: 0,
				activeTab: 'my',
				eventList: [],
				taeventList: [],

				Myoption: null,
				Taoption: null,
			}
		},
		onShow() {
			this.myHeartValue = 0
			this.taHeartValue = 0
			this.getLoveEvents() // 获取恋爱事件
		},
		onPullDownRefresh() {
			this.myHeartValue = 0
			this.taHeartValue = 0
			this.getLoveEvents() // 获取恋爱事件
			uni.stopPullDownRefresh() //停止刷新
		},
		methods: {
			// 前往设置页
			toSetting() {
				uni.navigateTo({
					url: '/pages/UserSettings/UserSettings'
				})
			},
			// 前往我的兑换页
			toShop() {
				uni.navigateTo({
					url: '/pages/shop/shop'
				})
			},
			// 前往设置Ta的奖励页
			toSetRewards() {
				uni.navigateTo({
					url: '/pages/setRewards/setRewards'
				})
			},
			// 前往添加心跳事件页
			toAddHeartEvent() {
				uni.navigateTo({
					url: '/pages/addHeartEvent/addHeartEvent'
				})
			},
			// 前往添加心碎事件页
			toAddBrokenHeartEvent() {
				uni.navigateTo({
					url: '/pages/addBrokenHeartEvent/addBrokenHeartEvent'
				})
			},
			// 前往显示心跳事件页
			toShowHeartEvent(id) {
				uni.navigateTo({
					url: '/pages/showHeartEvent/showHeartEvent?id=' + id
				})
			},
			// 前往显示心碎事件页
			toShowBrokenHeartEvent(id) {
				uni.navigateTo({
					url: '/pages/showBrokenHeartEvent/showBrokenHeartEvent?id=' + id
				})
			},
			// 处理Ta的心跳值页
			toTaPanel() {
				this.activeTab = 'ta'; // 修改活动标签
				
				// 延时渲染图表
				setTimeout(() => {
					this.drawTaChart() 
				}, 100)
			},
			// 获取恋爱事件
			getLoveEvents() {
				let that = this
				uni.request({
					url: 'https://lovestory.ljcljc.cn/api/queryEvent',
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
							that.eventList = res.data.data.myEvents
							that.taeventList = res.data.data.taEvents
							
							// 前端计算我的心跳值
							that.eventList.forEach(item => {
								if (item.changeMode == 'add') that.myHeartValue += parseInt(item.changeValue)
								if (item.changeMode == 'minus') that.myHeartValue -= parseInt(item.changeValue)
							})
							
							// 前端计算Ta的心跳值
							that.taeventList.forEach(item => {
								if (item.changeMode == 'add') that.taHeartValue += parseInt(item.changeValue)
								if (item.changeMode == 'minus') that.taHeartValue -= parseInt(item.changeValue)
							})
							
							// 处理我的心跳值的图表数据
							that.myChartData = JSON.parse(JSON.stringify(that.eventList))
							let MyNum = 0;
							for (let i = that.myChartData.length - 1; i >= 0; i--) {
								if (that.myChartData[i].changeMode === 'add') MyNum += parseInt(that.myChartData[i]
									.changeValue);
								if (that.myChartData[i].changeMode === 'minus') MyNum -= parseInt(that.myChartData[i].changeValue);
								that.myChartData[i].num = MyNum;
							}
							
							// 处理Ta的心跳值的图表数据
							that.taChartData = JSON.parse(JSON.stringify(that.taeventList))
							let TaNum = 0;
							for (let i = that.taChartData.length - 1; i >= 0; i--) {
								if (that.taChartData[i].changeMode === 'add') TaNum += parseInt(that.taChartData[i]
									.changeValue);
								if (that.taChartData[i].changeMode === 'minus') TaNum -= parseInt(that.taChartData[
									i].changeValue);
								that.taChartData[i].num = TaNum;
							}
							that.myChartData.reverse()
							that.taChartData.reverse()
							that.drawMyChart()
							console.log(that.taChartData)
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
			// 显示规则
			showRule() {
				uni.showModal({
					showCancel: false,
					title: '规则',
					content: '1. 你可以为Ta设置心跳值事件\n2. 你可以为Ta设置心跳值兑换的奖品\n3. 你可以兑换Ta为你设置的奖品',

				})
			},
			
			// 删除事件
			onLongPress(id) {
				let that = this
			    uni.showModal({    // 弹框询问是否进行下一步事件
			        title: '提示',
			        content: '是否删除该事件',
			        success(res) {
			            if (res.confirm) {
			                uni.request({
			                	url: 'https://lovestory.ljcljc.cn/api/delEvent?id=' + id,
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

										for(let i=0;i<that.taeventList.length;i++) {
											if(that.taeventList[i].id == id) {
											    that.taeventList.splice(i, 1);
											    break;
											}
										}
										that.drawTaChart();
								
										
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
			// 绘制我的心跳值的图表
			drawMyChart() {
				let myChart = echarts.init(document.getElementById('my-Chart'));
				if (this.myChartData.length > 5) {
					// 使用slice截取最新的5条数据
					this.myChartData = this.myChartData.slice(0, 5)
				}
				// 处理图标表现
				this.myMarkPointData = this.myChartData.map((item, index) => {
					return {
						coord: [index, item.num],
						value: item.changeValue,
						itemStyle: {
							color: item.changeMode === 'add' ? '#FFB3A3' : '#A2C7FE'
						},
						label: {
							formatter: `${item.changeMode === 'add' ? '+' : '-'}\n{c}`,
							position: 'insideTop'
						}
					};
				});
				this.Myoption = {
					xAxis: {
						type: 'category',
						// 隐藏横坐标
						show: false,
						data: ['1', '2', '3', '4', '5']
					},
					yAxis: {
						type: 'value',
						// 显示数值
						axisLabel: {
							formatter: '{value}',
							textStyle: {
								color: 'rgba(234, 234, 234, 0.4)',
							},
							verticalAlign: 'middle'
						},
						axisLine: {
							show: false,
							lineStyle: {
								color: 'rgba(234, 234, 234, 0.2)',
							}
						},
						// y轴显示在图表的右侧
						splitNumber: 2,
						position: 'right',
						splitLine: {
							lineStyle: {
								color: '#eaeaea',
								opacity: 0.2
							},
						}
					},
					series: [{
						data: this.myChartData.map(item => item.num),
						type: 'line',
						// 折线展示为平滑曲线
						smooth: true,
						lineStyle: {
							color: 'transparent'
						},
						markPoint: {
							data: this.myMarkPointData
						},
						symbol: 'circle',
						symbolSize: 10,
						itemStyle: {
							color: '#fff',
							shadowBlur: 3,
							shadowOffsetX: 0,
							shadowOffsetY: 1,
							shadowColor: 'rgba(0, 0, 0, 0.3)'
						},
						label: {
							show: false,
							position: 'top',
							formatter: '{b}'
						},
						areaStyle: {
							normal: {
								color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
										offset: 0,
										color: '#2C9AFF80'
									},
									{
										offset: 0.17,
										color: '#9C74F360'
									},
									{
										offset: 0.5417,
										color: '#EC81B450'
									},
									{
										offset: 0.7434,
										color: '#F1BC9740'
									},
									{
										offset: 1,
										color: '#F4E28313'
									}
								]),
								shadowColor: 'rgba(0, 0, 0, 0.1)',
							}
						},

					}]
				}


				myChart.setOption(this.Myoption);

				window.addEventListener("resize", function() {
					myChart.resize();
				});

			},
			// 绘制Ta的图表
			drawTaChart() {
				let taChart = echarts.init(document.getElementById('ta-Chart'));
				if (this.taChartData.length > 5) {
					// 使用slice截取最新的4条数据
					this.taChartData = this.taChartData.slice(0, 5)
				}

				this.taMarkPointData = this.taChartData.map((item, index) => {
					return {
						coord: [index, item.num],
						value: item.changeValue,
						itemStyle: {
							color: item.changeMode === 'add' ? '#FFB3A3' : '#A2C7FE'
						},
						label: {
							formatter: `${item.changeMode === 'add' ? '+' : '-'}\n{c}`,
							position: 'insideTop'
						}
					};
				});
				this.Taoption = {
					xAxis: {
						type: 'category',
						// 隐藏横坐标
						show: false,
						data: ['1', '2', '3', '4', '5']
					},
					yAxis: {
						type: 'value',
						// 显示数值
						axisLabel: {
							formatter: '{value}',
							textStyle: {
								color: 'rgba(234, 234, 234, 0.4)',
							},
							verticalAlign: 'middle'
						},
						axisLine: {
							show: false,
							lineStyle: {
								color: 'rgba(234, 234, 234, 0.2)',
							}
						},
						// y轴显示在图表的右侧
						splitNumber: 2,
						position: 'right',
						splitLine: {
							lineStyle: {
								color: '#eaeaea',
								opacity: 0.2
							},
						}
					},
					series: [{
						data: this.taChartData.map(item => item.num),
						type: 'line',
						// 折线展示为平滑曲线
						smooth: true,
						lineStyle: {
							color: 'transparent'
						},
						markPoint: {
							data: this.taMarkPointData
						},
						symbol: 'circle',
						symbolSize: 10,
						itemStyle: {
							color: '#fff',
							shadowBlur: 3,
							shadowOffsetX: 0,
							shadowOffsetY: 1,
							shadowColor: 'rgba(0, 0, 0, 0.3)'
						},
						label: {
							show: false,
							position: 'top',
							formatter: '{b}'
						},
						areaStyle: {
							normal: {
								color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
										offset: 0,
										color: '#2C9AFF80'
									},
									{
										offset: 0.17,
										color: '#9C74F360'
									},
									{
										offset: 0.5417,
										color: '#EC81B450'
									},
									{
										offset: 0.7434,
										color: '#F1BC9740'
									},
									{
										offset: 1,
										color: '#F4E28313'
									}
								]),
								shadowColor: 'rgba(0, 0, 0, 0.1)',
							}
						},

					}]
				}

				taChart.setOption(this.Taoption);
				window.addEventListener("resize", function() {
					taChart.resize();
				});
			}
		}
	}
</script>

<style>
	.bg {
		background: linear-gradient(197.99deg, #2C9AFF -14.52%, #9C74F3 16.96%, #EC81B4 54.17%, #F1BC97 74.34%, #F4E283 100%);
		min-height: 100vh;
		padding: 0 0;
		padding-top: 130px;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	.backIcon {
		width: 40px;
		background-color: rgba(200, 200, 200, 0.5);
		color: #fff;
		border-radius: 5px 0 0 5px;
		box-sizing: border-box;
		padding: 5px;
		display: flex;
		justify-content: center;
		align-items: center;
		position: absolute;
		right: 0;
		top: 64px;
		opacity: 0.8;
	}

	.headTitle {
		position: absolute;
		height: 36px;
		left: 26px;
		top: 109px;

		font-style: normal;
		font-weight: 500;
		font-size: 36px;
		line-height: 36px;


		color: #FFFFFF;
	}

	.headSubTitle {
		height: 25px;
		margin: 40px 26px;

		font-style: normal;
		font-weight: 500;
		font-size: 20px;
		line-height: 15px;
		text-align: left;

		color: #FFFFFF;
	}

	.ruleBox {
		position: absolute;
		width: 47px;
		height: 36px;
		right: 0;
		top: 109px;

		background: rgba(229, 229, 229, 0.3);
		mix-blend-mode: normal;
		border-radius: 30px 0px 0px 30px;
	}

	.ruleBox .title {
		color: #FFFFFF;
		opacity: 0.6;
		text-align: center;
		line-height: 36px;
	}

	.selectBox {
		position: relative;
		width: 100%;
		height: 150px;
		box-sizing: border-box;
	}

	.selectBoxItem {
		width: 90%;
		height: 36px;
		position: absolute;
		bottom: 0;
		top: 0;
		left: 0;
		right: 0;
		margin: 0 auto;

		display: flex;

		mix-blend-mode: normal;
		border: 2px solid #FFFFFF;
		border-radius: 36px;
	}

	.selectBoxItem .left {
		width: 50%;
		height: 36px;
		font-size: 18px;
		text-align: center;
		line-height: 36px;
		font-weight: 500;
		color: #FFFFFF;
		border-radius: 36px 0px 0px 36px;
	}

	.selectBoxItem .right {
		width: 50%;
		height: 36px;
		font-size: 18px;
		text-align: center;
		line-height: 36px;
		font-weight: 500;
		color: #FFFFFF;
		border-radius: 0px 36px 36px 0px;
	}

	.selectBoxItem .on {
		background: #FFFFFF;
		color: #FA57B6;
	}

	.heartBox {
		padding: 0;
		margin: 0;
		width: 100%;
	}

	.heartBoxDesc {
		margin: 5px 20px;
		font-size: 12px;
		color: #FFFFFF;
	}

	.eventBox {
		width: 100%;
		position: relative;
		margin-top: 40px;
	}

	.radius-boarder-box {
		width: 100%;
		height: 50vh;
		padding-top: 1px;
		margin-top: 30px;
		background-color: #FFFFFF;
		border-radius: 36px 36px 0 0;
	}

	.controller-box {
		float: right;
		margin-right: 30px;
		margin-top: -22px;
	}

	.eventBoxItem {
		margin: 12px auto;

		width: 90%;
		height: 50px;
		background: #FFFFFF;
		border-radius: 36px;

		display: flex;
		line-height: 50px;
	}

	.eventBoxItem .number {
		width: 10%;
		font-size: 14px;
		color: #000;
		opacity: 0.2;
		margin-left: 20px;
	}

	.eventBoxItem .event {
		width: 50%;
		font-size: 14px;
		color: #000;
		line-height: 35px;
	}

	.eventBoxItem .event .title {
		font-size: 16px;
		color: #121B36;
	}

	.eventBoxItem .event .time {
		font-size: 10px;
		color: #000000;
		opacity: 0.2;
		line-height: 0px;
	}

	.eventBoxItem .changeValue {
		width: 15%;
		line-height: 35px;
	}

	.eventBoxItem .valueBox {
		width: 10%;
		line-height: 40px;
	}

	.eventBoxItem .valueBox .title {
		font-size: 14px;
		color: #121B36;
		line-height: 0px;
		margin-top: -5px;
	}

	.eventBoxItem .back {
		width: 10%;
		line-height: 55px;
	}

	.uni-ec-canvas {
		width: 100%;
		height: 300px;
		display: block;
		margin-top: -50px;
	}
</style>