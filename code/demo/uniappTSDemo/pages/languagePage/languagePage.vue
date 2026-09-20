<template>
	<view class="page">
		<view class="picker-wrap" @click="showPicker = true">
			<text class="label">语言：</text>
			<text class="value">{{ languageList[selectedIndex] || '请选择' }}</text>
			<text class="arrow">▾</text>
		</view>

		<button class="btn" @click="bindLanguage">设置</button>

		<!-- 遮罩 + 滚轮选择器 -->
		<view class="mask" v-if="showPicker" @click="showPicker = false"></view>
		<view class="picker-panel" :class="{ 'picker-show': showPicker }">
			<view class="picker-header">
				<text class="cancel-btn" @click="showPicker = false">取消</text>
				<text class="title">选择语言</text>
				<text class="confirm-btn" @click="confirmPicker">确定</text>
			</view>
			<picker-view class="picker-view" :value="[tempIndex]" @change="onPickerChange">
				<picker-view-column>
					<view class="picker-item" v-for="(item, idx) in languageList" :key="idx">{{ item }}</view>
				</picker-view-column>
			</picker-view>
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
				selectedIndex: 0,
				tempIndex: 0,
				showPicker: false,
				languageList: [
					'中文',
					'English',
					'日本語',
					'한국어',
					'Français',
					'Deutsch',
					'Español',
					'Italiano',
					'Português',
					'Русский',
					'العربية'
				]
			}
		},

		onShow() {
			this.notifyMonitorValueChange();
		},

		methods: {
			notifyMonitorValueChange() {
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("  监听蓝牙回调=>", e);
				});
			},

			onPickerChange(e) {
				this.tempIndex = e.detail.value[0];
			},

			confirmPicker() {
				this.selectedIndex = this.tempIndex;
				this.showPicker = false;
			},

			bindLanguage() {
				// language 值从 1 开始，数组索引从 0 开始，所以 +1
				let val = {
					language: this.selectedIndex + 1
				}
				console.log('val==>', val)
				veepooFeature.veepooSendLanguageSetupManager(val)
			}
		}
	}
</script>

<style>
	.page {
		padding: 30rpx;
	}

	.picker-wrap {
		display: flex;
		align-items: center;
		background-color: #f5f5f5;
		border-radius: 12rpx;
		padding: 24rpx 20rpx;
		font-size: 30rpx;
	}

	.picker-wrap .label {
		color: #333;
	}

	.picker-wrap .value {
		flex: 1;
		color: #00b0fb;
		margin-left: 10rpx;
	}

	.picker-wrap .arrow {
		color: #999;
		font-size: 28rpx;
	}

	.btn {
		margin-top: 40rpx;
		background-color: #00b0fb;
		color: #fff;
		border-radius: 12rpx;
	}

	/* 遮罩 */
	.mask {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(0, 0, 0, 0.4);
		z-index: 99;
	}

	/* 滚轮面板 */
	.picker-panel {
		position: fixed;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: #fff;
		border-radius: 24rpx 24rpx 0 0;
		z-index: 100;
		transform: translateY(100%);
		transition: transform 0.3s ease;
	}

	.picker-show {
		transform: translateY(0);
	}

	.picker-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 24rpx 30rpx;
		border-bottom: 1rpx solid #eee;
	}

	.picker-header .cancel-btn {
		color: #999;
		font-size: 30rpx;
	}

	.picker-header .title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}

	.picker-header .confirm-btn {
		color: #00b0fb;
		font-size: 30rpx;
	}

	.picker-view {
		height: 420rpx;
	}

	.picker-item {
		display: flex;
		align-items: center;
		justify-content: center;
		height: 80rpx;
		font-size: 32rpx;
		color: #333;
	}
</style>
