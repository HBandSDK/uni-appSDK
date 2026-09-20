<template>
	<view class="page">
		<text class="tip-step">
			使用步骤
			1.读取表盘文件(在聊天信息中读取文件)
			2.开始传输(传输过程中保持在该页面)
		</text>
		<view style="height: 100rpx;"></view>
		<view class="file-info">文件名：
			<view style="color: blue;">{{fileData.fileName}}</view>
			<!-- <input class="weui-input" bindinput="inputFileNmae" value="{{fileName}}" placeholder="输入文件名" /> -->
		</view>
		<!-- <view class="file-info">文件读取状态:
		    <view wx:if="{{fileStatus==0}}" style="color: brown;">未读取</view>
		    <view wx:elif="{{fileStatus==1}}" style="color: chartreuse;">读取中</view>
		    <view wx:elif="{{fileStatus==2}}" style="color: blue;">已读取</view>
		</view> -->
		<view class="file-info">文件信息:
			<view style="color: blue;">{{fileData.fileInfo}}</view>
		</view>
		<view style="height: 100rpx;"></view>
		<view class="blue-btn" @click="clickReadDialFile">读取表盘文件</view>

		<view style="height: 100rpx;"></view>
		<view v-if="!isTransfering" class="blue-btn" @click="clickStartTransferDialFile">开始传输</view>
		<view v-else class="blue-btn" @click="clickCancelTransferDialFile">取消传输</view>
		<view>{{transferProgressText}}</view>
	</view>
</template>

<script>
	import {
		PackResFormat,
		Uint8ArrayToString
	} from "../../../jieli_sdk/jl_lib/jl_packResFormat_1.0.0";
	import {
		RCSPOpSystemInfo,
		RCSPOpWatchDial
	} from "../../../jieli_sdk/lib/rcsp-impl/rcsp";
	import {
		OPDirectoryBrowse,
		OPLargerFileTrans
	} from "../../../jieli_sdk/jl_lib/jl-rcsp-op/jl_op_watch_1.1.0";

	import {
		veepooJLGetFileDataManager,
		veepooJLAddDialTransferStartManager
	} from '../../../jieli_sdk/index'
	export default {
		data() {
			return {
				fileName: '',
				fileInfo: '',
				fileStatus: 0, //0：未读取，1：读取中，2：已读取
				transferProgressText: '',
				isTransfering: false,
				fileData: {},
				dialData: Uint8Array.prototype,
				lastModifyTime: 0,
			}
		},
		onUnload() {
			RCSPOpWatchDial?.cancelAddWatchResourseFile()
		},
		methods: {
			/**
			 * 点击事件--输入文件名
			 */
			inputFileNmae(e) {
				var value = e.detail.value
				console.log(" tas " + value);

				this.setData({
					fileName: value
				})
			},
			/**
			 * 点击事件--读取表盘文件
			 */
			clickReadDialFile() {
				let self = this;
				if (this.isTransfering) {
					return
				}
				// 小程序：chooseMessageFile 选文件
				// #ifdef MP
				uni.chooseMessageFile({
					count: 1,
					success: (res) => {
						const tempFilePaths = res.tempFiles[0];
						console.log("res==>", res)
						console.log("tempFilePaths : ", tempFilePaths);
						// 获取文件数据
						veepooJLGetFileDataManager(tempFilePaths, function(result) {
							console.log("result获取文件成功=>", result)
							self.setData({
								fileData: result
							})
						})

					}
				})
				// #endif
				// App：最小验证用，绕过文件选择器。
				// #ifdef APP-PLUS
				self._loadBundledFirmware();
				// #endif
			},

			// plus.io 读取文件
			// _loadBundledFirmware() {
			// 	const that = this;
			// 	const rel = '_www/static/dial/firmware.ufw'; //这里的firmware.ufw 只是一个示例文件
			// 	that.fileStatus = 1;
			// 	uni.showLoading({
			// 		title: '读取表盘文件中...',
			// 		mask: true
			// 	});
			// 	// @ts-ignore
			// 	plus.io.resolveLocalFileSystemURL(rel, (entry) => {
			// 		entry.file((file) => {
			// 			// file.size 就是权威长度，用它给 _onFirmwareLoaded 做字节数对账，
			// 			// 不必再为了拿 length() 去碰 plus.android
			// 			const expectSize = file.size || 0;
			// 			console.log('[固件] ' + rel + ' size=' + expectSize);
			// 			if (expectSize <= 0) {
			// 				that._onFirmwareFailed('固件为空(size=0)，请确认 static/dial/表盘文件 已同步到手机');
			// 				return;
			// 			}
			// 			// @ts-ignore
			// 			const reader = new plus.io.FileReader();
			// 			reader.onloadend = (evt) => {
			// 				const result = (evt.target && evt.target.result) || '';
			// 				// dataURL 形如 data:xxx;base64,AAAA...，只要逗号后面那段
			// 				const comma = result.indexOf(',');
			// 				const b64 = comma >= 0 ? result.substring(comma + 1) : result;
			// 				console.log('[固件] base64 len=' + b64.length);
			// 				that._onFirmwareLoaded(that._b64ToUint8(b64), expectSize);
			// 			};
			// 			reader.onerror = (e) => {
			// 				that._onFirmwareFailed('FileReader 失败: ' + ((e && e.message) || JSON
			// 					.stringify(e)));
			// 			};
			// 			reader.readAsDataURL(file);
			// 		}, (e) => {
			// 			that._onFirmwareFailed('entry.file 失败: ' + ((e && e.message) || JSON.stringify(
			// 				e)));
			// 		});
			// 	}, (e) => {
			// 		that._onFirmwareFailed('找不到固件文件 ' + rel + '：' + ((e && e.message) || JSON.stringify(e)) +
			// 			'。请确认 static/ota/firmware.ufw 已同步到手机(重新运行到手机)');
			// 	});
			// },
			/**
			 * 点击事件--开始传输
			 */
			clickStartTransferDialFile() {
				let self = this;
				const fileName = this.data.fileName.trim()
				let fileData = this.data.fileData;
				if (fileData.fileName.length == 0) {
					wx.showToast({
						title: "文件名不能为空",
						icon: 'error'
					})
					return
				}

				console.log("点击了开始传输")

				veepooJLAddDialTransferStartManager(fileData, function(result) {
					console.log("传输进度result=>", result);
					self.setData({
						transferProgressText: result.transferProgressText
					})
				})
				// RCSPOpSystemInfo?.getSystemInfo().then((systemInfo) => {
				//   console.log("开始传输执行")
				//   console.log("systemInfo=>", systemInfo)
				//   if (systemInfo.callStatus == 1) {//手表通话中
				//     wx.showToast({
				//       title: '通话中，不可操作',
				//       icon: 'none'
				//     })
				//   } else if (systemInfo.callStatus == 0) {//手表空闲
				//     if (this.data.fileStatus == 2 && this.dialData.length > 0) {
				//       const _callback: OPLargerFileTrans.TransferTaskCallback = {
				//         onError: (code: number) => {
				//           this.setData({
				//             transferProgressText: "传输失败，code:" + code
				//           })
				//         },
				//         onStart: () => {
				//           this.setData({
				//             transferProgressText: "开始传输",
				//             isTransfering: true
				//           })
				//         },
				//         onProgress: (progress: number) => {
				//           this.setData({
				//             transferProgressText: "正在传输，进度:" + progress
				//           })
				//         },
				//         onSuccess: () => {
				//           this.setData({
				//             transferProgressText: "传输成功",
				//             isTransfering: false
				//           })
				//         },
				//         onCancel: (_code: number) => {
				//           this.setData({
				//             transferProgressText: "传输取消",
				//             isTransfering: false
				//           })
				//         }
				//       }
				//       RCSPOpWatchDial?.addWatchResourseFile(this.dialData, fileName, this.lastModifyTime, true, _callback).then((res) => {
				//         if (res instanceof OPDirectoryBrowse.File) {//传输成功且刷新目录
				//           //设置为当前表盘
				//           RCSPOpWatchDial?.setUsingDial(res)
				//         } else if (res == undefined) {//传输完成后,目录浏览找不到该文件
				//           console.error("传输完成后找不到该文件，可能是文件名不符合标准，太长或者带中文");
				//         } else {//传输成功且不刷新目录
				//           console.log("目录浏览的相对路径：path：" + res);
				//         }
				//       }).catch((error) => {
				//         wx.showToast({ title: "添加表盘失败，" + error, icon: 'error' })
				//       })
				//     } else {
				//       wx.showToast({ title: "请先读取表盘文件", icon: 'error' })
				//       return
				//     }
				//   }
				// })
			},
			/**
			 * 点击事件--取消传输
			 */
			clickCancelTransferDialFile() {
				RCSPOpWatchDial?.cancelAddWatchResourseFile()
			}
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

	.tip-step {
		width: 100%;
	}

	.file-info {
		display: flex;
		flex-direction: row;
		align-items: center;
	}
</style>