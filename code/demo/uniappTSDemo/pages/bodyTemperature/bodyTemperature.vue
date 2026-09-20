<template>
	<view class="page">
		<!-- 标题 -->
		<view class="header">
			<text class="header-title">体温手动测量</text>
		</view>

		<!-- 数据卡片 -->
		<view class="card">
			<view class="card-row">
				<text class="card-label">体温</text>
				<text class="card-value">{{ deviceInfo.content?.bodyTemperature !== undefined ? deviceInfo.content.bodyTemperature : '--' }}</text>
				<text class="card-unit" v-if="deviceInfo.content?.bodyTemperature !== undefined">°C</text>
			</view>
			<view class="card-divider"></view>
			<view class="card-row">
				<text class="card-label">体表温度</text>
				<text class="card-value">{{ deviceInfo.content?.bodySurfaceTemperature !== undefined ? deviceInfo.content.bodySurfaceTemperature : '--' }}</text>
				<text class="card-unit" v-if="deviceInfo.content?.bodySurfaceTemperature !== undefined">°C</text>
			</view>
		</view>

		<!-- 进度 -->
		<view class="progress-section" v-if="isTest">
			<view class="progress-bar-bg">
				<view class="progress-bar-fill" :style="{ width: deviceInfo.progress + '%' }"></view>
			</view>
			<text class="progress-text">测试进度：{{ deviceInfo.progress }}%</text>
		</view>

		<!-- 按钮 -->
		<view class="btn-group">
			<button class="btn btn-start" :class="{ 'btn-disabled': isTest }" :disabled="isTest" @click="TemperatureMeasurementSwitchManager">
				开始测试
			</button>
			<button class="btn btn-stop" :class="{ 'btn-disabled': !isTest }" :disabled="!isTest" @click="stopTemperatureMeasurement">
				停止测试
			</button>
		</view>
	</view>
</template>

<script>
	import {
		veepooBle,
		veepooFeature
	} from '../../common/index.js'

	export default {
		data() {
			return {
				deviceInfo: {
					progress: 0,
					content: {
						bodyTemperature: undefined,
						bodySurfaceTemperature: undefined,
					}
				},
				isTest: false
			}
		},
		onShow() {
			this.notifyMonitorValueChange()
		},
		methods: {
			TemperatureMeasurementSwitchManager() {
				let self = this;
				self.isTest = true;
				self.deviceInfo.progress = 0;
				let data = {
					switch: true
				}
				veepooFeature.veepooSendTemperatureMeasurementSwitchManager(data)
			},
			stopTemperatureMeasurement() {
				let self = this;
				self.isTest = false;
				let data = {
					switch: false
				}
				veepooFeature.veepooSendTemperatureMeasurementSwitchManager(data)
			},
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("体温手动测量=>", e)
					if (e.type == 6) {
						self.deviceInfo.progress = e.progress;
						if (e.content) {
							self.deviceInfo.content = e.content
						}
						if (e.progress === 100) {
							self.isTest = false
						}
					}
				})
			},
		}
	}
</script>

<style>
	.page {
		padding: 30rpx;
	}

	.header {
		text-align: center;
		padding: 30rpx 0 40rpx;
	}

	.header-title {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
	}

	.card {
		background: #fff;
		border-radius: 16rpx;
		padding: 30rpx;
		box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.06);
	}

	.card-row {
		display: flex;
		align-items: baseline;
		justify-content: space-between;
		padding: 16rpx 0;
	}

	.card-label {
		font-size: 30rpx;
		color: #666;
	}

	.card-value {
		font-size: 48rpx;
		font-weight: bold;
		color: #00b0fb;
		margin-right: 8rpx;
	}

	.card-unit {
		font-size: 28rpx;
		color: #999;
	}

	.card-divider {
		height: 1rpx;
		background: #eee;
	}

	.progress-section {
		margin-top: 40rpx;
	}

	.progress-bar-bg {
		height: 20rpx;
		background: #eee;
		border-radius: 10rpx;
		overflow: hidden;
	}

	.progress-bar-fill {
		height: 100%;
		background: #00b0fb;
		border-radius: 10rpx;
		transition: width 0.3s ease;
	}

	.progress-text {
		display: block;
		text-align: center;
		margin-top: 12rpx;
		font-size: 26rpx;
		color: #999;
	}

	.btn-group {
		display: flex;
		gap: 20rpx;
		margin-top: 60rpx;
	}

	.btn {
		flex: 1;
		background-color: #00b0fb;
		color: #fff;
		border-radius: 12rpx;
		font-size: 32rpx;
	}

	.btn-stop {
		background-color: #ff6b6b;
	}

	.btn-disabled {
		background-color: #ccc;
	}
</style>
