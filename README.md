# UniApp_Ble_SDK

UniApp 蓝牙 SDK

## 1. 概述

UniAppSDK 由深圳维亿魄科技有限公司（以下简称"本公司"）开发，专门为本公司智能穿戴类产品提供 uni-app 跨端蓝牙控制开发平台，覆盖设备扫描、连接、健康数据读取与设备控制等能力，可用于 Android、iOS App 及微信小程序。

> 本仓库为公版 SDK，使用范围限于本公司智能穿戴产品；设备兼容性确认及定制需求，请联系本公司商务对接。

### 1.1 运行环境

| 类别     | 范围                                    | 备注                                                         |
| -------- | --------------------------------------- | ------------------------------------------------------------ |
| 客户端   | Android，iOS，HarmonyOS 4.0, 微信小程序 | 支持 BLE 功能；HarmonyOS Next 因兼容问题暂不支持（[查看说明](https://uniapp.dcloud.net.cn/api/system/bluetooth.html)） |
| 开发平台 | HBuilderX                               | 建议使用最新版本开发                                         |

注意：SDK 可在 Android、iOS App 与微信小程序项目中使用，仅支持真机调试，模拟器不支持蓝牙。

### 1.2 能力边界

| 模块             | 职责                                                         |
| ---------------- | ------------------------------------------------------------ |
| `lib/vp_sdk`     | 核心 SDK：扫描、连接、订阅监听、协议解析、健康数据与设备控制 |
| `lib/jieli_sdk`  | 杰理（Jieli）能力：OTA 升级、表盘传输等                      |

表盘传输、OTA 等功能目前只支持杰里类型设备，其他设备暂不支持。

### 1.3 接入前提

- 手机系统蓝牙已开启；
- Android 扫描设备需开启系统定位服务，并授予应用蓝牙/位置权限；
- 在 `manifest.json` 源码视图的 `distribute` 中补充以下配置（完整示例见 demo）：

安卓/HarmonyOS 4.0：

```json
/* android打包配置 */
"android": {
    "package": "客户填入自己的包名",
    "minSdkVersion": 21,
    "targetSdkVersion": 28,
    "permissions": [
        /*上面是默认的配置*/

        /*下面的几项必须加入*/
        "<uses-permission android:name=\"android.permission.BLUETOOTH\"/>",
        "<uses-permission android:name=\"android.permission.BLUETOOTH_ADMIN\"/>",
        "<uses-permission android:name=\"android.permission.ACCESS_FINE_LOCATION\"/>",
        "<uses-permission android:name=\"android.permission.ACCESS_COARSE_LOCATION\"/>"
    ]
}
```

iOS：

```json
/* ios打包配置 */
"ios": {
    "dSYMs": false,
    "privacyDescription": {
        "privacy": {
            "NSBluetoothAlwaysUsageDescription": "需要访问蓝牙以连接智能设备，实现数据同步与控制",
            "NSBluetoothPeripheralUsageDescription": "需要访问蓝牙以连接智能设备，实现数据同步与控制",
            "NSLocalNetworkUsageDescription": "需要获取局域网权限以发现并连接设备"
        }
    }
}
```

> **重要提示：** 正式版如需使用「网络表盘」或 SDK 中的付费功能，需要在小程序后台配置域名 `https://www.vphband.com`，并确保 SDK 内部使用的 `https://www.vphband.com:9001` 在 App 运行时可正常访问，否则相关功能不可用。

### 1.4 注意事项

⚠ OTA 跨项目/设备号升级错误的文件会导致设备变砖；如需验证升级，需配置指定项目匹配的固件。

### 1.5 方案选型与相关仓库

同一组织下另提供小程序版蓝牙 SDK：[WeChat_Mini_Program_Ble_SDK](https://github.com/HBandSDK/WeChat_Mini_Program_Ble_SDK)。

| 场景                                            | 推荐 SDK          |
| ----------------------------------------------- | ----------------- |
| 仅开发微信小程序，或使用微信官方 Donut 跨端方案 | 小程序 SDK        |
| 同时开发 Android/iOS App，或已有 uni-app 项目   | 本仓库 UniApp SDK |

## 2. 快速开始

### 2.1 引入 SDK

将 `lib/vp_sdk/index.js` 放入 uni-app 项目（demo 中对应 `code/demo/uniappTSDemo/common/index.js`），然后引入：

```js
import { veepooBle, veepooFeature, veepooLogger, init } from '../../common/index.js'
```

> 引入路径按 SDK 文件在项目中的实际位置调整。

### 2.2 最小接入流程

标准顺序为：**扫描设备 >> 连接设备 >> 注册全局监听 >> 密钥认证 >> 调用功能接口**。以下示例参考 demo 首页 `pages/index/index.vue` 的扫描、连接、监听与认证流程整理，保留了 demo 中的关键处理方式：

```js
import { veepooBle, veepooFeature } from '../../common/index.js'

let activeScanReceiver = null

export default {
  data() {
    return { bleList: [] }
  },
  onShow() {
    // 进入页面即开始扫描（首次进入与每次返回都会触发）
    this.startScan()
  },
  onHide() {
    this.stopScan()
  },
  onUnload() {
    this.stopScan()
  },
  methods: {
    startScan() {
      activeScanReceiver = res => {
        // demo 中扫描回调按数组处理，取第一个设备
        const device = res[0]
        if (!device || !device.deviceId) return
        // demo 中还做了 RSSI 过滤、deviceId 去重与数量上限处理
        this.bleList.push(device)
      }
      veepooBle.veepooUniAppSDKStartScanDeviceAndReceiveScanningDevice(res => {
        if (activeScanReceiver) activeScanReceiver(res)
      })
    },

    stopScan() {
      activeScanReceiver = null
      veepooBle.veepooUniAppSDKStopSearchBleManager(() => {})
    },

    connectBle(device) {
      this.stopScan()
      // demo 会缓存设备信息，供重连等功能使用
      uni.setStorageSync('bleInfo', device)
      veepooBle.veepooUniAppSDKBleConnectionServicesCharacteristicsNotifyManager(device, result => {
        if (result.connection) {
          // 1. 连接成功后注册全局监听（必须在认证前）
          this.notifyMonitorValueChange()
          // 2. 延时 500ms 后发起密钥认证，结果从全局监听 type=1 返回
          setTimeout(() => {
            veepooFeature.veepooBlePasswordCheckManager({ isPair: true })
          }, 500)
        }
      })
    },

    notifyMonitorValueChange() {
      // App（Android/iOS）使用此接口；微信小程序请改用
      // veepooUniAppWeiXinPlatformSDKNotifyMonitorValueChange
      veepooBle.veepooUniAppSDKNotifyMonitorValueChange(e => {
        switch (e.type) {
          case 1: // 密钥认证完成
            this.readBattery() // 认证通过后再调用功能接口
            break
          case 2: // 电池电量
            console.log('电量:', e.content)
            break
        }
      })
    },

    // 3. 读取电量，结果仍从全局监听 type=2 返回
    readBattery() {
      veepooFeature.veepooReadElectricQuantityManager()
    },
  },
}
```

### 2.3 关键说明

- 功能接口采用「发指令 + 全局监听」模式：接口调用本身不返回业务数据，结果统一从全局监听回调，通过 `res.type` 区分功能。
- App（Android/iOS）推荐使用 `veepooUniAppSDKNotifyMonitorValueChange`；微信小程序推荐使用 `veepooUniAppWeiXinPlatformSDKNotifyMonitorValueChange`，否则部分场景可能收不到监听数据。
- 全局监听必须在密钥认证前注册，且认证通过后才能调用数据类、控制类功能接口。demo 在连接成功后、认证前完成监听注册。
- 示例中的 `isPair: true` 与 demo 首页保持一致；无特殊说明时建议设为 `false`，OTA 固件升级前必须设为 `false`，详见使用文档 9.1 节。
- demo 的扫描与连接逻辑位于 `pages/index/index.vue`，设备信息展示、电量读取、重连等逻辑位于 `pages/operatePage/operatePage.vue`。
- 默认流程无需调用 `init`；仅在自定义蓝牙传输或复用已有连接时需要，详见使用文档第 6 节。
- 如自行封装蓝牙连接，连接成功且 MTU 设置成功后，必须调用 `veepooUniAppSDKHandoverServiceManager`，详见使用文档 7.4 节。
- demo 在 `App.vue` 的 `onLaunch` 中初始化杰理 SDK（`new veepooJLBle().init()`）并设置日志级别；基础蓝牙功能可不引入，使用 OTA、表盘传输时需引入 `lib/jieli_sdk`，详见使用文档 9.25 节。

完整初始化、接口参数与回调结构，请阅读 **[VeepooUniAppSDK 使用文档](docs/VeepooUniAppSDK使用文档.md)**。其中第 3 节「数据接收模型」为对接必读，第 10 节为 `type` 回调对照总表。

## 3. 功能概览

| 分类         | 主要能力                                                                                             | 参考章节              |
| ------------ | ---------------------------------------------------------------------------------------------------- | --------------------- |
| 设备连接     | 扫描、连接、重连、断开、连接状态监听、获取已连接设备                                                 | 文档第 7 章           |
| 基础信息     | 密钥认证、电池电量、个人信息同步、时间同步、单位/语言设置、肤色档位                                  | 9.1 ~ 9.3、9.9、9.22、9.23、9.29 |
| 健康数据     | 精准睡眠、日常数据、体温、计步、心率、血氧、血压、血糖、血液成分、身体成分、ECG、HRV/洛伦兹、压力、微体检、手动测量、B3 自动测量、女性经期、皮肤电活动 | 9.4 ~ 9.7、9.15 ~ 9.18、9.20 ~ 9.21、9.27 ~ 9.28、9.30 ~ 9.34、9.37 |
| 提醒与通知   | 天气、联系人/SOS、文字闹钟、消息推送（ANCS/Android 编码）、久坐/健康提醒、手机查找手环、抬手亮屏、屏幕亮度 | 9.8、9.10 ~ 9.12、9.14、9.19、9.25 |
| 运动         | 运动模式、运动数据、运动控制、世界时钟                                                               | 9.13、9.35、9.36      |
| 表盘与 OTA   | 自定义/网络表盘、UI 风格、杰理 OTA 与表盘传输（仅杰理设备）                                          | 9.24、9.26            |
| 定制项目     | JH58、ZT163、YM28PRO、4G 服务等                                                                       | 文档附录 A            |
| 日志         | 控制 SDK 内部日志输出，便于调试                                                                      | 文档第 8 章           |

各功能的调用方式、参数与返回结构见使用文档对应章节，demo 中也提供了主要功能的示例页面。

## 4. Demo

使用 HBuilderX 导入 `code/demo/uniappTSDemo`，运行到 Android、iOS 或微信小程序后使用真机调试；小程序需填入自己的 AppID。主要功能均提供示例页面，开发具体功能前建议先参考 demo。

## 5. 目录结构

```
├── code
│   └── demo
│       └── uniappTSDemo          # 演示工程（common/index.js 为可运行的 SDK）
├── docs
│   ├── txt                       # hrv、睡眠、相似度、ECG 文本
│   │   ├── ECG_Text.json
│   │   ├── hrv_font_tips.js
│   │   ├── similarity_tips.js
│   │   └── sleep_text.txt
│   ├── VeepooUniAppSDK使用文档.md
│   └── 照片表盘传输简易流程图.pdf
├── lib                           # 核心库
│   ├── jieli_sdk                 # 杰理 OTA/表盘 TS 源码
│   └── vp_sdk                    # SDK 构建产物（index.js）
├── LICENSE                       # Apache-2.0
└── README.md
```

## 6. 常见问题

| 问题                       | 排查方向                                                     |
| -------------------------- | ------------------------------------------------------------ |
| 搜不到设备                 | 确认使用真机、蓝牙已开启；Android 需开启系统定位并在 manifest 中配置蓝牙/位置权限 |
| 小程序收不到监听数据       | 微信小程序需使用 `veepooUniAppWeiXinPlatformSDKNotifyMonitorValueChange` |
| 密钥认证不通过             | 确认全局监听在认证前注册，且设备已成功连接                   |
| 调用接口后没有数据         | 功能结果从全局监听 `res.type` 返回，不是接口返回值           |
| 自行封装蓝牙后部分功能异常 | 连接成功且 MTU 设置成功后，调用 `veepooUniAppSDKHandoverServiceManager` |
| 网络表盘或付费功能不可用   | 小程序正式版需配置域名 `https://www.vphband.com`，并确保 `https://www.vphband.com:9001` 可访问 |
| OTA、表盘传输不可用        | 该能力仅支持杰理类型设备，且需引入 `lib/jieli_sdk`           |
| 页面退出后仍占用蓝牙资源   | 离开页面时调用 `veepooUniAppSDKStopSearchBleManager`，必要时断开连接 |
| OTA 升级后设备无法开机     | 确认 ufw 固件与当前项目/设备匹配，禁止跨项目或跨设备号升级   |

## 7. 当前版本

| 项目                     | 版本       | 发布日期  |
| ------------------------ | ---------- | --------- |
| UniAppSDK                | **V1.1.0** | 2026/11/6 |
| VeepooUniAppSDK 使用文档 | **V2.0.5** | 2026/11/6 |

## 8. 许可证

本项目基于 [Apache-2.0](LICENSE) 许可证开源。
