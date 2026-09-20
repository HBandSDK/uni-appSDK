<template>
	<view class="box">
		<view class="header">
			<text class="header-title">联系人管理</text>
		</view>

		<!-- 读取按钮 -->
		<view class="action-row">
			<button class="btn btn-primary" @click="ReadContactPersonDataManager">读取联系人</button>
		</view>

		<!-- 设置联系人 -->
		<view class="card">
			<view class="card-title">设置联系人</view>
			<view class="form-item">
				<text class="form-label">姓名</text>
				<input class="form-input" @input="getName" type="text" placeholder="请输入姓名" />
			</view>
			<view class="form-item">
				<text class="form-label">电话号码</text>
				<input class="form-input" @input="getPhone" type="text" placeholder="请输入手机号" />
			</view>
			<view class="form-item form-item-switch">
				<text class="form-label">紧急联系人</text>
				<switch :checked="sos" @change="getSOS" color="#00b0fb" />
			</view>
			<view class="form-item">
				<text class="form-label">编辑ID</text>
				<input class="form-input" @input="getPhoneId" type="number" placeholder="添加时不填" />
			</view>
			<view class="btn-group">
				<button class="btn btn-primary" @click="SettingContactPersonDataManager">添加联系人</button>
				<button class="btn btn-warn" @click="editContactPerson">编辑联系人</button>
			</view>
		</view>

		<!-- 删除联系人 -->
		<view class="card">
			<view class="card-title">删除联系人</view>
			<view class="form-item">
				<text class="form-label">联系人ID</text>
				<input class="form-input" @input="deleteId" type="text" placeholder="请输入要删除的ID" />
			</view>
			<button class="btn btn-danger" @click="deleteContactPersonDataManager">删除联系人</button>
		</view>

		<!-- 调整顺序 -->
		<view class="card">
			<view class="card-title">调整顺序</view>
			<view class="form-item">
				<text class="form-label">源ID</text>
				<input class="form-input" @input="getFromId" type="text" placeholder="需要调整的ID" />
			</view>
			<view class="form-item">
				<text class="form-label">目标ID</text>
				<input class="form-input" @input="getToId" type="text" placeholder="目标位置ID" />
			</view>
			<button class="btn btn-primary" @click="AdjustContactPersonDataManager">调整顺序</button>
		</view>

		<!-- 联系人列表 -->
		<view class="card" v-if="readList.length">
			<view class="card-title">联系人列表（{{readList.length}}）</view>
			<view class="contact-item" v-for="(item,index) in readList" :key="index">
				<view class="contact-main">
					<view class="contact-name-row">
						<text class="contact-name">{{item.name}}</text>
						<text class="contact-id">ID: {{item.id}}</text>
					</view>
					<text class="contact-phone">{{item.phone}}</text>
				</view>
			</view>
		</view>
		<view class="empty-tip" v-else>
			<text>暂无联系人数据</text>
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
				readList: [],
				sos: false,
				phone: '',
				name: '',
				delId: '',
				fromId: '',
				toId: '',
				phoneId: 0,
			}
		},
		onLoad(options) {

		},
		onReady() {

		},
		onShow() {
			this.notifyMonitorValueChange()
		},
		methods: {
			ReadContactPersonDataManager() {
				veepooFeature.veepooSendReadContactPersonDataManager();
			},
			SettingContactPersonDataManager() {
				let readList = this.readList;
				console.log("readList=>", readList.length + 1)

				// 注意：添加联系人，已有的联系人id 假设是1  那么在添加一个联系人的contactNumber 应该传入2
				let data = {
					isEdit: false,
					contactNumber: readList.length,
					name: this.name,
					phone: this.phone,
					sos: this.sos, // 设置为紧急联系人
				}
				console.log("data=>", data)
				if (!data.phone && !data.name) {
					uni.showToast({
						title: '填写手机号或名称',
						icon: 'none'
					})
					return
				}
				veepooFeature.veepooSendSettingContactPersonDataManager(data)

				setTimeout(() => {
					veepooFeature.veepooSendReadContactPersonDataManager();
				}, 1000);
			},

			editContactPerson() {
				let readList = this.readList;
				console.log("readList=>", readList.length + 1)

				// 注意：编辑联系人，因SDK内部做了自增，所以填入id需要减1，如id 是 2 则 填 1
				let data = {
					isEdit: true,
					contactNumber: Number(this.phoneId) - 1,
					name: this.name,
					phone: this.phone,
					sos: this.sos, // 设置为紧急联系人
				}
				console.log("data=>", data)
				if (!data.phone && !data.name) {
					uni.showToast({
						title: '填写手机号或名称',
						icon: 'none'
					})
					return
				}
				veepooFeature.veepooSendSettingContactPersonDataManager(data)

				setTimeout(() => {
					veepooFeature.veepooSendReadContactPersonDataManager();
				}, 1000);
			},


			deleteId(e) {
				let delId = e.detail.value;
				this.delId = delId
			},

			deleteContactPersonDataManager() {
				let data = {
					sosId: this.delId
				}
				veepooFeature.veepooSendDeleteContactPersonDataManager(data);
				setTimeout(() => {
					veepooFeature.veepooSendReadContactPersonDataManager();
				}, 1000);
			},
			AdjustContactPersonDataManager() {
				let readList = this.readList;
				console.log('readList.length=>', readList.length)
				if (readList.length == 1) {
					uni.showToast({
						title: '请先添加更多的联系人',
						icon: 'none'
					})
					return
				}
				let data = {
					fromId: this.fromId, // 需要调整的id
					toId: this.toId // 目标位置id
				}
				veepooFeature.veepooSendAdjustContactPersonDataManager(data)
				setTimeout(() => {
					veepooFeature.veepooSendReadContactPersonDataManager();
				}, 1000);
			},
			getFromId(e) {
				let fromId = e.detail.value;
				this.fromId = fromId
			},
			getToId(e) {
				let toId = e.detail.value;
				this.toId = toId
			},
			getSOS(e) {
				let sos = e.detail.value;
				this.sos = sos
			},

			getPhone(e) {
				let phone = e.detail.value;
				this.phone = phone
			},

			getPhoneId(e) {
				let phoneId = e.detail.value;
				this.phoneId = phoneId
			},

			getName(e) {
				let name = e.detail.value;
				this.name = name
			},
			// 监听订阅 notifyMonitorValueChange
			notifyMonitorValueChange() {
				let self = this;
				veepooBle.veepooUniAppSDKNotifyMonitorValueChange(function(e) {
					console.log(" 读取联系人 监听蓝牙回调=>", e);

					if (!e) {
						return
					}

					if (e.name == '读取联系人') {
						self.readList = e.content
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

	.form-item {
		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}

	.form-item-switch {
		justify-content: space-between;
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

	.btn-danger {
		background-color: #ff6b6b;
	}

	.action-row {
		margin-bottom: 24rpx;
	}

	.contact-item {
		padding: 20rpx 0;
		border-bottom: 1px solid #f0f0f0;
	}

	.contact-item:last-child {
		border-bottom: none;
	}

	.contact-main {
		display: flex;
		flex-direction: column;
		gap: 6rpx;
	}

	.contact-name-row {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.contact-name {
		font-size: 30rpx;
		font-weight: 500;
		color: #333;
	}

	.contact-id {
		font-size: 24rpx;
		color: #999;
	}

	.contact-phone {
		font-size: 28rpx;
		color: #00b0fb;
	}

	.empty-tip {
		text-align: center;
		padding: 60rpx 0;
		color: #999;
		font-size: 28rpx;
	}
</style>
