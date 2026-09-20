<template>
	<view class="container">
		<view class="section">
			<view class="section-title">读取世界时钟</view>
			<view class="btn-row">
				<button class="btn btn-primary" @click="readWorldClock">读取</button>
			</view>
			<view v-if="progress > 0 && progress < 100" class="progress">进度: {{progress}}%</view>
			<view v-if="readState" class="state">状态: {{readState}}</view>
			<view v-if="clockList.length > 0" class="clock-list">
				<view class="clock-item" v-for="(item, index) in clockList" :key="item.id">
					<view class="clock-line">ID: {{item.id}}</view>
					<view class="clock-line">城市: {{item.city}}</view>
					<view class="clock-line">时区偏移: {{item.timeZone}} (×15分钟)</view>
				</view>
			</view>
			<view v-else class="empty-tip">暂无数据</view>
		</view>

		<view class="divider"></view>

		<view class="section">
			<view class="section-title">添加世界时钟</view>
			<view class="input-row">
				<text class="label">id:</text>
				<input class="input" :value="id" @input="onIdInput" placeholder="如: 1、2、3" />
			</view>
			<view class="input-row">
				<text class="label">城市:</text>
				<input class="input" :value="addCity" @input="onCityInput" placeholder="如: 北京" />
			</view>
			<view class="input-row">
				<text class="label">时区偏移(×15分钟):</text>
				<input class="input" :value="addTimezone" @input="onTimezoneInput" placeholder="如: 0/-32/36" />
			</view>
			<view class="btn-row">
				<button class="btn btn-success" @click="addWorldClock">添加</button>
			</view>
		</view>

		<view class="divider"></view>

		<view class="section">
			<view class="section-title">调整时钟顺序</view>
			<view class="input-row">
				<text class="label">From ID:</text>
				<input class="input" type="number" :value="fromId" @input="onFromIdInput" />
			</view>
			<view class="input-row">
				<text class="label">To ID:</text>
				<input class="input" type="number" :value="toId" @input="onToIdInput" />
			</view>
			<view class="btn-row">
				<button class="btn btn-warning" @click="adjustWorldClock">调整</button>
			</view>
		</view>

		<view class="divider"></view>

		<view class="section">
			<view class="section-title">删除世界时钟</view>
			<view class="input-row">
				<text class="label">删除 ID:</text>
				<input class="input" type="number" :value="deleteId" @input="onDeleteIdInput" />
			</view>
			<view class="btn-row">
				<button class="btn btn-danger" @click="deleteWorldClock">删除</button>
			</view>
		</view>

		<view class="divider"></view>

		<!-- <view class="section">
			<view class="section-title">蓝牙回调日志</view>
			<scroll-view class="log-box" scroll-y enable-flex>
				<text class="log-text">{{logText || '暂无日志'}}</text>
			</scroll-view>
		</view> -->
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
				id: 1,
				clockList: [],
				progress: 0,
				readState: '',
				addCity: '北京',
				addTimezone: 0,
				fromId: 1,
				toId: 2,
				deleteId: 1,
				logText: '',
				_notifyRegistered: false
			}
		},
		onShow() {
			this.notifyMonitorValueChange();
		},
		methods: {
			notifyMonitorValueChange() {
				if (this._notifyRegistered) return;
				this._notifyRegistered = true;
				let that = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("世界时钟 监听蓝牙回调=>", e);
					if (e.type == 61) {
						that.appendLog(JSON.stringify(e));
						if (e.name == '读取世界时钟') {
							if (e.Progress !== undefined && e.Progress < 100) {
								that.progress = e.Progress;
							} else {
								that.clockList = e.content || [];
								that.progress = 100;
								that.readState = e.state;
							}
						} else if (e.name == '添加世界时钟') {
							that.appendLog('添加结果: ' + e.state + ' CRC: ' + e.CRC);
						} else if (e.name == '调整时钟顺序') {
							that.appendLog('调整结果: ' + e.state + ' CRC: ' + e.CRC);
						} else if (e.name == '删除世界时钟') {
							that.appendLog('删除结果: ' + e.state + ' ID: ' + e.worldClockId);
						}
					}
				});
			},

			appendLog(text) {
				let prev = this.logText || '';
				let now = new Date();
				let time =
					`${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}:${now.getSeconds().toString().padStart(2, '0')}`;
				this.logText = `[${time}] ${text}\n${prev}`;
			},

			readWorldClock() {
				console.log('读取世界时钟');
				this.clockList = [];
				this.progress = 0;
				this.readState = '';
				veepooFeature.veepooSendReadWorldClockDataManager({
					CRC: 0
				});
			},

			addWorldClock() {
				console.log('添加世界时钟');
				let item = {
					id: Number(this.id),
					timezone: Number(this.addTimezone),
					city: this.addCity
				};
				console.log('添加参数=>', item);
				veepooFeature.veepooSendAddWorldClockDataManager(item);
			},

			adjustWorldClock() {
				console.log('调整时钟顺序');
				let data = {
					fromId: Number(this.fromId),
					toId: Number(this.toId)
				};
				console.log('调整参数=>', data);
				veepooFeature.veepooSendAdujstWorldClockDataManager(data);
			},

			deleteWorldClock() {
				console.log('删除世界时钟');
				let data = {
					worldClockId: Number(this.deleteId)
				};
				console.log('删除参数=>', data);
				veepooFeature.veepooSendDeleteWorldClockDataManager(data);
			},

			onCityInput(e) {
				this.addCity = e.detail.value;
			},
			onIdInput(e) {
				this.id = e.detail.value;
			},
			onTimezoneInput(e) {
				this.addTimezone = e.detail.value;
			},
			onFromIdInput(e) {
				this.fromId = e.detail.value;
			},
			onToIdInput(e) {
				this.toId = e.detail.value;
			},
			onDeleteIdInput(e) {
				this.deleteId = e.detail.value;
			}
		}
	}
</script>

<style>
	/* pages/worldClock/index.wxss */
	.container {
		padding: 20rpx;
		background: #f5f5f5;
		min-height: 100vh;
	}

	.section {
		background: #fff;
		border-radius: 16rpx;
		padding: 24rpx;
		margin-bottom: 20rpx;
	}

	.section-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.btn-row {
		display: flex;
		gap: 16rpx;
		margin-top: 16rpx;
	}

	.btn {
		flex: 1;
		font-size: 28rpx;
		border-radius: 12rpx;
	}

	.btn-primary {
		background: #1890ff;
		color: #fff;
	}

	.btn-success {
		background: #52c41a;
		color: #fff;
	}

	.btn-warning {
		background: #faad14;
		color: #fff;
	}

	.btn-danger {
		background: #ff4d4f;
		color: #fff;
	}

	.progress {
		font-size: 26rpx;
		color: #1890ff;
		margin-top: 12rpx;
	}

	.state {
		font-size: 26rpx;
		color: #52c41a;
		margin-top: 8rpx;
	}

	.clock-list {
		margin-top: 16rpx;
	}

	.clock-item {
		background: #fafafa;
		border-radius: 12rpx;
		padding: 16rpx;
		margin-bottom: 12rpx;
	}

	.clock-line {
		font-size: 26rpx;
		color: #666;
		margin: 4rpx 0;
	}

	.empty-tip {
		font-size: 26rpx;
		color: #999;
		text-align: center;
		padding: 20rpx;
	}

	.input-row {
		display: flex;
		align-items: center;
		margin-bottom: 12rpx;
	}

	.label {
		font-size: 28rpx;
		color: #333;
		width: 260rpx;
	}

	.input {
		flex: 1;
		height: 64rpx;
		border: 1rpx solid #ddd;
		border-radius: 8rpx;
		padding: 0 16rpx;
		font-size: 28rpx;
	}

	.divider {
		height: 1rpx;
		background: #e8e8e8;
		margin: 10rpx 0;
	}

	.log-box {
		background: #1e1e1e;
		border-radius: 8rpx;
		padding: 16rpx;
		height: 400rpx;
		margin-top: 12rpx;
	}

	.log-text {
		font-size: 22rpx;
		color: #d4d4d4;
		font-family: monospace;
		white-space: pre-wrap;
	}
</style>