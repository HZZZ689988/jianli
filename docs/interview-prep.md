# 嵌入式软件实习面试备战提纲

## 简历主线

你的简历主线应稳定在：

> 嵌入式 Linux / Android 板端服务 + MCU/RTOS 固件 + 硬件链路联调。

两个项目的分工：

- 手持发射器项目：体现真实产品工程能力，重点是 MCU 固件、RK-MCU 协议、Bootloader/IAP、RK3562 板端服务和通信链路联调。
- IMX415 项目：体现嵌入式 Linux 深度，重点是 V4L2 sensor driver、设备树、MIPI/ISP 链路、内核镜像打包刷写和 AIQ 问题定位。

不建议把自己包装成纯算法、纯 Android App 或纯后端方向。更合适的岗位关键词：

- 嵌入式软件开发实习生
- 嵌入式 Linux 实习生
- BSP / 驱动开发实习生
- Android 系统/板端开发实习生
- MCU/RTOS 开发实习生
- Camera bring-up / 音视频底层实习生

## 30 秒自我介绍

我是杭州电子科技大学电子信息工程本科大三学生，目前有一段嵌入式软件实习经历，主要做 MCU 固件、RK3562 Android 板端服务和 RK3588 Linux camera bring-up。项目里我接触比较多的是 STM32/FreeRTOS、UART/CAN/MAVLink 通信、Bootloader/IAP、Android init 服务、Linux 设备树、V4L2 sensor driver 和板端问题定位。相比单纯写应用，我更熟悉从硬件接口、驱动配置、系统服务到日志抓包验证这一类嵌入式链路调试。

## 项目一：手持发射器

### 一句话概括

这是一个 RK3562 Android SoC + STM32F103 MCU 的手持设备控制链路项目，我主要参与 MCU 固件、RK-MCU 串口协议、Bootloader/IAP，以及 RK3562 板端服务、CAN/UART/UDP/MAVLink 链路和 P401 图传无感配对联调。

### 2 分钟讲法

这个产品里 STM32F103 负责采集和控制更贴近硬件的状态，比如按键、传感器、电池状态、开关机控制和部分外设状态；RK3562 运行 Android，负责板端服务、APK 显示和网络链路。两边通过 UART 通信，我参与设计了运行时帧格式，包含帧头、类型、序号、长度、TLV payload 和 CRC16 校验，用于状态摘要上传和命令交互。

MCU 侧我主要做 FreeRTOS 任务组织、电源/按键状态机、IWDG、GPIO/ADC/I2C/UART/CAN 外设相关逻辑，以及 UART-IAP Bootloader。电源部分不是简单读按键，而是区分短按和长按：短按显示四档电量，长按在关机态上电，在开机态通过 PC8 模拟 RK809 `PWRON` 长按流程让 RK3562 进入关机/睡眠相关流程，再切断系统供电；低电时会拒绝上电并用最低电量灯快速闪烁提示。电量显示通过 PB1 ADC 采样估算百分比，CH224 检测到充电时会做下一格闪烁提示，并加入显示平滑避免电量灯频繁跳变。

UART 协议部分，MCU APP 运行时通过 USART2 对接 RK 的 `/dev/ttyS3`，协议是 `0xA5 + type + seq + len + payload + CRC16-CCITT`，payload 里再放版本、flags、request_id、uptime 和 TLV。它支持 `HELLO`、`STATUS_SUMMARY`、`GPS_STATUS`、`COMMAND` 和 `ACK/ERROR`，RK 可以看到 MCU 电源状态、IWDG 状态、电量、充电、接触检测等字段，也可以下发 `ENTER_BOOTLOADER`。Bootloader 部分把 APP 区放到 `0x08004000`，由 RK3562 通过串口触发升级，流程包括 APP 写备份寄存器请求、复位进入 Bootloader、`IAP1` 握手、16 字节头校验、页擦写、CRC32 校验、设置 VTOR/MSP 并跳转 APP。

RK3562 侧我参与了 `CommRouter`、`PeripheralManager`、`LinkController` 相关联调。`CommRouter` 通过 `poll` 同时处理 APK UDP、P401 TUN UDP、UART MAVLink 和本地 UDS 外设消息，根据来源把 MAVLink 路由到 APK 或 P401；`PeripheralManager` 读取 input 按键、做短按/长按判断并构造 `COMMAND_LONG`，同时管理 `can1` 上的 CAN-MAC 配对；`LinkController` 做链路检测和网络侧辅助。后续我还参与了 P401 图传链路无感快速配对方案，基于天地端 `tun` 网卡 MAC，通过 CAN/DroneCAN 自定义报文做 MAC report、ACK 和去重，地面端在 `can1` 上完成启动上报、模拟天空端报文注入和 ACK 验证。调试时主要用 ADB、logcat、tcpdump、SocketCAN、串口日志、示波器和逻辑分析仪定位问题。

### 可追问点

- UART 协议为什么要有 `seq` 和 CRC？
- TLV 格式相比固定结构体有什么好处？
- MCU 开关机状态机怎么区分短按、长按、低电拒绝上电？
- 为什么 MCU 要通过 PC8/RK809 `PWRON` 脉冲配合 RK 息屏/唤醒/关机？
- 电量显示为什么要做显示平滑和低电保护？
- Bootloader 为什么要把 APP 放到 `0x08004000`？
- UART-IAP 如何处理断电或传输出错？
- FreeRTOS 任务之间如何避免阻塞？
- CAN `ERROR-ACTIVE`、`ERROR-PASSIVE` 表示什么？
- Android init 服务如何配置和验证？
- `tcpdump`、`candump`、`logcat` 分别解决哪类问题？
- P401 图传为什么要做 MAC 无感配对？
- CAN/DroneCAN MAC report 和 ACK 报文如何设计？
- 地面端验证通过和天空端自动化未完全验证的边界是什么？
- MCP2515 进入 `ERROR-PASSIVE` 或错误解码时怎么分层定位？

### 回答边界

可以说：

> 我主要负责 MCU 侧电源/按键状态机、状态采集、RK-MCU UART 协议和 Bootloader/IAP，参与 RK3562 板端服务和 P401/CAN 链路联调。

> P401 无感配对中，我参与了 CAN-MAC 交换协议和地面端链路实现/验证，能解释 report/ACK 帧格式、SocketCAN 调试和天空端 MCP2515 接收链路的硬件边界。

不要说：

> 整个 Android 系统、APK、全部板端服务和 MCU 都是我独立完成。

> P401 配对已经在所有自动化、反复接触/断开和高频天空端 TX 场景完全稳定。

## 项目二：RK3588 IMX415

### 一句话概括

这是 RK3588 Linux 平台上的 IMX415 摄像头 1080p60 bring-up 项目，我基于已有 V4L2 sensor driver 适配 1080p60 模式，调试设备树 media graph、MIPI/CSI/RKCIF/RKISP 链路，并定位到 `rkaiq_3A` 改写 `vertical_blanking` 导致 NV12 黑帧的问题。

### 2 分钟讲法

IMX415 通过 I2C 配置寄存器，通过 MIPI CSI-2 输出 RAW10 数据。RK3588 侧链路是 IMX415 -> DPHY -> CSI2 -> RKCIF -> RKISP，最终 RAW 节点用于前端验证，NV12 节点用于编码和显示。

我做的不是从零写驱动，而是在现有 IMX415 V4L2 sensor driver 基础上适配当前板子的 1920x1080@60fps 模式。驱动里重点关注 mode table、寄存器表、VTS/HTS、`vertical_blanking`、`pixel_rate`、`link_frequency` 和 MIPI lane 配置。当前稳定 timing 是 `VTS=0x08CA=2250`，有效高度是 1080，所以 `vertical_blanking=1170`。

设备树方面，我复核 IMX415 的 I2C 地址、MCLK、reset/power GPIO、MIPI lanes 和 remote endpoint，确保 media graph 能从 sensor 接到 DPHY、CSI2、RKCIF、RKISP。调试时用 `media-ctl` 看链路，用 `v4l2-ctl` 分别抓 RAW `/dev/video0` 和 NV12 `/dev/video11`，再结合 dmesg 和 GStreamer 推流验证。

遇到黑帧时，我分层排查：先用测试源证明 H.265/RTP 和接收端链路没问题，再看 RAW 是否非零，发现 RAW 正常但 NV12 黑。继续对比 `rkaiq_3A` 启停前后的 V4L2 controls，发现 AIQ 会把 `vertical_blanking` 从 1170 改成 46，导致 ISP/NV12 输出异常。当前方案是禁用不匹配的 AIQ，并用 systemd guard 固定已验证 controls；长期应该补齐匹配 1080p60 linear 模式的 IQ 文件。

### 可追问点

- I2C 和 MIPI CSI-2 在摄像头里分别做什么？
- sensor probe 成功为什么不等于出图成功？
- RAW10 Bayer 和 NV12 有什么区别？
- DPHY、CSI2、RKCIF、RKISP 分别是什么？
- VTS、HTS、vblank、exposure 的关系是什么？
- 为什么 `vertical_blanking=46` 会造成问题？
- `rkaiq_3A` 和 IQ 文件是什么？
- 为什么 RAW 正常但 NV12 黑说明问题可能在 ISP/AIQ？
- 为什么修改设备树后只需要重新打包 DTB/boot 镜像？
- boot、U-Boot、kernel Image、DTB、rootfs 的关系是什么？

### 必背数字

```text
/dev/video0: RAW path
/dev/video11: ISP/NV12 path
目标模式: 1920x1080@60fps
格式: RAW10 -> ISP -> NV12
VTS: 0x08CA = 2250
height: 1080
vertical_blanking: 1170
异常 vblank: 46
稳定 controls: vblank=1170, exposure=676, analogue_gain=80
```

### 回答边界

可以说：

> 我基于已有 IMX415 驱动做模式适配、设备树链路调试、板端验证和问题定位。

不要说：

> 我从零完整开发了 IMX415 sensor driver 或者彻底解决了 AIQ 算法。

## 常见综合问题

### 你最有含金量的能力是什么？

回答方向：

> 我能把嵌入式问题按链路拆开定位。比如 camera 项目里，我没有只看最终黑屏，而是从 sensor I2C、MIPI、RAW、ISP、AIQ、编码、网络和接收端逐段排除；手持项目里也是从 MCU、UART、CAN、Android 服务、APK 和网络链路逐段抓日志验证。

### 你和纯应用开发有什么不同？

回答方向：

> 我的工作更靠近板端和硬件接口，除了写业务代码，还要看设备树、内核日志、串口/CAN/网络抓包、示波器/逻辑分析仪和板端服务状态。很多问题不是代码语法错误，而是硬件连接、时序、服务启动顺序或多进程链路状态不一致。

### 你还有哪些短板？

回答方向：

> 内核驱动和 Camera/ISP 体系还在继续深入。比如 IMX415 项目中我已经能理解 V4L2 sensor driver、设备树和基本 timing，但 AIQ/IQ 文件调校属于更专业的 ISP 参数领域，当前项目采用工程规避，后续希望补齐匹配 IQ 配置和更系统的驱动层保护。

## 面试前检查清单

- 能手画手持项目链路：STM32F103 -> UART -> RK3562 -> CommRouter/PeripheralManager/LinkController -> APK/P401/CAN。
- 能手画 IMX415 链路：IMX415 -> DPHY -> CSI2 -> RKCIF -> RKISP -> RAW/NV12。
- 能解释 `0xA5 + type + seq + len + TLV + CRC16-CCITT`。
- 能解释 Bootloader/IAP 的地址划分和升级流程。
- 能解释 `VTS = height + vertical_blanking`。
- 能解释为什么 AIQ 会影响曝光、增益、vblank 和 ISP 输出。
- 能说清楚哪些是你主责，哪些是参与联调。
- 准备好 2 到 3 个具体问题定位案例，不要只背概念。
- 能讲 P401 MAC 配对链路：地面 `can1` / 天空 `can0`，`tun` MAC report/ACK，通过 `candump` 和服务日志验证，并说明天空端 MCP2515 接收链路仍是硬件验证边界。
