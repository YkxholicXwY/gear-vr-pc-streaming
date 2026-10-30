# Gear VR + Galaxy S6 Edge+ 串流 PC VR：资料汇编

> 整理日期：2026-10-30
> 目标设备：Galaxy S6 Edge+（Exynos 7420 / Android 7.0）+ Gear VR SM-R323
> 主机：RTX 4060 笔记本（借）
> 用途：VRChat 的 VR 模式
>
> **每条信息都标了来源。分三类：**
> - 【官方】= iVRy 开发者或 Steam 官方页面
> - 【实测】= 社区用户真实操作记录
> - 【推断】= 我根据信息做的判断，未经验证

---

## 一、结论先行

| 项目 | 判断 | 依据 |
|---|---|---|
| 技术上能不能跑通 | **能，路径清晰** | 【官方】iVRy 有专门的 GearVR DLC 和完整流程 |
| 你这个机型有没有人试过 | **没找到公开记录** | 搜遍中英文，iVRy+GearVR 的案例集中在 S8/S9/S10/Note 系列 |
| 视觉体验 | 中等 | 5.7 寸 2K + R323 的 101°，是当年 Gear VR 最佳搭配 |
| 交互上限 | **3DoF，坐着的 VR** | 硬件决定，无法突破 |
| 最大风险 | **三个，见第三节** | |

---

## 二、官方流程（iVRy 开发者写的）

> 来源：Steam 商店 iVRy for SteamVR (GearVR/Oculus App Installer) 页面说明【官方】

### 前置条件
- 电脑装好 **SteamVR**（iVRy 硬性要求，必须先装）
- 手机开启 **USB 调试**
- 一根能传数据的 micro USB 线

### 步骤

**1. 电脑端装三样（全免费）**
- `iVRy Driver for SteamVR`（基础驱动）
- `iVRy for SteamVR (GearVR/Oculus App Installer)`（DLC，用来把 app 推进手机）
- `iVRy GearVR System Files Installer`（DLC，修 Gear VR 系统文件用的，备用）

**2. 手机开 USB 调试，用数据线连电脑**

**3. 运行 iVRy Driver，选「Install GearVR/Oculus Mobile App」**
- 它会把 iVRy 的手机端应用通过 USB 推进手机

**4. 手机进 Gear VR Service 开发者选项，开两个开关**
- 路径：设置 → 应用 → Gear VR Service → 存储 → 管理存储 → **连点「VR Service Version」6 次**
- 【实测】gearvr.net 的 Gear VR Codex 明确写的是 **6 次**
- 如果应用列表里找不到 Gear VR Service：点右上角三点 → 显示系统应用
- 必须打开：
  - **Developer mode**（不用真插头显也能运行 VR 应用）
  - **Allow VR API without OSIG**（★ 这个最关键，它关掉 Gear VR 的签名校验，iVRy 才能跑）
- 建议顺手打开：
  - **Add icon to app list**（把 Gear VR Service 加到桌面，方便以后改设置）

**5. 手机上点 Android 启动器里的 iVRy 图标，然后插进 Gear VR**

**6. ★ 顺序关键：设备必须先装进 Gear VR，再启动 SteamVR**
- 官方原文："Device must be mounted in GearVR before starting SteamVR."

**7. 从 iVRy 的「Launch」按钮启动 SteamVR**
- 【官方】首次使用**必须**从 Launch 按钮启动，它要装一些附带的服务和驱动

**8. 正常进 SteamVR Home，然后开 VRChat**

---

## 三、六个已知坑（按踩到的概率排序）

### 坑 1：Gear VR Service 开发者选项会自己关掉
> 【实测】GitHub 上 S9+ 的 ALVR 教程明确提到

原话大意：以后可能还要回来重新开这些选项，因为它们会自己关掉（通常是在关闭 Gear VR 之后）。

**关闭 Gear VR 的正确方式**：下拉通知栏 → 点「Turn off」。不是直接拔手机。

**对策**：每次戴之前先确认这两个开关还在。

---

### 坑 2：Oculus 版本需要降级到 3.27.0.6.963
> 【实测】GitHub：mgrng/alvr_samsung_s9_tutorial

原话：「iVRy 安装的 Oculus 版本会在启动时崩溃。要解决这个问题，需要 Oculus 版本 3.27.0.6.963。」

**对策**
- 从 APKMirror 找 3.27.0.6.963 这个版本
- 先卸载手机上的 Oculus，再装旧版
- 可能需要 adb 安装
- 注意：这个教程是 S9+ 的，**你的 S6 Edge+ 是否同样需要降级，未知**（见第四节）

---

### 坑 3：DLC 没装，就看不到安装选项
> 【实测】Steam 社区讨论区，iVRy 开发者本人回复

症状：点了 Launch 按钮，没有「Install GearVR/Oculus Mobile App」这一项。

开发者给的三条排查：
1. DLC 没装 → 去装
2. DLC 装了但选项没出现 → 手动运行 `steamapps\common\iVRy\bin\win64\ivry_install_ovr.exe`
3. 还不行 → 直接侧载 APK，文件在 `steamapps\common\iVRy\APK` 文件夹里

**这条很实用**：即使自动安装失败，APK 就在本地，能手动装。

---

### 坑 4：分辨率可能被限制在 1920×864
> 【实测】Reddit r/iVRy_VR，2026 年 8 月

有用户报告：iVRy 里最高只能选 1920×864，他用 JPEG 能选到 2400×1080 但很卡，H.265 不能超过 1920×864。而且他说 **2-3 年前可以选 2400×1080，现在不行了**，H.264 还会直接黑屏。

**对你的影响**：S6 Edge+ 是 2560×1440，如果只能出 1920×864，画面会明显比原生模糊。

**但这条要打问号**：那个用户是骁龙 680 的中端机，限制可能来自设备性能而非 iVRy 本身。**你的情况未知。**

---

### 坑 5：Gear VR 系统文件已停止维护，可能卡更新循环
> 【官方】iVRy GearVR System Files Installer 页面说明

原文大意：2024 年 6 月起，三星和 Meta 不再为 Gear VR 提供可用的软件更新。除了恢复出厂设置的设备需要重装软件之外，**Gear VR 的软件更新器本身有 bug，会让已经装好软件的设备失效**。

**对你的影响**：你的 Gear VR 现在还能用（能进界面、能跑 Skybox），说明暂时没问题。**但一旦它开始卡更新循环，只有这个 Installer 能救。**

**对策**：现在就把 `iVRy GearVR System Files Installer` 装进 Steam 库，不用也放着。

---

### 坑 6：连接断断续续
> 【实测】Reddit r/iVRy_VR，10 天前

有用户报告：同一个 Wi-Fi 下，连上、断、连上、断，反复循环。

**对策**：优先走 USB（USB 网络共享），别用 Wi-Fi。这也是官方推荐的（"Wired connections for greater visual quality and performance"）。

---

## 四、没有公开案例的部分（必须说清楚）

**以下这些我给不了保证，标为未知：**

| 未知项 | 说明 |
|---|---|
| S6 Edge+ / Exynos 7420 能否跑通 | 搜遍中英文，**没有一份公开的成功或失败记录** |
| Android 7.0 上的兼容性 | iVRy 最低要求正好是 7.0，是"卡在及格线上" |
| 是否需要降级 Oculus | 坑 2 是 S9+ 的记录，S6 那一代是否一样，未知 |
| Lite 版现在限制多久 | 【实测】官方说"5 分钟后降低饱和度"，但 2026 年有用户说"现在是不是改成 10 分钟后直接关掉 SteamVR 了"——**说法不一** |
| 分辨率能不能上原生 | 见坑 4，有争议 |
| 3DoF 漂移程度 | 老手机陀螺仪精度下降，可能转头会慢慢跑偏 |

**一句话**：路径是通的，但你走的这一段没人留下路标。

---

## 五、建议的验证顺序（三段，每段都能退）

### 第一段：只验证"能不能连上"，不装 VRChat
- 装 SteamVR + iVRy Driver + GearVR DLC
- 手机开 USB 调试、开 OSIG 开关
- 推送 app，插进 Gear VR，启动 SteamVR
- **看到 SteamVR Home 出现在头显里 = 成功**
- 预计：25 分钟。**这一段失败就停，不要往下走。**

### 第二段：验证"晕不晕、清不清楚"
- 在 SteamVR Home 里待 5 分钟，转转头、走走
- 判断三件事：画面跟不跟得上、字看不清、眼睛累不累
- 顺便感受一下手机温度
- **这一段的结论决定值不值得继续**

### 第三段：装 VRChat，做一次 15 分钟的实测
- 进人少的 world
- 全程计时，到点了主动停

---

## 六、备选方案对比

| 方案 | 适配 Gear VR | 免费 | 维护状态 | 备注 |
|---|---|---|---|---|
| **iVRy** | ✅ 官方专门支持，有 GearVR DLC | Lite 免费 / Premium ¥31 | **活跃**（2026-10 还在更新） | 首选 |
| Trinus | ✅ 老版本支持 | 免费版单次 10 分钟 | 半停滞 | 备选 |
| PhoneVR | ❌ 偏 Cardboard | 完全免费开源 | **停更多年** | 不考虑 |
| ALVR | ✅ 但要配 GearVR Controller | 免费开源 | 活跃（主攻 Quest） | 教程针对 S9+，步骤更绕 |

---

## 七、来源清单

| 内容 | 来源 | 类型 |
|---|---|---|
| 完整流程、OSIG 要求、顺序要求 | Steam 商店页 `iVRy for SteamVR (GearVR/Oculus App Installer)`（app/1487090） | 【官方】 |
| Gear VR 系统文件失效说明 | Steam 商店页 `iVRy GearVR System Files Installer`（app/3260890） | 【官方】 |
| 开发者选项开启方式（点 6 次）、三个开关的作用 | gearvr.net/developer-mode（The Gear VR Codex） | 【实测】专业站 |
| DLC 排查、手动安装 exe、APK 侧载路径 | Steam 社区 app/992490 讨论区，iVRy 开发者回复（2021） | 【官方】 |
| Oculus 降级 3.27.0.6.963、开关会自动关 | github.com/mgrng/alvr_samsung_s9_tutorial | 【实测】 |
| 分辨率上限 1920×864 | Reddit r/iVRy_VR（2026-08） | 【实测】 |
| 连接断断续续 | Reddit r/iVRy_VR（近期） | 【实测】 |
| 5.7 寸是 Gear VR 最佳搭配 | CNET / Android Authority 等多家 Gear VR 评测 | 【实测】 |
| S6 Edge+ 玩 AltspaceVR 两年少过热 | XDA 论坛 Gear VR 版块（2017） | 【实测】 |
