<template>
	<view class="box">
		<view class="header">
			<text class="header-title">个人信息同步</text>
		</view>

		<view class="card">
			<view class="card-title">基本信息</view>
			<form @submit="onSubmit">
				<view class="form-item">
					<text class="form-label">身高</text>
					<view class="input-wrap">
						<input class="form-input" type="number" name="height" placeholder="请输入身高" />
						<text class="unit">cm</text>
					</view>
				</view>
				<view class="form-item">
					<text class="form-label">体重</text>
					<view class="input-wrap">
						<input class="form-input" type="number" name="weight" placeholder="请输入体重" />
						<text class="unit">kg</text>
					</view>
				</view>
				<view class="form-item">
					<text class="form-label">年龄</text>
					<view class="input-wrap">
						<input class="form-input" type="number" name="age" placeholder="请输入年龄" />
						<text class="unit">岁</text>
					</view>
				</view>
				<view class="form-item">
					<text class="form-label">性别</text>
					<view class="input-wrap">
						<input class="form-input" type="number" name="sex" placeholder="0女 1男" />
					</view>
				</view>

				<view class="card-title card-title-sub">目标设定</view>
				<view class="form-item">
					<text class="form-label">目标步数</text>
					<view class="input-wrap">
						<input class="form-input" type="number" name="steps" placeholder="如4000" />
						<text class="unit">步</text>
					</view>
				</view>
				<view class="form-item">
					<text class="form-label">目标睡眠</text>
					<view class="input-wrap">
						<input class="form-input" type="number" name="sleep" placeholder="如60" />
						<text class="unit">分钟</text>
					</view>
				</view>

				<button class="btn btn-primary" form-type="submit">开始同步</button>
			</form>
		</view>
	</view>
</template>

<script>
	// pages/personalInfo/index.js
	import {
		veepooBle,
		veepooFeature
	} from '../../common/index.js'
	export default {
		data() {
			return {
				device: {}
			}
		},

		onLoad() {

		},

		onShow() {
			this.notifyMonitorValueChange();
			let value = 123;
			let val = ''
			if (typeof value != 'string') {
				val = String(value)
			} else {
				val = value
			}

			let hex = value.toString(16);
		},

		methods: {
			onSubmit(e) {
				let info = e.detail.value;
				console.log(info);
				if (!info.height || !info.weight || !info.age || !info.sex || !info.steps || !info.sleep) {
					uni.showToast({
						title: '请输入完成相应值在提交！',
						icon: 'none'
					})
					return
				}

				let data = {
					height: info.height,
					weight: info.weight,
					age: info.age,
					sex: info.sex,
					steps: info.steps,
					sleep: info.sleep
				}
				veepooFeature.veepooSynchronizingPersonalInformationManager(data);
			},
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log("同步个人信息 监听蓝牙回调=>", e);
					if (e.content.settingState) {
						uni.showToast({
							title: '同步成功',
							icon: 'none'
						})
					}
					if (e) {
						self.device = e;
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
	}

	.card-title {
		font-size: 30rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.card-title-sub {
		margin-top: 30rpx;
	}

	.form-item {
		display: flex;
		align-items: center;
		margin-bottom: 24rpx;
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

	.btn {
		width: 100%;
		border-radius: 12rpx;
		font-size: 30rpx;
		color: #fff;
		margin-top: 20rpx;
	}

	.btn-primary {
		background-color: #00b0fb;
	}
</style>
