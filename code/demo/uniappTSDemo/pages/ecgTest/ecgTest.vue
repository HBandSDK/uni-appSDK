<template>
	<view class="box">
		<!-- 标题 -->
		<view class="header">
			<text class="header-title">ECG 心电测量</text>
		</view>

		<!-- 数据卡片 -->
		<view class="card">
			<view class="card-title">实时数据</view>
			<view class="data-grid">
				<view class="data-item">
					<text class="data-label">手环状态</text>
					<text class="data-value">{{device.content.wristbandStatus !== undefined ? device.content.wristbandStatus : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">导联佩戴</text>
					<text class="data-value">{{device.content.wearStatus !== undefined ? device.content.wearStatus : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">HR1每秒</text>
					<text class="data-value">{{device.content.HR1PerSecond !== undefined ? device.content.HR1PerSecond : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">HR2每分钟</text>
					<text class="data-value">{{device.content.HR2PerHour !== undefined ? device.content.HR2PerHour : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">Hrv每秒</text>
					<text class="data-value">{{device.content.Hrv !== undefined ? device.content.Hrv : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">RR1每秒</text>
					<text class="data-value">{{device.content.RR1PerSecond !== undefined ? device.content.RR1PerSecond : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">RR2每6秒</text>
					<text class="data-value">{{device.content.RR2Per6Second !== undefined ? device.content.RR2Per6Second : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">BR1每秒</text>
					<text class="data-value">{{device.content.BR2PerSecond !== undefined ? device.content.BR2PerSecond : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">BR2每分钟</text>
					<text class="data-value">{{device.content.BR2PerHours !== undefined ? device.content.BR2PerHours : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">M_ID</text>
					<text class="data-value">{{device.content.M_ID !== undefined ? device.content.M_ID : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">QTC</text>
					<text class="data-value">{{device.content.QTC !== undefined ? device.content.QTC : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">PWV</text>
					<text class="data-value">{{device.content.PWV !== undefined ? device.content.PWV : '--'}}</text>
				</view>
			</view>
		</view>

		<!-- 进度 -->
		<view class="ecg-item">测试进度：{{device && device.progress}}%</view>

		<!-- 按钮 -->
		<view class="btn-group">
			<button class="btn" @click="ECGmeasureStartDataManager">开启测试</button>
			<button class="btn btn-stop" @click="ECGmeasureStopDataManager">关闭测试</button>
		</view>

		<!-- 波形图 -->
		<scroll-view class="myScoll" scroll-x>
			<view>
				<view class="box1">
					<canvas canvas-id="ecg" id="ecg" style="width: 5000px; height: 300px;"></canvas>
				</view>
				<view class="box2">
					<canvas canvas-id="myCanvas" id="myCanvas" style="width: 5000px; height: 300px;"></canvas>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
	// 引入方式一:
	// // 整体引入
	// import sdk from '../../common/index.js'

	// // // 然后自己解构出来
	// const {
	// 	veepooBle,
	// 	veepooFeature
	// } = sdk;

	// 引入方式二：
	import {
		veepooBle,
		veepooFeature
	} from '../../common/index.js'
	import {
		ab2hex
	} from '../../utils/util.js'
	let totalArray = []

	export default {
		data() {
			return {
				// 必须预置 content，否则 {{device.content.wristbandStatus}} 等在 Vue3 下会因 content 为 undefined 抛错导致整页空白
				device: {
					progress: 0,
					content: {}
				},
				list: [],
				// list: [],
				height: 300,
				width: 5000,
				centerY: 240 // y轴中心作为绘画点，
			}
		},
		onReady() {

			this.drawCurve();
			let data = veepooFeature.veepooGetDiseaseTextManager({
				heartRate: 77,
				diseaseResult: [0, 0, 0, 0, 0, 0, 0, 0]
			});

			console.log('data==>', data);
		},
		onShow() {
			this.notifyMonitorValueChange();
		},
		methods: {
			// 无参数
			ECGmeasureStartDataManager() {
				// this.notifyMonitorValueChange();
				veepooFeature.veepooSendECGmeasureStartDataManager();
			},
			ECGmeasureStopDataManager() {
				veepooFeature.veepooSendECGmeasureStopDataManager();
			},
			// 监听订阅 notifyMonitorValueChange
			//  veepooUniAppSDKNotifyECGValueChange
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {

					console.log(" ECG 监听蓝牙回调=>", e);
					if (e.name == 'ecg测量') {
						console.log("========================================================")
						self.device = e

						if (e.progress == 100) {
							console.log("totalArray=>", totalArray)
							self.list = totalArray
							self.drawCurve();
						}
					}
				})
				// 这里主要是获取的是波形的数据 为
				veepooBle.veepooUniAppSDKNotifyECGValueChange(function(e) {
					if (e.name == 'ecg波形数据') {
						totalArray.push(...e.content)
						console.log('ecg波形数据', e.content)
					}
				})
			},



			drawCurve() {

				let ctx = uni.createCanvasContext('myCanvas', this)
				let list = this.list;
				console.log("list==>", list)
				let centerY = this.centerY;
				let width = this.width;
				let xScale = width / (list.length - 1); // 计算每个数据点占据的宽度
				let yScale = centerY / (Math.max(...list) - Math.min(...list))
				let x = 0;
				let y = centerY / 2 - (list[0] * yScale)
				ctx.beginPath(); // 开始绘制
				ctx.moveTo(x, y);
				ctx.setStrokeStyle('#c96d79'); // 设置线条颜色
				ctx.setLineWidth(2); // 设置线条宽度
				for (let i = 1; i < list.length; i++) {
					let x = i * xScale;
					let y = centerY / 2 - ((list[i] / 2) * yScale); // 负数在中心下方，正数在中心上方
					ctx.lineTo(x, y);
				}
				ctx.stroke(); // 绘制线条
				ctx.draw(false); // 绘制到canvas上，不需要等待上一步绘制完成
			}
		}
	}
</script>

<style>
	.box {
		padding: 30rpx;
	}

	.header {
		text-align: center;
		padding: 20rpx 0 30rpx;
	}

	.header-title {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
	}

	.card {
		background: #fff;
		border-radius: 16rpx;
		padding: 24rpx;
		box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.06);
	}

	.card-title {
		font-size: 30rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.data-grid {
		display: flex;
		flex-wrap: wrap;
	}

	.data-item {
		width: 33.33%;
		padding: 12rpx 0;
		display: flex;
		flex-direction: column;
	}

	.data-label {
		font-size: 24rpx;
		color: #999;
		margin-bottom: 6rpx;
	}

	.data-value {
		font-size: 28rpx;
		color: #333;
		font-weight: 500;
	}

	.ecg-item {
		line-height: 80rpx;
		font-size: 28rpx;
		color: #666;
		text-align: center;
	}

	.btn-group {
		display: flex;
		gap: 20rpx;
		margin: 30rpx 0;
	}

	.btn {
		flex: 1;
		background-color: #00b0fb;
		color: #fff;
		border-radius: 12rpx;
		font-size: 30rpx;
	}

	.btn-stop {
		background-color: #ff6b6b;
	}

	.box1 {
		position: absolute;
		left: 0px;
		top: 50px;
		width: 3000rpx;
	}

	.box2 {
		position: absolute;
		left: 0px;
		top: 50px;
		width: 3000rpx;
	}

	.myScoll {
		width: 100%;
		height: 500rpx;
		white-space: nowrap;
		position: relative;
	}
</style>
