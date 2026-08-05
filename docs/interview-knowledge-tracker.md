# 嵌入式面试知识掌握表

使用方式：

- `掌握程度`：填 `0-5`。`0`=没学，`1`=听过，`2`=能复述一点，`3`=能完整回答，`4`=能结合项目回答，`5`=能抗追问。
- `状态`：填 `未学`、`在学`、`待复盘`、`已掌握`、`重点薄弱`。
- `最近复盘`：填日期，例如 `2026-06-01`。
- `备注/卡点`：写你答不上来的点，后续我会优先围绕这里提问。

## 一、项目表达

| 模块 | 知识点 | 面试要求 | 项目关联 | 掌握程度 | 状态 | 最近复盘 | 备注/卡点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 项目表达 | 30 秒自我介绍 | 能说清方向、技术栈、项目主线 | 两个项目总览 |  | 未学 |  |  |
| 项目表达 | 手持发射器 2 分钟讲法 | 能讲架构、主责、难点、结果 | RK3562 + STM32F103 | 2 | 在学 | 2026-06-26 | 已补项目笔记：重点讲 MCU 电源/按键状态机、RK-MCU UART TLV 协议、UART-IAP、RK 用户态服务链路和 P401 CAN-MAC 边界。 |
| 项目表达 | IMX415 2 分钟讲法 | 能讲 camera pipeline、调试路径、黑帧定位 | RK3588 + IMX415 |  | 未学 |  |  |
| 项目表达 | 负责边界 | 能区分主责、参与、未完成 | 简历防追问 |  | 未学 |  |  |
| 项目表达 | 问题定位故事 | 能讲 2-3 个具体排查案例 | UART/CAN/APK、RAW/NV12 黑帧 |  | 未学 |  |  |

## 二、C 语言基础

| 模块 | 知识点 | 面试要求 | 项目关联 | 掌握程度 | 状态 | 最近复盘 | 备注/卡点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| C 语言 | 内存分区 | text/data/bss/heap/stack，MCU 链接脚本 | Bootloader/APP 地址、任务栈 |  | 未学 |  |  |
| C 语言 | 内存对齐 | 结构体 padding、跨平台通信风险 | RK-MCU 不直接发结构体 |  | 未学 |  |  |
| C 语言 | 大小端 | 多字节字段传输和解析 | UART/TLV/CRC 字段解析 |  | 未学 |  |  |
| C 语言 | `volatile` | 防编译优化，不保证原子性 | 寄存器、中断共享标志 |  | 未学 |  |  |
| C 语言 | 原子性/临界区 | 共享变量保护、关中断、mutex | ring buffer、状态结构体 |  | 未学 |  |  |
| C 语言 | 指针/数组 | 指针运算、数组退化、越界 | 协议 payload、buffer |  | 未学 |  |  |
| C 语言 | 函数指针 | callback、ops 表、handler 分发 | V4L2 ops、协议 type 分发 |  | 未学 |  |  |
| C 语言 | `memcpy`/`memmove` | 重叠内存区别，能手写 | buffer 处理 |  | 未学 |  |  |
| C 语言 | 编译链接 | `.elf`/`.bin`、map、链接地址 | IAP 固件、APP 偏移 |  | 未学 |  |  |

## 三、MCU / STM32 / FreeRTOS

| 模块 | 知识点 | 面试要求 | 项目关联 | 掌握程度 | 状态 | 最近复盘 | 备注/卡点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| STM32 | 启动流程 | 向量表、MSP、Reset_Handler、main | Bootloader 跳 APP |  | 未学 |  |  |
| STM32 | 中断机制 | NVIC、优先级、ISR 不能阻塞 | UART/CAN 接收 |  | 未学 |  |  |
| STM32 | GPIO/ADC/I2C/UART/CAN | 基本用途和调试方式 | 按键、电池、sensor、通信 |  | 未学 |  |  |
| STM32 | IWDG/WWDG | 喂狗策略、任务心跳 | 系统稳定性 |  | 未学 |  |  |
| STM32 | 电源/按键状态机 | 短按、长按、低电保护、RK PWRON 协同 | 开关机、息屏/唤醒、电量显示 | 2 | 在学 | 2026-06-26 | 新增项目核心：能讲去抖、短按电量显示、长按开关机、低电拒绝、软关机静默和 RK809/PWRON 时序。 |
| STM32 | Flash 擦写 | 页擦除、写入粒度、校验 | UART-IAP |  | 未学 |  |  |
| Bootloader | Flash 分区 | Bootloader/APP 地址划分 | APP `0x08004000` |  | 未学 |  |  |
| Bootloader | 跳转 APP | 检查 MSP/Reset_Handler、VTOR、关中断 | UART-IAP |  | 未学 |  |  |
| Bootloader | IAP 流程 | 握手、擦写、分包、CRC、复位 | RK3562 触发升级 |  | 未学 |  |  |
| Bootloader | 断电保护 | 状态标记、CRC、A/B 分区 | 防变砖 |  | 未学 |  |  |
| FreeRTOS | 任务状态 | Running/Ready/Blocked/Suspended | 任务拆分 |  | 未学 |  |  |
| FreeRTOS | 优先级调度 | 抢占、时间片、任务饥饿 | 通信/控制/日志任务 |  | 未学 |  |  |
| FreeRTOS | 队列 | 任务间传数据 | 命令/状态消息传递 |  | 未学 |  |  |
| FreeRTOS | 信号量 | 事件通知、资源计数 | UART 接收完成通知 |  | 未学 |  |  |
| FreeRTOS | 互斥量 | 共享资源保护、优先级继承 | 共享串口/状态表 |  | 未学 |  |  |
| FreeRTOS | FromISR API | ISR 中不能阻塞，唤醒任务 | UART/CAN ISR |  | 未学 |  |  |
| FreeRTOS | 栈溢出 | hook、水位检测、HardFault | 任务稳定性 |  | 未学 |  |  |

## 四、通信协议

| 模块 | 知识点 | 面试要求 | 项目关联 | 掌握程度 | 状态 | 最近复盘 | 备注/卡点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 通信 | UART | 字节流、波特率、帧边界 | RK-MCU `/dev/ttyS3` |  | 未学 |  |  |
| 通信 | UART 自定义帧 | 帧头/type/seq/len/TLV/CRC | RK-MCU 协议 |  | 未学 |  |  |
| 通信 | TLV | 可扩展、版本兼容 | 状态摘要/命令 payload |  | 未学 |  |  |
| 通信 | CRC16-CCITT | 错误检测、多项式、校验流程 | 串口协议、IAP |  | 未学 |  |  |
| 通信 | ACK/重传 | seq、timeout、retry、幂等 | 命令类消息 |  | 未学 |  |  |
| 通信 | 环形缓冲区 | head/tail、空满、并发保护 | UART ISR + task |  | 未学 |  |  |
| 通信 | I2C | 地址、ACK、上拉、控制面 | IMX415 寄存器配置 |  | 未学 |  |  |
| 通信 | SPI | CPOL/CPHA、片选、高速外设 | 备用八股 |  | 未学 |  |  |
| 通信 | CAN | 仲裁、错误检测、状态机 | CAN 外设/MAVLink-CAN |  | 未学 |  |  |
| 通信 | SocketCAN | `can0`、`candump`、`cansend` | RK3562 CAN 联调 |  | 未学 |  |  |
| 通信 | CAN MAC 交换 / P401 配对 | 10 Hz 触发、READY/ACK、八帧身份交换、DONE、验证边界 | P401 图传无感快速配对 | 2 | 在学 | 2026-08-05 | 最新实现已从早期 report/ACK DTID 演进为 `PAIRTRIG` 多阶段状态机；500 kbit/s、天空端 `clock 12500000` 已确认，但 3 秒目标仍缺健康接触多轮证据。 |
| 通信 | CAN 固件分包 / A/B 升级 | 分区、File.Read、超时重试、整包 CRC32、trial/confirmed、异常回滚 | AGS 驱动板安全升级 | 3 | 待复盘 | 2026-08-05 | 已确认 16 KiB Bootloader、双 52 KiB slot、256 B File.Read、20 ms 请求、1.5 s 超时和 5 次重试；10 轮正常升级及 150/150 CAN 恢复通过，断电/不健康回滚等异常路径未验证。 |
| 通信 | MAVLink | sysid/compid/msgid/seq/payload | CommRouter 路由 |  | 未学 |  |  |
| 通信 | 路由/去重 | endpoint、route matrix、时间窗口 | CommRouter |  | 未学 |  |  |
| 网络 | TCP/UDP | 可靠性、延迟、报文边界 | UDP 图传/遥测 |  | 未学 |  |  |
| 网络 | RTP/H.265 | 编码、封装、传输层区分 | IMX415 图传 |  | 未学 |  |  |

## 五、Linux / Android 板端

| 模块 | 知识点 | 面试要求 | 项目关联 | 掌握程度 | 状态 | 最近复盘 | 备注/卡点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Linux | 进程/线程 | 地址空间、资源、调度 | 板端服务 | 2 | 在学 | 2026-07-13 | 已结合 CommRouter 讲解单线程 poll 与多线程方案：多线程可隔离阻塞并利用多核，但共享路由状态需要锁和队列；当前仍需独立回答进程/线程基础。 |
| Linux | 同步机制 | mutex/cond/atomic/消息队列 | 路由表、链路状态 | 2 | 在学 | 2026-07-13 | 已理解 poll 单线程事件循环可以减少共享状态加锁；多线程方案仍需 mutex、队列和生命周期管理，待结合具体竞态案例复述。 |
| Linux | 文件描述符 | open/read/write/ioctl/socket | tty/CAN/video node | 3 | 待复盘 | 2026-07-13 | 已结合 CommRouter 复习：UART、UDP、UDS 均可作为 fd 交给 poll；能理解 fd/events/revents，但尚未闭卷手写完整事件循环。 |
| Linux | IO 多路复用 | select/poll/epoll、同时等待多个 fd | UART/UDP/CAN 事件循环 | 3 | 待复盘 | 2026-07-13 | 已讲解 poll 与多线程取舍、pollfd 遍历、POLLIN/POLLERR/POLLHUP；本轮未独立作答，下一次优先闭卷复述。 |
| Linux | socket 编程 | UDP send/recv、端口、抓包 | APK/P401 网络链路 | 3 | 待复盘 | 2026-07-13 | 已讲解 socket/bind/recvfrom/sendto、临时源端口、datagram 边界，以及实时数据与关键命令的不同可靠性策略；待独立复述。 |
| Linux | 阻塞/非阻塞 IO | blocking、nonblocking、EAGAIN、同步/异步区别 | UART/UDP/CAN 多链路处理 | 2 | 在学 | 2026-06-04 | 已讲解，待反馈：阻塞 read 会卡住线程；非阻塞没数据返回 EAGAIN，通常配合 poll/epoll，避免服务卡在某个 fd。 |
| Linux | `ioctl` | 设备控制命令、request/arg、V4L2 controls | 串口配置、V4L2 `/dev/videoX` | 2 | 在学 | 2026-06-04 | 已讲解，待反馈：read/write 传数据，ioctl 做设备配置；V4L2 设置格式、buffer、streamon、exposure/vblank 都走 ioctl。 |
| Linux | 用户态/内核态 | 系统调用、copy_from_user/to_user、应用层/驱动层关系 | CommRouter、V4L2、SocketCAN | 2 | 在学 | 2026-06-04 | 已讲解，待反馈：应用层通常在用户态，驱动层通常在内核态；用户态通过 open/read/write/ioctl/socket 进入内核驱动。 |
| Linux | 设备节点/字符设备 | `/dev/xxx`、char/block、major/minor | `/dev/ttyS3`、`/dev/video0/11` | 2 | 在学 | 2026-06-04 | 已讲解，待反馈：设备节点不是普通文件，open 后得到 fd；major 找驱动，minor 区分同类设备实例。 |
| Android | init 服务 | `.rc`、service、class、重启 | CommRouter 自启动 | 2 | 重点薄弱 | 2026-07-13 | 已讲解 `getprop init.svc.xxx`、`ps -A`、`setprop ctl.restart`、init `.rc`、日志和依赖资源的分层排查；仍未独立作答，继续列为重点薄弱。 |
| Linux | 权限/用户组 | `ls -l`、user/group、设备节点访问权限 | `/dev/ttyS3`、`/dev/videoX` 访问 | 3 | 待复盘 | 2026-07-13 | 已结合服务启动排查复习设备 owner/group/mode、进程用户和 init.rc group；需要与 SELinux domain/label 分层回答。 |
| Android | SELinux | enforcing/permissive、avc denied、`ls -Z`/`ps -AZ` | Android 服务访问设备节点 | 2 | 在学 | 2026-07-13 | 已理解普通权限正确仍可能被 SELinux 拒绝，应查 `avc: denied`、进程 domain 和设备 label；不能把关闭 SELinux 当作正式修复。 |
| Linux | 启动链与镜像组成 | U-Boot、Kernel、DTB、rootfs 的职责和启动关系 | RK3588 系统构建 | 1 | 重点薄弱 | 2026-08-05 | 远端审计新增项目主线；需要能从上电到挂载 rootfs 完整复述，并说明各镜像的构建和烧录位置。 |
| Linux | 外部内核模块 | ARCH/CROSS_COMPILE、内核构建目录、vermagic、依赖和加载验证 | RTL8188FU、tun.ko | 2 | 在学 | 2026-08-05 | RTL8188FU 的 USB ID、模块大小、vermagic、联网和吞吐数据已补；仍需闭卷复述构建依赖和方向不对称问题，不能声称 TCP/UDP 均大于 20 Mbit/s。 |
| Linux | SDK 构建与 rootfs 定制 | 构建入口、包裁剪、依赖预置、镜像打包、可重复构建 | RK3588 板端交付 | 3 | 待复盘 | 2026-08-05 | 已确认 `./build.sh rootfs`、no-Qt 真机闭环、3.60 GB 典型 rootfs、约 3.72 GB update.img 和反向解包/e2fsck 验证；现有证据不支持“缩减 500 MB”。 |
| Linux | 量产烧录与批量升级 | 设备身份、并发状态机、超时重试、版本检查、自检和日志归档 | SD 卡烧录、产线工具 | 1 | 重点薄弱 | 2026-08-05 | 需补充实际连接方式、并发数量、考核通过条件和失败样本，避免只说“脚本返回 0”。 |
| Android | ADB/logcat | 服务状态、日志定位 | RK3562 联调 | 4 | 已掌握 | 2026-06-01 | 熟悉：logcat 看 Android 用户态服务/APK/init 相关日志，可按 comm_router/mavlink/uart 等关键字筛。 |
| Linux | dmesg | 内核/驱动日志 | camera/CAN/tty | 4 | 已掌握 | 2026-06-01 | 熟悉：dmesg 看 kernel/driver/设备树/probe/I2C/MIPI/CAN/tty 等问题。 |
| Linux | tcpdump | 网络包定位 | UDP/MAVLink/RTP | 4 | 已掌握 | 2026-06-01 | 熟悉：tcpdump 看 UDP/RTP/MAVLink 包是否真实发出、目标 IP/端口是否正确。 |

## 六、Linux 驱动 / BSP / Camera

| 模块 | 知识点 | 面试要求 | 项目关联 | 掌握程度 | 状态 | 最近复盘 | 备注/卡点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 驱动 | 设备树 | DTS/DTSI/DTB、硬件描述 | IMX415 board config |  | 未学 |  |  |
| 驱动 | CAN 设备树与参考时钟 | SPI/IRQ/pinctrl/fixed-clock、实际晶振、位时序和真实收帧验证 | RK3588 外置 CAN 控制器 | 3 | 待复盘 | 2026-08-05 | 已确认旧 16 MHz DTS/`clock 8000000` 与实板 25 MHz 晶振不匹配，修正后 `clock 12500000`、500 kbit/s、sample point 0.680；物理触点/收发器稳定性需与时钟问题分开。 |
| 驱动 | `compatible`/`probe` | 驱动匹配、资源申请、注册设备 | IMX415 probe |  | 未学 |  |  |
| 驱动 | platform/I2C driver | SoC 外设 vs I2C 外设 | RKISP/RKCIF vs IMX415 |  | 未学 |  |  |
| 驱动 | 字符设备/ioctl | 用户态访问内核设备 | V4L2/ioctl 类比 |  | 未学 |  |  |
| 驱动 | 中断上下半部 | top half/bottom half/workqueue | 通用驱动八股 |  | 未学 |  |  |
| 驱动 | 锁机制 | spinlock/mutex/atomic | 驱动并发保护 |  | 未学 |  |  |
| Camera | I2C + MIPI | 控制面/数据面区别 | IMX415 |  | 未学 |  |  |
| Camera | media graph | entity/pad/link/endpoint | IMX415 -> RKISP |  | 未学 |  |  |
| Camera | V4L2 取帧流程 | S_FMT/REQBUFS/QBUF/DQBUF/STREAMON | `/dev/video0/11` |  | 未学 |  |  |
| Camera | RAW10/NV12 | Bayer 原始数据 vs YUV420 | RAW 正常、NV12 黑 |  | 未学 |  |  |
| Camera | VTS/HTS/vblank/exposure | timing 和帧率关系 | 1080p60 timing |  | 未学 |  |  |
| Camera | 3A/AIQ/IQ | 算法、服务、参数文件关系 | `rkaiq_3A` 黑帧 |  | 未学 |  |  |
| Camera | GStreamer | pipeline、编码、RTP 推流 | H.265/RTP 验证 |  | 未学 |  |  |
| Camera | 分层排查 | 测试源、RAW、NV12、编码、网络 | IMX415 黑帧定位 |  | 未学 |  |  |

## 七、手撕代码

| 模块 | 知识点 | 面试要求 | 项目关联 | 掌握程度 | 状态 | 最近复盘 | 备注/卡点 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 手撕 | 环形缓冲区 | 能写 put/get、空满判断 | UART 接收 |  | 未学 |  |  |
| 手撕 | CRC16 | 能说明/手写核心循环 | 协议校验 |  | 未学 |  |  |
| 手撕 | TLV 解析状态机 | 能写状态转移和边界检查 | RK-MCU 协议 |  | 未学 |  |  |
| 手撕 | `memmove` | 能处理重叠拷贝 | buffer 处理 |  | 未学 |  |  |
| 手撕 | 链表反转 | 基础数据结构 | 通用面试 |  | 未学 |  |  |
| 手撕 | 判断链表有环 | 快慢指针 | 通用面试 |  | 未学 |  |  |
| 手撕 | 字符串处理 | 反转、查找、去重 | 通用面试 |  | 未学 |  |  |
| 手撕 | 位操作 | set/clear/test bit、大小端转换 | 寄存器/协议字段 |  | 未学 |  |  |
| 手撕 | 简单状态机 | 状态枚举、事件驱动 | 协议解析/控制逻辑 |  | 未学 |  |  |

## 八、复盘记录

| 日期 | 复盘内容 | 做对的点 | 卡住的点 | 下一步 |
| --- | --- | --- | --- | --- |
| 2026-08-05 | 同步远端并复核最新版简历审计分支 | 保留当前项目证据文档，提取新增风险项，没有直接合并会删除证据的分支 | A/B 精确参数、RK3588 系统构建、CAN 时钟与产线数据仍缺闭卷和脱敏证据 | 优先完成关键红线问题的资料核对，再按项目做闭卷压力面试 |
| 2026-08-05 | 读取本机关联工程并补充参数 | 补齐两套 A/B、手持电源/电量、RK 端口、P401 新协议、CAN 时钟、rootfs 和 Wi-Fi 证据边界 | 手持 A/B 真机、两套异常回滚、P401 多轮时延和产线统计仍缺 | 先按证据索引闭卷复述，再补异常路径真机记录 |
|  |  |  |  |  |
