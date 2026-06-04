# 结合项目的嵌入式八股复习清单

> 目标：面试时不要只背概念，而是把高频八股都落到你的两个项目上：手持发射器 RK3562 + STM32F103 控制链路、RK3588 IMX415 摄像头 1080p60 bring-up。

## 牛客面经高频方向

参考牛客嵌入式/嵌入式 Linux 面经后，和你简历最匹配的高频问题集中在这些模块：

- C 语言基础：指针、内存分区、`volatile`、结构体对齐、函数指针、`memcpy`/`memmove`、堆栈、编译链接。
- MCU 与外设：STM32 启动流程、中断、GPIO、ADC、I2C、UART、CAN、看门狗、Bootloader/IAP。
- FreeRTOS：任务状态、调度、优先级、队列、信号量、互斥量、事件组、软件定时器、中断与任务通信、栈溢出。
- 通信协议：UART/I2C/SPI/CAN 区别，CRC，ACK/重传，帧格式，SocketCAN，TCP/UDP 基础。
- Linux/Android 板端：进程线程、同步、文件描述符、网络 IO、`init` 服务、`logcat`/`dmesg`/`tcpdump`/`adb`。
- Linux 驱动/BSP：设备树、platform/I2C 驱动匹配、`probe`、字符设备、`ioctl`、中断上下半部、锁、DMA、用户态和内核态交互。
- Camera/V4L2：I2C 配寄存器、MIPI CSI-2 传图、media graph、V4L2 ioctl、RAW/NV12、ISP/3A/IQ、GStreamer 推流。
- 项目深挖：项目架构、技术难点、问题定位、稳定性设计、为什么这么设计、哪里是自己主责。

复习优先级建议：

1. 先背项目相关的八股：FreeRTOS、UART/CAN、Bootloader、设备树、V4L2、调试工具。
2. 再补通用嵌入式基础：C、OS、Linux 进程线程、网络。
3. 最后准备算法/手撕：字符串、链表、数组、位运算、环形缓冲区、CRC、简单状态机。

## 项目一：RK3562 + STM32F103 控制链路

### 1. UART 协议设计

牛客高频问法：

- UART 为什么只需要 TX/RX？什么时候需要 RTS/CTS？
- 自定义通信协议一般怎么设计？
- 为什么要有帧头、长度、序号、校验？
- CRC 和 checksum 有什么区别？
- 串口丢包、粘包、错包怎么处理？

结合项目回答：

> 我们 RK3562 和 STM32F103 之间是 UART 运行时协议，帧格式是 `0xA5 + type + seq + len + TLV + CRC16-CCITT`。帧头用于快速重新同步，`type` 区分状态、命令、ACK、升级等消息，`seq` 用于命令应答和重复包识别，`len` 避免依赖固定结构体长度，TLV 便于后续扩展字段，CRC16 用于发现串口干扰或解析错位。接收侧一般按状态机解析：找帧头、读固定头、按长度收 payload、校验 CRC，失败就丢弃并重新找帧头。

追问补充：

- TLV 优点：字段可扩展、版本兼容好、不同消息可以复用同一解析框架。
- 固定结构体缺点：结构体对齐、大小端、版本变更、字段增删都更容易踩坑。
- CRC16 比简单累加和更能发现多位错误，适合串口这种有干扰风险的链路。
- 如果有可靠性要求，靠 `seq + ACK + timeout + retry` 做重传；状态类周期上报可以允许丢一帧，命令类必须有 ACK。

### 2. CAN 与 SocketCAN

牛客高频问法：

- CAN 和 UART/I2C/SPI 的区别？
- CAN 为什么适合车载/工业控制？
- CAN 仲裁机制是什么？
- `ERROR-ACTIVE`、`ERROR-PASSIVE`、`BUS-OFF` 是什么？
- Linux 下 SocketCAN 怎么收发？

结合项目回答：

> 项目里 CAN 用于板端外设和 MAVLink-CAN bridge 联调。CAN 和 UART 最大不同是它是多主总线，有硬件仲裁和错误检测，适合多节点、抗干扰和实时性要求较高的控制链路。Linux 侧用 SocketCAN，可以用 `candump` 看报文，用 `cansend` 构造测试帧，服务里把 CAN 报文和 MAVLink 消息做桥接。调试时我会先看接口是否 up、bitrate 是否一致，再看错误计数和总线状态，最后看应用层路由是否正确。

追问补充：

- CAN 仲裁：显性位覆盖隐性位，ID 越小优先级越高，仲裁失败节点自动退让。
- `ERROR-ACTIVE` 是正常可主动发错误帧；`ERROR-PASSIVE` 说明错误计数高，发送能力受限；`BUS-OFF` 说明节点被总线隔离，需要恢复流程。
- UART 是点对点异步串口；I2C 是两线多从、地址寻址；SPI 是同步全双工、片选扩展；CAN 是多主广播、硬件仲裁和错误管理。

### 3. FreeRTOS 任务与同步

牛客高频问法：

- FreeRTOS 任务有哪些状态？
- 优先级调度怎么做？
- 队列、信号量、互斥量、事件组有什么区别？
- 中断里能不能直接调用普通 API？
- 任务栈溢出怎么定位？
- PendSV/SysTick/SVC 在 Cortex-M RTOS 中分别做什么？

结合项目回答：

> STM32F103 侧我按硬件链路拆任务，比如状态采集、通信收发、控制逻辑和看门狗喂狗。实时性高的接收中断只做轻量动作，把数据放入缓冲区或用 FromISR API 通知任务，复杂解析放到任务里做，避免中断里阻塞。任务间传递消息优先用队列，保护共享资源用互斥量，多事件条件用事件组。对串口/CAN 接收这类场景，重点是避免任务长时间阻塞导致通信缓存溢出或看门狗超时。

追问补充：

- 任务状态：Running、Ready、Blocked、Suspended，部分实现还会讨论 Deleted。
- 同优先级任务通常时间片轮转，高优先级 Ready 会抢占低优先级。
- 二值信号量适合事件通知，计数信号量适合资源计数，互斥量带优先级继承，队列适合传数据。
- ISR 中使用 `xQueueSendFromISR`、`xSemaphoreGiveFromISR` 这类 API，并根据返回值决定是否触发任务切换。
- 栈溢出可通过 FreeRTOS hook、水位检测、HardFault 现场、map 文件和任务栈余量定位。

### 4. IWDG 看门狗与稳定性

牛客高频问法：

- 独立看门狗和窗口看门狗区别？
- 看门狗应该在哪喂？
- 系统卡死如何恢复？

结合项目回答：

> 项目里 IWDG 是系统自恢复兜底。我的理解是不能在一个高频定时器里无脑喂狗，否则任务死锁也可能继续喂狗。更合理的是由关键任务上报心跳，监控任务确认通信、采集、控制等关键链路都在运行，再统一喂狗。如果某个关键任务卡死、队列堵塞或通信长时间异常，就不喂狗，让系统复位进入可恢复状态。

追问补充：

- IWDG 通常由独立低速时钟驱动，适合系统异常兜底；WWDG 有窗口限制，可发现过早或过晚喂狗。
- 看门狗不能替代错误处理，应该配合错误计数、状态机复位、日志记录和升级回滚策略。

### 5. Bootloader / UART-IAP

牛客高频问法：

- STM32 启动流程是什么？
- Bootloader 和 APP 如何划分 Flash？
- IAP 升级流程怎么设计？
- 升级断电怎么办？
- 跳转 APP 前要做哪些事？
- 一个 `.bin` 和 `.elf` 有什么区别？

结合项目回答：

> 项目里 Bootloader 常驻前部 Flash，APP 偏移到 `0x08004000`。RK3562 通过 `/dev/ttyS3` 触发 UART-IAP，升级流程包括握手、擦除 APP 区、分包写入、CRC 校验、写升级状态、复位跳转。跳转 APP 前要检查 APP 栈顶地址和复位向量是否合法，关闭中断，必要时重设 MSP 和向量表偏移，然后跳转到 APP reset handler。

追问补充：

- STM32 上电后从向量表取初始 MSP 和 Reset_Handler，执行启动文件，初始化数据段/BSS，再进 `main`。
- `.elf` 包含符号表、段信息和调试信息，适合调试；`.bin` 是纯镜像，适合烧录或升级。
- 断电保护：至少要有升级状态标记和 CRC 校验；更稳的是双 APP 分区或 A/B 回滚。你的项目可以说“当前主要完成握手、擦写、CRC 校验和复位流程，断电回滚策略可继续增强”，不要过度夸大。

### 6. Android 板端服务与调试

牛客高频问法：

- Android/Linux 用户态服务怎么自启动？
- `logcat`、`dmesg`、`tcpdump` 分别看什么？
- UDP 和 TCP 区别？
- 多进程/多线程如何同步？
- 如何定位“APK 没显示数据”？

结合项目回答：

> RK3562 侧我参与 `CommRouter`、`PeripheralManager`、`LinkController` 的联调。服务通过 Android init 管理，调试时先看 init 服务是否启动，再看 `logcat` 应用和服务日志，看串口/CAN 是否有底层数据，看 `tcpdump` 是否有 UDP 报文，最后看 APK 解析和显示。`CommRouter` 的重点不是简单转发，而是多端点 MAVLink 路由、endpoint 标记和去重，避免同一消息在 UART、CAN、UDP 多链路之间重复绕回。

追问补充：

- `dmesg` 看内核、驱动、设备枚举和硬件错误；`logcat` 看 Android 用户态服务和应用日志；`tcpdump` 看网络包；串口日志看 MCU/RK 链路；`candump` 看 CAN 总线。
- UDP 无连接、低开销、可能丢包乱序，适合实时状态或视频 RTP；TCP 可靠有序但有重传和拥塞控制，延迟可能抖动。

## 项目二：RK3588 IMX415 Camera Bring-up

### 1. 设备树与驱动匹配

牛客高频问法：

- 为什么需要设备树？
- platform/I2C 驱动如何匹配？
- 为什么 `probe` 会被调用？
- 设备树里常见属性有哪些？
- 修改设备树后怎么验证？

结合项目回答：

> IMX415 是 I2C sensor，设备树负责描述硬件连接，比如 I2C 地址、MCLK、reset/power GPIO、MIPI data lanes 和 endpoint。驱动里通过 `of_match_table` 的 `compatible` 和设备树节点匹配，匹配后内核调用 `probe`，驱动再申请资源、读 sensor ID、注册 V4L2 subdev。设备树不是写行为逻辑，而是描述板级硬件差异，驱动负责具体控制。

追问补充：

- `compatible` 错了，驱动不会匹配；I2C 地址错了，可能 probe 读 ID 失败；GPIO/clock/pinctrl 错了，sensor 可能不上电或无 MCLK；endpoint/lane 错了，media graph 或 MIPI 数据链路会异常。
- 验证路径：`dmesg` 看 probe，`media-ctl -p` 看 graph，`v4l2-ctl --list-devices` 看节点，抓帧看是否有数据。

### 2. I2C 与 MIPI CSI-2

牛客高频问法：

- 摄像头里的 I2C 和 MIPI 分别做什么？
- sensor probe 成功为什么不等于出图成功？
- MIPI lane 数、link frequency、pixel rate 有什么关系？

结合项目回答：

> Camera sensor 通常用 I2C 配寄存器，比如模式、曝光、增益、VTS/HTS；真正图像数据通过 MIPI CSI-2 输出。probe 成功只能说明 I2C 通、sensor ID 可能读到了，不代表 MIPI lane、时钟、timing、media graph、ISP 都正确。我的 IMX415 项目就是按 IMX415 -> DPHY -> CSI2 -> RKCIF -> RKISP -> video node 分层验证，不能只看一个节点。

追问补充：

- I2C 是低速控制面，MIPI CSI-2 是高速数据面。
- lane 数不足、link frequency 不匹配、endpoint 配错，都可能 probe 成功但无帧或花屏。

### 3. V4L2 / Media Graph

牛客高频问法：

- V4L2 应用层如何取帧？
- `VIDIOC_S_FMT`、`REQBUFS`、`QBUF`、`DQBUF`、`STREAMON` 大概做什么？
- `/dev/video0` 和 subdev/media entity 是什么关系？
- RAW、YUV、NV12 有什么区别？

结合项目回答：

> V4L2 给用户态提供统一视频采集接口。应用层一般 open video node，通过 ioctl 设置格式、申请 buffer、入队、开流、出队拿帧。Rockchip camera 链路里 sensor 是 subdev，DPHY/CSI/RKCIF/RKISP 也在 media graph 里，最终导出 `/dev/videoX` 节点。我的项目里用 RAW `/dev/video0` 验证 sensor 前端采集，用 ISP/NV12 `/dev/video11` 验证 ISP 输出和后续编码链路。

追问补充：

- RAW10 Bayer 是 sensor 原始马赛克数据，还没做 demosaic、白平衡、降噪等 ISP 处理。
- NV12 是 YUV420 semi-planar 格式，常用于显示、编码和图传。
- RAW 正常但 NV12 黑，说明 sensor/I2C/MIPI/前端采集大概率不是第一嫌疑，重点看 ISP、AIQ、格式和 controls。

### 4. Sensor Timing：VTS/HTS/VBlank/Exposure

牛客高频问法：

- 帧率由什么决定？
- VTS、HTS、曝光、vblank 的关系？
- 为什么不能只改 fps 字段？
- 1080p60 适配要改哪些参数？

结合项目回答：

> 1080p60 不是简单改一个 fps 字段，而是 sensor 寄存器表、VTS/HTS、`vertical_blanking`、`pixel_rate`、`link_frequency`、lane 数和设备树链路要一致。项目里稳定参数是 `VTS=0x08CA=2250`，有效高度 `height=1080`，所以 `vertical_blanking=1170`。曝光时间通常受 VTS 限制，vblank 变小会压缩最大曝光范围，也会改变帧周期。

追问补充：

- 粗略关系：`VTS = height + vblank`。
- 帧率和 pixel clock、HTS、VTS 相关，通常 `fps ~= pixel_clock / (HTS * VTS)`，具体以 sensor 手册和驱动定义为准。
- 面试时强调“mode table、controls、media bus format 和链路频率要一致”，这是 bring-up 的重点。

### 5. ISP / 3A / IQ 黑帧定位

牛客高频问法：

- 3A 是什么？
- IQ 文件是什么？
- 为什么 RAW 有图但 NV12 黑？
- 你怎么定位黑屏问题？

结合项目回答：

> 3A 一般指 AE 自动曝光、AWB 自动白平衡、AF 自动对焦；IQ 文件是和 sensor、镜头、模式匹配的一组 ISP 调参配置。项目里黑帧不是一开始就假设 driver 错，而是分层验证：先用测试源证明 H.265/RTP 接收链路没问题，再抓 RAW `/dev/video0`，确认 RAW 非零；然后抓 NV12 `/dev/video11`，发现 Y 全 0、UV 约 128，表现为黑帧。继续对比 `rkaiq_3A` 启停前后的 controls，发现它把 `vertical_blanking` 从稳定值 1170 改成 46，导致 ISP/NV12 输出异常。当前工程方案是禁用不匹配 AIQ 或用 systemd guard 固定已验证 controls，长期应补齐匹配 1080p60 linear mode 的 IQ 文件。

追问补充：

- RAW 正常说明 sensor 到 RKCIF 前端采集基本通；NV12 黑说明 ISP/AIQ/格式转换/controls 是重点。
- `rkaiq_3A` 是用户态调参服务，它可能改曝光、增益、vblank 和 ISP 参数。
- 不要说“彻底解决 AIQ 算法”，准确说“定位到 IQ/AIQ 配置不匹配并做工程规避”。

### 6. GStreamer / H.265 / RTP / UDP

牛客高频问法：

- H.264/H.265 和 RTP/UDP 分别是什么层？
- 为什么图传常用 UDP？
- 如何定位视频推流黑屏/卡顿？

结合项目回答：

> 摄像头链路最终还要进入编码和网络传输。H.265 是视频编码格式，RTP 是实时媒体承载协议，UDP 是传输层协议。图传偏实时，宁可丢少量包也不希望 TCP 重传造成长时间卡顿，所以常用 RTP/UDP。定位时我会先用测试源排除编码和接收端，再用 V4L2 分别抓 RAW/NV12，最后看 GStreamer pipeline、编码器输入格式和网络抓包。

## 通用 C 语言八股：必须和项目挂钩

### 1. `volatile`

标准回答：

> `volatile` 告诉编译器这个变量可能被当前代码流之外的因素改变，不能随意优化读写。嵌入式里常见于寄存器映射、中断和主循环共享变量、多线程共享状态。但 `volatile` 不保证原子性，也不解决并发互斥。

项目挂钩：

- STM32 外设寄存器访问需要 `volatile`。
- 中断置位、任务读取的标志位可用 `volatile` 防止优化，但多字段共享状态仍要临界区、队列或锁。

### 2. 内存分区

标准回答：

> C 程序常见区域包括 text 代码段、rodata 只读常量、data 已初始化全局/静态变量、bss 未初始化全局/静态变量、heap 动态分配、stack 局部变量和函数调用栈。MCU 上这些区域由链接脚本映射到 Flash 和 SRAM。

项目挂钩：

- Bootloader/APP 分区本质依赖链接脚本和向量表地址。
- FreeRTOS 每个任务有独立任务栈，栈太小会 HardFault 或栈溢出。
- 嵌入式里动态内存要谨慎，避免碎片和不可控失败。

### 3. 结构体对齐和大小端

标准回答：

> 结构体成员会按对齐规则插入 padding，不同编译器/架构可能布局不同。大小端影响多字节整数的字节序。因此跨 MCU/RK 通信不建议直接发送结构体内存。

项目挂钩：

- RK-MCU 串口协议使用 TLV 和显式长度，而不是裸结构体，就是为了减少对齐、大小端和版本变化问题。

### 4. `memcpy` 和 `memmove`

标准回答：

> `memcpy` 假设源和目的区域不重叠，重叠时行为未定义；`memmove` 要处理重叠，必要时反向拷贝。

项目挂钩：

- 串口环形缓冲区、协议 payload 解析、图像 buffer 处理都要注意边界和重叠。
- 面试手撕时重点写出边界检查、空指针处理和重叠判断。

### 5. 函数指针和回调

标准回答：

> 函数指针保存函数入口地址，回调是把函数指针注册给框架，由框架在事件发生时调用。嵌入式里常见于驱动 ops、协议分发、中断回调、状态机处理。

项目挂钩：

- Linux 驱动里的 `file_operations`、V4L2 subdev ops 都是典型函数指针表。
- RK-MCU 协议解析可按 `type` 分发到不同 handler。

## 高频手撕题

优先准备这些，和你的项目最容易关联：

1. 环形缓冲区：串口接收、生产者消费者。
2. CRC16-CCITT：通信协议校验。
3. TLV 解析状态机：帧头、长度、payload、CRC。
4. `memmove`：内存重叠处理。
5. 链表反转/判断环：基础数据结构。
6. 字符串反转、查找子串、去重：牛客常见手撕。
7. 位操作：置位、清位、判断 bit、大小端转换。
8. 简单 LRU 或队列：考察数据结构和边界处理。

## 面试项目深挖模板

每个项目都按这 6 句话准备：

1. 背景：这个项目解决什么产品问题。
2. 架构：从硬件到系统服务的数据链路怎么走。
3. 主责：你具体负责哪几块，哪些是参与联调。
4. 难点：最容易出问题的地方是什么。
5. 定位：你怎么分层排查，证据是什么。
6. 结果：最终验证到什么程度，还有哪些边界。

### 手持发射器可用模板

> 这个项目是 RK3562 Android SoC + STM32F103 MCU 的手持设备控制链路。STM32 负责硬件状态采集和控制，RK3562 负责板端服务、APK 显示和网络链路。我主要做 MCU 固件、RK-MCU 串口协议、UART-IAP Bootloader，并参与 RK3562 服务链路联调。难点是链路长，问题可能出在 MCU、UART、CAN、Android init 服务、UDP、MAVLink 路由或 APK 显示。我的定位方法是分层抓证据：串口日志看 MCU/RK，SocketCAN 看 CAN，总线和服务日志看板端，tcpdump 看网络，最后看 APK 显示和消息解析。

### IMX415 可用模板

> 这个项目是 RK3588 Linux 平台 IMX415 摄像头 1080p60 bring-up。我基于已有 V4L2 sensor driver 适配 1080p60 RAW10 模式，复核设备树 media graph，并分层验证 RAW、ISP/NV12、H.265/RTP 链路。难点是 probe 成功不代表出图成功，I2C、MIPI、timing、media graph、ISP、AIQ 都可能影响结果。黑帧问题里我先证明编码/网络链路没问题，再确认 RAW 正常但 NV12 黑，最后对比 AIQ 启停前后 controls，定位到 `rkaiq_3A` 把 vblank 从 1170 改成 46，导致 ISP 输出异常。

## 七天突击安排

### Day 1：项目表达

- 背熟两个项目的一句话、两分钟讲法。
- 画出两张链路图：RK3562 + STM32 控制链路、IMX415 camera pipeline。
- 准备“我负责/我参与/我没做”的边界。

### Day 2：C 语言与手撕

- 指针、数组、函数指针、`volatile`、内存分区、结构体对齐。
- 手撕 `memmove`、环形缓冲区、位操作、链表反转。

### Day 3：MCU/FreeRTOS

- STM32 启动、中断、IWDG、Bootloader/IAP。
- FreeRTOS 调度、队列、信号量、互斥量、中断通知、栈溢出。

### Day 4：通信协议

- UART/I2C/SPI/CAN 对比。
- CRC、ACK、重传、TLV、MAVLink、SocketCAN、UDP/TCP/RTP。

### Day 5：Linux/Android 板端

- 进程线程、锁、文件描述符、网络 IO。
- Android init、`logcat`、`dmesg`、`tcpdump`、`adb`。

### Day 6：Linux 驱动/Camera

- 设备树、I2C/platform 匹配、`probe`、V4L2、media graph。
- RAW/NV12、ISP、3A/IQ、GStreamer、H.265/RTP。

### Day 7：模拟面试

- 每个项目连续讲 2 分钟。
- 每个项目准备 10 个追问。
- 做 2 道手撕题。
- 复盘回答是否夸大、是否有证据、是否能讲清定位过程。

## 参考的牛客面经

- [嵌入式 FreeRTOS 面经题目汇总](https://www.nowcoder.com/discuss/812234688495419392)：任务调度、同步机制、中断通信、定时器、系统移植。
- [影石嵌入式一面面经](https://www.nowcoder.com/discuss/793371834732335104)：C/内存、FreeRTOS 内核机制、项目深挖和手撕代码。
- [嵌入式全套面经总结](https://www.nowcoder.com/discuss/808356304229003264)：C 语言、STM32、RTOS、通信协议、驱动开发、项目经验。
- [嵌入式 Linux 驱动必考八股文清单](https://www.nowcoder.com/discuss/864181196555677696)：设备模型、platform/I2C 匹配、字符设备、中断、锁、设备树。
- [影石嵌入式 Linux 一面面经](https://www.nowcoder.com/discuss/857545753730822144)：设备树、驱动调试、模块加载、内核调试和项目经验。
- [嵌入式音视频必备 V4L2 架构](https://www.nowcoder.com/discuss/757617690482655232)：V4L2 应用层、核心层、驱动层、MIPI 摄像头数据流。
- [影石嵌入式面经](https://www.nowcoder.com/discuss/785858219989078016)：I2C 传感器 bring-up、设备树、字符设备、UART 流控、RTOS 调度。
- [2026 嵌入式面试热点](https://www.nowcoder.com/discuss/882528031858835456)：C/RTOS/Linux/驱动/项目深挖的准备主线。
