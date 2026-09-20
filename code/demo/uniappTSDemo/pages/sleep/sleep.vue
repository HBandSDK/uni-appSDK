<template>
	<view class="box">
		<view class="box-day">
			<view class="day-item" @click="ReadPreciseSleepManager(0)">今天</view>
			<view class="day-item" @click="ReadPreciseSleepManager(1)">昨天</view>
			<view class="day-item" @click="ReadPreciseSleepManager(2)">前天</view>
		</view>
		<view class="steep-date">
			<view class="steep-item-top">
				<view class="item-left">读取进度: {{ device.Progress || 0 }}%</view>
				<view class="item-left">{{ device.readDay || '--' }}</view>
			</view>
			<text>只显示第一段数据，剩余数据查看日志</text>
			<view class="steep-item">
				<view class="item-left">入睡时间</view>
				<view class="item-left">{{ device.content[0]?.fallAsleepTime || '--' }}</view>
			</view>
			<view class="steep-item">
				<view class="item-left">起床时间</view>
				<view class="item-left">{{ device.content[0]?.exitSleepTime || '--' }}</view>
			</view>
			<view class="steep-item">
				<view class="item-left">深睡时间</view>
				<view class="item-left">{{ device.content[0]?.deepSleepTime || 0 }}分钟</view>
			</view>
			<view class="steep-item">
				<view class="item-left">浅睡时间</view>
				<view class="item-left">{{ device.content[0]?.lightSleepTime || 0 }}分钟</view>
			</view>
			<view class="steep-item">
				<view class="item-left">苏醒时间</view>
				<view class="item-left">{{ device.content[0]?.nightTotalTime || 0 }}分钟</view>
			</view>
			<view class="steep-item">
				<view class="item-left">快速眼动</view>
				<view class="item-left">{{ device.content[0]?.otherSleepTime || 0 }}分钟</view>
			</view>
			<!-- 睡眠曲线 -->
			<view class="steep-line">
				<view>睡眠曲线：</view>
				<canvas canvas-id="sleepCurveCanvas" id="sleepCurveCanvas"
					style="width: 100%; height: 180rpx; background: #f7f7f7; border-radius: 16rpx;"
					@touchstart="onCanvasTouchStart" @touchmove="onCanvasTouchMove"
					@touchend="onCanvasTouchEnd"></canvas>
				<!-- 提示文本：选中段信息 -->
				<view v-if="showRuler" style="margin:10rpx 0; font-size:26rpx; color:#333;">
					选中片段：{{rulerTipText}}
				</view>
				<!-- 图例 -->
				<view class="legend">
					<text class="legend-item"><span class="dot" style="background:#FFB74D;"></span>苏醒</text>
					<text class="legend-item"><span class="dot" style="background:#42A5F5;"></span>快速眼动</text>
					<text class="legend-item"><span class="dot" style="background:#7C68ED;"></span>浅睡</text>
					<text class="legend-item"><span class="dot" style="background:#512DA8;"></span>深睡</text>
				</view>
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
				device: {
					readDay: '--',
					Progress: 0,
					content: [{
						fallAsleepTime: '--',
						exitSleepTime: '--',
						deepSleepTime: 0,
						lightSleepTime: 0,
						nightTotalTime: 0,
						sleepQuality: 0,
						sleepCurve: []
					}]
				},
				// 标尺相关变量
				showRuler: false,
				rulerX: 0,
				rulerTipText: "",
				lastClickTime: 0,
				clickTimeout: 300,
				canvasWidth: 375,
				canvasHeight: 90
			}
		},
		onShow() {
			this.notifyMonitorValueChange();
		},
		methods: {
			ReadPreciseSleepManager(e) {
				console.log('day:', e)
				veepooFeature.veepooSendReadPreciseSleepManager({
					day: e
				})
			},
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("睡眠监听回调=>", e);
					if (e && e.name == '精准睡眠数据') {
						self.device = e;
						self.$nextTick(() => {
							self.drawSleepCurve();
						});
					}
				})
			},
			// 绘制多层堆叠睡眠轨道
			drawSleepCurve() {
				const curve = this.device.content[0]?.sleepCurve;
				if (!curve || curve.length === 0) {
					// console.warn('无睡眠曲线数据');
					return;
				}
				const ctx = uni.createCanvasContext('sleepCurveCanvas', this);
				const query = uni.createSelectorQuery().in(this);
				query.select('#sleepCurveCanvas').boundingClientRect(rect => {
					if (!rect) {
						this.canvasWidth = 375;
						this.canvasHeight = 90;
						this._drawWithSize(ctx, curve, this.canvasWidth, this.canvasHeight);
						return;
					}
					this.canvasWidth = rect.width;
					this.canvasHeight = rect.height;
					this._drawWithSize(ctx, curve, this.canvasWidth, this.canvasHeight);
				}).exec();
			},
			_drawWithSize(ctx, curve, width, height) {
				// 层级定义：【顶层→底层】
				const layers = [{
						type: 4,
						color: '#FFB74D'
					},
					{
						type: 2,
						color: '#42A5F5'
					},
					{
						type: 1,
						color: '#7C68ED'
					},
					{
						type: 0,
						color: '#512DA8'
					},
				];
				const layerCount = layers.length;
				const layerHeight = height / layerCount; // 每一层轨道高度
				const totalPointCount = curve.length;
				// 循环每一层轨道
				layers.forEach((layer, layerIndex) => {
					// 当前层的y起始坐标
					const yStart = layerIndex * layerHeight;
					const targetType = layer.type;
					// 合并当前层中连续同状态分段（只保留等于targetType的片段）
					const segments = [];
					if (curve.length === 0) return;
					let isInSegment = false;
					let segStart = 0;
					for (let i = 0; i < curve.length; i++) {
						const val = curve[i];
						if (val === targetType) {
							if (!isInSegment) {
								isInSegment = true;
								segStart = i;
							}
						} else {
							if (isInSegment) {
								isInSegment = false;
								segments.push({
									start: segStart,
									end: i - 1
								})
							}
						}
					}
					// 补上最后一段
					if (isInSegment) {
						segments.push({
							start: segStart,
							end: curve.length - 1
						})
					}
					// 绘制当前层所有色块
					segments.forEach(seg => {
						const segWidth = (seg.end - seg.start + 1) / totalPointCount * width;
						const x = seg.start / totalPointCount * width;
						ctx.setFillStyle(layer.color);
						ctx.fillRect(x, yStart + 2, segWidth - 0.5, layerHeight - 4);
					})
				})

				// ========= 如果开启标尺，绘制竖线 =========
				if (this.showRuler) {
					ctx.setStrokeStyle('#333333');
					ctx.setLineWidth(1);
					ctx.beginPath();
					ctx.moveTo(this.rulerX, 0);
					ctx.lineTo(this.rulerX, height);
					ctx.stroke();
				}
				ctx.draw();
			},
			// Canvas点击：模拟双击
			onCanvasTouchStart(e) {
				const now = Date.now();
				const touchX = e.touches[0].x;
				// 双击判定
				if (now - this.lastClickTime < this.clickTimeout) {
					// 双击触发标尺
					this.showRuler = !this.showRuler;
					this.rulerX = touchX;
					this.updateRulerInfo(touchX);
				}
				this.lastClickTime = now;
			},
			// 滑动更新标尺位置
			onCanvasTouchMove(e) {
				if (!this.showRuler) return;
				const touchX = e.touches[0].x;
				this.rulerX = touchX;
				this.updateRulerInfo(touchX);
			},
			onCanvasTouchEnd() {
				// 抬起不关闭标尺，再次双击关闭
			},
			// 根据标尺X坐标，查找当前所在睡眠段，计算时长
			updateRulerInfo(xPos) {
				const curve = this.device.content[0]?.sleepCurve;
				if (!curve || curve.length === 0) {
					this.rulerTipText = "无数据";
					this.drawSleepCurve();
					return;
				}
				const totalPointCount = curve.length;
				// 根据x坐标换算到曲线数组索引
				let index = Math.round(xPos / this.canvasWidth * (totalPointCount - 1));
				index = Math.max(0, Math.min(totalPointCount - 1, index));
				const currentType = curve[index];

				// 找到当前连续片段的起始、结束索引
				let segStart = index;
				let segEnd = index;
				while (segStart > 0 && curve[segStart - 1] === currentType) {
					segStart--;
				}
				while (segEnd < totalPointCount - 1 && curve[segEnd + 1] === currentType) {
					segEnd++;
				}
				// 假设每个采样点代表1分钟，按需修改采样间隔
				const durationMin = segEnd - segStart + 1;
				const typeMap = {
					0: "深睡",
					1: "浅睡",
					2: "快速眼动",
					4: "苏醒"
				}
				const typeName = typeMap[currentType] || "未知";
				this.rulerTipText = `${typeName}，持续${durationMin}分钟`;
				// 重绘，刷新竖线
				this.drawSleepCurve();
			}
		}
	}
</script>
<style>
	.box {
		padding: 50rpx;
	}

	.box-day {
		display: flex;
		justify-content: space-around;
	}

	.day-item {
		width: 25%;
		background-color: #a8a8a8;
		height: 30px;
		line-height: 30px;
		text-align: center;
		color: white;
		border-radius: 15rpx;
	}

	.steep-date {
		margin-top: 20px;
	}

	.steep-item-top {
		display: flex;
		justify-content: space-between;
		height: 40px;
		line-height: 40px;
	}

	.steep-item {
		display: flex;
		justify-content: space-between;
		height: 40px;
		line-height: 40px;
		border-bottom: 1px solid #dadada;
	}

	.steep-line {
		margin-top: 20rpx;
		line-height: 40px;
	}

	.steep-line canvas {
		margin-top: 10rpx;
	}

	/* 图例样式 */
	.legend {
		display: flex;
		flex-wrap: wrap;
		margin-top: 10rpx;
		font-size: 24rpx;
		color: #333;
	}

	.legend-item {
		margin-right: 20rpx;
		display: inline-flex;
		align-items: center;
	}

	.legend-item .dot {
		display: inline-block;
		width: 30rpx;
		height: 30rpx;
		border-radius: 6rpx;
		margin-right: 8rpx;
	}
</style>