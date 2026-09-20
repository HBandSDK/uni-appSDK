<template>
	<view class="box">

		<!-- 标题 -->
		<view class="header">
			<text class="header-title">微体检测量</text>
		</view>

		<!-- 进度 -->
		<view class="progress-wrap">
			<view class="progress-text">测量进度：{{progress}}%</view>
			<view class="progress-bar">
				<view class="progress-bar-fill" :style="{ width: progress + '%' }"></view>
			</view>
			<view class="progress-sub" v-if="measuringTip && isMeasuring">{{measuringTip}}</view>
			<view class="progress-sub" v-else-if="liveHeartRate && isMeasuring">实时心率：{{liveHeartRate}} bpm</view>
		</view>

		<!-- 错误提示 -->
		<view class="error-text" v-if="errorMsg && !isMeasuring">{{errorMsg}}</view>

		<!-- 核心指标 -->
		<view class="card">
			<view class="card-title">核心指标</view>
			<view class="data-grid">
				<view class="data-item">
					<text class="data-label">心率(bpm)</text>
					<text class="data-value">{{report.heartRate !== undefined ? report.heartRate : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">血氧(%)</text>
					<text class="data-value">{{report.bloodOxygen !== undefined ? report.bloodOxygen : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">压力</text>
					<text class="data-value">{{report.pressure !== undefined ? report.pressure : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">HRV(ms)</text>
					<text class="data-value">{{report.hrv !== undefined ? report.hrv : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">血压(mmHg)</text>
					<text class="data-value">{{report.bloodPressure !== undefined ? report.bloodPressure : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">体温(℃)</text>
					<text class="data-value">{{report.bodyTemperature !== undefined ? report.bodyTemperature : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">血糖</text>
					<text class="data-value">{{report.bloodSugar !== undefined ? report.bloodSugar : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">情绪</text>
					<text class="data-value">{{report.emotion !== undefined ? report.emotion : '--'}}</text>
				</view>
				<view class="data-item">
					<text class="data-label">疲劳度</text>
					<text class="data-value">{{report.fatigueLevel !== undefined ? report.fatigueLevel : '--'}}</text>
				</view>
			</view>
		</view>

		<!-- 个人基本信息 -->
		<view class="card" v-if="report.basicInfoList.length">
			<view class="card-title">个人基本信息</view>
			<view class="data-grid">
				<view class="data-item" v-for="(item, i) in report.basicInfoList" :key="'b' + i">
					<text class="data-label">{{item.label}}</text>
					<text class="data-value">{{item.value}}</text>
				</view>
			</view>
		</view>

		<!-- 皮肤电检测 -->
		<view class="card" v-if="report.skinList.length">
			<view class="card-title">皮肤电检测</view>
			<view class="data-grid">
				<view class="data-item" v-for="(item, i) in report.skinList" :key="'s' + i">
					<text class="data-label">{{item.label}}</text>
					<text class="data-value">{{item.value}}</text>
				</view>
			</view>
		</view>

		<!-- 血液成分 -->
		<view class="card" v-if="report.bloodComponentList.length">
			<view class="card-title">血液成分</view>
			<view class="data-grid">
				<view class="data-item" v-for="(item, i) in report.bloodComponentList" :key="'bc' + i">
					<text class="data-label">{{item.label}}</text>
					<text class="data-value">{{item.value}}</text>
				</view>
			</view>
		</view>

		<!-- 身体成分 -->
		<view class="card" v-if="report.bodyCompositionList.length">
			<view class="card-title">身体成分</view>
			<view class="data-grid">
				<view class="data-item" v-for="(item, i) in report.bodyCompositionList" :key="'bo' + i">
					<text class="data-label">{{item.label}}</text>
					<text class="data-value">{{item.value}}</text>
				</view>
			</view>
		</view>

		<!-- 按钮 -->
		<view class="btn-group">
			<button class="btn btn-start" @click="microCheckStart" :disabled="isMeasuring">开始测量</button>
			<button class="btn btn-stop" @click="microCheckStop" :disabled="!isMeasuring">结束测量</button>
		</view>

		<!-- 说明 -->
		<view class="card">
			<view class="card-title">说明</view>
			<view class="info-item">· 微体检一次性检测心率、血氧、压力、HRV、血压、体温、血糖、情绪、疲劳度等指标</view>
			<view class="info-item">· 测量时请保持静止，确保设备贴合手腕</view>
			<view class="info-item">· 未返回的字段显示 --，测量结果仅供参考，如有异常请咨询医生</view>
		</view>

	</view>
</template>

<script>
	import {
		veepooBle,
		veepooFeature
	} from '../../common/index.js'

	// 非成功状态提示文案（2失败 3设备忙 4低电 6佩戴未通过 7导联脱落）
	const STATE_TIPS = {
		2: '测量失败，无结果数据',
		3: '设备正忙，正在测其它数据，请稍后再试',
		4: '设备电量低，请充电后再试',
		6: '佩戴未通过，请调整佩戴姿势',
		7: 'ECG导联脱落，请重新佩戴'
	}

	// 身体成分字段展示配置（设备返回哪些就展示哪些）
	const BODY_COMPOSITION_FIELDS = [
		{ key: 'bmi', label: 'BMI' },
		{ key: 'bodyFatRate', label: '体脂率(%)' },
		{ key: 'fatMass', label: '脂肪量(kg)' },
		{ key: 'leanBodyMass', label: '去脂体重(kg)' },
		{ key: 'muscleRate', label: '肌肉率(%)' },
		{ key: 'muscleMass', label: '肌肉量(kg)' },
		{ key: 'subcutaneousFat', label: '皮下脂肪(%)' },
		{ key: 'bodyWater', label: '体内水分(%)' },
		{ key: 'waterContent', label: '含水量(%)' },
		{ key: 'skeletalMuscleRate', label: '骨骼肌率(%)' },
		{ key: 'boneMass', label: '骨量(kg)' },
		{ key: 'proteinRate', label: '蛋白质占比(%)' },
		{ key: 'proteinMass', label: '蛋白质量(kg)' },
		{ key: 'basalMetabolicRate', label: '基础代谢率(kcal)' }
	]

	// 空报告：字段为空时页面显示 --
	const createEmptyReport = () => ({
		heartRate: undefined,
		bloodOxygen: undefined,
		pressure: undefined,
		hrv: undefined,
		bloodPressure: undefined,
		bodyTemperature: undefined,
		bloodSugar: undefined,
		emotion: undefined,
		fatigueLevel: undefined,
		basicInfoList: [],
		skinList: [],
		bloodComponentList: [],
		bodyCompositionList: []
	})

	// dataType=5 报告可能分包上报（current/total），缓存已收到的 content，收齐后合并展示
	let pendingReportContent = null

	export default {
		data() {
			return {
				isMeasuring: false,
				progress: 0,
				liveHeartRate: 0,   // 测量中每秒心率（type 51）
				hasResult: false,   // 是否已出报告
				errorMsg: '',       // 失败/忙/低电提示
				measuringTip: '',   // 测量中状态提示（佩戴未通过/导联脱落）
				report: createEmptyReport()
			}
		},
		onShow() {
			this.notifyMonitorValueChange()
		},
		onUnload() {
			// 页面卸载时停止测量
			if (this.isMeasuring) {
				this.microCheckStop()
			}
		},
		methods: {
			// 监听订阅 notifyMonitorValueChange
			notifyMonitorValueChange() {
				let self = this

				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					if (!e || e.type === undefined) {
						return
					}
					// type 53 微体检测量（进度/报告/各种状态）
					if (e.type == 53) {
						console.log('[微体检] 回调:', e)
						self.handleMicroCheckCallback(e)
					}
					// type 51 每秒心率，测量中实时显示
					else if (e.type == 51 && self.isMeasuring && e.content) {
						self.liveHeartRate = e.content.heartRate || 0
					}
					// type 54 ppg 原始数据，本页不绘制波形，忽略
				})

				// ppg 的通道与常规数据通道不一样（本页不绘制波形，仅保留监听避免误用）
				veepooBle.veepooUniAppSDKNotifyECGValueChange(function(e) {
					// type 36 ppg 原始数据，本页忽略
					if (!e || e.type === undefined) {
						return
					}
				})
			},

			// 处理微体检测量回调（type 53）
			// dataType: 0进度 1成功报告(平铺) 2失败 3设备忙 4低电 5成功报告 6佩戴未通过 7导联脱落
			handleMicroCheckCallback(e) {
				const dataType = e.dataType

				// 0 进度 / 6 佩戴未通过 / 7 导联脱落：测量仍在进行
				if (dataType === 0 || dataType === 6 || dataType === 7) {
					this.isMeasuring = true
					this.progress = e.progress || 0
					this.measuringTip = STATE_TIPS[dataType] || ''
					this.errorMsg = ''
					return
				}

				// 2 失败 / 3 设备忙 / 4 低电：结束测量
				if (dataType === 2 || dataType === 3 || dataType === 4) {
					pendingReportContent = null
					this.isMeasuring = false
					this.hasResult = false
					this.progress = 0
					this.measuringTip = ''
					this.errorMsg = STATE_TIPS[dataType] || ''
					return
				}

				// 1 成功报告（平铺）
				if (dataType === 1) {
					this.showReport(e.content || {})
					return
				}

				// 5 成功报告（可能分包）
				if (dataType === 5) {
					pendingReportContent = Object.assign({}, pendingReportContent, e.content || {})
					const total = Number(e.total) || 1
					const current = Number(e.current) || 1
					if (current >= total) {
						const content = pendingReportContent
						pendingReportContent = null
						this.showReport(content)
					}
				}
			},

			// 测量成功，归一化并展示报告
			showReport(content) {
				this.isMeasuring = false
				this.hasResult = true
				this.progress = 100
				this.errorMsg = ''
				this.measuringTip = ''
				this.liveHeartRate = 0
				this.report = this.buildReport(content)
			},

			// 归一化：兼容 uniapp 平铺字段与原生版的嵌套报告字段
			buildReport(content) {
				const c = content || {}

				// 血压：优先嵌套(optical/pump/置 05)，其次平铺 highPressure/lowPressure
				const optical = c.opticalBloodPressure
				const pump = c.pumpBloodPressure
				const bp = optical || pump
				let bloodPressure
				if (bp) {
					bloodPressure = bp.highPressure + '/' + bp.lowPressure
				} else if (c.highPressure !== undefined || c.lowPressure !== undefined) {
					bloodPressure = c.highPressure + '/' + c.lowPressure
				} else if (c.bloodPressure !== undefined) {
					bloodPressure = typeof c.bloodPressure === 'object' ?
						(c.bloodPressure.highPressure + '/' + c.bloodPressure.lowPressure) :
						c.bloodPressure
				}

				// 体温：05 为 {rawTemperature, bodyTemperature}，01 为数值
				let bodyTemperature
				if (c.bodyTemperature !== undefined) {
					bodyTemperature = typeof c.bodyTemperature === 'object' ?
						c.bodyTemperature.bodyTemperature : c.bodyTemperature
				}

				// 血糖：05 为 {displayType, value}，01 为数值(mmol/L)
				let bloodSugar
				if (c.bloodSugar !== undefined) {
					if (typeof c.bloodSugar === 'object') {
						bloodSugar = c.bloodSugar.displayType === 'level' ?
							('等级 ' + c.bloodSugar.value) : c.bloodSugar.value
					} else {
						bloodSugar = c.bloodSugar
					}
				}

				// 情绪/疲劳度：描述 + 数值
				const emotion = c.emotion !== undefined ?
					(this.getEmotionText(c.emotion) + '(' + c.emotion + ')') : undefined
				const fatigueLevel = c.fatigueLevel !== undefined ?
					(this.getFatigueText(c.fatigueLevel) + '(' + c.fatigueLevel + ')') : undefined

				// 个人基本信息（仅 05 报告有）
				const basicInfoList = []
				if (c.basicInfo) {
					basicInfoList.push({ label: '性别', value: c.basicInfo.gender === 'male' ? '男' : '女' })
					basicInfoList.push({ label: '年龄(岁)', value: c.basicInfo.age })
					basicInfoList.push({ label: '身高(cm)', value: c.basicInfo.height })
					basicInfoList.push({ label: '体重(kg)', value: c.basicInfo.weight })
				}

				// 皮电（仅 05 报告有）
				const skinList = []
				if (c.skinElectrical) {
					const s = c.skinElectrical
					const riskText = ['低', '中', '高']
					skinList.push({ label: '情绪', value: this.getEmotionText(s.emotion) + '(' + s.emotion + ')' })
					skinList.push({ label: '皮肤含水量(%)', value: s.skinMoisture })
					skinList.push({
						label: '抑郁症风险',
						value: riskText[s.depressionRisk] !== undefined ? riskText[s.depressionRisk] : s.depressionRisk
					})
					skinList.push({ label: '交感神经活跃度', value: s.snsActivation })
					skinList.push({ label: '皮质醇(ug/L)', value: s.cortisol })
				}

				// 血液成分（仅 05 报告有）
				const bloodComponentList = []
				if (c.bloodComponent) {
					const b = c.bloodComponent
					bloodComponentList.push({ label: '尿酸(μmol/L)', value: b.uricAcid })
					bloodComponentList.push({ label: '总胆固醇(mmol/L)', value: b.cholesterol })
					bloodComponentList.push({ label: '甘油三酯(mmol/L)', value: b.triglyceride })
					bloodComponentList.push({ label: '高密度脂蛋白(mmol/L)', value: b.highDensityLipoprotein })
					bloodComponentList.push({ label: '低密度脂蛋白(mmol/L)', value: b.lowDensityLipoprotein })
				}

				// 身体成分（仅 05 报告有，字段按设备支持情况返回）
				const bodyCompositionList = []
				if (c.bodyComposition) {
					for (let i = 0; i < BODY_COMPOSITION_FIELDS.length; i++) {
						const f = BODY_COMPOSITION_FIELDS[i]
						const v = c.bodyComposition[f.key]
						if (v !== undefined) {
							bodyCompositionList.push({ label: f.label, value: v })
						}
					}
				}

				return {
					heartRate: c.heartRate,
					bloodOxygen: c.bloodOxygen,
					pressure: c.pressure,
					hrv: c.hrv,
					bloodPressure: bloodPressure,
					bodyTemperature: bodyTemperature,
					bloodSugar: bloodSugar,
					emotion: emotion,
					fatigueLevel: fatigueLevel,
					basicInfoList: basicInfoList,
					skinList: skinList,
					bloodComponentList: bloodComponentList,
					bodyCompositionList: bodyCompositionList
				}
			},

			// 获取情绪描述
			getEmotionText(emotion) {
				if (emotion >= -10 && emotion <= -5) {
					return '情绪低落'
				} else if (emotion > -5 && emotion <= -2) {
					return '有些低落'
				} else if (emotion > -2 && emotion <= 2) {
					return '情绪平稳'
				} else if (emotion > 2 && emotion <= 5) {
					return '情绪较好'
				} else if (emotion > 5 && emotion <= 10) {
					return '情绪很好'
				}
				return '未知'
			},

			// 获取疲劳度描述
			getFatigueText(fatigue) {
				if (fatigue >= 0 && fatigue <= 2) {
					return '精力充沛'
				} else if (fatigue > 2 && fatigue <= 4) {
					return '轻度疲劳'
				} else if (fatigue > 4 && fatigue <= 6) {
					return '中度疲劳'
				} else if (fatigue > 6 && fatigue <= 8) {
					return '重度疲劳'
				} else if (fatigue > 8) {
					return '极度疲劳'
				}
				return '未知'
			},

			// 开始微体检测量
			microCheckStart() {
				pendingReportContent = null

				// 重置数据
				this.isMeasuring = true
				this.hasResult = false
				this.progress = 0
				this.errorMsg = ''
				this.measuringTip = ''
				this.liveHeartRate = 0
				this.report = createEmptyReport()

				// 发送开始微体检测量指令
				veepooFeature.veepooSendMicroCheckDataManager({ switch: 'start' })
			},

			// 停止微体检测量
			microCheckStop() {
				this.isMeasuring = false

				// 发送停止微体检测量指令
				veepooFeature.veepooSendMicroCheckDataManager({ switch: 'stop' })
			}
		}
	}
</script>

<style scoped>
	.box {
		padding: 30rpx;
		box-sizing: border-box;
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

	/* 进度 */
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

	.progress-sub {
		font-size: 24rpx;
		color: #00b0fb;
		margin-top: 10rpx;
	}

	/* 错误提示 */
	.error-text {
		font-size: 28rpx;
		color: #ff6b6b;
		text-align: center;
		margin-bottom: 20rpx;
	}

	/* 数据卡片 */
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
		box-sizing: border-box;
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

	/* 按钮 */
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

	.btn-start[disabled] {
		background-color: #9fdcf5;
	}

	.btn-stop {
		background-color: #ff6b6b;
	}

	.btn-stop[disabled] {
		background-color: #ffc1c1;
	}

	/* 说明 */
	.info-item {
		font-size: 26rpx;
		color: #666;
		line-height: 44rpx;
	}
</style>