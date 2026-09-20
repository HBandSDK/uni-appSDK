<template>
	<!--pages/function_test/dial_operate/dial_add_background/index.wxml-->
	<view class="page">
		<view style="height: 100rpx;"></view>
		<view class="blue-btn" @click="clickWay1">裁剪图片</view>
		<view style="height: 100rpx;"></view>
		<!-- <view  class="blue-btn" catchtap="clickWay2">方式二:传输资源文件</view> -->
		<view style="height: 100rpx;"></view>
		<view class="blue-btn" @click="clickCancelTransfer">取消传输</view>
		<view>{{transferProgressText}}</view>

		<view>{{fileName}}</view>
		<image :src="pathImage" mode="" />
	</view>

</template>

<script lang="ts">
	import {
		RCSPOpSystemInfo,
		RCSPOpWatch,
		RCSPOpWatchDial
	} from "../../../jieli_sdk/lib/rcsp-impl/rcsp";
	import {
		OPDirectoryBrowse,
		OPLargerFileTrans
	} from "../../../jieli_sdk/jl_lib/jl-rcsp-op/jl_op_watch_1.1.0";
	import {
		veepooBle,
		veepooFeature
	} from '../../../common/index.js';
	export default {
		data() {
			return {
				isTransfering: false,
				transferProgressText: "",
				imgs: "../../../image/dial_icon3_main.png",
				fileName: "",
				devScreenWidth: 240,
				devScreenHeight: 280,
				pathImage: ''
			}
		},
		onLoad(options) {
			this.pathImage = options.pathImage

			console.log("options.pathImage=>", options)

			// 获取屏幕信息
			RCSPOpWatch?.getFlashInfo().then((res) => {
				this.devScreenWidth = res.width
				this.devScreenHeight = res.height
				console.log("devScreenWidth : " + this.devScreenWidth);
				console.log("devScreenHeight : " + this.devScreenHeight);
			})
		},
		methods: {
			/**
			  * 生命周期函数--监听页面加载
			  */
			// onLoad(options) {

			// 	this.pathImage = options.pathImage

			// 	console.log("options.pathImage=>", options)

			// 	// 获取屏幕信息
			// 	RCSPOpWatch?.getFlashInfo().then((res) => {
			// 		this.devScreenWidth = res.width
			// 		this.devScreenHeight = res.height
			// 		console.log("addBg-devScreenWidth : " + res.width);
			// 		console.log("addBg-devScreenHeight : " + res.height);
			// 	})
			// },
			onUnload() {
				RCSPOpWatchDial?.cancelAddWatchResourseFile()
			},
			saveFile(fs : WechatMiniprogram.FileSystemManager, filePath : string, data : Uint8Array) {
				fs.writeFileSync(filePath, data.buffer)
				console.log("writeFileSync : ", data);
				console.log("filePath : ", filePath);

				uni.saveImageToPhotosAlbum({
					filePath: filePath,
					success() {
						uni.showToast({
							title: '保存成功'
						})
					},
					fail() {
						uni.showToast({
							title: '保存失败',
							icon: 'none'
						})
					}
				})
			},
			onShow() {

			},
			uint8ArrayToHex(uInt8Array : any) {
				return uInt8Array.map((byte : any) => byte.toString(16).padStart(2, '0')).join('');
			},
			//裁剪图片
			clickWay1() {
				let self = this;
				uni.chooseMedia({
					count: 1,
					mediaType: ['image'],
					sizeType: ['original'],
					success: (res) => {
						console.log("选择图片", res);
						const imagePath = res.tempFiles[0].tempFilePath
						uni.navigateTo({
							url: '/pages/dial/addBg/dial_bg_image/dial_bg_image?imagePath=' + imagePath + "&width=" + this.devScreenWidth + "&height=" + this.devScreenHeight,
							events: {
								// 返回的数据
								onDialBgData: (resDialBg : { data : any }) => {
									console.log("resDialBg==+>", resDialBg);
									// 发送数据 分词发送，先发送缩略图，在发送背景图，固定名字
									this._handleDialBgData(resDialBg.data);

								}
							}
						})
					}
				})
				return
			},
			clickCancelTransfer() {
				RCSPOpWatchDial?.cancelAddWatchResourseFile()
			},

			// 发送缩略图
			_handleDialBgData(data : any) {
				let self = this;
				console.log("onDialBgData :  ", data);
				const lastModifyTime = (new Date()).getTime()
				console.log(" lastModifyTime " + lastModifyTime);
				setTimeout(() => {
					uni.showModal({
						title: "输入文件名",
						editable: true,
						content: "bgp_w000", // 缩略图名称 bgp_w000  背景图 bgp_w001
						success: (res) => {
							if (res.confirm) {
								const lastModifyTime = (new Date()).getTime()
								console.log(" lastModifyTime " + lastModifyTime);

								const fileName = res.content
								if (!fileName.toUpperCase().startsWith("BGP_")) {
									uni.showToast({ title: "文件名应为bgp_xxxx或者BGP_xxxx", icon: "error" })
									return
								}
								const transferCallback : OPLargerFileTrans.TransferTaskCallback = {
									onError: (code : number) => {
										self.transferProgressText = "传输失败，code:" + code
										self.isTransfering = false
									},
									onStart: () => {
										self.transferProgressText = "开始传输"
										self.isTransfering = true
									},
									onProgress: (progress : number) => {
										self.transferProgressText = "正在传输，进度:" + progress
									},
									onSuccess: () => {
										self.transferProgressText = "传输成功"
										self.isTransfering = false
									},
									onCancel: (_code : number) => {
										self.transferProgressText = "传输取消"
										self.isTransfering = false
									}
								}
								console.log("data=>", data)
								console.log("RCSPOpWatchDial=>", RCSPOpWatchDial)
								console.log("data.data1=>", data.data1)
								console.log("data.data1 instanceof Uint8Array=>", data.data1 instanceof Uint8Array)
								if (!RCSPOpWatchDial) {
									console.error("RCSPOpWatchDial 为 undefined，请检查蓝牙是否连接")
									uni.showToast({ title: "蓝牙未连接", icon: "error" })
									return
								}
								if (!data.data1) {
									console.error("data.data1 为空，bmpConvert 可能返回 undefined，请检查图片像素尺寸是否匹配宽*高*4")
									uni.showToast({ title: "图片数据转换失败", icon: "error" })
									return
								}
								const fileDataBuffer = data.data1 instanceof ArrayBuffer ? data.data1 : (data.data1.buffer ? data.data1.buffer : data.data1)
								RCSPOpWatchDial.addWatchResourseFile(fileDataBuffer, fileName, lastModifyTime, true, transferCallback).then((res) => {
									self.fileName = fileName
									if (res instanceof OPDirectoryBrowse.File) {
										console.log("res====>", res)
										console.log("res====>", res.getName())
										RCSPOpWatchDial?.setDialCustomBackground(res).then((res) => {
											self._handleDialBgData2(data)
											console.log("发送数据")
											console.log("res=>", res)
											//设置成功
										}).catch((error) => {
											console.error("setDialCustomBackground 失败=>", error)
										})

									} else {
										console.warn("addWatchResourseFile 返回非 File 对象=>", res)
									}
								}).catch((error) => {
									console.error("addWatchResourseFile 失败=>", error)
								})
							}
						}
					})
				}, 1500);
			},

			// 发送背景图片
			_handleDialBgData2(data : any) {
				let self = this;
				console.log("onDialBgData :  ", data);
				const lastModifyTime = (new Date()).getTime()
				console.log(" lastModifyTime " + lastModifyTime);

				if (data) {
					const lastModifyTime = (new Date()).getTime()
					console.log(" lastModifyTime " + lastModifyTime);

					const fileName = 'bgp_w001'
					if (!fileName.toUpperCase().startsWith("BGP_")) {
						uni.showToast({ title: "文件名应为bgp_xxxx或者BGP_xxxx", icon: "error" })
						return
					}
					const transferCallback : OPLargerFileTrans.TransferTaskCallback = {
						onError: (code : number) => {
							self.transferProgressText = "传输失败，code:" + code
							self.isTransfering = false
						},
						onStart: () => {
							self.transferProgressText = "开始传输"
							self.isTransfering = true
						},
						onProgress: (progress : number) => {
							self.transferProgressText = "正在传输，进度:" + progress
						},
						onSuccess: () => {
							self.transferProgressText = "传输成功"
							self.isTransfering = false


							// 切换自定义背景表盘
							setTimeout(() => {

								let value = {
									control: 1,// 设置 1 读取
									style: 0, // 风格
									styleType: 2 // 0 默认表盘 1 表盘市场  2 自定义表盘
								}
								console.log("value=>", value)
								veepooFeature.veepooSendSwitchCustomBGUIDialManager(value)
							}, 500);

							// 切换自定义UI风格，切换
							setTimeout(() => {
								let value = {
									timePosition: 7,
									timeTopPosition: 2,
									timeButtomPosition: 3,
									isDefaultBg: 0,
									timeColor: [251, 251, 251]
								}
								console.log("value=>", value);
								veepooFeature.veepooSendSetupCustomBackgroundDialDataManager(value)
							}, 1000);


						},
						onCancel: (_code : number) => {
							self.transferProgressText = "传输取消"
							self.isTransfering = false
						}
					}


					console.log("fileName===>", fileName)
					console.log("data.data2=>", data.data2)
					if (!RCSPOpWatchDial) {
						console.error("RCSPOpWatchDial 为 undefined，请检查蓝牙是否连接")
						uni.showToast({ title: "蓝牙未连接", icon: "error" })
						return
					}
					if (!data.data2) {
						console.error("data.data2 为空，bmpConvert 可能返回 undefined，请检查图片像素尺寸是否匹配宽*高*4")
						uni.showToast({ title: "图片数据转换失败", icon: "error" })
						return
					}
					const fileDataBuffer2 = data.data2 instanceof ArrayBuffer ? data.data2 : (data.data2.buffer ? data.data2.buffer : data.data2)
					RCSPOpWatchDial.addWatchResourseFile(fileDataBuffer2, fileName, lastModifyTime, true, transferCallback).then((res) => {
						self.fileName = fileName
						if (res instanceof OPDirectoryBrowse.File) {
							console.log("res====>", res)
							console.log("res====>", res.getName())
							RCSPOpWatchDial?.setDialCustomBackground(res).then((res) => {
								console.log("发送数据")
								console.log("res=>", res)
								//设置成功
							}).catch((error) => {
								console.error("setDialCustomBackground 失败=>", error)
							})

						} else {
							console.warn("addWatchResourseFile 返回非 File 对象=>", res)
						}
					}).catch((error) => {
						console.error("addWatchResourseFile 失败=>", error)
					})
				}

			},
		}
	}
</script>

<style>
	.page {
		width: 100%;
		height: 100%;
		background-color: #F8FAFCFF;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.blue-btn {
		width: 686rpx;
		height: 96rpx;
		background: #398BFF;
		border-radius: 48rpx;
		align-content: center;

		font-size: 30rpx;
		font-family: PingFangSC-Medium, PingFang SC;
		font-weight: 500;
		color: #FFFFFF;
		line-height: 96rpx;
		text-align: center;
	}
</style>