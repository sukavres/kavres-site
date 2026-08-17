---
title: 'H3C_S2126-EI_SNMP配置指南'
published: 2026-08-17
category: '数据通信'
draft: false
---

# H3C S2126-EI SNMP 配置指南

> **命名规范**：`品牌_型号_SNMP配置指南.md`（示例：`H3C_S2126-EI_SNMP配置指南.md`）。
> **本文定位**：**SNMP 及相关监控运维**配置指南（时间 / VLAN 管理 IP / SNMP / Syslog / 端口镜像），单独成文以控制单篇长度。
> **命令强绑定品牌/版本**：以下命令均在 H3C S2126-EI（Comware V3.10）真机验证，其他设备请以实际 `?` 帮助信息为准，不可跨设备套用。
> 🔐 **SSH 登录配置不在本文**，详见同设备《H3C_S2126-EI_SSH配置指南》。
> 📌 **本文档范围**：覆盖时间（⚠️ 存疑）、VLAN 管理接口、SNMP、Syslog、端口镜像；端口镜像章节命令标注「存疑」，需真机验证后使用。时间章节命令同样基于 Comware V3.10 通用语法整理，尚未在本机真实验证，部署前请以 `?` 确认。

---

## 一、设备信息

| 项目 | 内容 |
|---|---|
| 设备型号 | H3C S2126-EI |
| 软件版本 | Comware V3.10, Release 2211P06 |
| 硬件版本 | REV.A |
| Bootrom | 610 |
| 内存 / Flash | 64MB SDRAM / 8MB |
| 端口 | 24×FE + 2×GE（Subslot 0/1/2） |
| 管理 IP | 192.168.1.250（Vlan-interface 1） |
| 网管 / Syslog 服务器 | 192.168.1.240 |
| 文档生成日期 | 2026-08-13 |

> SSH 登录相关（用户名 / 密码 / SSH 状态）见《H3C_S2126-EI_SSH配置指南》。

---

## 二、时间配置（⚠️ 存疑，需真机验证）

> ⚠️ **本节命令尚未在本机真实验证**，基于 H3C Comware V3.10 通用语法整理，仅供参照。实际部署前请在真机用 `?` 确认命令是否存在、参数格式是否一致，跑通后再 `save`。时间准确性直接影响 Syslog / Trap 时间戳，建议配置。

### 2.1 设置系统时间

> 注意：时间相关命令在**用户视图 `<H3C>`** 下执行，**不在系统视图**。

```bash
# 在用户视图执行（不要进 system-view）
<H3C> clock datetime 21:00:00 2026/08/17
```

> ⚠️ **存疑点（需真机确认）**：日期格式各版本可能不同，常见写法有 `2026/08/17`、`2026-08-17`、`08/17/2026`，以本机 `clock datetime ?` 帮助信息为准。将示例时间替换为你配置时的实际时间。

### 2.2 设置时区

```bash
# 在用户视图执行
<H3C> clock timezone Beijing add 08:00:00
```

> ⚠️ **存疑点（需真机确认）**：
> - 时区名 `Beijing` 各版本可能不同，部分版本用 `GMT+8` 或 `UTC+8`，以 `clock timezone ?` 为准。
> - `add 08:00:00` 表示在 UTC 基础上**加 8 小时**（即东八区）。
> - ⚠️ Comware V3.10 的时区/夏令时设置通常**无法随 `save` 持久化**，设备重启后可能丢失，需依赖 NTP 或每次手动设定。真机确认本机行为。

### 2.3 验证

```bash
display clock
```

> 预期：显示正确的日期、时间与时区（如 `Beijing Time`）。若重启后时间归零，说明未持久化，需加装 NTP 或纳入开机脚本。

---

## 三、VLAN / 管理接口配置

### 4.1 配置管理 VLAN 接口 IP

```bash
system-view
interface Vlan-interface 1
 ip address 192.168.1.250 255.255.255.0
 quit
```

> 说明：本设备管理 IP 为 `192.168.1.250`（位于 Vlan-interface 1），与 SSH 文档、Trap/Syslog 源接口保持一致。若管理 VLAN 不是 VLAN 1，替换成实际的 VLAN 接口号。

### 4.2 验证

```bash
display ip interface Vlan-interface 1
```

> 状态应为 up，IP 与设备信息表一致。若此前已配置管理 IP，可跳过本节。

---

## 四、SNMP 配置

### 4.1 前置知识（关键概念与命令视图）

**关键概念**

| 概念 | 说明 |
|---|---|
| **SNMP Agent** | 设备上的 SNMP 服务进程，负责响应网管请求 |
| **Community（团体名）** | SNMPv1/v2c 的"密码"，分只读（RO）和读写（RW） |
| **Trap** | 设备**主动**向网管推送的告警消息 |
| **Target Host** | 接收 Trap 的网管服务器地址 |
| **MIB** | 管理信息库，定义设备可被监控的参数树 |

> ⚠️ **Comware V3.10 是老版本**，很多命令的视图层级和新版（V7）不同：
> - `snmp-agent` 相关命令 → **系统视图 `[H3C]`**
> - `clock` 相关命令 → **用户视图 `<H3C>`**（不在系统视图！）
> - `info-center` 相关命令 → **系统视图 `[H3C]`**

### 4.2 开启 SNMP Agent

```bash
system-view
snmp-agent
```

> 这是最基础的一步，**不开启 Agent，后面所有配置都不会生效**。执行后无回显提示即为成功。

### 4.3 配置版本与团体名

```bash
snmp-agent sys-info version v2c
snmp-agent community read public
snmp-agent community write private
```

> 说明：
> - `v2c` 当前最常用；若网管同时支持 v1/v2c，可写 `snmp-agent sys-info version v1 v2c`。
> - 将 `public` / `private` 替换为你自己的团体名，建议用复杂字符串，如 `MyNMS@2026!`。
> - ⚠️ 读写团体名权限很大，若网管只做监控，配只读即可。

### 4.4 启用 Trap（逐类开启）

在 Comware V3.10 中，`snmp-agent trap enable` **必须指定类别**，不能直接回车。

| 类别 | 说明 | 推荐 |
|---|---|---|
| `system` | 系统级告警（端口 UP/DOWN、风扇/电源故障） | ✅ 必开 |
| `standard` | RFC 标准通用告警 | ✅ 推荐 |
| `configuration` | 配置变更告警（有人改了配置） | ✅ 推荐 |
| `flash` | Flash 操作告警（备份/擦除配置） | 按需 |

```bash
snmp-agent trap enable system
snmp-agent trap enable standard
snmp-agent trap enable configuration
snmp-agent trap enable flash
```

> 💡 **最关键的是 `system`**！不开启它，端口 DOWN、风扇故障等告警**不会**推送给网管。

### 4.5 配置 Trap 目标主机与源接口

```bash
snmp-agent target-host trap address udp-domain 192.168.1.240 params securityname public v2c
snmp-agent trap source Vlan-interface 1
```

| 参数 | 含义 | 示例 |
|---|---|---|
| `192.168.1.240` | 网管服务器 IP 地址 | 替换成你的实际 IP |
| `public` | 团体名（要和网管平台一致） | 替换成你的团体名 |
| `v2c` | SNMP 版本 | 和网管平台保持一致 |

> 指定 Trap 源接口后，Trap 将以管理 IP `192.168.1.250` 为源，便于网管识别设备。若有**多台网管服务器**，重复执行 `target-host` 命令即可。

### 4.6 验证

```bash
display snmp-agent
display snmp-agent community
display snmp-agent trap
display snmp-agent target-host
```

预期：Agent 为 `Enabled`、版本 `v2c`；各模块 Trap（`system/standard/configuration/flash`）为 `Enabled`；目标主机 IP / 端口（默认 162）/ 版本正确。

**测试 Trap 是否可达**：在网管服务器监听 UDP 162（如 Wireshark），手动触发端口状态变化：

```bash
system-view
interface Ethernet 1/0/1
shutdown
undo shutdown
quit
```

网管应在几秒内收到 `linkDown` / `linkUp` 的 Trap 报文。

---

## 五、Syslog 配置

> 说明：Syslog（远程日志）与 SNMP Trap 是**互补**关系——Trap 是"告警主动推送"，Syslog 是"全量日志外发"。三条通道（**Console 显示 / SNMP Trap / Syslog**）彼此独立，互不影响。

### 5.1 关键概念

| 概念 | 说明 |
| --- | --- |
| **Info-Center** | H3C 的日志中心，统一管理日志 / 告警 / 调试信息的输出通道 |
| **Loghost（日志主机）** | 接收 Syslog 报文的远程服务器（如 Graylog / ELK / Splunk / Kiwi） |
| **facility** | Syslog 设施标识（local0~local7），用于在日志服务器侧区分不同设备 |
| **source 接口** | 发送 Syslog 报文所使用的源 IP，建议固定为管理 VLAN 接口 |

### 5.2 开启日志中心

```bash
info-center enable
```

> 部分版本默认已开启，但显式执行一次最稳妥；可用 `display info-center` 确认首行状态为 `Enabled`。

### 5.3 指定日志主机

```bash
info-center loghost 192.168.1.240
```

> 默认使用 **UDP 514** 端口。若日志服务器端口不同，可追加 `port <端口号>`；如需指定 facility，追加 `facility local4`（local0~local7 任选，需与服务器侧一致）。

### 5.4 指定日志源接口

```bash
info-center loghost source Vlan-interface 1
```

> 固定用管理 VLAN 接口的 IP（192.168.1.250）作为 Syslog 源地址，便于日志服务器识别设备。若交换机有多个 IP，不配此项可能选到不可达地址，导致服务器收不到日志。

### 5.5 配置日志级别（可选）

```bash
info-center source default channel loghost log level informational
```

> 允许 `informational` 及以上级别日志发往日志主机（覆盖告警、通知、普通事件）。
> ⚠️ 若你的版本对该命令报语法错误，可先只配 `info-center loghost` + `source`，默认即发送 `warning` 及以上日志；再用 `info-center source ?` 确认本机支持的写法。

### 5.6 验证

```bash
display info-center
display info-center loghost
```

> 测试：在日志服务器侧监听 UDP 514，手动 `shutdown` / `undo shutdown` 某个端口，应在数秒内收到对应日志。

---

## 六、端口镜像配置（⚠️ 存疑，需真机验证）

> ⚠️ **本节命令尚未在本机真实验证**，基于 H3C Comware V3.10 通用语法整理，仅供参照。实际部署前请在真机用 `?` 确认命令是否存在、端口命名是否正确，跑通后再 `save`。若命令报错，以本机 `?` 帮助信息为准。

### 6.1 关键概念

| 概念 | 说明 |
| --- | --- |
| **本地端口镜像** | 将指定源端口的进出流量复制到监控端口，供抓包机 / IDS 分析 |
| **mirroring-port（源端口）** | 被监听的端口（业务流量经过的口） |
| **monitor-port（监控端口 / 目的端口）** | 接抓包机的端口，只收不发 |
| **mirroring-group** | 镜像组，本地镜像用 `local` 类型 |

> 典型场景：把上联口或某业务口的流量镜像到接了 Wireshark 电脑的口，排查异常流量、协议交互、环路等。

### 6.2 配置端口镜像（存疑）

```bash
system-view
# 创建本地镜像组 1
mirroring-group 1 local

# 指定被镜像的源端口（both = 双向；可改为 inbound / outbound）
# ⚠️ 端口命名以本机为准：S2126-EI 为 24×FE + 2×GE，
#    FE 口形如 Ethernet 1/0/X，GE 口形如 GigabitEthernet 1/0/X（槽位号以 display interface brief 为准）
mirroring-group 1 mirroring-port GigabitEthernet 1/0/1 both

# 指定监控端口（接抓包机）
mirroring-group 1 monitor-port GigabitEthernet 1/0/2

# 在监控端口上关闭 STP，避免其参与拓扑计算（推荐）
interface GigabitEthernet 1/0/2
 undo stp enable
 quit
```

> ⚠️ **存疑点（需真机确认）**：
> - `mirroring-group` 系列命令在 Comware V3.10 是否完整支持，需真机 `?` 确认；个别老版本可能改用接口视图下的 `mirroring-port` / `monitor-port` 写法。
> - 端口槽位号、类型（Ethernet / GigabitEthernet）必须按本机 `display interface brief` 实际值填写，不可照搬示例。
> - 监控端口不要同时承载业务流量；监控口关闭 STP 后切勿再接环路。

### 6.3 验证

```bash
display mirroring-group all
```

> 预期：Type 为 `Local`，Status 为 `Active`，Mirroring port / Monitor port 显示正确。若 Status 为 `Incomplete`，说明源口或监控口未配全。

### 6.4 保存配置

> 本设备所有配置完成后统一执行一次 `save` 即可（见第七章一键脚本末尾），本节不单独保存。

---

## 七、完整一键配置脚本

以下是从头到尾的完整配置，可直接复制粘贴（末尾统一保存一次）：

```bash
# === 系统时间（⚠️ 存疑，用户视图执行，需在 system-view 之前）===
clock datetime 21:00:00 2026/08/17
clock timezone Beijing add 08:00:00

system-view

# === VLAN 管理接口 IP（与 SSH 文档统一为 192.168.1.250）===
interface Vlan-interface 1
 ip address 192.168.1.250 255.255.255.0
 quit

# === SNMP 基础配置 ===
snmp-agent
snmp-agent sys-info version v2c
snmp-agent community read public
snmp-agent community write private

# === SNMP Trap 配置 ===
snmp-agent trap enable system
snmp-agent trap enable standard
snmp-agent trap enable configuration
snmp-agent trap enable flash
snmp-agent target-host trap address udp-domain 192.168.1.240 params securityname public v2c
snmp-agent trap source Vlan-interface 1

# === Syslog 日志服务器配置 ===
info-center enable
info-center loghost 192.168.1.240
info-center loghost source Vlan-interface 1
info-center source default channel loghost log level informational

# === 端口镜像（⚠️ 存疑，需真机验证后使用）===
mirroring-group 1 local
mirroring-group 1 mirroring-port GigabitEthernet 1/0/1 both
mirroring-group 1 monitor-port GigabitEthernet 1/0/2
interface GigabitEthernet 1/0/2
 undo stp enable
 quit

return
save
```

> 📌 **使用前请修改以下参数：**
> - `21:00:00 2026/08/17` → 你配置时的实际时间（⚠️ 时间命令尚未真机验证，日期格式/时区名以本机 `?` 为准）
> - `192.168.1.250` → 你的交换机管理 IP（若已配置可跳过 VLAN 段）
> - `public` → 你的只读团体名
> - `private` → 你的读写团体名
> - `192.168.1.240` → 你的网管 / Syslog 服务器 IP
> - `Vlan-interface 1` → 你的管理 VLAN 接口
> - 端口镜像段端口号（GigabitEthernet 1/0/1 / 1/0/2）需按本机 `display interface brief` 实际值替换，且该段命令尚未真机验证，务必先单独测试

---

## 八、功能验证清单

配置完成后，逐项确认：

- [ ] `display clock` → 日期/时间/时区正确（⚠️ 存疑命令，需真机验证）
- [ ] `display ip interface Vlan-interface 1` → 管理 IP 为 192.168.1.250 且状态 up
- [ ] `display snmp-agent` → Agent 状态为 Enabled
- [ ] `display snmp-agent community` → 团体名正确
- [ ] `display snmp-agent trap` → 各模块 Enabled
- [ ] `display snmp-agent target-host` → 网管 IP 正确（192.168.1.240:162）
- [ ] `display info-center` → 状态为 Enabled（Syslog 前置）
- [ ] `display info-center loghost` → 日志服务器 IP 正确（192.168.1.240:514）
- [ ] `display mirroring-group all` → 镜像组状态为 Active，源 / 监控口正确（⚠️ 存疑命令，需真机验证）
- [ ] 网管平台能正常采集数据
- [ ] 网管平台能收到 Trap 告警
- [ ] 日志服务器能收到 Syslog 日志
- [ ] Console 口不再刷屏（见第九章）
- [ ] 配置已 `save` 保存

---

## 九、Console 口日志与 SNMP Trap / Syslog 的关系

### 9.1 三条完全独立的通道

| 通道 | 控制命令 | 作用 |
| --- | --- | --- |
| **Console 显示** | `info-center console ...` | 控制屏幕上的日志显示 |
| **SNMP Trap 上报** | `snmp-agent trap ...` | 控制发给网管的告警 |
| **Syslog 上报** | `info-center loghost ...` | 控制发给日志服务器的运行日志 |

**关闭 Console 日志显示，完全不会影响 Trap 与 Syslog 上报！**

### 9.2 推荐的 Console 静音配置

```bash
system-view
# 方案 A：只屏蔽端口 UP/DOWN 刷屏（推荐）
info-center source L2INF channel console deny

# 方案 B：所有 Console 日志级别提到 Warning
info-center console level warning

return
save
```

**效果：**
- ✅ Console 口清净，不再刷端口日志
- ✅ SNMP Trap 照常发给网管主机
- ✅ Syslog 照常发给日志服务器
- ✅ 网管平台依然能收到 linkDown/linkUp 告警

---

## 十、常见问题排错

### 问题 1：`snmp-agent trap enable` 后面只能接类别名，不能直接回车

**原因：** Comware V3.10 不支持 `snmp-agent trap enable` 一键全开，必须逐个指定。

**解决：** 依次执行 `system` / `standard` / `configuration` / `flash`（见 4.4）。

---

### 问题 2：网管平台收不到 Trap

**排查清单：**

| 检查项 | 命令 | 确认内容 |
| --- | --- | --- |
| 网管 IP 是否正确 | `display snmp-agent target-host` | IP 地址无误 |
| 团体名是否一致 | `display snmp-agent community` | 和网管平台配置一致 |
| 版本是否匹配 | `display snmp-agent` | 网管平台选的 v2c |
| 网络是否可达 | `ping 192.168.1.240` | 交换机到网管服务器通 |
| 防火墙是否放行 | — | UDP 162 端口未拦截 |
| Trap 功能是否开启 | `display snmp-agent trap` | 各模块为 Enabled |

---

### 问题 3：Trap 报文里的源 IP 不是管理地址

**原因：** 未指定 Trap 源接口，设备用出接口 IP 作为源地址。

**解决：**
```bash
system-view
snmp-agent trap source Vlan-interface 1
return
save
```

---

### 问题 4：日志服务器收不到 Syslog

**排查清单：**

| 检查项 | 命令 | 确认内容 |
| --- | --- | --- |
| 信息中心是否开启 | `display info-center` | 首行状态为 `Enabled` |
| 日志主机 IP 是否正确 | `display info-center loghost` | IP 地址无误 |
| 源接口是否配置 | `info-center loghost source Vlan-interface 1` | 该接口已配 IP 且可达 |
| 网络是否可达 | `ping 192.168.1.240` | 交换机到日志服务器通 |
| 端口是否放行 | — | UDP 514 端口未拦截 |
| facility 是否匹配 | `info-center loghost ... facility local4` | 与日志服务器侧一致 |

---

### 问题 5：Console 口狂刷端口 UP/DOWN 日志

**原因：** 某端口链路不稳定（如 Ethernet1/0/17），反复震荡。

**临时解决（止住刷屏）：**
```bash
system-view
info-center source L2INF channel console deny
return
save
```

**根因排查：**
1. 检查该端口网线水晶头是否氧化/松动，换线测试
2. 检查对端设备是否反复重启
3. 尝试手动固定速率双工：
```bash
system-view
interface Ethernet 1/0/17
speed 100
duplex full
return
save
```

---

### 问题 6：系统时间/时区不对或重启后丢失（⚠️ 存疑章节相关）

**排查清单（针对第二章，需真机验证）：**

| 检查项 | 命令 | 确认内容 |
| --- | --- | --- |
| 当前时间 | `display clock` | 日期/时间正确 |
| 时区是否生效 | `display clock` | 显示 `Beijing Time` 等本地时区 |
| 命令是否支持 | `clock datetime ?` / `clock timezone ?` | 本机日期格式、`Beijing` 时区名可用 |
| 重启后是否丢失 | 重启后 `display clock` | V3.10 时区常无法 `save` 持久化 |

> 若时区重启后丢失，建议加装 NTP 时钟同步（需网内有 NTP 服务器），或在开机/上线脚本中固化时间设置命令。

---

### 问题 7：端口镜像不生效 / 命令报错（⚠️ 存疑章节相关）

**排查清单（针对第六章，需真机验证）：**

| 检查项 | 命令 | 确认内容 |
| --- | --- | --- |
| 镜像组是否创建 | `display mirroring-group all` | 存在 group 1 且 Type 为 Local |
| 状态是否 Active | `display mirroring-group all` | Status 为 `Active`（若为 `Incomplete` 说明源/监控口未配全） |
| 命令是否支持 | `mirroring-group ?` | 本机确实有 `local` / `mirroring-port` / `monitor-port` 关键字 |
| 端口命名是否正确 | `display interface brief` | 源/监控口的实际名称与配置一致 |
| 监控口是否接对 | — | 抓包机接在 monitor-port 所在物理口 |

> 若 `mirroring-group` 命令在本机报"未知命令"，说明该 build 可能用接口视图写法（进源口/监控口后配 `mirroring-port` / `monitor-port`），请以 `?` 帮助信息为准重新组织。

---

## 十一、安全加固建议

### 11.1 使用复杂团体名

不要用 `public`、`private` 这种默认值，容易被扫描工具利用：

```bash
snmp-agent community read SnmP@R0_M0n1t0r_2026
```

### 11.2 限制 SNMP 访问源（ACL）

只允许网管服务器访问 SNMP Agent：

```bash
system-view
acl number 2000
 rule permit source 192.168.1.240 0   # 只允许网管/Syslog 服务器
 rule deny source any                  # 拒绝其他所有
 quit

snmp-agent community read SnmP@R0_M0n1t0r_2026 acl 2000
return
save
```

### 11.3 如条件允许，升级到 SNMPv3

SNMPv3 支持用户认证和报文加密，安全性远高于 v2c。S2126-EI 的 Comware V3.10 支持 v3，配置方式如下（简要）：

```bash
snmp-agent sys-info version v3
snmp-agent group v3 NMSGroup privacy
snmp-agent usm-user v3 nmsadmin NMSGroup cipher Auth@2026! privacy Priv@2026!
snmp-agent target-host trap address udp-domain 192.168.1.240 params securityname nmsadmin v3 privacy
```

---

## 十二、配置前后对比

| 项目 | 配置前 | 配置后 |
| --- | --- | --- |
| 系统时间 | 可能不准确/未设时区 | ⚠️ 已补充配置（命令存疑，待真机验证） |
| SNMP Agent | 未开启 | ✅ 已开启（v2c） |
| 网管监控 | 不可监控 | ✅ 可采集数据 |
| Trap 上报 | 无 | ✅ system/standard/configuration/flash 全开 |
| 网管目标 | 无 | ✅ 192.168.1.240:162 |
| Syslog 外发 | 无 | ✅ 192.168.1.240:514 |
| 端口镜像 | 无 | ⚠️ 已补充配置（命令存疑，待真机验证） |
| Console 刷屏 | 狂刷端口日志 | ✅ 已静音（L2INF 屏蔽） |
| 监控连续性 | 断断续续 | ✅ 稳定上报 |

---

## 十三、注意事项

1. **统一保存**：所有配置完成后执行一次 `save` 即可（见第七章一键脚本末尾），无需每章单独保存；排错/加固章节里的 `save` 为独立故障处理动作，可视情况执行。
2. **命令品牌相关**：SNMP 的 Trap 开启方式（一行全开 vs 逐类开）、Syslog 的日志中心命令（`info-center` 系）在不同品牌/版本差异大，本文命令仅适用于 Comware V3.10，不可跨设备套用。
3. **管理 IP**：设备管理 IP `192.168.1.250` 与网管/Syslog 服务器 `192.168.1.240` 需同步修改 VLAN 段与一键脚本、Syslog 段。
4. **SSH 不在本文**：登录认证配置见同设备《H3C_S2126-EI_SSH配置指南》。
5. **Console 静音独立**：关闭 Console 显示不影响 Trap/Syslog，三条通道互不影响（见第九章）。
6. **团体名安全**：生产环境务必使用复杂团体名并配合 ACL（见第十一章），避免 `public`/`private` 默认值。
7. **端口镜像存疑**：第六章端口镜像命令基于 Comware V3.10 通用语法整理，未在本机真实验证；实际部署前请用 `?` 确认命令与端口命名，在真机跑通后再 `save`（见问题 7 排查）。
8. **时间配置存疑**：第二章时间命令基于 Comware V3.10 通用语法整理，未在本机真实验证；日期格式（`2026/08/17` 等）、时区名（`Beijing` 等）及持久化行为请以本机 `?` 与 `display clock` 为准（见问题 6 排查）。

---

> 📝 **文档说明：** 本文档基于 H3C S2126-EI（Comware V3.10, Release 2211P06）实测整理，不同版本命令可能略有差异，请以设备实际 `?` 帮助信息为准。时间、端口镜像章节为补充内容，命令未经验证，标注「存疑」。

---

## 文档信息

- 文档生成时间：2026-08-17
- 适用设备：H3C S2126-EI（Comware V3.10，Release 2211P06）
- 命令来源：真机 + AI 沟通验证（时间配置、端口镜像章节标注「存疑」需真机复核）

