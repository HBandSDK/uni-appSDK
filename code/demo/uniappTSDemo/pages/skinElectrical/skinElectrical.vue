<template>
	<view class="page-wrap">
		<view class="page-title">皮肤电活动测量</view>

		<!-- 数据卡片 -->
		<view class="data-card" v-if="hasData">
			<view class="card-item">
				<text class="label">运行模式</text>
				<text class="value">{{skinData.model === 'start' ? '测量中' : '已停止'}}</text>
			</view>
			<view class="card-item">
				<text class="label">设备状态</text>
				<text class="value">{{skinData.state}}</text>
			</view>
			<view class="card-item">
				<text class="label">测量进度</text>
				<text class="value">{{skinData.Progress}}</text>
			</view>
			<view class="card-item">
				<text class="label">情绪等级 [-10 ~ 10]</text>
				<text class="value">{{skinData.content.emotionLevel || 0}}</text>
			</view>
			<view class="card-item">
				<text class="label">皮肤含水量 [1 ~ 99]</text>
				<text class="value">{{skinData.content.skinMoisture || 0}}</text>
			</view>
			<view class="card-item">
				<text class="label">抑郁风险等级</text>
				<text class="value">{{skinData.content.depressionRisk || 0}}</text>
			</view>
			<view class="card-item">
				<text class="label">交感神经活跃度 [1 ~ 99]</text>
				<text class="value">{{skinData.content.snsActivation || 0}}</text>
			</view>
			<view class="card-item">
				<text class="label">皮质醇浓度</text>
				<text class="value">{{skinData.content.cortisolValue || 0}}</text>
			</view>
		</view>

		<!-- 空状态 -->
		<view class="empty-tip" v-else>
			<text>暂无测量数据，请点击下方按钮开始测量</text>
		</view>

		<!-- 操作按钮 -->
		<view class="btn-group">
			<button @click="startSkinElectricalActivityTest" class="btn-start">开始测量</button>
			<button @click="closeSkinElectricalActivityTest" class="btn-stop">停止测量</button>
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
				skinData: {
					content: {}
				},
				_notifyRegistered: false
			}
		},
		computed: {
			hasData() {
				return this.skinData && Number(this.skinData.type) === 62
			}
		},
		onShow() {
			this.notifyMonitorValueChange();
		},
		onUnload() {
			// 若 SDK 提供取消监听方法，请在此调用；否则重置标志，避免重复注册
			this._notifyRegistered = false;
		},
		methods: {
			// 开始皮肤电活动测试
			startSkinElectricalActivityTest() {
				veepooFeature.veepooSkinElectricalActivityStartManager()
			},

			// 关闭皮肤电活动测试
			closeSkinElectricalActivityTest() {
				veepooFeature.veepooSkinElectricalActivityCloseManager()
			},

			// 监听订阅 notifyMonitorValueChange
			notifyMonitorValueChange() {
				if (this._notifyRegistered) return;
				this._notifyRegistered = true;
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log(" 皮肤电活动测量 监听蓝牙回调=>", e);
					if (e && Number(e.type) === 62) {
						self.skinData = {
							...e,
							content: e.content || {}
						};
					}
				})
			},
		}
	}
</script>

<style>
	.page-wrap {
		padding: 32rpx;
		background-color: #f6f7f9;
		min-height: 100vh;
		box-sizing: border-box;
	}

	.page-title {
		font-size: 38rpx;
		font-weight: 600;
		text-align: center;
		margin-bottom: 40rpx;
		color: #222;
	}

	.data-card {
		background-color: #ffffff;
		border-radius: 20rpx;
		padding: 36rpx;
		box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.06);
	}

	.data-card .card-item {
		display: flex;
		justify-content: space-between;
		padding: 20rpx 0;
		border-bottom: 1rpx solid #eee;
	}

	.data-card .card-item:last-child {
		border-bottom: none;
	}

	.data-card .card-item .label {
		font-size: 28rpx;
		color: #666;
	}

	.data-card .card-item .value {
		font-size: 28rpx;
		color: #222;
		font-weight: 500;
	}

	.empty-tip {
		text-align: center;
		padding: 80rpx 0;
		color: #999;
		font-size: 28rpx;
	}

	.btn-group {
		margin-top: 60rpx;
		display: flex;
		gap: 24rpx;
	}

	.btn-group button {
		flex: 1;
		border-radius: 12rpx;
		font-size: 30rpx;
	}

	.btn-group .btn-start {
		background-color: #07c160;
		color: #fff;
	}

	.btn-group .btn-stop {
		background-color: #fff;
		color: #f56c6c;
		border: 1rpx solid #f56c6c;
	}
</style>