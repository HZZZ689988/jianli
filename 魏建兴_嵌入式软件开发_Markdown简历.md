# 魏建兴

男｜20岁｜本科｜157-1584-1972｜w15715841972@163.com  
求职方向：嵌入式软件开发实习生 / 嵌入式 Linux 实习生 / BSP 驱动开发实习生

## 教育背景

**杭州电子科技大学｜电子信息工程｜本科**　2023.09 - 2027.06  
主修课程：嵌入式系统设计、数字电路、模拟电路、通信原理、C语言、FPGA应用技术

## 实习经历

**奇创未来科技公司｜嵌入式软件开发实习生**　2025.12 - 至今  
参与企业级手持设备嵌入式软件开发，覆盖 STM32F103 MCU 固件、RK3562 Android 板端服务、RK-MCU 串口协议、CAN/UART/UDP/MAVLink 通信链路、Bootloader/IAP 升级、板端部署、硬件联调与问题定位。

## 项目经历

### 手持发射器 RK3562 + STM32F103 控制链路开发

**平台 / 技术：** STM32F103、RK3562 Android、C、FreeRTOS、UART、CAN、SocketCAN、MAVLink、DroneCAN、Bootloader、IWDG、Android init

- 主要负责 STM32F103 MCU 侧底层控制固件，基于 FreeRTOS 实现按键/电源状态机、IWDG 看门狗、PB1 电压采样、四档电量显示、CH224 充电检测、低电保护与 RK3562 `PWRON` 协同控制。
- 设计 RK3562 与 MCU 的 UART 运行时通信协议，采用 `0xA5 + type + seq + len + TLV + CRC16-CCITT` 帧格式，实现电量、充电、看门狗、电源状态等信息上报，并支持 RK 侧命令下发与 ACK 响应。
- 设计并实现 MCU Bootloader 与 UART-IAP 升级流程，将 APP 区迁移至 `0x08004000`，支持 RK3562 通过 `/dev/ttyS3` 完成 `IAP1` 握手、Flash 页擦写、CRC32 校验、APP 合法性检查和复位跳转。
- 参与 RK3562 用户态支撑服务联调，配合 `CommRouter` / `PeripheralManager` 打通 APK UDP、P401 TUN UDP、UART MAVLink、UDS 外设事件和 `can1` SocketCAN 链路，实现 MAVLink 消息路由、按键 `COMMAND_LONG` 映射和日志验证。
- 参与 P401 图传无感配对地面端开发，基于 `can1` 和 CAN/DroneCAN 自定义报文实现 `tun` MAC 上报、ACK 确认和去重处理，完成地面端启动上报、模拟天空端报文注入和 ACK 验证。

### RK3588 IMX415 摄像头 1080p60 适配与问题定位

**平台 / 技术：** RK3588 Linux、IMX415、V4L2、Device Tree、MIPI CSI-2、RKCIF、RKISP、GStreamer、H.265/RTP、systemd

- 基于现有 IMX415 V4L2 sensor driver 适配 `1920x1080@60fps` RAW10 模式，修改并验证 mode table、寄存器表、VTS/HTS、`vertical_blanking`、`pixel_rate`、`link_frequency` 和 MIPI lane 配置。
- 复核并调试 RK3588 摄像头设备树链路，覆盖 I2C 地址、MCLK、reset/power GPIO、MIPI data lanes、endpoint 连接，以及 IMX415 -> DPHY -> CSI2 -> RKCIF -> RKISP -> `/dev/video11` media graph。
- 使用 `media-ctl`、`v4l2-ctl`、dmesg 和 GStreamer 分层验证 RAW `/dev/video0`、ISP/NV12 `/dev/video11`、MPP H.265 编码和 RTP/UDP 图传路径。
- 定位 3A/IQ 配置导致的黑屏问题，确认 `rkaiq_3A` 运行时将 `vertical_blanking` 从稳定值 `1170` 改写为 `46` 后影响 ISP/NV12 输出，并编写 systemd guard 与验证脚本固定已验证控制参数。
- 更换 CAM2 方位摄像头后定位严重色差问题，通过测试源排除编码和接收端链路，结合 media graph 与 V4L2 bus format 判断为 Bayer 相位不匹配，并修改驱动中 `MEDIA_BUS_FMT_*` 配置恢复颜色显示。

## 专业技能

- **MCU / RTOS：** 熟悉 STM32F103 固件开发、FreeRTOS 任务组织、IWDG 看门狗、电源/按键状态机、低电保护、UART-IAP Bootloader、GPIO/ADC/I2C/UART/CAN 外设开发与板级调试。
- **嵌入式 Linux / Android：** 熟悉 Linux/Android 用户态板端服务开发、进程管理、Android init 服务、ADB 调试、logcat/dmesg 日志分析，具备设备树、V4L2 sensor driver 修改和 Camera bring-up 经验。
- **通信协议：** 熟悉 UART、CAN、SocketCAN、UDP、MAVLink、DroneCAN/libcanard，具备 RK-MCU 协议设计、TLV 状态上报、命令 ACK、CAN MAC 交换、链路抓包与问题定位经验。
- **音视频 / Camera：** 熟悉 V4L2 取帧流程、media graph、RAW10 Bayer、NV12、MIPI CSI-2、RKCIF/RKISP、GStreamer、H.265/RTP 推流和基础 ISP/AIQ 问题定位。
- **工具链：** 熟悉 Git、VS Code、Keil MDK、STM32CubeMX、Makefile、ADB、MobaXterm、tcpdump、candump、cansend、v4l2-ctl、media-ctl、GStreamer、示波器和逻辑分析仪。
