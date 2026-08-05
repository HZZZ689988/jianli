# 简历项目证据索引

这份索引用于面试前复盘，不建议把原始公司源码、完整日志、内网地址和板端账号直接公开。公开简历中只保留可解释的技术事实；更完整的材料放在本地私有项目仓库中备查。

## 项目一：手持发射器 RK3562 + STM32F103 控制链路

### 简历主张

- 负责 STM32F103 MCU 侧底层控制固件，覆盖 FreeRTOS 任务、电源/按键状态机、RK3562 `PWRON` 协同控制、电量/充电显示、IWDG 看门狗、UART 运行时协议和 UART-IAP Bootloader。
- 参与企业级手持设备嵌入式软件开发，覆盖 STM32F103 MCU、RK3562 Android 板端服务、RK-MCU 通信协议、CAN/UART/UDP/MAVLink 链路、Bootloader/IAP、硬件联调。
- 参与 RK3562 侧 `CommRouter`、`PeripheralManager`、`LinkController` 等服务，完成 UART、CAN、GPIO、APK、P401 网络链路之间的 MAVLink 消息路由、按键 COMMAND_LONG 映射和联调验证。
- 参与 P401 图传无感快速配对链路开发，基于天地端 `tun` 网卡 MAC 和 CAN/DroneCAN 自定义报文完成地面端 MAC 上报、ACK、去重和 dry-run 配对验证。
- 使用 `logcat`、`tcpdump`、串口日志、SocketCAN、ADB、示波器/逻辑分析仪等工具定位链路问题。

### 本地证据位置

- `D:\WORK\CAN_TEST1\CAN_TEST_1\README.md`
  - 记录 STM32F103 控制固件的系统视图：RK3562 `/dev/ttyS3` 对 MCU USART2、USART1 debug log、CAN/DroneCAN、CH224、QMA6100P、PB1 电量采样、四颗电量 LED 和电源 GPIO。
  - 记录两阶段固件布局：Bootloader `0x08000000`、APP `0x08004000`、RK UART-IAP 升级和 `IAP1` 协议。
  - 记录电源键行为：软关机、2s 长按切换、短按电量显示、长按 LED 动画、CH224 充电闪烁显示。

- `D:\WORK\CAN_TEST1\CAN_TEST_1\Core\Src\power_ctrl.c`
  - 记录 MCU 侧电源/按键状态机：去抖、短按电量显示、关机态长按上电、开机态长按触发 RK3562 `PWRON` 低电平保持，再进入关机流程。
  - 记录低电保护：外部电源不允许时拒绝上电并触发低电量提示。

- `D:\WORK\CAN_TEST1\CAN_TEST_1\Core\Src\battery_led.c`
  - 记录 PB1 / `Vbus_ADC` 采样、电量百分比估算、四档 LED 显示、CH224 充电检测、下一格闪烁、低电快闪和显示平滑逻辑。

- `D:\WORK\CAN_TEST1\CAN_TEST_1\Core\Src\rk_uart_proto.c`
  - 记录 RK-MCU UART runtime protocol：USART2 中断接收队列、状态机解析、`0xA5 + type + seq + len + payload + CRC16-CCITT`、TLV 组包、`HELLO`、`STATUS_SUMMARY`、`GPS_STATUS`、`COMMAND`、`ACK/ERROR`。
  - 记录 `ENTER_BOOTLOADER` 命令如何置位 bootloader 请求，再由 APP 复位进入 Bootloader。

- `D:\WORK\CAN_TEST1\CAN_TEST_1\Bootloader\Src\bootloader_main.c`
  - 记录 Bootloader 合法 APP 检查：MSP 必须在 SRAM 范围内且 4 字节对齐，Reset_Handler 必须在 APP flash 范围内且 Thumb bit 有效。
  - 记录 UART-IAP 流程：等待 `IAP1`、校验 16 字节 header、页擦除、写入 APP、CRC32 校验、首 APP 页延后写入、失败后继续等待升级，成功后复位。
  - 记录跳转 APP 前关闭 SysTick/NVIC、设置 `SCB->VTOR`、加载 MSP 并跳转 Reset_Handler。

- `D:\WORK\CAN_TEST1\mcu_link\README.md`
  - 记录 RK3562 与 STM32F103 共享 UART 协议库和 RK 侧 monitor / Android native daemon 原型。
  - 记录协议常量、TLV 语义、RK `/dev/ttyS3` 独占访问和 Health HAL 电量集成方向。

- `D:\WORK\handle_fire_software\handle_fire_software_link\README.md`
  - 说明该仓库是 RK3562 Android support-service workspace。
  - 明确服务名：`CommRouter`、`PeripheralManager`、`LinkController`。
  - 记录板端部署、板端构建、Python 单元测试入口。

- `D:\WORK\handle_fire_software\handle_fire_software_link\docs\COMM_ROUTER_REPO_MAP.md`
  - 记录 `CommRouter` 当前源码、`PeripheralManager` MAVLink/CAN bridge、`LinkController` QoS/链路检测脚本。
  - 记录 `APK_UDP`、`UART_TELEM`、`LINK_CONTROLLER`、`LOCAL_PERIPHERAL`、`LOCAL_CAN` 等端点类型。
  - 记录 MAVLink route matrix 和去重策略。

- `D:\WORK\handle_fire_software\handle_fire_software_link\docs\project-status.md`
  - 记录板端手动链路验证、CAN 状态、GPIO 按键、APK 通信、P401 UART 和服务链路阶段性结论。
  - 可支撑“参与通信链路联调、MAVLink 路由、CAN/GPIO/APK 联调、板端验证”的说法。

- `D:\WORK\handle_fire_software\handle_fire_software_link\docs\verification\rk3562-android-services-bringup.md`
  - 记录更完整的 RK3562 服务 bring-up 和验证过程。

- `D:\WORK\handle_fire_software\apk\shooter-apk\README.md`
  - 记录 Android 端 H.265 RTP、MAVLink、BBox OSD、UDP 端口、Compose/MediaCodec 等应用侧信息。

- `D:\WORK\CAN_TEST1\.codex_gcs_support_lf\docs\can_p401_mac_pairing.md`
  - 记录 P401 `tun` MAC 通过 CAN/DroneCAN 链路交换的地面端实现。
  - 记录地面端默认配置：`can1`、本地节点 `2`、对端节点 `1`、MAC report DTID `8195/0x2003`、ACK DTID `8196/0x2004`、result DTID `8197/0x2005`、启动上报 5 帧、`dry_run=true`。
  - 记录扩展 CAN 帧格式：`CAN ID = priority(7) << 24 | DTID << 8 | source_node_id`，payload 前 6 字节为 `tun` MAC，第 7 字节为 MAC 类型，第 8 字节为 DroneCAN 单帧 tail byte。
  - 记录地面端模拟天空端报文注入、ACK 日志和 `candump -L can1` 验证方式。

- `D:\WORK\CAN_TEST1\.codex_gcs_support_lf\comm-routerd\src\main.cpp`
  - 记录 RK3562 侧 `comm-routerd` 的 UDP/UDS/UART 多 fd `poll` 事件循环。
  - 记录 APK、P401 TUN、UART telemetry、LocalButton、LocalCan 等 source 分类和路由规则：APK/本地外设上行到 P401，P401/UART/本地外设下行到 APK。

- `D:\WORK\CAN_TEST1\.codex_gcs_support_lf\peripheral-managerd\src\main.cpp`
  - 记录 RK3562 侧 `peripheral-managerd` 读取 input 事件，做去抖和短按/长按识别，按 JSON 映射构造 MAVLink `COMMAND_LONG`，通过 UDS 发给 `comm-routerd`。
  - 记录 `peripheral-managerd` 集成 CAN pairing 子模块，统一处理按键和 CAN-MAC 配对事件。

- `D:\WORK\CAN_TEST1\.codex_gcs_support_lf\peripheral-managerd\src\can_pairing.cpp`
  - 记录 CAN pairing 的 SocketCAN raw socket、扩展帧过滤、MAC report/ACK 发送、重复 report 去重、`dry_run=true` 配对钩子边界。

- `D:\WORK\CAN_TEST1\docs\project-status.md`
  - 记录 RK3562 地面端和 RK3588 天空端 CAN 链路 bring-up、MAC 交换、接触状态机、MCP2515/XL2515 接收链路问题和验证边界。
  - 记录地面端 `can1` 为 RK3562 SoC CAN，500 kbit/s，`sample-point 0.868`，`restart-ms 100`，由 `PeripheralManager` 等服务配置。
  - 记录地面端 MAC pairing 已部署并验证启动上报、模拟天空端 `07200301` 报文后的 `07200402` ACK，以及 `dry_run=true skip_pairing` 日志。
  - 记录天空端外置 MCP2515-over-SPI `can0` 接收链路存在错误状态、错误解码或 RX 为 0 的硬件/电气/采样边界，自动天空端服务和高频自动 TX 仍需谨慎。

- `D:\WORK\CAN_TEST1\rk3588_rk3562_can_mac_issue_summary_zh.md`
  - 汇总 MAC 交换协议、天地端脚本路径、手动 MAC 交换成功片段和当前阻塞点。
  - 明确 MAC 交换协议和脚本基本可用，但当前工程重点应放在天空端外部 CAN 接收链路的硬件恢复、电气测量和真实接收验证。

### 已核对的关键参数（2026-08-05）

#### STM32F103RCT6 UART-IAP A/B

| 项目 | 确认值 | 证据边界 |
| --- | --- | --- |
| Flash 布局 | Bootloader `0x08000000`，16 KiB；Slot A `0x08004000`；Slot B `0x08021800`；单槽 `0x1D800`，即 118 KiB | 链接脚本和 Bootloader 源码已确认 |
| metadata | `0x0803F000`、`0x0803F800` 两页；页大小 2 KiB；Flash 结束 `0x08040000` | 双页交替写，magic 最后写入 |
| 启动策略 | IAP 等待窗口 120 s；试启动上限 3 次；APP 健康确认延迟 10 s | A/B 构建和主机验证通过，尚无 A/B 真机闭环证据 |
| UART | USART2 对 RK3562 `/dev/ttyS3`，`115200 8N1` | 当前源码配置 |
| 升级头 | 魔术 `IAP1`；16 字节小端头：`size/crc32/base/reserved`；`reserved` 必须为 0 | 只允许写非活动槽；活动槽和已有 pending 均会拒绝 |
| CRC32 | 初值 `0xFFFFFFFF`；反射多项式 `0xEDB88320`；结果异或 `0xFFFFFFFF` | 标准 reflected CRC32 |
| 传输行为 | RK updater 默认 512 B burst；阶段响应 `READY -> RX -> ERASED -> OK`；没有逐块 ACK | `OK` 在整包 CRC 校验通过后返回 |
| 构建产物 | Bootloader BIN 7,052 B；Slot A/B BIN 58,468 B | 仅表示构建产物大小，不等于真机验证 |

历史单槽 UART-IAP 有板端传输证据；当前 A/B 实现已经完成构建和主机侧校验，但文档仍标记 `hardware-validated: no`。面试时必须把这两层证据分开。

#### MCU 电源、电量和运行时 UART

| 项目 | 确认值 |
| --- | --- |
| 按键时序 | 去抖 50 ms；短按上限 500 ms；长按关机 2 s；手势窗口 3 s |
| RK 电源协同 | RK809/PWRON 短脉冲 250 ms；关机长按模拟 6 s |
| 电量采样/显示 | ADC 周期 1 s；日志 5 s；充电 LED 闪烁 500 ms；短按显示 5 s；低电提示 2 s；快闪半周期 100 ms |
| ADC 标定 | 3.3 V、12 bit、满量程 4095；raw `2792 -> 0%`，`3392 -> 100%`；四档 LED |
| 电量状态 | 档位变化需持续 10 min；低电进入 1%，退出 10% |
| 电压保护边界 | `APP_BATTERY_PROTECT_VOLTAGE_ENABLE=0`；预留 7.1 V 阈值未启用，不能说成已验证保护 |
| runtime UART 帧 | `0xA5 + type + seq + 1-byte len + payload + CRC16`；最大 payload 128 B |
| 接收行为 | 接收队列 128 B；每次 poll 最多取 64 B；HELLO/STATUS/GPS 默认各 1 Hz |
| 消息类型 | HELLO `0x00`、PING `0x01`、STATUS `0x02`、GPS `0x04`、COMMAND `0x08`、ACK/ERROR `0x0B` |
| 命令/TLV | `ENTER_BOOTLOADER` command ID 为 5；TLV type 为 16 bit，可携带 APP/git、电源、IWDG、电量、充电、接触/GPS 等状态 |

#### RK3562 服务端口

| 链路 | 地址/端口 |
| --- | --- |
| CommRouter APK endpoint | `127.0.0.1:14566` |
| APK peer | `127.0.0.1:14550` |
| UART_TELEM / LINK_CONTROLLER | `14561` / `14562` |
| LOCAL_PERIPHERAL / LOCAL_CAN | `14563` / `14564` |
| LinkController local | `0.0.0.0:14565` |
| P401 peer | `192.168.144.66:14550` |
| MAVLink 去重 | 500 ms；key 为 `sysid/compid/msgid/payload CRC32`，忽略 seq |
| 视频 | H.265/RTP，UDP `5600` |

#### P401 自动配对当前协议

- 当前实现位于 `D:\WORK\CAN_TEST1\p401-auto-pair`，不能再只按早期 `8195/8196` DTID 简化协议描述。
- 地面端约 10 Hz 发送 `07200702#5041495254524947`，payload 为 ASCII `PAIRTRIG`；天空端 READY 为 `07200501`，地面 READY ACK 为 `07200502`，之后继续完成八帧身份交换以及 DONE/DONE_ACK。
- CAN 配置为 500 kbit/s、normal、`ERROR-ACTIVE`；天空端必须观察到 SocketCAN `clock 12500000`。
- 快速版本普通协议间隔和接收轮询均约 50 ms；READY ACK 次数为配置项，当前快速包装器默认 1 次。完成 `PASS_P401_PAIR` 后停止 10 Hz 触发。
- `BACKUP_ENABLE=0` 是当前默认值；clear 后 commit 失败会落到 `FAIL_NO_ROLLBACK`。天空端 systemd 失败重启间隔为 5 s。
- “3 秒内自动配对”目前仍是目标口径，文档明确要求健康接触状态下多轮验证，不能表述成已稳定通过。

### 面试中可说的证据点

- MCU 侧不是单纯外设 demo，而是有完整按键/电源状态机：去抖、短按显示电量、长按开关机、低电拒绝上电、RK `PWRON` 脉冲协同和软关机静默。
- RK-MCU UART 协议有明确二进制帧和 TLV schema，状态摘要中能携带 app/git、电源、IWDG、电量、充电、接触检测等字段，并支持 RK 命令进入 Bootloader。
- UART-IAP 有可讲的安全点：APP 偏移、向量表合法性检查、CRC32、先写非首 APP 页、成功后首 APP 页生效、失败后继续停留在升级窗口。
- `CommRouter` 不是单纯转发 demo，而是围绕多来源 MAVLink 做 endpoint 标记、route matrix 和下行去重。
- `PeripheralManager` 覆盖 CAN/GPIO 节点和 MAVLink-CAN bridge，属于板端外设管理和消息桥接角色。
- `LinkController` 覆盖链路可达性检测、QoS 或网络侧辅助控制。
- 板端验证不只看代码编译，还包括 Android init 服务状态、ADB/logcat、tcpdump、SocketCAN、串口收发和 APK 显示。
- P401 无感配对不是单纯手动配置 MAC；早期版本通过 CAN/DroneCAN 自定义 DTID 交换 `tun` MAC，当前版本已演进为约 10 Hz `PAIRTRIG`、READY/ACK、八帧身份交换和 DONE/DONE_ACK 状态机。
- 地面端 RK3562 `can1` 的 MAC 上报和 ACK 链路已经能通过模拟天空端报文和 `candump`/日志验证；天空端自动化和长期稳定性要结合 MCP2515 外部接收链路状态谨慎表述。

### 需要谨慎表达

- 不要把整个 RK3562 Android 系统、APK、所有服务都说成独立完成。
- MCU Bootloader/IAP 如果被追问，应结合你的实际源码和板端记录说明；公开简历仓库不放完整升级协议和公司代码。
- 涉及真实设备 IP、公司项目代号、固件包、日志原文时，面试中只解释技术过程，不展示敏感信息。
- P401 CAN-MAC 配对不要说“全自动量产闭环已完全稳定”或“3 秒目标已经多轮通过”。准确表达是：参与协议演进和天地端联调，已有触发、身份交换与配对脚本；健康接触状态下的多轮时延和稳定性仍需继续验证。

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

## 项目三：AGS STM32F103CB CAN A/B 固件升级

### 本地证据位置

- `D:\WORK\AGS_ctrl_board\firmware\Common\Inc\ab_boot.h`
  - 定义 128 KiB Flash 的 Bootloader、A/B slot、reserved 和双 metadata 布局。
- `D:\WORK\AGS_ctrl_board\docs\design\rk-mcu-ab-firmware-update.md`
  - 记录 RK 文件服务端、DroneCAN `BeginFirmwareUpdate` / `File.Read`、AGUP 包和状态机设计。
- `D:\WORK\AGS_ctrl_board\docs\verification\mcu-ab-firmware-update.md`
  - 记录构建、主机测试、板端验证范围和仍未关闭的异常场景。
- `D:\WORK\AGS_ctrl_board\docs\evidence\2026-08-03-mcu-ab-10-round-and-slot-limit-hardware-pass.md`
  - 记录 6 轮普通包和 4 轮满槽包交替升级、CAN 恢复探测及实际 `File.Read` 次数。

### 已核对的关键参数

| 项目 | 确认值 |
| --- | --- |
| Flash 布局 | Bootloader `0x08000000`，16 KiB；Slot A `0x08004000`，52 KiB；Slot B `0x08011000`，52 KiB；reserved `0x0801E000`，6 KiB |
| metadata | `0x0801F800`、`0x0801FC00`；每页 1 KiB；record 64 B；Flash page 1 KiB |
| 试启动 | trial boot count 为 1；健康确认窗口 2 s；IWDG 停喂上限 5 s |
| CAN/节点 | 500 kbit/s；MCU node ID 42；RK/file server node ID 120 |
| 升级传输 | DroneCAN `BeginFirmwareUpdate + File.Read`；单次 capacity 256 B；严格顺序 offset |
| 超时重试 | 请求间隔 20 ms；响应超时 1.5 s；退避 100 ms；最多 5 次；完成后延迟 500 ms reset |
| AGUP 包头 | 32 B little-endian；magic `AGUP` / `0x50554741`；version 1；目标槽 A=0/B=1；包含 image size、image CRC32、image version、flags、header CRC32 |
| CRC32 | 初值 `0xFFFFFFFF`；反射多项式 `0xEDB88320`；结果异或 `0xFFFFFFFF` |

### 硬件验证结果与边界

- 普通包约 32,984 B，理论 129 次 `File.Read`；6 轮实测分别为 `130/130/131/130/129/130`。
- 满槽 image 53,248 B，AGUP 53,280 B，理论 209 次 `File.Read`；4 轮实测分别为 `210/210/210/211`。
- 10 轮 A/B 交替升级完成，最终 Slot A version 4；升级后完成 150/150 ping/pong CAN 恢复验证，证据标记 `hardware-validated: yes`。
- 整体任务仍标记 `done: no`：未覆盖升级中断电、不健康 trial 回滚、文件服务中断、多节点总线负载、签名、反回滚和生产授权。
- 不把“约 20 秒”写成固定升级耗时。现有日志中的更新态约为普通包 8 个、满槽包 12 个 1 Hz 观测周期，另有 10 s post-probe，统计口径不同。

## 项目四：RK3588 系统构建、外置 CAN 与 Wi-Fi 模块

### RK3588 外置 MCP2515/XL2515 CAN

- 证据位于 `D:\WORK\CAN_TEST1\docs\evidence`，包括时钟修正构建、受控部署、开机自启和 DTS pin 审计记录。
- 实板晶振为 25 MHz，驱动上报 SocketCAN `clock 12500000`；旧设备树错误配置 16 MHz，对应 `clock 8000000`。mcp251x 驱动会将输入 oscillator 除以 2。
- 错误设备树下用 320 kbit/s 配置时，按 `320 * 25 / 16` 会接近实际 500 kbit/s，因此曾出现“320 kbit/s 反而可解码”的诊断现象。
- 修正后目标为 bitrate 500000、sample point 0.680、clock 12500000；控制器位于 `spi0.1`，中断为 GPIO3_D2，reset 原理图为 GPIO0_C7。
- 修正内核为 `5.10.160-rt77-g6f3eebb34bd6`；boot.img 为 38,653,440 B，SHA256 以 `38f893af` 开头、`7755af` 结尾。
- 时钟配置问题已经关闭，但后续物理 CAN 稳定性仍受触点/收发器路径影响，不能说成“改时钟后所有 CAN 问题彻底解决”。

### rootfs 与整机镜像

- 工程位于 `D:\WORK\RK3588\rk3588_sdk_work_rootfs`，标准入口为 `./build.sh rootfs`。
- 旧板 vendor Qt 5.15.8 文件约 116 MB；Debian Qt plugin 约 8.4 MB；purge simulation 涉及 22 个 GUI Qt 包。现有证据不支持“rootfs 缩减约 500 MB”。
- 当前典型 rootfs image 为 3,602,907,136 B；完整 update.img 常见约 3.72 GB。10-entry 固件反向解包逐项一致，rootfs `e2fsck -fn` 五阶段通过。
- no-Qt rootfs 证据标记 `hardware-validated: yes`、`done: yes`；真机验证覆盖无 Qt、ROS pub/sub、Wi-Fi、SSH、IMX415 60 帧、MPP H.265 和 UDP 5600。
- SD 升级曾因过早拔卡造成 ext4 损坏，重新制作并完整等待写入后复验通过；该案例可用于说明升级介质完整性和落盘时序。

### RTL8188FU

| 项目 | 确认值 |
| --- | --- |
| 工程/USB ID | `D:\WORK\RK3588\rtl8188ftv`；`0bda:f179` |
| 内核模块 | 3,888,888 B；vermagic `5.10.160-rt77-g6f3eebb34bd6-dirty SMP mod_unload aarch64` |
| 联网结果 | 扫描到 27 个 AP；PHY 72.2 Mbit/s；DHCP `192.168.101.105/24`；网关和公网 ping 0% loss |
| 吞吐边界 | UDP receive 曾达 19.98 Mbit/s、loss 0.012%；UDP transmit 曾仅 1.26 Mbit/s；10 min 稳定测试 TCP TX 6.223、RX 12.344、双向合计 13.641 Mbit/s |

因此不使用“TCP/UDP 均超过 20 Mbit/s”的统一口径。更准确的说法是：完成外部 Wi-Fi 模块移植、联网、吞吐和稳定性验证，并定位到收发方向不对称与网络可达性问题。

## 简历证据使用方式

面试前建议按下面顺序准备：

1. 先背熟两个项目的一句话概括。
2. 再分别画出手持项目通信链路图和 IMX415 camera pipeline。
3. 准备 3 个“我怎么定位问题”的故事：CAN/GPIO/APK 链路、P401/UART/MAVLink 路由、IMX415 RAW 正常但 NV12 黑帧。
4. 准备 3 组关键数字：UART 协议帧格式、IMX415 `VTS/vblank`、RTP/MAVLink 默认端口。
5. 对所有“参与/配合/负责”的边界保持一致，不在面试现场临时夸大。
