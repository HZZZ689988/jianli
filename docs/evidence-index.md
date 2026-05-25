# 简历项目证据索引

这份索引用于面试前复盘，不建议把原始公司源码、完整日志、内网地址和板端账号直接公开。公开简历中只保留可解释的技术事实；更完整的材料放在本地私有项目仓库中备查。

## 项目一：手持发射器 RK3562 + STM32F103 控制链路

### 简历主张

- 参与企业级手持设备嵌入式软件开发，覆盖 STM32F103 MCU、RK3562 Android 板端服务、RK-MCU 通信协议、CAN/UART/UDP/MAVLink 链路、Bootloader/IAP、硬件联调。
- 参与 RK3562 侧 `CommRouter`、`PeripheralManager`、`LinkController` 等服务，完成 UART、CAN、GPIO、APK、P401 网络链路之间的 MAVLink 消息路由和联调验证。
- 使用 `logcat`、`tcpdump`、串口日志、SocketCAN、ADB、示波器/逻辑分析仪等工具定位链路问题。

### 本地证据位置

- `D:\handle_fire_software\handle_fire_software_link\README.md`
  - 说明该仓库是 RK3562 Android support-service workspace。
  - 明确服务名：`CommRouter`、`PeripheralManager`、`LinkController`。
  - 记录板端部署、板端构建、Python 单元测试入口。

- `D:\handle_fire_software\handle_fire_software_link\docs\COMM_ROUTER_REPO_MAP.md`
  - 记录 `CommRouter` 当前源码、`PeripheralManager` MAVLink/CAN bridge、`LinkController` QoS/链路检测脚本。
  - 记录 `APK_UDP`、`UART_TELEM`、`LINK_CONTROLLER`、`LOCAL_PERIPHERAL`、`LOCAL_CAN` 等端点类型。
  - 记录 MAVLink route matrix 和去重策略。

- `D:\handle_fire_software\handle_fire_software_link\docs\project-status.md`
  - 记录板端手动链路验证、CAN 状态、GPIO 按键、APK 通信、P401 UART 和服务链路阶段性结论。
  - 可支撑“参与通信链路联调、MAVLink 路由、CAN/GPIO/APK 联调、板端验证”的说法。

- `D:\handle_fire_software\handle_fire_software_link\docs\verification\rk3562-android-services-bringup.md`
  - 记录更完整的 RK3562 服务 bring-up 和验证过程。

- `D:\handle_fire_software\apk\shooter-apk\README.md`
  - 记录 Android 端 H.265 RTP、MAVLink、BBox OSD、UDP 端口、Compose/MediaCodec 等应用侧信息。

### 面试中可说的证据点

- `CommRouter` 不是单纯转发 demo，而是围绕多来源 MAVLink 做 endpoint 标记、route matrix 和下行去重。
- `PeripheralManager` 覆盖 CAN/GPIO 节点和 MAVLink-CAN bridge，属于板端外设管理和消息桥接角色。
- `LinkController` 覆盖链路可达性检测、QoS 或网络侧辅助控制。
- 板端验证不只看代码编译，还包括 Android init 服务状态、ADB/logcat、tcpdump、SocketCAN、串口收发和 APK 显示。

### 需要谨慎表达

- 不要把整个 RK3562 Android 系统、APK、所有服务都说成独立完成。
- MCU Bootloader/IAP 如果被追问，应结合你的实际源码和板端记录说明；公开简历仓库不放完整升级协议和公司代码。
- 涉及真实设备 IP、公司项目代号、固件包、日志原文时，面试中只解释技术过程，不展示敏感信息。

## 项目二：RK3588 IMX415 摄像头 1080p60 适配与 3A 问题定位

### 简历主张

- 基于现有 IMX415 V4L2 sensor driver 适配 `1920x1080@60fps` RAW10 模式。
- 修改并验证 mode table、VTS/HTS、`vertical_blanking`、`pixel_rate` / `link_frequency` 等 sensor timing 参数。
- 调试 RK3588 设备树链路，覆盖 I2C、MCLK、GPIO、MIPI lanes、endpoint、DPHY、CSI2、RKCIF、RKISP。
- 分层验证 RAW `/dev/video0`、NV12 `/dev/video11`、H.265/RTP 图传路径。
- 定位 `rkaiq_3A` 改写 `vertical_blanking` 导致 ISP/NV12 黑帧的问题。

### 本地证据位置

- `C:\Users\w1571\Desktop\imx415\handoff\codex_imx415_work\docs\tasks\imx415-highfps-status.md`
  - 状态记录：`implemented: yes`、`hardware-validated: yes`、`done: yes`。
  - 记录 `1920x1080@60`、`vts_def=0x08CA`、`hts_def=0x0226 * IMX415_4LANES * 2`。
  - 记录 RAW `/dev/video0`、NV12 `/dev/video11` 600 frame 验证通过。

- `C:\Users\w1571\Desktop\imx415\handoff\codex_imx415_work\docs\verification\imx415-highfps-bringup.md`
  - 记录 RK3588-Tronlong 板端、kernel source、boot partition、构建命令、刷写命令和验证命令。
  - 记录 RAW/NV12 测试结果和 `60.00 fps` 验证。

- `C:\Users\w1571\Desktop\imx415\handoff\codex_imx415_work\docs\evidence\2026-05-13-rkaiq-forces-vblank-black.md`
  - 记录 `rkaiq_3A` 启动前后 controls 对比。
  - 已验证稳定状态：`vertical_blanking=1170`、`exposure=676`、`analogue_gain=80`。
  - 异常状态：`rkaiq_3A` 将 `vertical_blanking` 改为 `46`，NV12 Y plane 全 0、UV 约 128，表现为黑帧。

- `C:\Users\w1571\Desktop\imx415\docs\rk3588-imx415-1080p60-porting-guide.md`
  - 记录 1080p60 移植说明、driver mode notes、DTS routing notes、build/deploy/validation 命令。

- `C:\Users\w1571\Desktop\imx415\docs\interview-imx415-rk3588-camera-bringup.md`
  - 记录面试口径、camera link 基础概念、驱动构成、问题定位过程和常见追问。

### 面试中可说的证据点

- 1080p60 不是简单改 fps 字段，而是 mode table、VTS/HTS、vblank、pixel rate、link frequency、MIPI lane 和寄存器表要一致。
- 当前稳定 timing：`VTS = 0x08CA = 2250`，`height = 1080`，所以 `vertical_blanking = 1170`。
- RAW 正常但 NV12 黑，说明 sensor 到前端采集路径和 ISP/AIQ 输出路径要分开排查。
- `rkaiq_3A` 是用户态 3A/ISP 调参服务，会根据 IQ 文件控制曝光、增益、vblank 和 ISP 参数；当前 IQ/AIQ 与 1080p60 linear mode 不匹配。

### 需要谨慎表达

- 不要说“从零写了 IMX415 驱动”。准确表述是：基于 Rockchip/现有 IMX415 驱动做板级适配、模式修正、链路调试和稳定性处理。
- 不要说“彻底解决了 AIQ 算法”。准确表述是：定位到 AIQ/IQ 配置不匹配导致黑帧，当前采用关闭不匹配 AIQ + 固定 controls 的工程规避方案，长期应补齐匹配 IQ 文件。
- 如果被问到 1080p90，只说曾探索过但板端出现 CSI/MIPI 错误，最终以 1080p60 作为验证目标。

## 简历证据使用方式

面试前建议按下面顺序准备：

1. 先背熟两个项目的一句话概括。
2. 再分别画出手持项目通信链路图和 IMX415 camera pipeline。
3. 准备 3 个“我怎么定位问题”的故事：CAN/GPIO/APK 链路、P401/UART/MAVLink 路由、IMX415 RAW 正常但 NV12 黑帧。
4. 准备 3 组关键数字：UART 协议帧格式、IMX415 `VTS/vblank`、RTP/MAVLink 默认端口。
5. 对所有“参与/配合/负责”的边界保持一致，不在面试现场临时夸大。
