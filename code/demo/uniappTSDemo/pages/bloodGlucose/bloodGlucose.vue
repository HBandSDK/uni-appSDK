<template>
	<view class="box">
		<view class="header">
			<text class="header-title">血糖测量</text>
		</view>

		<!-- 血糖测量 -->
		<view class="card">
			<view class="card-title">血糖测量</view>
			<view class="progress-wrap">
				<view class="progress-text">进度：{{bloodGlucoseData.Progress || 0}}</view>
				<view class="progress-bar">
					<view class="progress-bar-fill" :style="{ width: (bloodGlucoseData.Progress || 0) + '%' }"></view>
				</view>
			</view>
			<view class="data-grid">
				<view class="data-item">
					<text class="data-label">血糖值</text>
					<text class="data-value">{{(bloodGlucoseData.content && bloodGlucoseData.content.bloodGlucose) !== undefined ? bloodGlucoseData.content.bloodGlucose : '--'}}</text>
				</view>
			</view>
			<view class="btn-group">
				<button class="btn btn-primary" @click="startTest">校准测量</button>
				<button class="btn btn-warn" @click="noCalibrationStartTest">不校准测量</button>
				<button class="btn btn-stop" @click="stopTest">停止测量</button>
			</view>
		</view>

		<!-- 血糖校准 -->
		<view class="card">
			<view class="card-title">血糖校准</view>
			<view class="form-item">
				<text class="form-label">校准值</text>
				<view class="input-wrap">
					<input class="form-input" type="text" @input="inputValue1" placeholder="3-15" />
					<text class="unit">mmol/L</text>
				</view>
			</view>
			<view class="form-item form-item-switch">
				<text class="form-label">校准开关</text>
				<switch :checked="verifySwitch" @change="bindVerifySwitch" color="#00b0fb" />
			</view>
			<view class="btn-group">
				<button class="btn btn-primary" @click="startBloodVerify">设置校准</button>
				<button class="btn btn-warn" @click="stopBloodVerify">读取校准</button>
			</view>
			<view class="result-row">
				<text class="result-label">读取的血糖：</text>
				<text class="result-value">{{(bloodGlucoseData.content && bloodGlucoseData.content.bloodGlucoseValue) !== undefined ? bloodGlucoseData.content.bloodGlucoseValue : '--'}}</text>
			</view>
		</view>

		<!-- 六个血糖校准 -->
		<view class="card">
			<view class="card-title">六个时段血糖校准</view>
			<view class="six-item" v-for="(item, idx) in sixList" :key="idx">
				<view class="six-name">{{item.label}}</view>
				<view class="six-row">
					<view class="six-field">
						<text class="six-field-label">时间</text>
						<picker class="six-picker" mode="time" :value="item.time" start="00:00" end="23:59" @change="bindSixTime($event, idx)">
							<view class="picker-text">{{item.time}}</view>
						</picker>
					</view>
					<view class="six-field">
						<text class="six-field-label">血糖</text>
						<view class="input-wrap six-input-wrap">
							<input class="form-input" type="digit" :value="item.bloodGlucoseValue" @input="bindSixValue($event, idx)" placeholder="mmol/L" />
							<text class="unit">mmol/L</text>
						</view>
					</view>
				</view>
			</view>
			<view class="btn-group">
				<button class="btn btn-primary" @click="startBloodSixVerify">设置校准</button>
				<button class="btn btn-warn" @click="stopBloodSixVerify">读取校准</button>
			</view>
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
				// 必须预置 content，否则 {{bloodGlucoseData.content.bloodGlucose}} 在 Vue3 下会因 content 为 undefined 抛错导致整页空白
				bloodGlucoseData: {
					Progress: 0,
					content: {}
				},
				value1: 3,
				verifySwitch: false,
				sixList: [{
						label: '早餐前',
						key: 'beforeBreakfast',
						time: '07:05',
						bloodGlucoseValue: 8.2
					},
					{
						label: '早餐后',
						key: 'afterBreakfast',
						time: '10:55',
						bloodGlucoseValue: 8.2
					},
					{
						label: '午餐前',
						key: 'beforeLunch',
						time: '14:00',
						bloodGlucoseValue: 8.2
					},
					{
						label: '午餐后',
						key: 'afterLunch',
						time: '15:13',
						bloodGlucoseValue: 9.2
					},
					{
						label: '晚餐前',
						key: 'beforeDinner',
						time: '21:50',
						bloodGlucoseValue: 8.2
					},
					{
						label: '晚餐后',
						key: 'afterDinner',
						time: '21:46',
						bloodGlucoseValue: 8.2
					}
				]
			}
		},
		onLoad() {

		},
		onShow() {
			this.notifyMonitorValueChange()
		},
		methods: {
			bindVerifySwitch(e) {
				console.log('e==>', e);
				let val = e.detail.value
				let self = this;
				self.verifySwitch = val
			},

			// veepooSendBloodGlucoseMeasurementDataManager
			startTest() {
				let data = {
					switch: 'start',
					calibration: true, // true 开启校准模式  false 关闭校准模式
				}
				veepooFeature.veepooSendBloodGlucoseMeasurementDataManager(data);
			},

			inputValue1(e) {
				this.value1 = e.detail.value
			},
			noCalibrationStartTest() {
				let data = {
					switch: 'start',
					calibration: false, // true 开启校准模式  false 关闭校准模式
				}
				veepooFeature.veepooSendBloodGlucoseMeasurementDataManager(data);
			},
			stopTest() {
				let data = {
					switch: 'stop',
					calibration: true, // true 开启校准模式  false 关闭校准模式
				}
				veepooFeature.veepooSendBloodGlucoseMeasurementDataManager(data);
			},

			// 设置
			startBloodVerify() {
				// 注意：每次发送都需要将血糖转换为 mmol/L
				// mg/dl  转mmol/L 公式：血糖水平（mg/dl）= 血糖水平（mmol/L）× 18   血糖水平（mmol/L）= 血糖水平（mg/dl）/ 18
				let verifySwitch = this.verifySwitch;
				let data = {
					deviceControl: 'setup', // setup 设置  read 读取
					switch: verifySwitch ? 'start' : 'stop', // start 开启  stop 关闭
					bloodGlucoseValue: this.value1
				}
				console.log("data=>", data)
				veepooFeature.veepooSendBloodGlucoseCalibrateModuleDataManager(data);
			},
			// 读取
			stopBloodVerify() {
				let verifySwitch = this.verifySwitch;
				let data = {
					deviceControl: 'read', // setup 设置  read 读取
					switch: verifySwitch ? 'start' : 'stop', // start 开启  stop 关闭
					bloodGlucoseValue: this.value1
				}
				console.log("data=>", data)
				veepooFeature.veepooSendBloodGlucoseCalibrateModuleDataManager(data);
			},
			stopBloodSixVerify() {
				/*
				beforeBreakfast
				afterBreakfast
				beforeLunch
				afterLunch
				beforeDinner
				afterDinner
				*/
				let data = {
					conSwitch: 'start',
					switch: 'read', // setup 开启 read 关闭
				}
				veepooFeature.veepooSendSixBloodGlucoseCalibrateValueDataManager(data);
			},

			bindSixTime(e, idx) {
				let time = e.detail.value;
				let parts = time.split(':');
				this.$set(this.sixList[idx], 'time', time);
				this.$set(this.sixList[idx], 'hour', parts[0]);
				this.$set(this.sixList[idx], 'minute', parts[1]);
			},

			bindSixValue(e, idx) {
				let val = e.detail.value;
				this.$set(this.sixList[idx], 'bloodGlucoseValue', val);
			},


			startBloodSixVerify() {
				let sixData = {};
				this.sixList.forEach((item) => {
					let parts = (item.time || '00:00').split(':');
					sixData[item.key] = {
						hour: parts[0],
						minute: parts[1],
						bloodGlucoseValue: Number(item.bloodGlucoseValue) || 0
					};
				});
				let data = {
					conSwitch: 'start', // start 开启  stop 关闭
					switch: 'setup', // setup 设置 read 读取
					...sixData
				}
				console.log("data=>", data)
				// uni.navigateBack()
				veepooFeature.veepooSendSixBloodGlucoseCalibrateValueDataManager(data);
			},
			// 监听订阅 notifyMonitorValueChange
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("监听蓝牙回调=>", e);
					if (e.type) {
						self.bloodGlucoseData = e
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
		font-size: 36rpx;
		color: #333;
		font-weight: bold;
	}

	.form-item {
		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}

	.form-item-switch {
		justify-content: space-between;
	}

	.form-label {
		width: 160rpx;
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

	.btn-primary {
		background-color: #00b0fb;
	}

	.btn-warn {
		background-color: #ffa726;
	}

	.btn-stop {
		background-color: #ff6b6b;
	}

	.result-row {
		display: flex;
		align-items: center;
		margin-top: 20rpx;
		padding-top: 20rpx;
		border-top: 1px solid #f0f0f0;
	}

	.result-label {
		font-size: 28rpx;
		color: #666;
	}

	.result-value {
		font-size: 32rpx;
		color: #333;
		font-weight: bold;
	}

	.tip-text {
		font-size: 26rpx;
		color: #999;
		margin-bottom: 16rpx;
	}

	.six-item {
		padding: 20rpx 0;
		border-bottom: 1px solid #f0f0f0;
	}

	.six-item:last-child {
		border-bottom: none;
	}

	.six-name {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 12rpx;
	}

	.six-row {
		display: flex;
		gap: 20rpx;
	}

	.six-field {
		flex: 1;
		display: flex;
		flex-direction: column;
	}

	.six-field-label {
		font-size: 24rpx;
		color: #999;
		margin-bottom: 8rpx;
	}

	.six-picker {
		flex: 1;
	}

	.six-input-wrap {
		flex: 1;
	}
</style>
