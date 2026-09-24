# Locus 1.0.3 (4) 汉化增强稳定优化版

[![Build unsigned IPA](https://github.com/Catalvic/Locus-1.0.3-optimized-build/actions/workflows/build-locus.yml/badge.svg)](https://github.com/Catalvic/Locus-1.0.3-optimized-build/actions/workflows/build-locus.yml)

免费、开源的 iPhone 系统级虚拟定位工具。可在地图上选点或搜索地点后传送位置，也包含摇杆、道路路线、手绘路线、GPX、收藏、最近使用、Live Activity、本地备份和中国大陆坐标修正。

> [!IMPORTANT]
> 本仓库是 **1.0.3 (4) · iOS 18.0+ · 普通未签名 IPA**。下载后必须使用你自己的证书完成签名，不能直接安装到普通 iPhone。它与 [iOS 16+ TrollStore 专版 1.2.2 (9)](https://github.com/Catalvic/Locus-iOS16-TrollStore-optimized-build) 是两条独立构建线，请按设备系统和安装方式选择。

## 下载

- **未签名 IPA：** [下载成功编译 Run 的 Artifact](https://github.com/Catalvic/Locus-1.0.3-optimized-build/actions/runs/35949299099/artifacts/10787568110)（需要登录 GitHub；如果 Artifact 已按保留策略过期，可在 Actions 重新运行工作流）
- **完整源码：** [下载 1.0.3 汉化增强稳定优化版源码 ZIP](https://github.com/Catalvic/Locus-1.0.3-optimized-build/raw/refs/heads/main/Locus_1.0.3_%E6%B1%89%E5%8C%96%E5%A2%9E%E5%BC%BA%E7%A8%B3%E5%AE%9A%E4%BC%98%E5%8C%96%E7%89%88_%E6%BA%90%E7%A0%81.zip)
- **编译记录：** [GitHub Actions Run 35949299099](https://github.com/Catalvic/Locus-1.0.3-optimized-build/actions/runs/35949299099) · `Success` · Xcode 26.6 · Release/arm64

Artifact 解压后的 IPA 文件名为 `Locus-1.0.3-optimized-unsigned.ipa`。它与本项目交付的中文文件名 `Locus_1.0.3_汉化增强稳定优化版_未签名.ipa` 内容相同。

## 1.0.3 升级了什么

本版本从公开汉化增强基线的 `1.0.2 (3)` 升级为 `1.0.3 (4)`。升级重点是减少连接卡顿、停止后位置回跳、配对文件损坏、VPN 误判以及路线和 GPX 边界问题。

### 1. 连接更快，也更不容易连错设备

- 优先尝试最近一次成功端口和标准端口 `49152`，失败后再自动发现 `_remotepairing._tcp` 动态端口。
- 所有候选端口都在当前 Tunnel IP 上做真实握手验证，降低同一局域网有多台 iPhone 时连错设备的概率。
- 隧道、RemoteXPC、定位写入和清除放到后台串行执行，减少地图和按钮长时间卡住。
- 修复 RemoteXPC 资源释放顺序，连接失败后能更干净地重新尝试。

### 2. 停止定位更可靠

- “暂停路线”“继续路线”和“停止定位”分别处理，不再混用同一状态。
- 路线、摇杆、健康检查和静止坐标重发都加入任务代次控制；点击停止后，旧任务不能再次写回虚拟位置。
- 移动时暂停旧的静止坐标重发，减少路线行进中突然跳回旧位置。
- 只有系统确认清除成功后才显示“已恢复真实 GPS”。清除失败或隧道掉线时仍保留停止入口，方便重连后再次清除。

### 3. 配对文件与 LocalDevVPN 检测更安全

- RPPairing 文件增加大小限制、plist 初检和 idevice 原生解析验证。
- 新配对文件先写入候选文件并设置 `0600` 权限；候选文件验证或写入失败时不会覆盖原有配对，验证通过后再原子替换。
- LocalDevVPN 只接受已启用的 `utun` 接口；默认配置只把 `10.7.0.x` 和 `10.7.1.x` 识别为兼容地址。
- 不再把 `10.7.2.x`、`10.7.99.x` 等其他 VPN 误判为 LocalDevVPN 已连接。
- 手机内配对进行时会暂时禁用关闭和下滑退出，避免重复启动不能安全取消的底层任务。

### 4. 路线与摇杆边界修复

- 路线按累计距离重采样，单段和多段路线统一最多 `30,000` 个采样点，并保留最终终点。
- 最多支持 `20` 个途经点；道路规划失败会显示真实 MapKit 错误，不再把起终点直线误报成道路路线。
- 修复国际日期变更线：`179° → -179°` 按约 2° 的短方向插值，不会绕地球一圈。
- 摇杆只有在定位会话确实建立后才显示开启，并增加极区除零保护和经度归一化。
- 新增交换起终点和距离过近提示。

### 5. GPX 导入导出加固

- 改用系统 `XMLParser`，文件最大 `8 MB`，轨迹点总数最多 `30,000`。
- 拒绝 NaN、Infinity、越界坐标、单点轨迹和所有坐标完全重复的轨迹。
- 优先选择点数最多的完整 `trkseg`，其次选择最长 `rte`，最后才使用 `wpt`。
- 不再强行连接多个彼此不连续的轨迹段或路线。
- 使用正确的 `.gpx` 文件类型和文件名；大文件的解析和导出放到后台执行，并显示进度。

### 6. 中文体验、无障碍和发布体积

- 首次设置、配对、VPN、权限、通知、设置和主要错误提示统一为简体中文。
- 首次设置可保存中断进度，并在 iOS 18–26 明确提示首次配对需要电脑。
- 主按钮会按状态显示“请先选点”“传送到此处”“正在连接”。
- 主要按钮和图标的触控区域至少为 44 pt，并补充 VoiceOver 名称。
- 停止、删除等危险操作使用深红底白字，静态对比度约 `6.61:1`。
- Live Activity 跟随系统明暗模式，同状态刷新间隔缩短到 5 秒，并在反向地理编码完成后主动刷新城市名称。
- Release 构建启用无用代码裁剪、Swift 符号裁剪、部署后处理和 dSYM 分离。

## 功能介绍

以下功能来自公开汉化增强基线，并在 1.0.3 优化版中保留：

| 功能 | 说明 |
| --- | --- |
| 地图选点与地点搜索 | 点地图、放置图钉或搜索地点，然后把系统位置传送到目标坐标 |
| 摇杆移动 | 步行、跑步、骑行、驾车四种速度模式，适合连续移动 |
| 道路路线 | 使用 MapKit 生成步行或驾车路线，可查看候选路线、距离和预计时间 |
| 手绘路线 | 直接在地图上绘制要运行的轨迹 |
| GPX | 导入已有 GPX 轨迹，或把当前路线导出为 GPX |
| 收藏与记录 | 收藏地点、最近使用、搜索历史 |
| 中国坐标修正 | 在中国大陆处理 Apple 地图 GCJ-02 与系统定位 WGS-84 的偏差 |
| Live Activity | 在锁屏和灵动岛显示当前定位或路线状态 |
| 后台状态 | 后台会话与状态处理、状态更新和掉线通知；实际持续时间受 iOS 后台策略限制，尚待真机验证 |
| 本地备份 | 备份和恢复收藏、搜索历史及安全的界面设置 |
| 地图源识别 | 识别 Apple/高德地图数据源，配合坐标修正 |
| 用户偏好 | 保存外观、选点方式、搜索历史和地图视角等设置 |

Locus 通过 Apple 开发者定位服务向 `locationd` 写入模拟坐标，属于系统级位置模拟。它不会修改其他 App，也不是游戏客户端或反检测工具。对开发者模拟位置有额外检测的 App 可能拒绝使用该位置。

## 安装前准备

1. 一台 **iOS 18.0 或更高版本**的 iPhone。
2. 用于侧载的个人证书或开发者证书，以及 Feather、SideStore、AltStore、Sideloadly 等兼容签名工具。
3. iOS 18–26 还需要一台电脑，用 `idevice_pair` 生成一次 RPPairing 文件。
4. 在 iPhone 上安装并连接 [LocalDevVPN](https://apps.apple.com/us/app/localdevvpn/id6755608044)。默认 Tunnel IP 通常为 `10.7.0.1`。

> [!WARNING]
> GitHub 下载的 IPA 没有 `LC_CODE_SIGNATURE`、`_CodeSignature` 或 `embedded.mobileprovision`。请先用自己的证书签名；GitHub 编译成功不等于已经签名或可以直接安装。

## 首次安装与配对

### 第一步：签名并安装 IPA

1. 下载并解压 GitHub Artifact。
2. 把 `Locus-1.0.3-optimized-unsigned.ipa` 导入你的签名工具。
3. 使用自己的证书签名并安装到 iPhone。
4. Bundle ID 为 `com.chrismack.locus`，建议保留原标识。确需修改时，签名工具必须同步重写主应用、嵌入的 Live Activity 扩展及相关签名标识，不能只修改主应用。

### 第二步：iOS 18–26 导入 RPPairing

1. 在电脑下载 [idevice_pair](https://github.com/jkcoxson/idevice_pair/releases)。
2. 用数据线连接 iPhone，解锁并在手机上选择“信任”。
3. 生成 **RPPairing** 文件。
4. 通过隔空投送、文件 App、系统分享页或剪贴板导入 Locus。

请使用 idevice_pair 生成的 RPPairing plist，不能误用 lockdown 配对文件或 SideStore 的 `.mobiledevicepairing` 文件。

> [!WARNING]
> RPPairing 是敏感配对凭据。只通过本地、可信方式传输和保存，不要上传到 GitHub、公共网盘或聊天群，也不要发给他人。使用剪贴板导入后请及时清除剪贴板内容。

### 第三步：iOS 27 手机内配对

源码已经提供手机内配对流程，但当前构建尚未完成 iOS 27 真机验证：

1. 打开 Locus，进入“设置 → 在这台 iPhone 上配对 → 开始配对”。
2. 允许本地网络权限，并保持 Locus 打开。
3. 前往“系统设置 → 隐私与安全性 → 开发者模式 → 与主机配对”。
4. 选择 Locus，先输入 iPhone 解锁密码，再输入 Locus 显示的 6 位配对码。

### 第四步：连接 LocalDevVPN

1. 打开 LocalDevVPN 并启动 VPN。
2. 回到 Locus，确认 Tunnel IP；默认通常是 `10.7.0.1`。
3. 第一次传送建议在 Wi-Fi 下进行。会话建立后再测试蜂窝网络、摇杆、路线和 GPX。

## 日常使用

### 单点传送

1. 在地图上点选目标，或搜索地点。
2. 确认底部显示的目标坐标。
3. 点击“传送到此处”。
4. 等待连接完成；成功后再打开地图或目标 App 检查位置。

### 摇杆

先完成一次传送并建立定位会话，再打开摇杆。选择步行、跑步、骑行或驾车速度，推动摇杆开始连续移动。若连接失败，摇杆不会显示为已开启。

### 道路路线和手绘路线

1. 进入路线页，设置起点、终点和途经点。
2. 选择步行或驾车道路路线，也可以切换为手绘轨迹。
3. 设定速度后开始路线。
4. “暂停路线”只暂停行进；要退出虚拟定位，请使用“停止定位”。

### GPX

在路线页选择导入 GPX，检查生成的轨迹后开始运行。需要保存当前路线时选择导出 GPX。超过 8 MB、超过 30,000 点或坐标无效的文件会被拒绝。

### 恢复真实位置

1. 在 Locus 中点击“停止定位”。
2. 等到页面明确显示系统已经清除模拟定位。
3. 如果清除失败或 VPN 已掉线，重新连接 LocalDevVPN，再次点击“停止定位”。
4. 必要时关闭并重新打开地图或目标 App，让它重新读取真实 GPS。

## LiveContainer 文件选择器无响应

本节只适用于选择在 LiveContainer 中运行 Locus 的用户。Live Activity、后台持续时间和扩展功能在该容器中的表现尚未真机验证。

可以任选一种方式导入 RPPairing：

1. 在 LiveContainer 长按 Locus → Settings → 开启 **Fix File Picker**，再重新导入。
2. 在文件 App 中通过系统分享页选择“打开到 LiveContainer → Locus”。
3. 复制 RPPairing plist 的完整内容，在 Locus 的首次设置或设置页选择“从剪贴板粘贴 RPPairing”。

## 常见问题

| 问题 | 处理方法 |
| --- | --- |
| IPA 点了不能安装 | 这是未签名 IPA，先用自己的证书和兼容工具签名 |
| 提示配对文件无效 | 重新用 idevice_pair 生成 RPPairing；不要导入 lockdown 或 `.mobiledevicepairing` 文件 |
| 提示 VPN 未连接 | 启动 LocalDevVPN，检查是否为有效 `utun`，确认 Tunnel IP；默认通常是 `10.7.0.1` |
| 一直正在连接 | 确认首次使用在 Wi-Fi 下、配对文件有效且 LocalDevVPN 已连接，然后重新尝试 |
| 路线规划失败 | 查看页面显示的 MapKit 错误，检查网络、起终点和出行方式 |
| GPX 无法导入 | 检查文件是否超过 8 MB/30,000 点，是否只有一个点、全部重复或存在非法坐标 |
| 停止后仍是虚拟位置 | 重连 LocalDevVPN 后再次点击“停止定位”，直到系统确认清除成功 |
| 某些游戏不接受位置 | 该 App 可能检测并拒绝 Apple 开发者模拟位置；Locus 不提供反检测或游戏客户端修改 |

## 构建和校验信息

| 项目 | 值 |
| --- | --- |
| 版本 | `1.0.3 (4)` |
| 最低系统 | `iOS 18.0` |
| Bundle ID | `com.chrismack.locus` |
| 架构 | 主应用和 Live Activity 扩展均为 `arm64` |
| 编译环境 | Xcode 26.6 · iPhoneOS 26.5 SDK · Release |
| 成功 Run | [`35949299099`](https://github.com/Catalvic/Locus-1.0.3-optimized-build/actions/runs/35949299099) |
| Artifact ZIP SHA-256 | `7712DA2B7426C9B421C529F160A5A152C15B5C0B484A722F5EEDB9CB0E803560` |
| IPA SHA-256 | `F3D5869CCAF3BBBA398B756CC831ED540FD28109F20D6A08DD89620DB5E9985C` |
| 源码 ZIP SHA-256 | `C15EABF1622BC3BE4703DE947E6E36BA63A6272A57630513BA5317FABF058B20` |

IPA 已完成 ZIP 完整性、路径安全、版本、Bundle ID、arm64 Mach-O、扩展嵌入和无签名状态检查。主应用与扩展均没有 `LC_CODE_SIGNATURE`，包内也没有 `_CodeSignature` 或 `embedded.mobileprovision`。

## 当前验证状态

### 已完成

- GitHub Actions 在 macOS/Xcode 上真实完成 Release 编译，日志以 `** BUILD SUCCEEDED **` 结束。
- 主应用和 Live Activity 扩展都成功链接，扩展已嵌入主应用。
- 源码语法、plist/YAML、XcodeGen target、版本号、路线边界和 GPX 接受/拒绝策略检查已通过。
- 下载后的 Artifact digest 与 GitHub 显示一致，IPA 哈希与工作流生成的 `.sha256` 一致。

### 尚未完成

- 使用个人或开发者证书签名。
- 在真实 iPhone 上安装、启动和授权。
- iOS 18、26、27 的定位、停止定位、路线、摇杆、GPX、后台运行和 Live Activity 实测。
- 动态端口变化、同网多台 iPhone、Wi-Fi/蜂窝切换和 iPad 文件导入导出测试。

> [!NOTE]
> 页面中的“已优化”和“已支持”表示对应源码已经实现并进入成功编译的 IPA，不表示已经在真实 iPhone 上体验验证。GitHub 编译成功也不等于已经完成签名或真机验证。

## 版本来源与官方 v1.0.3 的关系

本项目以 MIT 许可证公开源码 [`jzksnsjswkw/locus-ZH@5685ee3`](https://github.com/jzksnsjswkw/locus-ZH/commit/5685ee3456f4cc83e332b8dde1124a4988988431) 为基线。该基线保留简体中文、中国坐标修正、RPPairing、LocalDevVPN、Live Activity、备份、地图源识别、搜索历史、摇杆、路线和 GPX 等增强功能；本仓库在此基础上实现上述稳定性优化并把版本调整为 `1.0.3 (4)`。

这是一条汉化增强分支上的独立稳定优化构建，**不代表完整合并了 [`ChrisMack32/Locus` 官方 `v1.0.3`](https://github.com/ChrisMack32/Locus/releases/tag/v1.0.3)**。官方标签中的路线方向箭头和亚米级位置抖动设置没有进入本构建，因此本页面不会把它们列为已有功能。官方项目、汉化分支与本优化构建各自独立维护。

## 开源与隐私

- 许可证：MIT。
- idevice FFI 来源：[jkcoxson/idevice](https://github.com/jkcoxson/idevice)，MIT 许可证。
- 没有自建的分析统计或上传服务；MapKit 搜索和反向地理编码可能连接 Apple 服务。
- 本项目与 Apple、Mirage、Wapixel 及各游戏厂商无隶属关系。

如需自行构建，下载源码 ZIP 后运行 `xcodegen generate`，再使用 Xcode 或仓库内的 GitHub Actions 工作流构建。工作流产物仍是未签名 IPA，安装前必须使用你自己的证书签名。
