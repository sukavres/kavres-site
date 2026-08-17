---
title: 'Quidway_S3328TP-EI_SNMP配置指南'
published: 2026-08-17
category: '数据通信'
draft: false
---

# Quidway S3328TP-EI 配置指南

> **命名规范**：`品牌_型号_SNMP配置指南.md`（本文件：`Quidway_S3328TP-EI_SNMP配置指南.md`）。
> **本文定位**：**SNMP 及相关监控运维**配置文档（时间 / VLAN 管理 IP / SNMP / Syslog / 端口镜像）。SSH 登录配置不在本文，详见《Quidway_S3328TP-EI_SSH配置指南》。
> ⚠️ **命令强绑定版本**：本文命令均按本机 VRP 5.30（V100R003C00SPC301）真机验证写法，不可跨设备套用。

---

## 一、设备信息

| 项目 | 值 |
|------|----|
| 型号 | Quidway S3328TP-EI |
| 软件版本 | VRP (R) Software, Version 5.30 (S3328 V100R003C00SPC301) |
| BootROM | 315 (2009-11-04) |
| 管理 IP | 192.168.1.101 (Vlanif1) |
| 网管/日志服务器 | 192.168.1.240 |
| SSH 用户名 | sshadmin |
| SSH 密码 | Admin@2024! |

---

## 二、时间配置

> ⚠️ 说明：本版本（V100R003）**系统视图下无 `clock` 命令**，`clock timezone` 无法持久化生效。  
> 务实方案：直接在用户视图下把时间设成北京时间即可。时区行保持 `Default Zone Name add 00:00:00`，不影响日志时间戳显示。

### 2.1 查看当前时间

```
<Quidway> display clock
```

### 2.2 设置北京时间（用户视图下执行）

```
<Quidway> clock datetime 17:10:00 2026-08-14
```

### 2.3 验证

```
<Quidway> display clock
2026-08-14 17:10:05 Friday
Time Zone : Default Zone Name add 00:00:00
```

> 显示时间即为北京时间，日志/Syslog 时间戳正确。

---

## 三、VLAN 1 配置（管理接口）

> 说明：VLAN 1 为默认管理 VLAN，本机管理 IP `192.168.1.101` 配置在 `Vlanif1` 三层接口上。  
> Syslog 源接口（`info-center loghost source Vlanif1`）与 SSH 登录均依赖此 IP，需先确保接口可达。

### 3.1 配置 Vlanif1 管理 IP

```
<Quidway> system-view
[Quidway] interface Vlanif 1
[Quidway-Vlanif1] ip address 192.168.1.101 255.255.255.0
[Quidway-Vlanif1] quit
```

> 若需修改管理地址，替换 `192.168.1.101` 与掩码即可；掩码 `255.255.255.0` 即 `/24`。

### 3.2 验证

```
<Quidway> display ip interface brief
<Quidway> display ip interface Vlanif 1
```

应能看到 `Vlanif1` 的 IP 为 `192.168.1.101/24`，且物理/协议状态为 `up`。

---

## 四、SNMP 配置（v2c）

### 4.1 启用 SNMP Agent

```
<Quidway> system-view
[Quidway] snmp-agent
```

### 4.2 设置 SNMP 版本

```
[Quidway] snmp-agent sys-info version v2c
```

### 4.3 配置只读团体字（无 ACL 限制）

```
[Quidway] snmp-agent community read cipher public@123
```

> 如需恢复 ACL 限制，可执行：  
> `snmp-agent community read cipher public@123 acl 2000`

### 4.4 设置设备位置与联系人

```
[Quidway] snmp-agent sys-info location DC3F-B04-Rack01
[Quidway] snmp-agent sys-info contact Admin-010-88888888
```

### 4.5 启用 Trap 并配置 Trap 目标

```
[Quidway] snmp-agent trap enable
[Quidway] snmp-agent target-host trap address udp-domain 192.168.1.240 params securityname monitor v2c
```

### 4.6 验证

```
<Quidway> display snmp-agent sys-info
<Quidway> display snmp-agent community read
<Quidway> display snmp-agent target-host
```

---

## 五、Syslog（Info-center）配置

### 5.1 启用 Info-center

```
<Quidway> system-view
[Quidway] info-center enable
```

### 5.2 配置日志主机

```
[Quidway] info-center loghost 192.168.1.240 facility local6
```

### 5.3 指定日志源接口（可选，推荐）

```
[Quidway] info-center loghost source Vlanif1
```

### 5.4 设置日志级别

```
[Quidway] info-center loghost level informational
```

### 5.5 验证

```
[Quidway] display info-center
```

应能看到：`Log host: 192.168.1.240, facility local6`

---

## 六、端口镜像（Port Mirroring）配置

> 说明：本版本镜像命令使用 `port-mirroring to observe-port X both` 语法（带 `to`）。  
> 观察口（Observe Port）= Ethernet0/0/10  
> 镜像源口（Mirrored Ports）= Ethernet0/0/1 ~ 0/0/9（双向）

### 6.1 定义观察口

```
<Quidway> system-view
[Quidway] observing-port 1 interface Ethernet 0/0/10
```

> ⚠️ 必须确保 Ethernet0/0/10 物理已接线（接抓包服务器）且端口为 up 状态。如被 shutdown：  
> `[Quidway] interface Ethernet 0/0/10` → `undo shutdown`

### 6.2 绑定镜像源口（1-9 口双向镜像到观察口 1）

```
[Quidway] interface Ethernet 0/0/1
[Quidway-Ethernet0/0/1] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/1] quit

[Quidway] interface Ethernet 0/0/2
[Quidway-Ethernet0/0/2] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/2] quit

[Quidway] interface Ethernet 0/0/3
[Quidway-Ethernet0/0/3] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/3] quit

[Quidway] interface Ethernet 0/0/4
[Quidway-Ethernet0/0/4] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/4] quit

[Quidway] interface Ethernet 0/0/5
[Quidway-Ethernet0/0/5] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/5] quit

[Quidway] interface Ethernet 0/0/6
[Quidway-Ethernet0/0/6] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/6] quit

[Quidway] interface Ethernet 0/0/7
[Quidway-Ethernet0/0/7] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/7] quit

[Quidway] interface Ethernet 0/0/8
[Quidway-Ethernet0/0/8] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/8] quit

[Quidway] interface Ethernet 0/0/9
[Quidway-Ethernet0/0/9] port-mirroring to observe-port 1 both
[Quidway-Ethernet0/0/9] quit
```

### 6.3 验证

```
<Quidway> display observing-port
<Quidway> display port-mirroring
```

### 6.4 保存配置

> 说明：以上所有配置（时间 / VLAN 1 / SNMP / Syslog / 端口镜像）完成后，只需执行一次 `save` 即可，**无需每个章节单独保存**。

```
<Quidway> save
```

---

## 七、完整一键配置脚本（汇总）

> 可直接复制粘贴到 Console 口执行（除镜像口需逐条进接口外）。

```
! ===== 时间 =====
clock datetime 17:10:00 2026-08-14

system-view

! ===== VLAN 1 管理接口 =====
interface Vlanif 1
 ip address 192.168.1.101 255.255.255.0
quit

! ===== SNMPv2c =====
snmp-agent
snmp-agent sys-info version v2c
snmp-agent community read cipher public@123
snmp-agent sys-info location DC3F-B04-Rack01
snmp-agent sys-info contact Admin-010-88888888
snmp-agent trap enable
snmp-agent target-host trap address udp-domain 192.168.1.240 params securityname monitor v2c

! ===== Syslog =====
info-center enable
info-center loghost 192.168.1.240 facility local6
info-center loghost source Vlanif1
info-center loghost level informational

! ===== 端口镜像（观察口） =====
observing-port 1 interface Ethernet 0/0/10

! ===== 端口镜像（源口 1-9，逐口绑定） =====
interface Ethernet 0/0/1
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/2
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/3
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/4
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/5
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/6
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/7
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/8
 port-mirroring to observe-port 1 both
quit
interface Ethernet 0/0/9
 port-mirroring to observe-port 1 both
quit

! ===== 保存（全部配置完成后统一保存） =====
quit
save
```

---

## 八、功能验证清单

| 功能 | 验证命令 | 期望结果 |
|------|---------|----------|
| 时间 | `display clock` | 显示北京时间 17:10 左右 |
| VLAN 1 管理 IP | `display ip interface Vlanif 1` | IP: 192.168.1.101/24，状态 up |
| SNMP 版本 | `display snmp-agent sys-info` | Version: v2c |
| SNMP 团体字 | `display snmp-agent community read` | 显示 public@123，无 ACL |
| SNMP Trap 目标 | `display snmp-agent target-host` | 192.168.1.240, v2c |
| Syslog | `display info-center` | Log host: 192.168.1.240, local6 |
| 镜像观察口 | `display observing-port` | Index 1, Eth0/0/10 |
| 镜像源口 | `display port-mirroring` | Eth0/0/1~9 → Observe-port 1, Both |

---

## 九、注意事项

1. **时区与保存**：本版本时区无法持久化，手动设北京时间即可；所有配置（时间 / VLAN 1 / SNMP / Syslog / 端口镜像）完成后统一 `save` 一次（见 六·6.4），无需每章单独保存。
2. **SNMP 安全性**：当前 community 无 ACL 限制，仅适合内网隔离环境。生产环境建议恢复 `acl 2000` 限制。
3. **镜像观察口**：Ethernet0/0/10 必须物理连接抓包服务器且端口 up，否则镜像流量丢弃。
4. **NTP**：本设备未配置 NTP，时间久了会漂移，建议定期手动校准或升级版本后配 NTP。
5. **VLAN 1 管理 IP**：本文档默认 `192.168.1.101/24`，需与实际网络一致；若修改，同步更新设备信息表与一键脚本中的地址。
6. **登录密码复杂度**：若修改设备登录密码（配置见 `Quidway_S3328TP-EI_SSH配置指南.md`），请保持大小写+数字+特殊字符的同等复杂度。

---

## 文档信息

- 文档生成时间：2026-08-17
- 适用设备：Quidway S3328TP-EI（VRP 5.30 V100R003C00SPC301）

