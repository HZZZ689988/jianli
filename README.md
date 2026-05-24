# 魏建兴

男｜20岁｜本科｜157-1584-1972｜w15715841972@163.com  
求职方向：嵌入式软件开发实习生

## 教育背景

**杭州电子科技大学｜电子信息工程｜本科**　2023.09 - 2027.06  
主修课程：嵌入式系统设计、数字电路、模拟电路、通信原理、C语言、FPGA应用技术

## 实习经历

**奇创未来科技公司｜嵌入式软件开发实习生**　2025.12 - 至今  
参与企业级手持设备嵌入式软件开发，覆盖 STM32F103 MCU 固件、RK3562 Android 板端服务、RK-MCU 串口协议、CAN/UART/UDP/MAVLink 通信链路、Bootloader/IAP 升级、硬件联调与问题定位。

## 项目经历

### 手持发射器 RK3562 + STM32F103 控制链路开发

**平台 / 技术：** STM32F103RCT6、RK3562 Android、C、FreeRTOS、UART、CAN、MAVLink、Android init、Bootloader、IWDG、APK联调

- 主要负责 STM32F103 MCU 侧固件开发，配合硬件工程师完成板级功能联调，覆盖 FreeRTOS 任务框架、IWDG 看门狗、开关机控制、GPS、加速度计、电池电压、剩余电量和充电状态采集。
- 设计 RK3562 与 MCU 的 UART 运行时通信协议，采用 `0xA5 + type + seq + len + TLV + CRC16-CCITT` 帧格式，实现 MCU 状态摘要上传，并支持 RK 侧解析后在 APK 显示。
- 设计并实现 MCU Bootloader 与 UART-IAP 升级流程，将 APP 区迁移至 `0x08004000`，支持 RK3562 通过 `/dev/ttyS3` 触发固件升级，完成升级握手、擦写、CRC 校验和复位流程。
- 参与 RK3562 侧通信链路开发，配合 CommRouter / PeripheralManager / LinkController 等板端服务，完成 UART、CAN、GPIO 按键、APK、P401 网络链路之间的 MAVLink 消息路由与联调验证。
- 使用 logcat、tcpdump、串口日志、SocketCAN、ADB、示波器/逻辑分析仪等工具定位串口收发、CAN 通信、服务自启动、按键上报和 APK 显示链路问题。

### RK3588 IMX415 摄像头驱动适配与 3A 问题定位

**平台 / 技术：** RK3588 Linux、IMX415、V4L2、Device Tree、MIPI CSI-2、RKISP、GStreamer、H.265/RTP、systemd

- 基于现有 IMX415 V4L2 sensor driver 进行 1080p60 适配排查，修改并验证 sensor timing、mode table、`vertical_blanking` 等关键参数，将链路从 30fps 调试至 60fps 工作状态。
- 参与 RK3588 摄像头链路验证，覆盖 IMX415 sensor、MIPI CSI-2、RKCIF、RKISP、RAW `/dev/video0`、NV12 `/dev/video11`、MPP H.265 编码和 RTP/UDP 图传路径。
- 定位 3A/IQ 配置导致的黑屏问题，通过 RAW/NV12 抓帧统计、v4l2-ctl、media graph、dmesg 和运行时 control 对比，确认 rkaiq_3A 改写 sensor control 后影响 ISP/NV12 输出。
- 编写 systemd guard 与验证脚本，固定已验证的 sensor control 参数，辅助完成启动后恢复、异常状态复现和链路稳定性验证。

## 专业技能

- **嵌入式 Linux / Android：** 熟悉 Linux/Android 用户态板端服务开发、进程管理、网络配置、ADB 调试、logcat/dmesg 日志分析，具备设备树、V4L2 sensor driver 修改与 BSP 适配经验。
- **MCU & RTOS：** 熟练使用 C 进行 STM32 固件开发，熟悉 FreeRTOS、IWDG、UART-IAP Bootloader、GPIO/ADC/I2C/UART/CAN 外设开发与板级调试，了解 C++。
- **通信协议：** 熟悉 UART、CAN、SocketCAN、UDP、MAVLink、DroneCAN/libcanard，具备 RK-MCU 协议设计、状态上报、命令 ACK、链路抓包与问题定位经验。
- **工具链与辅助：** 熟悉 Git、VS Code、Keil MDK、STM32CubeMX、Makefile、ADB、MobaXterm、tcpdump、candump、cansend、v4l2-ctl、GStreamer、示波器、逻辑分析仪；能使用 AI 辅助代码检索、日志分析和技术文档整理。
