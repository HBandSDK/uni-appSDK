<template>
	<view class="box">
		<view class="header">
			<text class="header-title">血压测量</text>
		</view>

		<!-- 通用血压测量 -->
		<view class="card">
			<view class="card-title">通用血压测量</view>
			<view class="progress-wrap">
				<view class="progress-text">进度：{{univerData.Progress || 0}}</view>
				<view class="progress-bar">
					<view class="progress-bar-fill" :style="{ width: (univerData.Progress || 0) + '%' }"></view>
				</view>
			</view>
			<view class="data-grid">
				<view class="data-item">
					<text class="data-label">高压</text>
					<text class="data-value">{{univerData.content?.bloodPressureHigh !== undefined ? univerData.content?.bloodPressureHigh : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">低压</text>
					<text class="data-value">{{univerData.content?.bloodPressureLow !== undefined ? univerData.content?.bloodPressureLow : '--'}}</text>
				</view>
			</view>
			<view class="btn-group">
				<button class="btn btn-start" @click="startTest">开启测量</button>
				<button class="btn btn-stop" @click="stopTest">停止测量</button>
			</view>
		</view>

		<!-- 私人血压定制 -->
		<view class="card">
			<view class="card-title">私人血压定制</view>
			<view class="form-item">
				<text class="form-label">高压</text>
				<view class="input-wrap">
					<input class="form-input" @input="value1" type="text" placeholder="请输入高压" />
					<text class="unit">mmHg</text>
				</view>
			</view>
			<view class="form-item">
				<text class="form-label">低压</text>
				<view class="input-wrap">
					<input class="form-input" @input="value2" type="text" placeholder="请输入低压" />
					<text class="unit">mmHg</text>
				</view>
			</view>
			<view class="btn-group">
				<button class="btn btn-primary" @click="startBlood">开启定制</button>
				<button class="btn btn-danger" @click="stopBlood">关闭定制</button>
				<button class="btn btn-warn" @click="readBlood">读取定制</button>
			</view>
		</view>

		<!-- 血压私人测量 -->
		<view class="card">
			<view class="card-title">血压私人测量</view>
			<view class="btn-group">
				<button class="btn btn-start" @click="startPrivateTest">开启测量</button>
				<button class="btn btn-stop" @click="stopPrivateTest">停止测量</button>
			</view>
		</view>
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
	export default {
		data() {
			return {
				// 必须预置 content，否则 {{univerData.content.bloodPressureHigh}} 在 Vue3 下会因 content 为 undefined 抛错导致整页空白
				univerData: {
					Progress: 0,
					content: {}
				},
				high: 0,
				low: 0
			}
		},
		onShow() {
			this.notifyMonitorValueChange()
		},
		methods: {
			startPrivateTest() {
				let data = {
					switch: 'start'
				}
				veepooFeature.veepooSendPrivateBloodPressureStupDataManager(data);
			},
			stopPrivateTest() {
				let data = {
					switch: 'stop'
				}
				veepooFeature.veepooSendPrivateBloodPressureStupDataManager(data);
			},

			startTest() {
				let data = {
					switch: 'start'
				}
				veepooFeature.veepooSendReadUniversalBloodPressureDataManager(data);
			},

			stopTest() {
				let data = {
					switch: 'stop'
				}
				veepooFeature.veepooSendReadUniversalBloodPressureDataManager(data);
			},

			startBlood() {
				let self = this;
				let data = {
					switch: 'start',
					bloodPressureHigh: self.high,
					bloodPressureLow: self.low
				}
				console.log('data==>', data)
				veepooFeature.veepooSendBloodPressurePrivateDataManager(data)
			},

			stopBlood() {
				let self = this;
				let data = {
					switch: 'stop',
					bloodPressureHigh: self.high,
					bloodPressureLow: self.low
				}
				veepooFeature.veepooSendBloodPressurePrivateDataManager(data)
			},

			readBlood() {
				let data = {
					switch: 'read',
					bloodPressureHigh: '0',
					bloodPressureLow: '0'
				}
				veepooFeature.veepooSendBloodPressurePrivateDataManager(data)
			},
			value1(e) {
				let self = this;
				self.high = e.detail.value
			},
			value2(e) {
				let self = this;
				self.low = e.detail.value
			},
			// 监听订阅 notifyMonitorValueChange
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("监听蓝牙回调=>", e);
					if (e.type == 18) {
						self.univerData = e
					}
					if (e.type == 28) {

					}
				})
			},
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
		margin-bottom: 24rpx;
	}

	.card-title {
		font-size: 30rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.progress-wrap {
		margin-bottom: 20rpx;
	}

	.progress-text {
		font-size: 28rpx;
		color: #666;
		margin-bottom: 12rpx;
	}

	.progress-bar {
		width: 100%;
		height: 16rpx;
		background: #f0f0f0;
		border-radius: 10rpx;
		overflow: hidden;
	}

	.progress-bar-fill {
		height: 100%;
		background: #00b0fb;
		border-radius: 10rpx;
		transition: width 0.3s ease;
	}

	.data-grid {
		display: flex;
		flex-wrap: wrap;
		margin-bottom: 20rpx;
	}

	.data-item {
		width: 50%;
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
		font-size: 32rpx;
		color: #333;
		font-weight: bold;
	}

	.form-item {
		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}

	.form-label {
		width: 140rpx;
		font-size: 28rpx;
		color: #666;
		flex-shrink: 0;
	}

	.input-wrap {
		flex: 1;
		display: flex;
		align-items: center;
		background-color: #f5f7fa;
		border-radius: 8rpx;
		padding-right: 20rpx;
	}

	.form-input {
		flex: 1;
		padding: 15rpx 20rpx;
		font-size: 28rpx;
		color: #333;
	}

	.unit {
		font-size: 24rpx;
		color: #999;
		flex-shrink: 0;
	}

	.btn-group {
		display: flex;
		gap: 20rpx;
		margin-top: 10rpx;
	}

	.btn {
		flex: 1;
		border-radius: 12rpx;
		font-size: 30rpx;
		color: #fff;
	}

	.btn-start {
		background-color: #00b0fb;
	}

	.btn-stop {
		background-color: #ff6b6b;
	}

	.btn-primary {
		background-color: #00b0fb;
	}

	.btn-warn {
		background-color: #ffa726;
	}

	.btn-danger {
		background-color: #ff6b6b;
	}
</style>
