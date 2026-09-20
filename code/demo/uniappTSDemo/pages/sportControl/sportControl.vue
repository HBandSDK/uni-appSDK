<template>
	<view class="box">
		<!-- 运动控制按钮组 -->
		<text>1.输入运动模式点击控制按钮</text>
		<view class="btn_box">
			<view class="box-mode-state">
				<view class="mode-state-item" @click="startSport">开启设备运动</view>
				<view class="mode-state-item" @click="pausedSport">暂停设备运动</view>
			</view>
			<view class="box-mode-state">
				<view class="mode-state-item" @click="continueSport">继续设备运动</view>
				<view class="mode-state-item" @click="stopSport">停止设备运动</view>
			</view>
		</view>

		<!-- 运动模式输入 -->
		<view class="input">
			<text>运动模式：</text>
			<input type="number" placeholder="输入运动模式" v-model.number="sportMode" />
		</view>

		<text>2.点击操作按钮后开始读取</text>
		<!-- 读取控制按钮 -->
		<button style="margin-top: 20px;" @click="toggleRead">
			{{ isReading ? '停止读取' : '开始读取' }}
		</button>

		<!-- 实时运动数据显示 -->
		<view class="data-box" style="margin-top: 20px; padding: 10px; border: 1px solid #ccc; border-radius: 8px;">
			<view class="data-title" style="font-weight: bold; margin-bottom: 8px;">实时运动数据</view>
			<view>运动模式：{{ sportData.sportModel }}</view>
			<view>运行状态：{{ sportData.runState }}</view>
			<view>设备状态：{{ sportData.deviceState }}</view>
			<view>运动时间：{{ sportData.exerciseTimeStamp }}</view>
			<view>运动距离：{{ sportData.exerciseDistance }} 公里</view>
			<view>心率：{{ sportData.heartRate }} 次/分</view>
			<view>卡路里：{{ sportData.calories }} 千卡</view>
			<view>配速：{{ sportData.pace }} /公里</view>
			<view>速度：{{ sportData.speed }} 公里/小时</view>
			<view v-if="sportData.gnssInfo">
				<view>GNSS类型：{{ sportData.gnssInfo.isGnssType ? '是' : '否' }}</view>
				<view>GNSS信号：{{ sportData.gnssInfo.gnssSignal }}</view>
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
				opCode: 1,
				sportMode: 0,
				isReading: false,
				sportData: {
					sportModel: 0,
					opCode: 0,
					runState: 'Unknown',
					deviceState: 'Unknown',
					exerciseTimeStamp: '00:00:00',
					exerciseDistance: '0.00',
					heartRate: 0,
					calories: '0.0',
					pace: "0'00\"",
					speed: '0.00',
					gnssInfo: null,
					opCodeText: '',
				},
				timerId: null,
			}
		},
		mounted() {
			this.notifyMonitorValueChange()
		},
		beforeUnmount() {
			if (this.isReading) {
				this.isReading = false
				if (this.timerId) {
					clearTimeout(this.timerId)
					this.timerId = null
				}
			}
		},
		methods: {
			// ----- 运动控制指令 -----
			startSport() {
				const data = {
					switch: 'setup',
					sportMode: this.sportMode,
					opCode: 1,
				}
				console.log('运动控制下发数据:', data)
				veepooFeature.veepooSendSportControlDataManager(data)
			},
			pausedSport() {
				const data = {
					switch: 'setup',
					sportMode: this.sportMode,
					opCode: 2,
				}
				console.log('运动控制下发数据:', data)
				veepooFeature.veepooSendSportControlDataManager(data)
			},
			continueSport() {
				const data = {
					switch: 'setup',
					sportMode: this.sportMode,
					opCode: 3,
				}
				console.log('运动控制下发数据:', data)
				veepooFeature.veepooSendSportControlDataManager(data)
			},
			stopSport() {
				const data = {
					switch: 'setup',
					sportMode: this.sportMode,
					opCode: 4,
				}
				console.log('运动控制下发数据:', data)
				veepooFeature.veepooSendSportControlDataManager(data)
			},

			// 切换读取状态
			toggleRead() {
				if (this.isReading) {
					this.isReading = false
					if (this.timerId) {
						clearTimeout(this.timerId)
						this.timerId = null
					}
					console.log('已停止读取')
					return
				}
				this.isReading = true
				console.log('开始读取')
				const sendRequest = () => {
					if (!this.isReading) return
					const data = {
						switch: 'read',
						sportMode: this.sportMode,
					}
					veepooFeature.veepooSendSportControlDataManager(data)
					this.timerId = setTimeout(sendRequest, 300)
				}
				sendRequest()
			},

			// 注册回调并转换单位
			notifyMonitorValueChange() {
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange((e) => {
					console.log('运动控制回调 =>', e)
					if (!e || !e.content) return
					const content = e.content

					// --- 单位转换 ---
					const rawTime = content.exerciseTimeStamp ?? 0
					const rawDist = content.exerciseDistance ?? 0
					const rawSpeed = content.speed ?? 0
					const rawPace = content.pace ?? 0
					const rawCal = content.calories ?? 0

					// 时间：秒 → HH:MM:SS
					const hours = Math.floor(rawTime / 3600)
					const minutes = Math.floor((rawTime % 3600) / 60)
					const seconds = Math.floor(rawTime % 60)
					const timeStr = [hours, minutes, seconds]
						.map(v => String(v).padStart(2, '0'))
						.join(':')

					// 距离：米 → 公里
					const distKm = (rawDist / 1000).toFixed(2)

					// 速度：除以 1000，保留 1 位小数
					let speedKmh = "-.-"
					if (rawSpeed !== 0) {
						speedKmh = (rawSpeed / 1000).toFixed(1)
					}

					// 配速：限制最大 59999
					let paceStr = "--'--\""
					if (rawPace !== 0) {
						let limitedPace = rawPace
						if (limitedPace > 59999) {
							limitedPace = 59999
						}
						const paceMin = Math.floor(limitedPace / 60)
						const paceSecRem = limitedPace % 60
						paceStr = `${paceMin}'${String(paceSecRem).padStart(2, '0')}"`
					}

					// 卡路里：原始为卡 → 千卡（保留 1 位小数）
					const kcal = (rawCal / 1000).toFixed(1)

					this.sportData = {
						sportModel: content.sportModel ?? 0,
						opCode: content.opCode ?? 0,
						runState: content.runState ?? 'Unknown',
						deviceState: content.deviceState ?? 'Unknown',
						exerciseTimeStamp: timeStr,
						exerciseDistance: distKm,
						heartRate: content.heartRate ?? 0,
						calories: kcal,
						pace: paceStr,
						speed: speedKmh,
						gnssInfo: content.gnssInfo ?? null,
					}

					this.sportData.opCodeText = this.getOpCodeText(this.sportData.opCode)
				})
			},

			getOpCodeText(opCode) {
				const map = {
					1: '开启运动',
					2: '暂停运动',
					3: '继续运动',
					4: '停止运动',
					5: '运动数据上报',
				}
				return map[opCode] || '未知操作'
			},
		},
	}
</script>

<style scoped>
	.box {
		padding: 50rpx;
	}

	.btn_box {
		padding: 20px;
	}

	.opCodeText {
		display: flex;
		flex-direction: column;
	}

	.box-mode-state {
		display: flex;
		justify-content: space-between;
		margin-bottom: 30rpx;
		gap: 10px;
	}

	.mode-state-item {
		width: 45%;
		height: 70rpx;
		background-color: #494949;
		color: white;
		line-height: 70rpx;
		text-align: center;
		cursor: pointer;
	}

	.box-input {
		padding: 20rpx 50rpx;
	}

	.input {
		display: flex;
		height: 40px;
		line-height: 40px;
		align-items: center;
	}

	input {
		background-color: #e7e7e7;
		border: none;
		padding: 0 10px;
		flex: 1;
	}

	.data-box {
		margin-top: 20px;
		padding: 10px;
		border: 1px solid #ccc;
		border-radius: 8px;
	}

	.data-title {
		font-weight: bold;
		margin-bottom: 8px;
	}

	button {
		margin-top: 20px;
	}
</style>