# Gear VR 接 PC 串流：完整实践记录


> 结果：**技术链路打通，画质可看，但帧率/发热撑不住，判定为硬件极限**

---

## 一、结论先行

### 做到了
- 把**已经崩溃**的 Gear VR 系统修回来了（组件版本错配 → 重新装配套版本）
- 画面成功传到头显：SteamVR 的完整界面可见、可辨认游戏封面
- 数据传输速率 9.3 MB/s、网络延迟 7ms（链路本身质量很好）
- 摸清了 Android 7 + Exynos 7420 上跑 iVRy 的**全部必要条件**

### 没做到
- 转头一卡一卡（帧率 40/58，丢 18 帧）
- 手机十几分钟就烫，之后降频
- 连接断断续续（R323 不支持 USB 直连，只能 Wi-Fi）

### 判定
**不是设置问题，是 2015 年的芯片在跑 2026 年的串流负载。**
Exynos 7420（14nm，2015）+ Gear VR 全封闭空间 + 持续实时解码 = 物理极限。

---

## 二、设备清单

| 项目 | 具体 | 备注 |
|---|---|---|
| 手机 | **SM-G9280**（Galaxy S6 Edge+，国行） | Exynos 7420 / Android 7.0 / arm64-v8a / 5.7寸 2560×1440 |
| 头显 | **Gear VR SM-R323**（2016 版） | 101° 视野，micro USB 模块，**不支持 USB 数据直连** |
| 电脑 | 雷神s2c i7 4720hq 960m

---

## 三、完整流程（下次照这个走）

### 阶段 1：装软件（电脑端，全免费）
1. Steam 装 **SteamVR**
2. Steam 装 **iVRy Driver for SteamVR**（appid 992490）
3. Steam 装 **iVRy for SteamVR (GearVR/Oculus App Installer)**（DLC，appid 1487090）
4. Steam 装 **iVRy GearVR System Files Installer**（DLC，appid 3260890）——**这个的加密包解不开，实际没用上，可以跳过**



### 阶段 2：手机推送 app
1. 手机开 **USB 调试**
2. 手机开 **「验证 USB 传输的应用」→ 关**（不然 adb 安装被拦）
3. USB 连电脑，Steam 启动 iVRy Driver → 选「Launch Install GearVR/Oculus Mobile App」
4. **如果自动安装失败**（本次就失败了），走手动：
   - APK 在 `G:\SteamLibrary\steamapps\common\iVRy\APK\iVRy-ovr.apk`
   - 或直接用 adb：`adb install -r -d "路径\iVRy-ovr.apk"`
   - 本次成功方式是 adb 直装

### 阶段 3：Gear VR Service 设置（★ 最关键，错一个就出不来）
```
设置 → 应用 → 显示系统应用 → Gear VR Service → 存储 → 管理存储
→ 连点「VR Service Version」6 次 → 开发者选项出现
```

| 开关 | 设置 | 为什么 |
|---|---|---|
| **Allow VR API without OSIG** | ✅ **必须开** | iVRy 靠它绕过签名校验 |
| **Developer mode** | ❌ **必须关** | ★ 本次最大发现，见坑 1 |
| **Mirroring mode** | ❌ **必须关** | 老机型不支持，会干扰 VR 显示 |

### 阶段 4：修复 Oculus 组件版本错配（★ 本次核心技术成果）
**症状**：装完 iVRy 后，插头显没反应，日志刷 `NoClassDefFoundError: com.oculus.os.SettingsObserverCallback`

**诊断**：各组件版本混血
```
systemdriver        9.0.0.169.450
ocms                9.0.0.130.473
vrshell             9.0.0.170.450
systemactivities    9.0.0.169.450
systemutilities     9.0.0.232.450
horizon             9.0.0.130.473
```

**修复**：从 `vault.gearvr.net` 下载 `Oculus_Install_Files.zip`（261MB，S8 提取的一套配套版本），逐个 adb 覆盖安装：
```bash
adb install -r -d "Oculus System Driver.apk"
adb install -r -d "Oculus System Utilities.apk"
adb install -r -d "Oculus System Activities.apk"
adb install -r -d "Oculus Home.apk"
adb install -r -d "Oculus Core Mobile Services.apk"
adb install -r -d "Oculus app runtime.apk"
adb install -r -d "Oculus VR.apk"
adb install -r -d "Oculus.apk"
adb install -r -d "Oculus Browser.apk"
```
**修复后版本统一为 232.450 / 257.473 两批，ocms 不再崩溃。**
APK 备份位置：`D:\GearVR_Fix\apks\`

### 阶段 5：连接与启动
1. 拔 USB，手机插进 Gear VR（**R323 用 A 档**，5.7寸机型用 A）
2. 电脑 Steam 启动 iVRy Driver → 点 **Launch**
3. iVRy 会自动识别手机（Bonjour 服务发现）
4. 首次**必须**从 Launch 按钮启动（官方要求，会装附带服务）


**现象**：插进去显示灰色等待页，`sys.hmt.mounted = false`（灰色是正常的，这个是后来才发现的，灰色显示出来以后可以开始调试电脑了）

### 坑 3 Oculus 组件版本错配
见阶段 4。**这是 Meta 在 2023–2024 年反复搞坏 Gear VR 的后果**，官方更新器有 bug，会把已装好的设备弄成混血状态。


### 坑 5 Subsampling 4:2:2 导致解码失败
4:2:2 色度抽样，老硬解只吃 4:2:0
**解法**：Subsampling 拉到最左 = **4:2:0**

### 坑 6 JPEG 是软解，CPU 解不动
JPEG 不走硬解电路，全靠 CPU 一帧帧解。Exynos 7420 解 768×768@60 撑不住 → 一卡一卡
**解法**：用 H265（硬解）

### 坑 7 设置改完必须重启 SteamVR
iVRy 面板顶部明写：「SteamVR must be restarted for some changes to take effect」
**解法**：改完点主界面左下角「Restart SteamVR」

### 坑 8（环境）无线 adb 会被 PowerShell 进程回收
每次新 PowerShell 进程会重启 adb daemon，TCP 连接丢失
**解法**：在同一个命令里先 `adb connect` 再操作

---

## 五、iVRy 最终配置

### Performance 面板
```
Frame Encoder:          H265（硬解，最重要）
Subsampling:            4:2:0
Max Output Resolution:  768×768（per eye）
Auto-adjust Resolution: 关
Auto-adjust Quality:    关
```

### SteamVR 面板
```
Render Resolution (per eye):  1344×1344
Use Room Setup:               关
Head Height:                  165cm
```

### Audio/Video 面板
```
Stream Audio:   可开（声音不传到手机，耳机接电脑）
```

### 配置文件（只记录改动过的项）
`D:\steam\config\steamvr.vrsettings` 的 `driver_ivry` 段：
```json
"driver_ivry" : {
    "address" : "192.168.1.100",
    "linkOutputToRenderResolution" : false,
    "renderHeight" : 768,
    "renderWidth" : 1536
}
```
**注意：编码器设置不在这个文件里，存在程序内部，只能界面改。**

---

## 六、为什么到极限（数据）

### 实测数据
| 指标 | 数值 | 正常值 |
|---|---|---|
| 传输速率 | 9.3 MB/s | ✅ 够 |
| 网络延迟 | 7 ms | ✅ 优秀 |
| 带宽 | 13.5 MB/s | ✅ 够 |
| **渲染帧率** | **58 Hz** | 目标 60 |
| **显示帧率** | **40 Hz** | 目标 60 ❌ 丢 18 帧 |

### 端到端延迟链路
```
手机陀螺仪采样 → Wi-Fi 传输 → 电脑渲染 → NVENC 编码 → 回传 → 手机解码 → 显示
└────────────────── 合计 60–100ms（真头显 <20ms）──────────────────┘
```

### 发热曲线
- 前 15–20 分钟：正常
- 之后：边框明显发烫 → 系统降频 → 帧率掉
- Exynos 7420 是 14nm LPE（2015），能效比现代芯片差一个数量级

---

## 七、操作上限（这台机器能做到什么）

| 能做 | 勉强 | 做不到 |
|---|---|---|
| 坐着转头看环境 | 一般社交 world | 凑近看细节（头不能挪） |
| 看片 / 360 视频 | 人数中等的房间 | 蹲下 / 低头看身体 |
| 在 SteamVR 里逛 | 需要走位的 world | 用手抓东西、做手势 |
| 静态体验类内容 | | 快速甩头（必拖影） |

**交互手段只剩**：转头（3DoF）+ Gear VR 触摸板 + 蓝牙手柄

---

## 八、下一步（未完成）

### Moonlight 平面串流（推荐）
- 电脑端 **Sunshine 已安装并启动**：`C:\Program Files\Sunshine`，端口 47984/47989/47990/48010 全部就绪
- 手机端需要 **月光·阿西西**：`Axixi2233/moonlight-android`，最新 `axi260912.apk`
  - 直链：`https://github.com/Axixi2233/moonlight-android/releases/download/260912/axi260912.apk`
  - SHA256：`473185d98aa61b5c3f79a40559986ccd9ead6ed23af8b2ab93d5e3db74e440b4`
  - 镜像：`https://ghproxy.net/` + 完整 GitHub 地址（速度约 14KB/s，很慢）

- **优点**：只走 H.264 硬解，不烫、不卡、延迟 10–20ms
- **代价**：平面画面，无立体感、无头部追踪

### 硬件升级（如果真要做 VR）
| 选项 | 价格 | 评价 |
|---|---|---|
| 二手 S10+ | 500–800 | △ 不推荐：仍是 60Hz + 3DoF + 串流 |
| **二手 Quest 2** | **800–1200** | ✅ 正解：6DoF + 90Hz + 独立运行 |

---

## 九、命令与路径速查

```bash
# adb（iVRy 自带）
G:\SteamLibrary\steamapps\common\iVRy\ADB\adb.exe

# 无线调试
adb -s <serial> tcpip 5555
adb connect 192.168.1.100:5555

# 查头显状态（最重要的一行）
adb shell getprop sys.hmt.connected   # 物理连接
adb shell getprop sys.hmt.developer   # 开发者模式（应为 false）
adb shell getprop sys.hmt.mounted     # 挂载完成（应为 true）

gearvr应该是完球了，手机硬件卡死了，当前用的是s6edge＋，不确定s8，s9，s10会强多少，现在这几款手机太难凑出来完美的了，而且脱离性价比了完全没折腾的必要。
可惜三星没有继续做下去，gearvr或许未来已尽。。。

