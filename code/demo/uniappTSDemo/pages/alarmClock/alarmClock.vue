<template>
	<view class="box">
		<view class="header">
			<text class="header-title">闹钟管理</text>
		</view>

		<!-- 读取按钮 -->
		<view class="action-row">
			<button class="btn btn-primary" @click="ReadAlarmClockDataManager">读取闹钟</button>
		</view>

		<!-- 设置闹钟 -->
		<view class="card">
			<view class="card-title">设置闹钟</view>
			<view class="form-item">
				<text class="form-label">闹钟ID</text>
				<input class="form-input" @input="getId" type="text" placeholder="请输入闹钟ID" />
			</view>
			<view class="form-item">
				<text class="form-label">闹钟时间</text>
				<picker class="form-picker" mode="time" :value="startTime" start="00:00" end="23:59" @change="bindStartTimeChange">
					<view class="picker-text">{{startTime}}</view>
				</picker>
			</view>
			<view class="form-item">
				<text class="form-label">闹钟标签</text>
				<input class="form-input" @input="getLabel" type="text" placeholder="请输入闹钟标签" />
			</view>
			<view class="form-item form-item-switch">
				<text class="form-label">是否开启</text>
				<switch :checked="alarmSwitch" @change="getalarmSwitch" color="#00b0fb" />
			</view>
			<view class="form-item form-item-column">
				<text class="form-label">重复</text>
				<view class="week-grid">
					<view class="week-item" :class="{ 'week-item-active': alarmRepeat.Monday }" @click="toggleWeekday('Monday')">一</view>
					<view class="week-item" :class="{ 'week-item-active': alarmRepeat.Tuesday }" @click="toggleWeekday('Tuesday')">二</view>
					<view class="week-item" :class="{ 'week-item-active': alarmRepeat.Wednesday }" @click="toggleWeekday('Wednesday')">三</view>
					<view class="week-item" :class="{ 'week-item-active': alarmRepeat.Thursday }" @click="toggleWeekday('Thursday')">四</view>
					<view class="week-item" :class="{ 'week-item-active': alarmRepeat.Friday }" @click="toggleWeekday('Friday')">五</view>
					<view class="week-item" :class="{ 'week-item-active': alarmRepeat.Saturday }" @click="toggleWeekday('Saturday')">六</view>
					<view class="week-item" :class="{ 'week-item-active': alarmRepeat.Sunday }" @click="toggleWeekday('Sunday')">日</view>
				</view>
			</view>
			<button class="btn btn-primary" @click="SetAlarmClockDataManager">设置闹钟</button>
		</view>

		<!-- 删除闹钟 -->
		<view class="card">
			<view class="card-title">删除闹钟</view>
			<view class="form-item">
				<text class="form-label">闹钟ID</text>
				<input class="form-input" @input="deleteAlId" type="text" placeholder="请输入要删除的ID" />
			</view>
			<button class="btn btn-danger" @click="deleteAlarmClockDataManager">删除闹钟</button>
		</view>

		<!-- 修改闹钟 -->
		<view class="card">
			<view class="card-title">修改闹钟</view>
			<view class="form-item">
				<text class="form-label">闹钟ID</text>
				<input class="form-input" @input="UPgetId" type="text" placeholder="请输入闹钟ID" />
			</view>
			<view class="form-item">
				<text class="form-label">闹钟时间</text>
				<picker class="form-picker" mode="time" :value="UPstartTime" start="00:00" end="23:59" @change="UPbindStartTimeChange">
					<view class="picker-text">{{UPstartTime}}</view>
				</picker>
			</view>
			<view class="form-item form-item-switch">
				<text class="form-label">是否开启</text>
				<switch :checked="UPalarmSwitch" @change="UPgetalarmSwitch" color="#00b0fb" />
			</view>
			<button class="btn btn-warn" @click="updateAlarmClockDataManager">修改闹钟</button>
		</view>

		<!-- 闹钟列表 -->
		<view class="card" v-if="alarmList && alarmList.length">
			<view class="card-title">闹钟列表（{{alarmList.length}}）</view>
			<view class="alarm-item" v-for="(item, index) in alarmList" :key="index">
				<view class="alarm-row">
					<text class="alarm-time">{{item.time || '--'}}</text>
					<text class="alarm-id">ID: {{item.alarmId}}</text>
				</view>
				<view class="alarm-status">
					<text class="alarm-label">{{item.alarmSwitch ? '已开启' : '已关闭'}}</text>
				</view>
			</view>
		</view>
		<view class="empty-tip" v-else>
			<text>暂无闹钟数据</text>
		</view>
	</view>
</template>

<script>
	// 引入方式二：
	import {
		veepooBle,
		veepooFeature
	} from '../../common/index.js'
	export default {
		data() {
			return {
				index: 1,
				startTime: '00:00',
				alarmSwitch: false,
				alarmId: '',
				deleteAlarmId: '',
				UPstartTime: '00:00',
				UPalarmSwitch: false,
				UPalarmId: '',
				alarmList: [],
				lable: "",
				alarmRepeat: {
					"Monday": true,
					"Tuesday": true,
					"Wednesday": true,
					"Thursday": true,
					"Friday": true,
					"Saturday": false,
					"Sunday": false
				}
			}
		},
		onLoad(options) {
			let check = [0xB9, 0x02, 0x01, 0x01, 0xA0, 0x02, 0x60, 0x55, 0xA1, 0x12, 0xB1, 0x08, 0x02, 0x01, 0x21, 0x09, 0x2E, 0x00, 0x00, 0x00, 0xB2, 0x06, 0xE5, 0x93, 0x88, 0xE5, 0x93, 0x88]


			let arr = [160, 33, 0, 161, 1, 1, 162, 9, 233, 153, 136, 230, 152, 190, 230, 150, 135, 163, 11, 49, 53, 50, 56, 57, 53, 57, 54, 50, 52, 50, 164, 1, 1];
			// ["b9", "02", "01", "01", "a0", "02", "07", "bc", "a1", "24", "b1", "08", "03", "01", "1f", "15", "00", "00", "00", "00", "b2", "18", "e6", "96", "87", "e5", "ad", "97", "e9", "97", "b9", "e9", "92", "9f", "e6", "b5", "8b", "e8", "af", "95", "e6", "a0", "87", "e7", "ad", "be"];

			let hex = [];
			for (let i = 0; i < arr.length; i++) {
				let item = arr[i];
				hex.push(item.toString(16).padStart(2, '0'))
			}
			console.log("hex====>", hex);
			this.notifyMonitorValueChange();
		},
		onShow() {

		},
		methods: {
			// 读取
			ReadAlarmClockDataManager() {
				veepooFeature.veepooSendReadAlarmClockDataManager();
			},
			// 监听订阅 notifyMonitorValueChange
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log(" ss 监听蓝牙回调=>", e);
					self.alarmList = e.content
				})
			},
			bindStartTimeChange(e) {
				console.log('picker发送选择改变，携带值为', e.detail.value)
				this.startTime = e.detail.value
			},
			getId(e) {
				let alarmId = e.detail.value;
				this.alarmId = alarmId
			},
			getLabel(e) {
				let lable = e.detail.value;
				this.lable = lable
			},
			getalarmSwitch(e) {
				let alarmSwitch = e.detail.value;
				this.alarmSwitch = alarmSwitch
			},
			toggleWeekday(day) {
				this.alarmRepeat[day] = !this.alarmRepeat[day];
			},
			UPbindStartTimeChange(e) {
				console.log('picker发送选择改变，携带值为', e.detail.value)
				this.UPstartTime = e.detail.value
			},
			UPgetId(e) {
				let UPalarmId = e.detail.value;
				this.UPalarmId = UPalarmId
			},
			UPgetalarmSwitch(e) {
				let UPalarmSwitch = e.detail.value;
				this.UPalarmSwitch = UPalarmSwitch
			},
			deleteAlId(e) {
				let deleteAlarmId = e.detail.value;
				this.deleteAlarmId = deleteAlarmId
			},
			deleteAlarmClockDataManager() {
				let data = {
					alarmId: this.deleteAlarmId,
					switch: true,
					time: '08:00',
					alarmRepeat: {
						"Monday": true,
						"Tuesday": true,
						"Wednesday": true,
						"Thursday": true,
						"Friday": true,
						"Saturday": false,
						"Sunday": false
					}
				}
				veepooFeature.veepooSendDeleteAlarmClockDataManager(data);
			},
			SetAlarmClockDataManager() {
				let data = {
					alarmId: this.alarmId,
					switch: this.alarmSwitch,
					time: this.startTime,
					alarmRepeat: this.alarmRepeat,
					name: this.lable
				}
				console.log("data=>", data)
				veepooFeature.veepooSendSetAlarmClockDataManager(data);
			},
			updateAlarmClockDataManager() {
				let data = {
					alarmId: this.UPalarmId,
					switch: this.UPalarmSwitch,
					time: this.UPstartTime,
					alarmRepeat: this.alarmRepeat,
					name: "猪猪小猪猪"
				}
				console.log("data=>", data)
				veepooFeature.veepooSendSetAlarmClockDataManager(data);
				this.index = this.index + 1
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

	.form-item {
		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}

	.form-item-switch {
		justify-content: space-between;
	}

	.form-item-column {
		flex-direction: column;
		align-items: flex-start;
	}

	.form-item-column .form-label {
		margin-bottom: 16rpx;
	}

	.week-grid {
		display: flex;
		gap: 12rpx;
		flex-wrap: wrap;
		width: 100%;
	}

	.week-item {
		width: 70rpx;
		height: 70rpx;
		border-radius: 50%;
		background-color: #f5f7fa;
		color: #666;
		font-size: 26rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.week-item-active {
		background-color: #00b0fb;
		color: #fff;
	}

	.form-label {
		width: 180rpx;
		font-size: 28rpx;
		color: #666;
		flex-shrink: 0;
	}

	.form-input {
		flex: 1;
		background-color: #f5f7fa;
		padding: 15rpx 20rpx;
		border-radius: 8rpx;
		font-size: 28rpx;
	}

	.form-picker {
		flex: 1;
	}

	.picker-text {
		background-color: #f5f7fa;
		padding: 15rpx 20rpx;
		border-radius: 8rpx;
		font-size: 28rpx;
		color: #333;
	}

	.btn {
		width: 100%;
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

	.btn-danger {
		background-color: #ff6b6b;
	}

	.action-row {
		margin-bottom: 24rpx;
	}

	.alarm-item {
		padding: 20rpx 0;
		border-bottom: 1px solid #f0f0f0;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.alarm-item:last-child {
		border-bottom: none;
	}

	.alarm-row {
		display: flex;
		flex-direction: column;
		gap: 6rpx;
	}

	.alarm-time {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}

	.alarm-id {
		font-size: 24rpx;
		color: #999;
	}

	.alarm-status {
		display: flex;
		align-items: center;
	}

	.alarm-label {
		font-size: 24rpx;
		color: #00b0fb;
		padding: 6rpx 16rpx;
		background-color: #e6f5ff;
		border-radius: 20rpx;
	}

	.empty-tip {
		text-align: center;
		padding: 60rpx 0;
		color: #999;
		font-size: 28rpx;
	}
</style>
