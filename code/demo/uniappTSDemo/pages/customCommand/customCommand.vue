<template>
	<view class="page">
		<view class="tip">
			<text>每行一帧，支持空格 / 逗号 / 0x 分隔。</text>
			<text>SDK 只校验帧头（0x04 开头、≥3 字节、长度限制），内容原样下发。</text>
		</view>

		<view class="card">
			<view class="section-title">当前链路单包大小</view>
			<view class="packet-info" v-if="packetInfo">{{ packetInfo }}</view>
			<view class="empty" v-else>尚未查询，点击下方按钮获取</view>
			<button class="btn btn-block" @click="onQueryPacketSize">查询单包大小</button>
		</view>

		<view class="card">
			<view class="section-title">指令内容</view>
			<textarea class="input" v-model="input" auto-height :maxlength="-1" placeholder="例如：04 01 00 02 01 01" />
			<view class="btn-group">
				<button class="btn" @click="onTemplate">读取示例</button>
				<button class="btn" @click="onPreview">解析预览</button>
				<button class="btn" @click="onClear">清空</button>
			</view>
			<button class="btn-primary" @click="onSend">下发</button>
		</view>

		<block v-if="frames.length > 0">
			<view class="section-title">解析结果（{{ frames.length }} 帧）</view>
			<view class="card">
				<view class="frame-item" v-for="item in frames" :key="item.index">
					<view class="frame-head">
						<text class="frame-index">#{{ item.index }}</text>
						<text class="frame-len">{{ item.len }} 字节</text>
						<text
							:class="['frame-flag', item.isCustom ? 'ok' : 'bad']">{{ item.isCustom ? '0x04' : '非 0x04' }}</text>
					</view>
					<text class="frame-text">{{ item.text }}</text>
				</view>
			</view>
		</block>

		<view class="section-title">最近接收</view>
		<view class="card">
			<text class="recv-text" v-if="lastRecv">{{ lastRecv }}</text>
			<view class="empty" v-else>暂无数据</view>
		</view>

		<view class="section-title">日志</view>
		<view class="log-box">
			<view class="log-item" v-for="item in logs" :key="item.id">
				<view class="log-head">
					<text class="log-time">{{ item.time }}</text>
					<text :class="['log-tag', item.kind]">{{ item.kindText }}</text>
				</view>
				<text class="log-text">{{ item.text }}</text>
			</view>
			<view class="empty" v-if="logs.length === 0">暂无日志</view>
		</view>
	</view>
</template>

<script>
	import {
		veepooBle
	} from '../../common/index.js'

	const MAX_LOG = 100
	let logSeq = 0

	export default {
		data() {
			return {
				input: '04 01 00 02 01 01',
				frames: [],
				lastRecv: '',
				packetInfo: '',
				logs: []
			}
		},

		onLoad() {
			this.registerNotify()
		},

		methods: {
			// 监听设备上报，只处理 0x04 自定义协议
			registerNotify() {
				const that = this
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					if (!Array.isArray(e) || e[0] !== '04') return
					const text = e.join(' ')
					that.lastRecv = text
					that.log('接收（' + e.length + ' 字节）：' + text, 'recv')
				})
			},

			// 查询当前链路单包大小（MTU - 3），App 端取不到时按 20 字节兜底
			onQueryPacketSize() {
				const that = this
				veepooBle.veepooUniAppSDKGetPacketSizeManager(function(res) {
					if (!res || !res.success) {
						that.packetInfo = ''
						that.log('查询单包大小失败：' + ((res && res.errMsg) || '未知错误'))
						return
					}
					const info = 'MTU ' + res.mtu + ' 字节，单包最大可写 ' + res.packetSize + ' 字节'
					that.packetInfo = info
					that.log('查询单包大小：' + info)
				})
			},

			// 解析输入：每行一帧，支持空格、逗号、0x 等分隔
			parseInput(text) {
				const lines = (text || '').split('\n')
				const frames = []
				for (let i = 0; i < lines.length; i++) {
					const raw = lines[i].replace(/0x/gi, '').replace(/[\s,;:_-]/g, '')
					if (!raw) continue
					if (!/^[0-9a-fA-F]+$/.test(raw) || raw.length % 2 !== 0) {
						return {
							error: '第 ' + (i + 1) + ' 行不是合法的十六进制数据（需为偶数个字符）'
						}
					}
					const frame = []
					for (let j = 0; j < raw.length; j += 2) {
						frame.push(raw.substr(j, 2).toUpperCase())
					}
					frames.push(frame)
				}
				if (frames.length === 0) {
					return {
						error: '请输入至少一帧数据'
					}
				}
				return {
					frames: frames
				}
			},

			// 预览解析结果
			onPreview() {
				const res = this.parseInput(this.input)
				if (res.error) {
					this.frames = []
					this.log(res.error)
					return
				}
				this.frames = res.frames.map(function(f, i) {
					return {
						index: i + 1,
						len: f.length,
						isCustom: f[0] === '04',
						text: f.join(' ')
					}
				})
				this.log('已解析 ' + this.frames.length + ' 帧')
			},

			// 下发：SDK 内部会做连接、帧头、长度校验，并按 30ms 间隔逐帧写入
			onSend() {
				const res = this.parseInput(this.input)
				if (res.error) {
					this.log(res.error)
					return
				}
				this.onPreview()
				const that = this
				veepooBle.veepooUniAppSDKSendCustomDataFramesManager(res.frames, function(r) {
					if (r && r.success) {
						that.log('下发成功：共 ' + r.total + ' 帧', 'send')
						return
					}
					// 结果结构：code 数值码 + codeName 名称 + codeDesc 中文描述 + errMsg 具体原因
					const desc = (r && r.codeDesc) || '未知错误'
					const name = (r && r.codeName) || 'Unknown'
					const code = (r && typeof r.code === 'number') ? r.code : -1
					let msg = '下发失败：' + desc + '（' + name + '，code=' + code + '）'
					if (r && r.errMsg) msg += '，' + r.errMsg
					if (r && typeof r.failIndex === 'number' && r.failIndex >= 0) {
						msg += '（失败帧 ' + (r.failIndex + 1) + '）'
					}
					that.log(msg, 'send')
				})
			},

			// 填入礼拜提醒读取指令示例
			onTemplate() {
				this.input = '04 01 00 02 01 01'
				this.onPreview()
			},

			onClear() {
				this.frames = []
				this.lastRecv = ''
				this.logs = []
			},

			log(text, kind) {
				const time = new Date().toTimeString().slice(0, 8)
				const kindText = kind === 'send' ? '下发' : (kind === 'recv' ? '接收' : '信息')
				const logs = this.logs || []
				logs.unshift({
					id: time + '_' + (logSeq++),
					time: time,
					kind: kind || 'info',
					kindText: kindText,
					text: text
				})
				if (logs.length > MAX_LOG) logs.length = MAX_LOG
				this.logs = logs
			}
		}
	}
</script>

<style scoped>
	.page {
		padding: 24rpx;
		background-color: #f5f6f7;
		min-height: 100vh;
		box-sizing: border-box;
	}

	.tip {
		display: flex;
		flex-direction: column;
		font-size: 24rpx;
		color: #666666;
		line-height: 1.7;
		margin-bottom: 20rpx;
	}

	.card {
		background-color: #ffffff;
		border-radius: 16rpx;
		padding: 24rpx;
		margin-bottom: 20rpx;
	}

	.section-title {
		margin: 6rpx 0 16rpx;
		font-size: 28rpx;
		font-weight: bold;
		color: #333333;
	}

	.packet-info {
		font-size: 26rpx;
		color: #07c160;
		margin-bottom: 16rpx;
	}

	.empty {
		padding: 24rpx 0;
		font-size: 24rpx;
		color: #bbbbbb;
	}

	.input {
		width: 100%;
		min-height: 160rpx;
		padding: 16rpx;
		box-sizing: border-box;
		background-color: #f7f8fa;
		border-radius: 12rpx;
		font-size: 26rpx;
		font-family: monospace;
		color: #333333;
	}

	.btn-group {
		display: flex;
		margin: 20rpx 0;
	}

	.btn {
		flex: 1;
		margin-right: 16rpx;
		font-size: 26rpx;
	}

	.btn:last-child {
		margin-right: 0;
	}

	.btn-block {
		width: 100%;
		font-size: 26rpx;
	}

	.btn-primary {
		width: 100%;
		font-size: 28rpx;
		background-color: #07c160;
		color: #ffffff;
	}

	.frame-item {
		padding: 16rpx 0;
		border-bottom: 1rpx solid #f3f3f3;
	}

	.frame-head {
		display: flex;
		align-items: center;
		margin-bottom: 8rpx;
	}

	.frame-index {
		font-size: 24rpx;
		color: #333333;
		font-weight: bold;
	}

	.frame-len {
		margin-left: 16rpx;
		font-size: 22rpx;
		color: #999999;
	}

	.frame-flag {
		margin-left: auto;
		padding: 2rpx 10rpx;
		border-radius: 6rpx;
		font-size: 20rpx;
	}

	.frame-flag.ok {
		background-color: #e8f8ee;
		color: #07c160;
	}

	.frame-flag.bad {
		background-color: #fdecec;
		color: #fa5151;
	}

	.frame-text {
		font-size: 22rpx;
		font-family: monospace;
		color: #333333;
		line-height: 1.5;
		word-break: break-all;
	}

	.recv-text {
		font-size: 24rpx;
		font-family: monospace;
		color: #333333;
		line-height: 1.5;
		word-break: break-all;
	}

	.log-box {
		background-color: #ffffff;
		border-radius: 16rpx;
		padding: 10rpx 20rpx;
		max-height: 620rpx;
		overflow-y: auto;
	}

	.log-item {
		padding: 14rpx 0;
		border-bottom: 1rpx solid #f3f3f3;
	}

	.log-head {
		display: flex;
		align-items: center;
	}

	.log-time {
		font-size: 22rpx;
		color: #999999;
	}

	.log-tag {
		margin-left: 12rpx;
		padding: 2rpx 10rpx;
		border-radius: 6rpx;
		font-size: 20rpx;
	}

	.log-tag.send {
		background-color: #e8f8ee;
		color: #07c160;
	}

	.log-tag.recv {
		background-color: #e8f1ff;
		color: #576b95;
	}

	.log-tag.info {
		background-color: #f2f3f5;
		color: #666666;
	}

	.log-text {
		display: block;
		margin-top: 6rpx;
		font-size: 22rpx;
		color: #333333;
		line-height: 1.5;
		word-break: break-all;
	}
</style>