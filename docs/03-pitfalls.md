# 避坑速查

> 一页纸版本。遇到问题按「症状」查。

---

## 一、症状 → 原因 → 解法

| 症状 | 最可能的原因 | 解法 |
|---|---|---|
| 插进 Gear VR **完全没反应** | Oculus 组件版本错配 | 见 [01 篇 阶段4](01-practice-log.md)，重装配套版本 |
| 插进去是**灰色等待页** | `Developer mode` 开着，系统不走自动挂载 | 关掉 Developer mode，只留 OSIG |
| 手机启动 iVRy 后**纯灰，没文字** | 正常，这是「请插入头显」的等待页 | 插进头显即可 |
| **画面全黑，但数据在传**（吞吐量几 MB/s） | H.264 编码黑屏 / Subsampling 4:2:2 | 编码器改 H265，Subsampling 改 4:2:0 |
| **一卡一卡** | JPEG 软解 CPU 解不动 / 发热降频 | 换 H265 硬解；降分辨率；让手机凉下来 |
| 电脑上 iVRy 提示 **communication errors** | 手机掉线（Wi-Fi 休眠 / app 被杀） | 关 Wi-Fi 省电；`设置→Wi-Fi→高级→休眠期间保持开启` |
| 改了设置**没生效** | 没重启 SteamVR | 点 iVRy 主界面「Restart SteamVR」 |
| iVRy **双击 exe 就闪退** | 脱离 Steam 启动，拿不到 Steam API | 必须从 Steam 启动（`steam.exe -applaunch 992490`） |
| Gear VR 插上后**自动进 Oculus Home** | 正常行为 | 若要跑 iVRy，先开 iVRy 再插 |
| Gear VR 的开发者选项**自己关掉了** | 系统行为，关掉 Gear VR 后会复位 | 每次使用前确认一遍 |

---

## 二、必开 / 必关清单

### Gear VR Service → 开发者选项

```
设置 → 应用 → 显示系统应用 → Gear VR Service
→ 存储 → 管理存储 → 连点「VR Service Version」6 次
```

| 开关 | 设置 | 说明 |
|---|---|---|
| **Allow VR API without OSIG** | ✅ **必须开** | 允许运行未签名的 VR 应用 |
| **Developer mode** | ❌ **必须关** | 开了会导致头显不挂载 |
| **Mirroring mode** | ❌ **必须关** | 老机型（S7/S8 及更早）不支持 |
| Add icon to app list | 随意 | 只是桌面快捷方式 |

### iVRy → Performance

```
Frame Encoder:          H265         ← 别用 H264（黑屏）/ JPEG（卡）
Subsampling:            4:2:0        ← 别用 4:2:2
Auto-adjust Resolution: 关
Auto-adjust Quality:    关
```

### 手机系统

```
设置 → 开发者选项 → 验证 USB 传输的应用 → 关    （不然 adb 装不上）
设置 → Wi-Fi → 高级 → 休眠期间保持 Wi-Fi 开启 → 始终
```

---

## 三、诊断命令

```bash
adb = G:\SteamLibrary\steamapps\common\iVRy\ADB\adb.exe   # iVRy 自带

# ★ 三条最有用的状态查询
adb shell getprop sys.hmt.connected   # 头显物理连接（应为 true）
adb shell getprop sys.hmt.developer   # 开发者模式（应为 false）
adb shell getprop sys.hmt.mounted     # 挂载完成（应为 true，否则必灰屏）

# 当前前台是什么
adb shell dumpsys activity activities | grep mFocusedActivity

# 查组件版本（判断是否错配）
adb shell dumpsys package com.oculus.systemdriver | grep versionName

# 无线调试（插头显后 USB 被占，只能走这个）
adb -s <序列号> tcpip 5555
adb connect <手机IP>:5555
```

### 判读方法

**不要靠“屏幕看起来黑/灰”判断**，直接查 `sys.hmt.mounted`：

| 值 | 含义 |
|---|---|
| `connected=false` | 没检测到头显 → 检查 micro USB 模块 / 挡位（R323 大屏用 A 档） |
| `connected=true, mounted=false` | 检测到了但没挂载 → 八成是 Developer mode 开着 |
| `mounted=true` 但画面黑 | 系统层没问题，是编码/解码问题 |

---

## 四、连接与启动顺序（顺序错了就出不来）

```
1. 手机开 USB 调试，USB 连电脑
2. Steam 启动 iVRy Driver（首次要选 Launch，装附带服务）
3. 拔 USB
4. 手机上点 iVRy 图标，按提示插进 Gear VR
5. ★ 确认已插好，再点 iVRy 的 Launch 启动 SteamVR
```

**官方原话**：`Device must be mounted in GearVR before starting SteamVR.`

---

## 五、如果全部照做了还是不行

先确认这三条，任一不成立就别往下调参：

1. `sys.hmt.mounted` 是 `true` 吗？
2. iVRy 设置里 `Connection` 显示 `Wifi` 且 `Address` 有正确的手机 IP 吗？
3. `Throughput` 有没有到 **MB/s 级**（不是 KB/s）？

三条都满足但画面仍是黑的 → 编码/解码问题，换 H265 + 4:2:0。
三条中有不满足的 → 回到对应章节。

**如果都正常但还是卡、还是烫** → 到硬件极限了，换设备或换方案。详见 [01 篇 第六节](01-practice-log.md)。
