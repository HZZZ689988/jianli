# P401 图传无感快速配对面试记录

## 一句话概括

这是 RK3562 地面端与 RK3588 天空端之间的 P401 图传无感快速配对链路：通过 CAN/DroneCAN 自定义报文交换天地端 `tun` 网卡 MAC 地址，让两端在物理接触建立 CAN 链路后自动完成 MAC 上报、ACK 确认和后续配对触发准备。

## 链路角色

```text
RK3562 地面端
  can1: RK3562 SoC CAN
  tun: P401 图传相关网卡
  PeripheralManager / gcs_peripheral_manager: 地面端 CAN pairing 支持

RK3588 天空端
  can0: 外置 MCP2515/XL2515 over SPI
  tun: P401 图传相关网卡
  sky_can_mac_exchange.sh / rk3588_mac_pair_node.py: 天空端 MAC 交换脚本/节点
```

## 协议设计

当前协议基于扩展 CAN 帧，沿用 DroneCAN/UAVCAN v0 的消息布局思路：

```text
CAN ID = priority(7) << 24 | DTID << 8 | source_node_id
Data[0..5] = tun MAC，二进制 6 字节
Data[6] = mac_type，1 表示 TUN MAC
Data[7] = DroneCAN single-frame tail byte，0xC0 | transfer_id
```

关键 DTID：

```text
MAC report: 8195 / 0x2003
ACK:        8196 / 0x2004
Result:     8197 / 0x2005，预留
```

节点约定：

```text
天空端 node id: 1
地面端 node id: 2
```

典型报文：

```text
天空 -> 地面: 07200301#<天空_tun_mac>01C0
地面 -> 天空: 07200402#<天空_tun_mac>01C0  ACK

地面 -> 天空: 07200302#<地面_tun_mac>01C0
天空 -> 地面: 07200401#<地面_tun_mac>01C0  ACK
```

## 当前验证状态

可以稳妥表达：

- 地面端 RK3562 `can1` 上的 CAN pairing 支持已部署。
- 地面端能读取本地 `tun` MAC，并在启动阶段发送多帧 MAC report。
- 地面端收到模拟天空端 `07200301` report 后，能解析 peer MAC、发送 `07200402` ACK，并在 `dry_run=true` 下跳过真实 P401 配对钩子。
- 手动恢复到稳定 CAN 状态时，曾观察到天空 report、地面 ACK、地面 report 的有效交换帧。

不能夸大：

- 真实 P401 配对 API 尚未确认，当前保持 `dry_run=true`。
- 天空端自动服务、高频自动 TX、反复接触/断开后的长期稳定性尚未完整验证。
- 当前主要风险不在 MAC 报文格式，而在天空端外置 MCP2515/XL2515 CAN 接收链路。

## 问题定位边界

当前关键工程结论：

```text
MAC 交换协议和地面端脚本基本可用。
阻塞点主要在天空端外置 MCP2515/XL2515 接收链路：
  SocketCAN 可能显示 ERROR-ACTIVE
  但 sky candump 仍可能 RX=0 或错误解码
```

因此不能只用：

```text
ip -details link show can0 显示 ERROR-ACTIVE
```

就判断链路恢复。更可靠的成功判据是：

```text
天空端 candump can0 能正确看到地面端 07015501 / 07015502 / 5B5 / 07200302
```

## 面试讲法

可以这样讲：

> P401 图传无感配对的目标是减少人工配置，让天地两端在 CAN 接触链路建立后自动交换 `tun` 网卡 MAC。我的工作集中在 RK3562 地面端和 CAN/DroneCAN 报文链路：定义 MAC report、ACK 和预留 result DTID，payload 中携带 6 字节 MAC、MAC 类型和 DroneCAN tail byte；地面端启动时读取本地 `tun` MAC 并周期上报，收到天空端 report 后解析 peer MAC、回 ACK，并在 dry-run 下记录配对钩子。验证时用 `candump`、服务日志和模拟天空端报文确认地面端上报、解析和 ACK 都正常。后续联调发现主要阻塞不在 MAC 协议，而在天空端外置 MCP2515 接收链路，存在 ERROR-PASSIVE、RX 为 0 或错误解码，所以自动天空端服务和反复接触场景还需要硬件链路继续验证。

## 高频追问

### 为什么要交换 MAC？

P401 图传链路涉及天地两端网络接口识别和后续配对。通过 CAN 在物理接触阶段交换 `tun` MAC，可以减少手动读取/输入 MAC 的步骤，为后续自动配对或 vendor pairing API 调用提供 peer 标识。

### 为什么用 CAN/DroneCAN 报文？

现有系统里已经有 CAN/DroneCAN 链路和 SocketCAN 调试工具。把 MAC exchange 做成 CAN report/ACK，可以复用现有 `can1/can0` 通道、节点 ID、DTID 和抓包验证方式。

### 为什么要 ACK？

MAC report 是配对流程里的关键状态。ACK 能确认对端确实收到并解析了 MAC，避免只发送不确认。重复 report 可以通过去重窗口处理，重复 report 仍可 ACK，但不重复触发真实配对钩子。

### 为什么保留 `dry_run=true`？

真实 P401 配对命令尚未最终确认，且天空端 CAN 接收链路仍有稳定性风险。`dry_run=true` 可以先验证 CAN 协议、日志和 ACK 链路，避免在硬件链路不稳定时误触发真实配对。

### 天空端问题怎么定位？

先看 SocketCAN 状态和错误计数，再用 `candump` 观察是否能正确接收地面端已知帧。如果 `ERROR-ACTIVE` 但 RX 为 0 或解码出错误 ID，就继续往 MCP2515/XL2515、SIT1042 收发器、RXD/RXCAN、复位/供电、共模、终端电阻、采样点和接触状态排查。

## 简历边界

建议简历写：

> 参与 P401 图传链路无感快速配对方案开发，基于天地端 `tun` MAC 和 CAN/DroneCAN 自定义报文实现 MAC 上报、ACK 确认和去重处理，并在 RK3562 地面端完成启动上报、模拟天空端报文注入和 ACK 验证。

不建议写：

> 独立完成 P401 全自动无感配对并验证量产稳定。
