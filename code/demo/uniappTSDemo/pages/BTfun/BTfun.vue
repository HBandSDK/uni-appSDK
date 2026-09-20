<template>
	<view class="box">
		<button class="btn" :disabled="isTest" @click="BTconnect">开始BT连接</button>
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

			}
		},
		methods: {
			BTconnect() {
				let data2 = {
					model: 'setting',
					switch: 'open',
					mediaSwitch: 'close',
					isPair: true
				}
				console.log('data2=>', data2)
				veepooFeature.veepooSendBTPairFunctionManager(data2)


			}
		}
	}
</script>

<style>
	.box {
		margin: 50px;
	}

	.item {
		line-height: 80rpx;
		font-size: 18px;
	}

	.btn {
		margin-top: 50rpx;
		background-color: #a8a8a8;
		width: 100px;
		height: 35px;
		text-align: center;
		margin: 50px auto;
		font-size: 14px;
		color: white;
	}
</style>