# 魏建兴

男｜20岁｜本科｜157-1584-1972｜w15715841972@163.com  
求职方向：嵌入式软件开发实习生 / 嵌入式 Linux 实习生

## 教育背景

**杭州电子科技大学｜电子信息工程｜本科**　2023.09 - 2027.06  
主修课程：嵌入式系统设计、数字电路、模拟电路、通信原理、C语言、FPGA应用技术

## 实习经历

**奇创未来科技公司｜嵌入式软件开发实习生**　2025.12 - 至今  
参与企业级手持设备嵌入式软件开发，覆盖 STM32F103 MCU 固件、RK3562 Android 板端服务、RK-MCU 串口协议、CAN/UART/UDP/MAVLink 通信链路、Bootloader/IAP 升级、板端部署、硬件联调与问题定位。

## 项目经历

### 手持发射器 RK3562 + STM32F103 控制链路开发

**平台 / 技术：** STM32F103RCT6、RK3562 Android、C、FreeRTOS、UART、CAN、SocketCAN、MAVLink、DroneCAN、Android init、Bootloader、IWDG、Shell脚本、APK/P401联调

- 主要负责 STM32F103 MCU 侧底层控制固件，基于 FreeRTOS 实现按键/电源状态机、IWDG 看门狗、GPS、QMA6100P 加速度计、CH224 充电检测和 PB1 电压采样；支持 ON/OFF 短按电量显示、长按开关机、低电保护、软关机静默、充电闪烁显示，以及通过 RK3562 `PWRON` 脉冲配合息屏/唤醒/关机相关联调。
- 设计并实现 RK3562 与 MCU 的 UART 运行时通信协议，采用 `0xA5 + type + seq + len + TLV + CRC16-CCITT` 帧格式，支持 `HELLO`、状态摘要、GPS 状态、`COMMAND`、`ACK/ERROR` 和 `ENTER_BOOTLOADER` 命令，RK 侧可解析电量、充电、看门狗、电源和接触状态。
- 设计并实现 MCU Bootloader 与 UART-IAP 升级流程，将 APP 区迁移至 `0x08004000`，支持 RK3562 通过 `/dev/ttyS3` 触发升级；Bootloader 检查 MSP/Reset_Handler 合法性后设置 VTOR/MSP 跳转 APP，升级时使用 `IAP1 + size/crc32/base` 头、页擦写、CRC32 校验和首 APP 页延后写入降低异常中断风险。
- 参与 RK3562 用户态支撑服务开发与部署，配合 CommRouter / PeripheralManager / LinkController 打通 APK UDP、P401 TUN UDP、UART MAVLink、外设 UDS 和 `can1` SocketCAN 链路，实现 MAVLink 消息路由、按键 COMMAND_LONG 映射、链路可达检测和日志验证。
- 参与 P401 图传链路无感快速配对方案开发，基于天地端 `tun` 网卡 MAC 和 CAN/DroneCAN 自定义报文实现 MAC 上报、ACK 确认和去重处理，并在 RK3562 地面端完成 `can1` 配置、启动上报、模拟天空端报文注入和 ACK 验证。
- 使用 logcat、tcpdump、串口日志、SocketCAN、ADB、示波器/逻辑分析仪等工具定位串口收发、CAN 通信、服务自启动、按键上报和 APK 显示链路问题。

### RK3588 IMX415 摄像头 1080p60 适配与 3A 问题定位

**平台 / 技术：** RK3588 Linux、IMX415、V4L2、Device Tree、MIPI CSI-2、RKISP、GStreamer、H.265/RTP、systemd

- 基于现有 IMX415 V4L2 sensor driver 适配 `1920x1080@60fps` RAW10 模式，修改并验证 mode table、VTS/HTS、`vertical_blanking`、`pixel_rate` / `link_frequency` 等 sensor timing 参数。
- 复核并调试 RK3588 摄像头设备树链路，覆盖 I2C 地址、MCLK、reset/power GPIO、MIPI data lanes、endpoint 连接，以及 IMX415 -> DPHY -> CSI2 -> RKCIF -> RKISP -> `/dev/video11` media graph。
- 分层验证 RAW `/dev/video0`、ISP/NV12 `/dev/video11`、MPP H.265 编码和 RTP/UDP 图传路径，使用 `media-ctl`、`v4l2-ctl`、dmesg 和抓帧统计定位链路问题。
- 定位 3A/IQ 配置导致的黑屏问题：确认 `rkaiq_3A` 运行时将 `vertical_blanking` 从稳定值 `1170` 改写为 `46` 后影响 ISP/NV12 输出，并编写 systemd guard 与验证脚本固定已验证控制参数。

## 专业技能

- **嵌入式 Linux / Android：** 熟悉 Linux/Android 用户态板端服务开发、进程管理、网络配置、ADB 调试、logcat/dmesg 日志分析，具备设备树、V4L2 sensor driver 修改、Camera bring-up 与 BSP 适配经验。
- **MCU & RTOS：** 熟练使用 C 进行 STM32 固件开发，熟悉 FreeRTOS、IWDG、UART-IAP Bootloader、电源/按键状态机、低电保护、GPIO/ADC/I2C/UART/CAN 外设开发与板级调试，了解 C++。
- **通信协议：** 熟悉 UART、CAN、SocketCAN、UDP、MAVLink、DroneCAN/libcanard，具备 RK-MCU 协议设计、状态上报、命令 ACK、CAN MAC 交换、链路抓包与问题定位经验。
- **工具链与辅助：** 熟悉 Git、VS Code、Keil MDK、STM32CubeMX、Makefile、ADB、MobaXterm、tcpdump、candump、cansend、v4l2-ctl、GStreamer、示波器、逻辑分析仪；能使用 AI 辅助代码检索、日志分析和技术文档整理。
