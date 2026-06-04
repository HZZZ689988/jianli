# Linux / Android 板端学习记录 2026-06-04

这份文档整理本轮面试复习对话中已经讲过的 Linux / Android 板端知识点。用途是：后续复盘时可以快速知道“讲过什么、你反馈到什么程度、下一步该追问哪里”。

## 当前学习原则

- 解释风格：后续默认使用更适合初学者的语言，少堆概念，多用“是什么、为什么需要、项目里怎么用、面试怎么说”。
- 复习目标：能把 Linux 基础知识和你的两个项目挂钩，而不是孤立背八股。
- 当前主线：RK3562 Android 板端服务、RK3588 IMX415 Camera bring-up。

## 已整理进知识表格的状态

知识表格位置：[interview-knowledge-tracker.md](./interview-knowledge-tracker.md)

### 已掌握或较熟悉

| 知识点 | 当前状态 | 项目化表达 |
| --- | --- | --- |
| `logcat` | 已掌握 | 看 Android 用户态服务、APK、init 相关日志；排查 CommRouter/MAVLink/APK 显示链路。 |
| `dmesg` | 已掌握 | 看 kernel/driver/设备树/probe/I2C/MIPI/CAN/tty 等问题。 |
| `tcpdump` | 已掌握 | 看 UDP/RTP/MAVLink 包是否真实发出、目标 IP/端口是否正确。 |
| 文件描述符 fd | 待复盘 | fd 是用户态访问内核资源的句柄；UART `/dev/ttyS3`、V4L2 `/dev/videoX`、UDP/CAN socket 都可纳入 fd 模型。 |
| IO 多路复用 | 待复盘 | `poll` 同时等待 UART、UDP、CAN fd，避免阻塞在某一个 `read` 上。 |
| UDP socket 编程 | 待复盘 | `bind` 绑定本地监听端口，`recvfrom` 收一个 datagram 并拿到对端 IP/端口，`sendto` 指定目标发送。 |
| 权限/用户组 | 待复盘 | 服务打不开 `/dev/ttyS3` 时查设备节点权限、进程用户、init.rc 的 user/group 和 Permission denied 日志。 |

### 重点薄弱

| 知识点 | 当前状态 | 后续要强化 |
| --- | --- | --- |
| Android init 服务 | 重点薄弱 | `/vendor/etc/init/*.rc`、`getprop init.svc.xxx`、`setprop ctl.start/stop/restart xxx`、`ps -A`、`logcat -b all`、服务权限/SELinux。 |

### 已讲解但待你反馈

| 知识点 | 当前状态 | 复盘重点 |
| --- | --- | --- |
| 进程/线程 | 在学 | 进程隔离、线程共享地址空间；独立服务靠 IPC，一个服务内多线程要同步共享状态。 |
| 同步机制 | 在学 | mutex、atomic、condition variable、死锁；什么时候用单线程 `poll` 简化设计。 |
| 阻塞/非阻塞 IO | 在学 | 阻塞 `read` 会卡住线程；非阻塞没数据返回 `EAGAIN`，通常配合 `poll/epoll`。 |
| `ioctl` | 在学 | `read/write` 传数据，`ioctl` 做设备配置；V4L2 的格式、buffer、streamon、controls 都走 ioctl。 |
| 用户态/内核态 | 在学 | 用户态应用通过系统调用进入内核驱动；应用层/驱动层是功能分层，用户态/内核态是权限分层。 |
| 设备节点/字符设备 | 在学 | `/dev/xxx` 设备节点、字符设备/块设备、major/minor、用户态 open 后得到 fd。 |
| SELinux | 在学 | 普通权限之外的第二道门；文件权限正确但仍打不开设备时查 `avc: denied`。 |

## 对话知识整理

### 1. fd 文件描述符

你的理解已经基本正确：

> 可以把 fd 理解为 Linux 把 socket、UART、V4L2 video node 等资源交给用户态程序的统一句柄。用户态通过 `read/write/ioctl/poll/close` 操作 fd，背后由内核驱动或协议栈完成真实工作。

更准确表达：

> fd 不是物理接口本身，而是进程操作某个内核对象的整数编号。

项目挂钩：

- `CommRouter` 打开 `/dev/ttyS3` 得到串口 fd，收发 RK-MCU 协议。
- `v4l2-ctl` 打开 `/dev/video0` 或 `/dev/video11` 得到 video fd，通过 V4L2 ioctl 抓 RAW/NV12。
- UDP 和 SocketCAN 通过 `socket()` 创建 fd，可以放进 `poll` 事件循环。

### 2. `poll/select/epoll`

核心理解：

> `poll` 用来同时等待多个 fd 的事件，解决“不能阻塞在某一个 read 上”的问题。

项目挂钩：

```text
UART fd        RK-MCU 协议
UDP socket fd  APK/P401 MAVLink
CAN socket fd  CAN bridge
timer fd       链路心跳
```

如果直接阻塞读 UART，UDP/CAN 可能处理不及时。使用 `poll` 后，哪个 fd 有数据就处理哪个。

### 3. Android init 服务

当前薄弱，需要多复盘。

初学者理解：

> `init.rc` 就是 Android 的开机服务配置。它告诉系统要启动哪个程序、用什么用户/用户组、属于哪个 class、退出后是否重启。

常用命令：

```bash
adb shell
getprop init.svc.comm_router
ps -A | grep comm_router
setprop ctl.start comm_router
setprop ctl.stop comm_router
setprop ctl.restart comm_router
logcat -b all | grep -i comm_router
```

排查服务起不来：

```text
1. rc 文件是否在 /vendor/etc/init/
2. service 名称是否正确
3. /vendor/bin/xxx 是否存在、有执行权限
4. 依赖设备节点是否存在，比如 /dev/ttyS3
5. user/group 是否有权限
6. SELinux 是否 avc denied
```

### 4. `logcat` / `dmesg` / `tcpdump`

你已标记熟悉。

简单记：

```text
logcat：用户态服务和 APK
dmesg：内核、驱动、设备树、硬件链路
tcpdump：网络包
```

项目排查示例：

- APK 没显示 MCU 状态：先看串口/CAN 底层，再看 `logcat` 服务路由日志，再用 `tcpdump` 看 UDP 包，最后看 APK。
- Camera 黑屏：先看 `dmesg` 里 IMX415 probe/MIPI/ISP，再用 V4L2 抓 RAW/NV12，再看 RTP/UDP。

### 5. UDP socket、`bind`、`recvfrom`

你的当前理解：

> UDP/TCP 大致了解，`bind` 和 `recvfrom` 已基本了解。

简化记忆：

```text
bind：我在哪个本地端口等包
recvfrom：从这个端口收一个 UDP 包，并知道是谁发来的
sendto：按目标 IP/端口发一个 UDP 包
```

项目挂钩：

> `CommRouter` 如果要接收 APK 发来的 MAVLink/UDP 命令，就需要 bind 到约定端口。APK 往板子的这个端口发包，服务通过 `recvfrom` 收到，并能拿到 APK 的 IP/端口用于回包。

### 6. 进程/线程

已讲解，待反馈。

简单记：

```text
进程：独立房间，资源隔离强
线程：同一个房间里的多个人，共享东西多，所以容易抢
```

项目挂钩：

- `CommRouter`、`PeripheralManager`、`LinkController` 如果是独立进程，彼此不能直接访问变量，需要 socket/pipe/shared memory/Binder 等 IPC。
- 如果 `CommRouter` 内部开 UART/UDP/CAN 多线程，就要保护 route table、endpoint 状态、去重缓存。
- 如果吞吐量不高，`poll` 单线程事件循环可以减少锁复杂度。

### 7. 线程同步

已讲解，待反馈。

简单记：

```text
mutex：保护一段共享数据，同一时间只让一个线程改
atomic：简单计数/标志的原子操作
condition variable：一个线程等条件，另一个线程通知
deadlock：两个线程互相等对方的锁
```

项目挂钩：

> 多线程服务如果同时更新 MAVLink 路由表、endpoint 状态或 packet_count，需要 mutex/atomic；也可以通过单线程事件循环减少共享数据。

### 8. 阻塞 / 非阻塞 IO

已讲解，待反馈。

简单记：

```text
阻塞：没数据就卡住等
非阻塞：没数据立刻返回 EAGAIN
poll：睡觉等多个 fd，谁有数据叫醒我
```

项目挂钩：

> RK3562 多链路服务不适合卡在一个 `read` 上。UART、UDP、CAN 可以设为非阻塞并配合 `poll`，保证多条链路都能及时处理。

### 9. `ioctl`

已讲解，待反馈。

简单记：

```text
read/write：读写数据
ioctl：控制设备、配置设备
```

项目挂钩：

- 串口 fd：`read/write` 收发字节，termios/ioctl 配波特率、数据位、停止位。
- V4L2 fd：`ioctl` 设置分辨率、格式、buffer、开流、抓帧、controls。
- IMX415：`v4l2-ctl` 设置 `vertical_blanking`、`exposure`、`analogue_gain` 本质是 V4L2 ioctl。

### 10. 用户态/内核态、应用层/驱动层

已讲解，待反馈。

简单区别：

```text
用户态/内核态：按权限和运行空间划分
应用层/驱动层：按软件功能划分
```

通常对应：

```text
应用层通常在用户态
驱动层通常在内核态
```

但不是绝对，例如用户态驱动、FUSE、UIO 等。

项目挂钩：

```text
CommRouter / APK / GStreamer / v4l2-ctl：用户态应用层
tty/UART / CAN / V4L2 sensor / RKISP：内核态驱动层
```

应用层通过 `open/read/write/ioctl/socket` 进入内核，内核驱动操作硬件。

### 11. 设备节点、字符设备、major/minor

已讲解，待反馈。

简单记：

> `/dev/ttyS3`、`/dev/video0` 不是普通文件，而是设备节点。用户态 open 它得到 fd，内核根据设备号找到对应驱动。

```text
字符设备：串口、摄像头、I2C、GPIO 等，常用 read/write/ioctl
块设备：磁盘、SD 卡、eMMC 等，按 block 读写
major：找哪类驱动
minor：同一驱动下的第几个设备
```

项目挂钩：

- `/dev/ttyS3` -> tty/UART 驱动 -> RK3562 UART -> STM32F103。
- `/dev/video0` / `/dev/video11` -> V4L2/RKCIF/RKISP -> IMX415 camera pipeline。

### 12. 设备树、probe、设备节点关系

已讲解，待反馈。

简单链路：

```text
设备树描述硬件
  -> compatible 匹配驱动
  -> probe 初始化硬件
  -> 驱动注册设备
  -> /dev/xxx 设备节点出现
  -> 用户态 open/read/write/ioctl
```

项目挂钩：

- IMX415：设备树描述 I2C 地址、MCLK、reset/power GPIO、MIPI lanes、endpoint；driver probe 后注册 V4L2 subdev，最终通过 RKCIF/RKISP 暴露 video node。
- RK3562 UART：设备树打开 UART 控制器并配置 pinctrl，驱动 probe 成功后出现 `/dev/ttyS3`，`CommRouter` 才能打开它。

### 13. 权限/用户组

你反馈：大致了解。

简单记：

```text
ls -l /dev/ttyS3
crw-rw---- root system ...
```

含义：

```text
root 用户可以读写
system 组可以读写
其他用户不能访问
```

项目排查：

```text
1. /dev/ttyS3 是否存在
2. ls -l 看权限
3. ps 看服务用什么用户运行
4. init.rc 里 user/group 是否正确
5. logcat 看 Permission denied
```

### 14. SELinux

已讲解，待反馈。

简单记：

> 普通权限是第一道门，SELinux 是第二道门。

会出现：

```text
ls -l 看起来有权限
但 logcat/dmesg 有 avc: denied
服务仍然打不开设备
```

常用命令：

```bash
getenforce
setenforce 0
setenforce 1
ls -Z /dev/ttyS3
ps -AZ | grep comm_router
logcat -b all | grep -i avc
dmesg | grep -i avc
```

面试边界：

> `setenforce 0` 可以作为调试验证手段，正式产品不能靠关闭 SELinux，应该补正确策略。

## 下一步建议

优先复盘顺序：

1. Android init 服务：用真实命令串起来讲服务起不来怎么查。
2. SELinux：会看 `avc: denied`，能解释普通权限和 SELinux 的区别。
3. 设备树 -> probe -> `/dev/xxx`：把 IMX415 和 `/dev/ttyS3` 两个例子讲熟。
4. 进程/线程 + 同步：能结合 `CommRouter` 路由表和去重缓存说明。
5. `ioctl` + V4L2：能讲 `v4l2-ctl` 背后就是 V4L2 ioctl。
