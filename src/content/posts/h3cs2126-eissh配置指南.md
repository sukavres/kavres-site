---
title: 'H3C_S2126-EI_SSH配置指南'
published: 2026-08-17
category: '数据通信'
draft: false
---

# H3C S2126-EI SSH 配置指南

> **命名规范**：`品牌_型号_SSH配置指南.md`（本文件：`H3C_S2126-EI_SSH配置指南.md`）。
> **本文定位**：**登录与安全基线**专用文档（SSH 远程管理 + 可选 Console 刷屏抑制）。同设备的 SNMP / Syslog / 端口镜像等监控运维配置见《H3C_S2126-EI_SNMP配置指南》。
> ⚠️ **命令强绑定版本**：本文命令均按本机 Comware V3.10 真机验证写法，不可跨设备套用（包括华为、思科等其它品牌）。

## 一、设备信息

| 项目 | 内容 |
| --- | --- |
| 设备型号 | H3C S2126-EI |
| 系统软件 | Comware V3.10, Release 2211P06 |
| 硬件 / 内存 | 64M SDRAM / 8M Flash |
| 端口 | 24×FE + 2×FE（堆叠口） |
| 管理 IP | 192.168.1.250/24 |
| 管理 VLAN | VLAN 1（Vlan-interface 1） |
| SSH 用户名 | sshadmin |
| SSH 密码 | Admin@2024! |
| 文档生成日期 | 2026-08-17 |

> ⚠️ **版本差异提醒（本机实测）**：
> - **SSH 默认即运行**：生成 RSA 密钥后 SSH 服务默认开启，`ssh server enable` 在本机不存在属正常现象，跳过即可（见 6.2）。
> - **权限写法不同**：不支持 `class manage`，权限级别直接写在 `service-type` 后，如 `service-type ssh level 3`。
> - **命令隐藏机制**：未生成 RSA 密钥前，SSH 相关命令会被系统隐藏，表现为"命令未识别"；生成密钥后才会显现。
> - **管理 IP 一致性**：本机管理 IP `192.168.1.250`（Vlan-interface 1）须与《H3C_S2126-EI_SNMP配置指南》保持一致。

## 二、SSH 配置

### 2.0 配置管理 IP（VLAN 接口，SSH 登录地址，前置）

```text
interface Vlan-interface 1
 ip address 192.168.1.250 255.255.255.0
 quit
```

> 配置交换机管理 IP，即 SSH 客户端连接时填的地址。须与《H3C_S2126-EI_SNMP配置指南》中的设备管理 IP 一致（本例 `192.168.1.250` / `Vlan-interface 1`）。若管理 VLAN 不是 1，替换接口名与对应 IP 即可。

### 2.1 生成本地 RSA 密钥对（必须第一步）

```text
public-key local create rsa
```

> SSH 的前置条件。执行后系统提示输入密钥长度，**直接回车使用默认值 1024** 即可。看到 `Create RSA keys successfully`（界面显示 `++++++` 进度条）即表示成功。
>
> ⚠️ **关键原理**：未生成密钥时，`ssh server enable` 等 SSH 命令会被系统隐藏，导致"命令未识别"报错。生成密钥后所有 SSH 命令才会显现。

### 2.2 开启 SSH 服务（本机默认已开启）

> 本机 Comware V3.10 在生成 RSA 密钥后 SSH 服务**默认开启**，**无需也不支持 `ssh server enable`**，跳过此步即可。可用 `display ssh server status` 确认状态为 Enable / RUNNING。

### 2.3 配置 VTY 通道（本地认证 + 仅允许 SSH）

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

### 2.4 AAA 本地用户（本地账号 / 密码 / 权限）

```text
local-user sshadmin
 password simple Admin@2024!
 service-type ssh level 3
 quit
```

> - `sshadmin`：用户名，按需修改。
> - `Admin@2024!`：密码，建议改为强密码（含大小写+数字+特殊字符）。
> - `level 3`：最高管理权限级别。
> - ⚠️ 此设备**不支持** V7 的 `class manage` 写法，权限级别直接写在 `service-type` 后。
> - 💡 `password simple` 以明文形式保存配置中的密码；若设备支持 `password cipher`，建议改用 `password cipher <密文>` 提高安全性。

### 2.5 配置 SSH 用户（认证方式 / 服务类型）

```text
ssh user sshadmin authentication-type password
```

> 老版本 Comware V3/V5 必须显式指定 SSH 用户的认证方式，否则可能登录失败。

### 2.6 保存配置

```text
save
```

> 输入 `y` 确认覆盖保存，否则重启后配置丢失。所有 SSH 配置完成后执行一次 `save` 即可，无需中间反复保存。

## 三、完整一键配置脚本

```text
system-view

# === 配置管理 IP（SSH 登录地址，与 SNMP 文档一致）===
interface Vlan-interface 1
 ip address 192.168.1.250 255.255.255.0
 quit

# === 生成 RSA 密钥（SSH 前置，须第一步）===
public-key local create rsa
# 提示时直接回车，使用默认值 1024

# === VTY 仅允许 SSH ===
user-interface vty 0 4
 authentication-mode scheme
 protocol inbound ssh
 quit

# === 创建本地用户 ===
local-user sshadmin
 password simple Admin@2024!
 service-type ssh level 3
 quit

# === 指定 SSH 用户认证方式 ===
ssh user sshadmin authentication-type password

return
save
```

> 注：本机生成 RSA 密钥后 SSH 服务默认开启，`ssh server enable` 不存在，无需添加；脚本末尾统一 `save` 一次即可。

## 四、功能验证清单

- [ ] `display ssh server status` 显示 Enable / RUNNING（本机无需 `ssh server enable`）
- [ ] `display local-user` 确认 `sshadmin` 存在且 `service-type ssh level 3`
- [ ] 客户端（PuTTY / Xshell / ssh 命令）能登录，认证方式 password
- [ ] 仅 SSH 可登，Telnet 被拒（`protocol inbound ssh` 生效）
- [ ] 配置已 `save`，重启不丢

### 4.1 SSH 客户端登录测试

| 项目 | 值 |
| --- | --- |
| 协议 | SSH |
| 端口 | 22 |
| 用户名 | sshadmin |
| 密码 | Admin@2024! |
| 地址 | 交换机管理 IP（即 VLAN 1 的 192.168.1.250，见 2.0） |

> 登录成功并显示命令行提示符即配置完成 ✅

## 五、注意事项

1. **密钥是前提**：未生成 RSA 密钥前 SSH 命令被隐藏、STelnet 无法协商，务必 2.1 先执行并确认。
2. **SSH 默认已运行**：本机 V3.10 生成密钥后 SSH 自动开启，`ssh server enable` 不存在属正常，跳过即可；以 `display ssh server status` 确认。
3. **权限写法**：不支持 `class manage`，用 `service-type ssh level 3`；`level` 直接写在 `service-type` 后。
4. **双用户实体**：本机需同时存在「本地用户（密码/权限）」与「SSH 用户（认证方式）」两个实体，缺一不可。
5. **保存一次即可**：本模板所有配置完成后统一 `save` 一次。
6. **管理 IP 一致性**：管理 IP `192.168.1.250`（Vlan-interface 1）须与《H3C_S2126-EI_SNMP配置指南》一致。
7. **客户端登录参考**：Linux `ssh sshadmin@192.168.1.250`；Windows PuTTY 选 SSH、端口 22；首次连接确认主机指纹。

## 六、常见问题与排错

> 以下按**故障现象**组织，涵盖配置阶段和运维阶段最常遇到的问题。H3C 特有项基于本机 Comware V3.10 实测。

### 6.1 连接类

| # | 现象 | 可能原因 | 排查命令 / 解决 |
| --- | --- | --- | --- |
| 1 | `Connection refused` / 超时 | SSH 服务未开启（本机应先确认已生成密钥） | `display ssh server status` 确认 Enable；本机生成密钥即自动开启 |
| 2 | 能 ping 通但 SSH 不通 | ACL / 防火墙拦截 TCP 22 | 检查 VTY 是否绑 ACL；确认源 IP 放行 |
| 3 | 连上后立刻断开 | VTY idle-timeout 过短 | `user-interface vty 0 4` 下 `idle-timeout 10 0` |

### 6.2 认证 / 命令识别类（本机版本特性）

| # | 现象 | 原因 | 解决 |
| --- | --- | --- | --- |
| 4 | 输入 `ssh server enable` 提示"命令未识别" | 本机 V3.10 无此命令 | **正常现象**：生成 RSA 密钥后 SSH 已默认运行，跳过即可；若仍不识别，先 `public-key local create rsa` 让 SSH 命令显现 |
| 5 | `local-user admin class manage` 提示"参数过多" | 不支持 `class manage` | 改用 `local-user <用户名>` 进子视图，直接配 `password` 与 `service-type ssh level X` |
| 6 | `undo service-type` 提示"命令不完整" | V3.10 语法要求跟具体参数 | 改用 `service-type ssh level 3` 直接覆盖，不需 undo |

### 6.3 配置前后对比

| 维度 | 配置前 | 配置后 |
| --- | --- | --- |
| 远程管理 | 仅 Console 口 | 支持 SSH 安全登录 |
| 登录方式 | Telnet / Console | 仅 SSH（已禁用 Telnet） |
| 用户权限 | — | sshadmin @ level 3（最高） |

### 6.4 协商 / 算法类

| # | 现象 | 可能原因 | 解决方法 |
| --- | --- | --- | --- |
| 7 | `Unable to negotiate` / `no matching key exchange` | 新客户端算法太新，老设备只支持旧算法（RSA/AES128-CBC 等） | **客户端侧**：PuTTY 无此问题；Linux/macOS 在 `~/.ssh/config` 加 `HostKeyAlgorithms +ssh-rsa` 等；或换 PuTTY / 旧版 SecureCRT / Xshell |
| 8 | `Host key verification failed` | 设备重置后指纹变了 | 客户端 `ssh-keygen -R 192.168.1.250` 清除旧指纹 |

### 6.5 IP 封锁处理（暴力破解防护）

多数设备在连续认证失败后会自动封锁源 IP。以下为通用流程，命令需按品牌替换（H3C 具体命令请以 `?` 确认）：

```text
# ① 查看当前被封锁的 IP 列表
<H3C> display ssh server ip-block list

# ② 解封指定 IP
[H3C] activate ssh server ip-block ip-address X.X.X.X

# ③ （慎用）关闭整个 IP 锁定功能 —— 效果≈解封所有，但降低安全性
[H3C] ssh server ip-block disable

# ④ 重新开启锁定功能（生产建议保持开启）
[H3C] undo ssh server ip-block disable
```

> ⚠️ 默认封锁规则因品牌/版本而异；生产环境建议保持锁定开启，仅用 ② 逐个解封可信 IP。

### 6.6 可选加固配置

#### 6.6.1 限制 VTY 只允许指定 IP 登录

```text
acl number 3000
 rule permit source {{网管IP}} 0
 rule deny
 quit
user-interface vty 0 4
 acl 3000 inbound
 quit
```

#### 6.6.2 修改 SSH 监听端口（降低扫描风险）

```text
ssh server port 2222
```

> 改后客户端需加 `-p 2222` 参数。

#### 6.6.3 调整认证重试次数

```text
ssh server authentication-retries 3
```

#### 6.6.4 缩短空闲超时

```text
user-interface vty 0 4
 idle-timeout 5 0
 quit
```

## 七、客户端登录速查

| 客户端 | 命令 / 操作 |
| --- | --- |
| Linux / macOS / WSL | `ssh sshadmin@192.168.1.250` |
| Windows PowerShell | `ssh sshadmin@192.168.1.250`（Win10 1809+ 内置 OpenSSH） |
| PuTTY | Host: `192.168.1.250` → Connection → SSH → Auth → 用户名 `sshadmin` |
| MobaXterm / SecureCRT / Xshell | Session → SSH → Host `192.168.1.250` → Port `22` |

---

## 八、附录：关闭 Console 刷屏日志（可选）

> 该需求与 SSH 配置**相互独立**，仅在"Console 口端口震荡日志刷屏影响运维"时才需要，故置于文末。

### 8.1 方案 A：日志级别提到 warning（推荐，最简单）

```text
system-view
info-center console level warning
quit
save
```

> **注释**：将 Console 通道日志级别提到 `warning`，端口 up/down 等 info 级震荡日志不再刷屏，但 warning 及以上仍可见。

### 8.2 方案 B：屏蔽 L2INF 模块（端口状态模块）

```text
system-view
info-center source L2INF channel console deny
quit
save
```

> **注释**：精确屏蔽 L2INF（二层接口）模块向 Console 通道的输出，针对性更强。

### 8.3 关键提醒：Console 与 SNMP Trap 是两条独立通道

> ⚠️ **核心要点**：Console 显示 ≠ SNMP Trap 上报。关闭 Console 刷屏**完全不影响** Trap 上报网管，两者互不影响，可放心配置。

---

> 📌 **总结**：H3C S2126-EI（Comware V3.10）配置 SSH 三步核心——`public-key local create rsa` → 本地用户 `service-type ssh level 3` → VTY `protocol inbound ssh`；注意老设备需用兼容写法，且 SSH 无需 `enable` 命令。Console 刷屏抑制为独立可选项，置于文末。

---

## 文档信息

- 文档生成时间：2026-08-17
- 适用设备：H3C S2126-EI（Comware V3.10）
- 命令来源：真机 + AI 沟通验证

