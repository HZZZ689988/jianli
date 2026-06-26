# 手持发射器项目面试优化笔记

## 一句话概括

这是一个 RK3562 Android 主控板 + STM32F103 MCU 的手持发射器控制链路项目。我的主责集中在 STM32F103 MCU 底层控制、RK-MCU UART 协议和 UART-IAP Bootloader，RK3562 侧参与 Android 用户态支撑服务、MAVLink 路由、SocketCAN 和 P401 图传配对链路联调。

## 项目分层

```text
APK / P401 / 网络链路
        |
RK3562 Android 用户态服务
  CommRouter / PeripheralManager / LinkController
        |
UART /dev/ttyS3、SocketCAN can1、UDP、UDS
        |
STM32F103 MCU
  FreeRTOS / 电源状态机 / 电量显示 / UART 协议 / Bootloader
```

## MCU 侧主责

可以重点讲四块：

1. 电源/按键状态机

- ON/OFF 按键做去抖。
- 短按显示电量。
- 关机态长按上电。
- 开机态长按通过 PC8 模拟 RK809 `PWRON` 长按，再进入关机流程。
- 低电时拒绝上电，并用最低电量灯快闪提示。
- 软关机状态下抑制重复运行日志，避免误以为系统还在正常开机态。

2. 电量和充电显示

- PB1 / `Vbus_ADC` 采样估算电量百分比。
- 四颗电量 LED 显示 1-4 档电量。
- CH224 检测到充电时，下一档 LED 闪烁。
- 电量显示加入平滑策略，避免 ADC 波动导致 LED 来回跳。
- 低电保护和充电状态会影响是否允许系统上电。

3. RK-MCU UART 协议

帧格式：

```text
SOF      0xA5
type     消息类型
seq      序号
len      payload 长度
payload  common header + TLV
crc16    CRC16-CCITT-FALSE，小端
```

payload 公共头：

```text
payload_version
flags
request_id
uptime_ms
TLV list
```

支持消息：

```text
HELLO
STATUS_SUMMARY
GPS_STATUS
COMMAND
ACK/ERROR
```

可上报字段包括 app 版本、git hash、电源状态、IWDG 状态、电量百分比、充电状态、接触检测、GPS 状态等。RK 侧可以发 `ENTER_BOOTLOADER` 命令让 MCU 复位进入升级窗口。

4. Bootloader / UART-IAP

- Bootloader 放在 `0x08000000`。
- APP 放在 `0x08004000`。
- APP 请求升级时写 backup register，然后 MCU 复位进入 Bootloader。
- Bootloader 等待 RK 通过 `/dev/ttyS3` 发送 `IAP1`。
- RK 再发送 16 字节 header：`size + crc32 + base + reserved`。
- Bootloader 校验 base、size、reserved，擦除 APP 区，接收 APP bin。
- 写 flash 时先暂存首 APP 页，先写后续页，整包 CRC32 通过后再写首 APP 页。
- 成功后返回 `OK` 并复位；失败后继续留在 Bootloader 等待下一次升级。
- 正常启动前检查 APP 栈顶和 Reset_Handler 合法性，设置 VTOR/MSP 后跳转 APP。

## RK3562 侧参与内容

RK 侧不要讲成“我独立做完整 Android 系统”，更准确是：

> 我参与了 RK3562 用户态支撑服务和通信链路联调，重点是让 APK、P401、UART、CAN、GPIO/input 和本地服务之间的数据能按预期流转。

可以拆成三块：

1. `CommRouter`

- 通过 UDP 对接 APK。
- 通过 UDP 对接 P401 TUN。
- 通过 UART 读取 MAVLink telemetry。
- 通过 UDS 接收本地外设消息。
- 使用 `poll` 同时等待多个 fd。
- 根据 source 做 MAVLink 路由：
  - APK -> P401
  - P401 -> APK
  - UART_TELEM -> APK
  - LOCAL_BUTTON / LOCAL_CAN -> APK + P401

2. `PeripheralManager`

- 读取 `/dev/input/eventX` 按键事件。
- 做去抖、短按/长按识别。
- 按 JSON 配置映射成 MAVLink `COMMAND_LONG`。
- 通过 UDS 发给 `CommRouter`。
- 同时集成 CAN pairing 子模块。

3. P401 CAN-MAC 配对

- 地面端 RK3562 使用 `can1`。
- 读取本地 `tun` 网卡 MAC。
- 通过 CAN/DroneCAN 自定义 report 报文发送。
- 收到天空端 report 后解析 peer MAC，回 ACK。
- 重复 report 会继续 ACK，但不重复触发配对钩子。
- 当前真实 P401 pairing API 未确认，所以保持 `dry_run=true`。
- 地面端 report/ACK 已验证；天空端 MCP2515/XL2515 接收链路仍是硬件验证边界。

## 面试官可能追问

### 你这个项目的主责是什么？

回答：

> 我的主责在 STM32 MCU 侧，包括电源/按键状态机、电量/充电显示、RK-MCU UART 协议和 Bootloader/IAP。RK3562 侧我参与用户态服务和链路联调，能讲清 CommRouter、PeripheralManager、SocketCAN 和 P401 配对链路，但不把整个 Android 系统说成我独立完成。

### 你的 MCU 状态机难点在哪里？

回答：

> 难点是按键动作会影响整机电源状态，不能只按一次就翻转。要区分短按、长按、低电拒绝、开机态和关机态。开机态长按还要先通过 RK809 `PWRON` 给 RK 关机/睡眠机会，再切断系统供电。软关机后要避免后台任务继续打印大量运行日志，避免误判状态。

### UART 协议为什么不用直接发结构体？

回答：

> 直接发结构体有对齐、大小端、版本兼容问题。TLV 可以新增字段而不破坏旧解析器；CRC16 可以检测串口干扰或丢字节；seq 和 request_id 可以做请求响应对应；unknown TLV 可以按长度跳过。

### UART-IAP 怎么避免升级失败变砖？

回答：

> 首先 Bootloader 独立在 `0x08000000`，APP 在 `0x08004000`。Bootloader 正常跳转前会检查 APP 栈顶和 Reset_Handler。升级时先校验 header，再擦写 APP 区，接收完成后做 CRC32。关键是首 APP 页延后写入，只有整包校验通过才写首页，这样异常中断时更容易让 Bootloader 判断 APP 无效并继续等待升级。

### RK 侧你具体做了什么？

回答：

> 我参与的是用户态通信链路和服务联调。`CommRouter` 负责 APK、P401、UART、UDS 多来源 MAVLink 路由；`PeripheralManager` 把按键/input 和 CAN 事件转换为本地 MAVLink 消息；我还参与了 `can1` SocketCAN 配置、`candump/cansend` 验证、P401 `tun` MAC report/ACK 交换和服务日志排查。

## 不足和边界

- MCU 电量百分比依赖 PB1 / `Vbus_ADC` 的经验映射，严格 SOC 估算还需要电池曲线和实测校准。
- RK 侧服务是参与和联调，不要说完整 Android BSP、APK 和所有 daemon 都由自己独立完成。
- P401 CAN-MAC 配对目前是协议和地面端验证闭环，真实 vendor pairing API 与天空端 MCP2515 接收稳定性不能夸大。
- UART-IAP、看门狗、电源状态机要区分 host/build 验证和真实板端验证，面试中按已有证据说。

## 推荐简历表述

> 主要负责 STM32F103 MCU 侧底层控制固件，基于 FreeRTOS 实现按键/电源状态机、IWDG 看门狗、PB1 电压采样、四档电量显示、CH224 充电检测与 RK3562 `PWRON` 协同控制；设计 RK-MCU UART TLV 协议和 UART-IAP Bootloader，支持状态摘要上报、命令 ACK、RK 侧触发升级和 APP 合法性校验。

> 参与 RK3562 Android 用户态通信链路联调，配合 CommRouter / PeripheralManager 打通 APK UDP、P401 TUN UDP、UART MAVLink、UDS 外设事件和 `can1` SocketCAN 链路；参与 P401 图传无感配对地面端实现，通过 CAN/DroneCAN 自定义报文完成 `tun` MAC report/ACK、去重和 dry-run 验证。
