---
title: 'H3C-S2126-EI交换机-SSH配置指南'
published: 2026-08-16
category: '数据通信'
draft: false
---

# H3C S2126-EI SSH 配置指南（规整版）

> 设备型号：H3C S2126-EI
> 系统版本：Comware V3.10, Release 2211P06
> 本文重点：**SSH 远程管理配置**（主体）。

> ⚠️ **版本差异提醒**：此设备运行 **Comware V5 内核 + V3.10 命令行风格**，部分 V7 命令（如 `authorization-attribute`、`ssh server enable`、`local-user xxx class manage`）**不存在或行为不同**，本文一律使用兼容写法。

---

## 一、设备信息与重要说明

### 1.1 设备基本信息

| 项目 | 内容 |
| --- | --- |
| 设备型号 | H3C S2126-EI |
| 系统软件 | Comware V3.10, Release 2211P06 |
| 运行时间 | 124 周 + 2 天 |
| 内存 / Flash | 64M SDRAM / 8M Flash |
| 端口 | 24 个 FE + 2 个 FE（堆叠口） |

### 1.2 与本指南相关的版本要点

- **SSH 默认即运行**：V3.10 上 SSH 服务在生成 RSA 密钥后默认开启，`ssh server enable` 这条命令在部分版本**不存在**，属正常现象（见排错 `5.1`）。
- **权限写法不同**：不支持 `class manage`，权限级别直接写在 `service-type` 后面，如 `service-type ssh level 3`。
- **命令隐藏机制**：未生成 RSA 密钥前，SSH 相关命令会被系统隐藏，表现为"命令未识别"。

---

## 二、配置目标

| 需求 | 实现方式 |
| --- | --- |
| 通过 SSH 安全远程管理 | 创建本地用户 + 配置 VTY 仅允许 SSH |
| （可选）Console 口不再刷端口震荡日志 | 日志级别提到 warning，或屏蔽 L2INF 模块（见文末附录） |
| SNMP Trap 照常上报网管 | 不受影响，Trap 走独立通道（与 Console 显示互不干扰） |

---

## 三、SSH 配置完整流程（主体）

### 3.1 生成 RSA 密钥（SSH 前置，必须第一步）

```text
public-key local create rsa
```

> **注释**：SSH 的前置条件。执行后系统提示输入密钥长度，**直接回车使用默认值 1024** 即可。看到 `Create RSA keys successfully`（界面显示 `++++++` 进度条）即表示成功。
>
> ⚠️ **关键原理**：未生成密钥时，`ssh server enable` 等 SSH 命令会被系统隐藏，导致"命令未识别"报错。生成密钥后所有 SSH 命令才会显现。

### 3.2 配置管理 IP（VLAN 接口，SSH 登录地址）

```text
interface Vlan-interface 1
 ip address 192.168.1.250 255.255.255.0
 quit
```

> **注释**：配置交换机的管理 IP，即 SSH 客户端连接时填的地址。必须与本文 **4.2 登录测试** 以及 **SNMP 文档中的设备管理 IP** 保持一致——本例统一为 `192.168.1.250`、位于 `Vlan-interface 1`。若管理 VLAN 不是 1，替换接口名与对应 IP 即可。

### 3.3 创建本地用户并配置权限

```text
system-view
local-user sshadmin
 password simple Admin@2024!
 service-type ssh level 3
 quit
```

> **注释**：
> - `sshadmin`：用户名，按需修改。
> - `Admin@2024!`：密码，建议改为强密码（含大小写+数字+特殊字符）。
> - `level 3`：最高管理权限级别。
> - ⚠️ 此设备**不支持** V7 的 `class manage` 写法，权限级别直接写在 `service-type` 后。
> - 💡 `password simple` 以明文形式保存配置中的密码；若设备支持 `password cipher`，建议改用 `password cipher <密文>` 提高安全性。

### 3.4 指定 SSH 用户认证方式

```text
ssh user sshadmin authentication-type password
```

> **注释**：老版本 Comware V5/V3 必须显式指定 SSH 用户的认证方式，否则可能登录失败。

### 3.5 配置 VTY 线路（强制仅允许 SSH）

```text
user-interface vty 0 4
 authentication-mode scheme
 protocol inbound ssh
 quit
```

| 命令 | 作用 |
| --- | --- |
| `authentication-mode scheme` | 使用本地账号密码认证（AAA 本地方案） |
| `protocol inbound ssh` | 关闭 Telnet，仅允许 SSH 接入 |

### 3.6 保存配置

```text
save
```

> **注释**：输入 `y` 确认覆盖保存，否则重启后配置丢失。

---

## 四、验证与 SSH 登录测试

### 4.1 验证命令

```text
# 查看 SSH 服务状态（应显示 Running）
display ssh server status

# 查看本地用户配置
display local-user username sshadmin

# 查看 VTY 配置
display user-interface vty 0 4
```

### 4.2 SSH 客户端登录测试

使用 PuTTY / Xshell / Terminal 等 SSH 客户端连接：

| 项目 | 值 |
| --- | --- |
| 协议 | SSH |
| 端口 | 22 |
| 用户名 | sshadmin |
| 密码 | Admin@2024! |
| 地址 | 交换机管理 IP（即 VLAN 1 的 192.168.1.250，见 3.2 步配置） |

> 登录成功并显示命令行提示符即配置完成 ✅

---

## 五、常见问题与排错

### 5.1 输入 `ssh server enable` 提示"命令未识别"

| 原因 | 解决 |
| --- | --- |
| 设备版本无此命令 | **正常现象**，V3.10 默认 SSH 已运行，跳过即可 |
| RSA 密钥未生成 | 先执行 `public-key local create rsa`，密钥生成后 SSH 命令才会显现 |

### 5.2 输入 `local-user admin class manage` 提示"参数过多"

| 原因 | 解决 |
| --- | --- |
| 此设备不支持 `class manage` | 改用 `local-user <用户名>` 进入子视图，直接配 `password` 和 `service-type ssh level X` |

### 5.3 `undo service-type` 提示"命令不完整"

| 原因 | 解决 |
| --- | --- |
| V5 语法要求跟具体参数 | 改用 `service-type ssh level 3` 直接覆盖，不需要 undo |

### 5.4 配置前后对比

| 维度 | 配置前 | 配置后 |
| --- | --- | --- |
| 远程管理 | 仅 Console 口 | 支持 SSH 安全登录 |
| 登录方式 | Telnet/Console | 仅 SSH（已禁用 Telnet） |
| 用户权限 | — | sshadmin @ level 3（最高） |

---

## 六、完整命令流（可直接复制）

> 以下为 SSH 配置核心命令，复制后粘贴即可（不含文末 Console 可选部分）。

```text
system-view

# ---- 生成 RSA 密钥（SSH 前置）----
public-key local create rsa
# 提示时直接回车，使用默认值 1024

# ---- 配置管理 IP（SSH 登录地址，与 SNMP 文档一致）----
interface Vlan-interface 1
 ip address 192.168.1.250 255.255.255.0
 quit

# ---- 创建本地用户 ----
local-user sshadmin
 password simple Admin@2024!
 service-type ssh level 3
 quit

# ---- 指定 SSH 用户认证方式 ----
ssh user sshadmin authentication-type password

# ---- VTY 仅允许 SSH ----
user-interface vty 0 4
 authentication-mode scheme
 protocol inbound ssh
 quit

# ---- 保存 ----
save
```

---

## 七、附录：关闭 Console 刷屏日志（可选，推荐）

> 该需求与 SSH 配置**相互独立**，仅在"Console 口端口震荡日志刷屏影响运维"时才需要，故放在最后。

### 7.1 方案 A：日志级别提到 warning（推荐，最简单）

```text
system-view
info-center console level warning
quit
save
```

> **注释**：将 Console 通道日志级别提到 `warning`，端口 up/down 等 info 级震荡日志不再刷屏，但 warning 及以上仍可见。

### 7.2 方案 B：屏蔽 L2INF 模块（端口状态模块）

```text
system-view
info-center source L2INF channel console deny
quit
save
```

> **注释**：精确屏蔽 L2INF（二层接口）模块向 Console 通道的输出，针对性更强。

### 7.3 关键提醒：Console 与 SNMP Trap 是两条独立通道

> ⚠️ **核心要点**：Console 显示 ≠ SNMP Trap 上报。关闭 Console 刷屏**完全不影响** Trap 上报网管，两者互不影响，可放心配置。

---

> 📌 **总结**：H3C S2126-EI（Comware V3.10）配置 SSH 三步核心——`public-key local create rsa` → 本地用户 `service-type ssh level 3` → VTY `protocol inbound ssh`；注意老设备需用兼容写法，且 SSH 无需 `enable` 命令。Console 刷屏抑制为独立可选项，置于文末。

