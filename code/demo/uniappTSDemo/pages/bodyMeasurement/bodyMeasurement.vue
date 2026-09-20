<template>
	<view class="box">
		<view class="header">
			<text class="header-title">身体成分测量</text>
		</view>

		<!-- 进度 -->
		<view class="progress-wrap">
			<view class="progress-text">测试进度：{{device.progress || 0}}%</view>
			<view class="progress-bar">
				<view class="progress-bar-fill" :style="{ width: (device.progress || 0) + '%' }"></view>
			</view>
		</view>

		<!-- 数据卡片 -->
		<view class="card">
			<view class="card-title">身体数据</view>
			<view class="data-grid">
				<view class="data-item">
					<text class="data-label">BMI</text>
					<text class="data-value">{{device.content?.BMI !== undefined ? device.content?.BMI : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">体脂率</text>
					<text class="data-value">{{device.content?.bodyFatPercentage !== undefined ? device.content?.bodyFatPercentage : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">脂肪量</text>
					<text class="data-value">{{device.content?.fatMass !== undefined ? device.content?.fatMass : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">去脂体重</text>
					<text class="data-value">{{device.content?.leanBodyMass !== undefined ? device.content?.leanBodyMass : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">肌肉率</text>
					<text class="data-value">{{device.content?.muscleRate !== undefined ? device.content?.muscleRate : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">肌肉量</text>
					<text class="data-value">{{device.content?.muscleMass !== undefined ? device.content?.muscleMass : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">皮下脂肪</text>
					<text class="data-value">{{device.content?.subcutaneousFat !== undefined ? device.content?.subcutaneousFat : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">体内水分</text>
					<text class="data-value">{{device.content?.bodyMoisture !== undefined ? device.content?.bodyMoisture : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">含水量</text>
					<text class="data-value">{{device.content?.waterContent !== undefined ? device.content?.waterContent : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">骨骼肌率</text>
					<text class="data-value">{{device.content?.skeletalMuscleRate !== undefined ? device.content?.skeletalMuscleRate : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">骨量</text>
					<text class="data-value">{{device.content?.boneMass !== undefined ? device.content?.boneMass : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">蛋白质占比</text>
					<text class="data-value">{{device.content?.proportionOfProtein !== undefined ? device.content?.proportionOfProtein : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">蛋白质量</text>
					<text class="data-value">{{device.content?.proteinAmount !== undefined ? device.content?.proteinAmount : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">基础代谢率</text>
					<text class="data-value">{{device.content?.basalMetabolicRate !== undefined ? device.content?.basalMetabolicRate : '--'}}</text>
				</view>
			</view>
		</view>

		<!-- 按钮 -->
		<view class="btn-group">
			<button class="btn btn-start" @click="BodyCompositionTestStartDataManager">开始检测</button>
			<button class="btn btn-stop" @click="BodyCompositionTestStopDataManager">停止检测</button>
		</view>

		<!-- 历史数据 -->
		<view class="card">
			<view class="card-title">历史数据</view>
			<view class="tip-text" v-if="!deviceIdList.length">为空表示没有ID</view>
			<view class="id-item" v-for="(item, index) in deviceIdList" :key="index">
				<text>数据ID：{{item.dataId}}</text>
			</view>
			<view class="btn-group">
				<button class="btn btn-primary" @click="startGetDataId">获取ID列表</button>
				<button class="btn btn-warn" @click="dataIdGetData">按ID读取</button>
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
				// 必须预置 content，否则 {{device.content.BMI}} 在 Vue3 下会因 content 为 undefined 抛错导致整页空白
				device: {
					progress: 0,
					content: {}
				},
				deviceIdList: []
			}
		},
		onLoad() {

		},
		onShow() {
			this.notifyMonitorValueChange();
		},
		methods: {
			BodyCompositionTestStartDataManager() {
				veepooFeature.veepooSendBodyCompositionTestStartDataManager();
			},
			BodyCompositionTestStopDataManager() {
				veepooFeature.veepooSendBodyCompositionTestStopDataManager()
			},
			startGetDataId() {
				veepooFeature.veepooSendReadBodyCompositionTestIdDataManager()
			},
			dataIdGetData() {
				let self = this;
				let deviceIdList = this.deviceIdList[0];
				console.log("deviceIdList=>", deviceIdList)
				let data = {
					dataId: deviceIdList.dataId
				}
				veepooFeature.veepooSendBodyCompositionIdReadDataManager(data)
			},
			// 监听订阅 notifyMonitorValueChange
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("身体成分 监听蓝牙回调=>", e);
					if (e.type == 32) {
						if (e.name == '身体成分检测') {
							self.device = e;
						} else if (e.name == '根据Id获取身体成分数据') {
							self.device = e;
						} else if (e.name == '身体成分读取测量保存的数据ID') {
							self.deviceIdList = e.content;
						}
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

	.progress-wrap {
		margin-bottom: 30rpx;
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

	.btn-group {
		display: flex;
		gap: 20rpx;
		margin: 20rpx 0;
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

	.tip-text {
		font-size: 26rpx;
		color: #999;
		margin-bottom: 16rpx;
	}

	.id-item {
		padding: 12rpx 0;
		font-size: 28rpx;
		color: #333;
	}
</style>
