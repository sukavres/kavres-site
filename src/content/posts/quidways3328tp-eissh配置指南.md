---
title: 'Quidway_S3328TP-EI_SSH配置指南'
published: 2026-08-19
category: '数据通信'
draft: false
---

# Quidway S3328TP-EI SSH 配置指南

> **命名规范**：`品牌_型号_SSH配置指南.md`（本文件：`Quidway_S3328TP-EI_SSH配置指南.md`）。
> **本文定位**：**登录与安全基线**专用文档。同设备的 SNMP / Syslog / 端口镜像等监控运维配置见《Quidway_S3328TP-EI_SNMP配置指南》。
> ⚠️ **命令强绑定版本**：本文命令均按本机 VRP 5.30（V100R003C00SPC301）真机验证写法，不可跨设备套用（包括华为 V600、H3C 等其它版本/品牌）。

## 一、设备信息

| 项目 | 值 |
|------|----|
| 型号 | Quidway S3328TP-EI |
| 软件版本 | VRP (R) Software, Version 5.30 (S3328 V100R003C00SPC301) |
| BootROM | 315 (2009-11-04) |
| 管理 IP | 192.168.1.101 (Vlanif1) |
| SSH 用户名 | sshadmin |
| SSH 密码 | Admin@2024! |

---

## 二、配置过程（完整命令序列）

### 2.1 生成本地 RSA 密钥对

```
<Quidway> system-view
[Quidway] rsa local-key-pair create
The key name will be: Quidway_Host
The range of public key size is (512 ~ 2048).
Input the bits in the modulus[default = 512]:
Generating keys...
```
> 直接回车用默认 512 位即可。位数越大生成越慢，老设备建议 512~1024。

### 2.2 开启 STelnet 服务

```
[Quidway] stelnet server enable
Info: Start STELNET server
```

### 2.3 配置 VTY 通道（AAA 认证 + 仅允许 SSH）

```
[Quidway] user-interface vty 0 4
[Quidway-ui-vty0-4] authentication-mode aaa
[Quidway-ui-vty0-4] protocol inbound ssh
[Quidway-ui-vty0-4] quit
```

### 2.4 AAA 本地用户配置

```
[Quidway] aaa
[Quidway-aaa] local-user sshadmin password cipher Admin@2024!
Info: A new user added
[Quidway-aaa] local-user sshadmin service-type ssh
[Quidway-aaa] local-user sshadmin level 3
[Quidway-aaa] quit
```

> **注意**：VRP 5.30 里权限等级关键字是 `level`，不是 `privilege-level`。

### 2.5 配置 SSH 用户

```
[Quidway] ssh user sshadmin
Info: A new ssh user added
[Quidway] ssh user sshadmin authentication-type password
[Quidway] ssh user sshadmin service-type stelnet
```

### 2.6 保存配置

```
[Quidway] quit
<Quidway> save
```

---

## 三、完整一键脚本（可直接粘贴）

```
system-view
rsa local-key-pair create

stelnet server enable

user-interface vty 0 4
 authentication-mode aaa
 protocol inbound ssh
 quit

aaa
 local-user sshadmin password cipher Admin@2024!
 local-user sshadmin service-type ssh
 local-user sshadmin level 3
 quit

ssh user sshadmin
ssh user sshadmin authentication-type password
ssh user sshadmin service-type stelnet

quit
save
```

---

## 四、验证命令

```
# 1. SSH 服务状态
<Quidway> display ssh server status
# 期望：STelnet server: Enable

# 2. SSH 用户信息
<Quidway> display ssh user sshadmin
# 期望：Authentication type: password
#       Service type: stelnet

# 3. 本地用户
<Quidway> display local-user username sshadmin
# 期望：State: active
#       Service-type-mask: S(Ssh)

# 4. VTY 配置
<Quidway> display user-interface vty 0 4
# 期望：Authentication mode: aaa
#       Protocol inbound: ssh
```

---

## 五、客户端登录方式

### Linux / Mac / WSL

```bash
# 标准连接
ssh sshadmin@192.168.1.101

# 老设备算法兼容（如协商失败时使用）
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 \
    -oHostKeyAlgorithms=+ssh-rsa \
    -oCiphers=+aes128-cbc \
    sshadmin@192.168.1.101
```

### Windows（PuTTY）

- Host Name: `192.168.1.101`
- Port: `22`
- Connection type: `SSH`
- 建议使用 **PuTTY 0.70 及以下版本**（对新算法兼容性更好）

### 用户名 / 密码

| 项目 | 值 |
|------|----|
| Username | `sshadmin` |
| Password | `Admin@2024!` |

---

## 六、老设备特有的坑

### 坑 1：连续输错密码 → IP 被封锁

VRP 5.30 有防暴力破解机制，**连续输错 6 次密码，客户端 IP 会被封锁**。

**查看被封锁的 IP：**
```
<Quidway> display ssh server ip-block list
```

**解封指定 IP：**
```
<Quidway> activate ssh server ip-block ip-address 192.168.1.240
```

### 坑 2：SSH 客户端算法太新连不上

老设备只支持旧算法（RSA、AES128-CBC、3DES、DH-group1）。新版 OpenSSH 默认禁用了这些算法。

**解决方法：** 用上面的 `-oKexAlgorithms=+...` 参数降级，或换老版本客户端（PuTTY 0.70-、SecureCRT 7.x、Xshell 5/6）。

### 坑 3：没有"首次强制改密码"功能

VRP 5.30 **没有** `password-force-change disable` 这条命令，也没有首次登录强制改密机制。设的密码 `Admin@2024!` 可以直接登录，无需修改。

### 坑 4：密码加密方式

`local-user password cipher` 用的是华为私有可逆加密，配置文件中看到的是密文但可被设备还原。如果迁移到新版本（V600），密码不能直接复制，需要重新设置。

---

## 七、可选加固配置

### 限制 VTY 只允许指定 IP 登录

```
[Quidway] acl 2000
[Quidway-acl-basic-2000] rule permit source 192.168.1.240 0.0.0.0
[Quidway-acl-basic-2000] rule deny source any
[Quidway-acl-basic-2000] quit

[Quidway] user-interface vty 0 4
[Quidway-ui-vty0-4] acl 2000 inbound
[Quidway-ui-vty0-4] quit
```

### 设置空闲超时（10 分钟自动断开）

```
[Quidway] user-interface vty 0 4
[Quidway-ui-vty0-4] idle-timeout 10 0
[Quidway-ui-vty0-4] quit
```

---

## 八、与 S5735 V2（V600）SSH 配置对比

| 项目 | S3328TP-EI (VRP 5.30) | S5735 V2 (V600) |
|------|----------------------|-----------------|
| 密钥生成 | `rsa local-key-pair create` | `rsa local-key-pair create` |
| 开启服务 | `stelnet server enable` | `stelnet server enable` |
| `ssh server-source` | ❌ 不存在 | ✅ 必配 |
| `password-force-change disable` | ❌ 不存在（默认不强制） | ✅ 需要 |
| 权限关键字 | `level 3` | `privilege-level 3` |
| 用户名长度限制 | 无 | ≥ 6 位 |
| 密码复杂度强制 | 无 | 大小写+数字+特殊字符 |
| VTY 协议限制 | `protocol inbound ssh` | `protocol inbound ssh` |

---

## 九、排错速查

| 现象 | 原因 | 解决 |
|------|------|------|
| 连接被拒绝 | STelnet 未开启 | `stelnet server enable` |
| 算法协商失败 | 客户端太新 | 加 `-oKexAlgorithms=+...` 参数 |
| 登录提示密码错误 | 用户名/密码错或用户未绑定 SSH | 检查 `display ssh user` |
| 连接超时 | 网络不通或管理口 Down | 检查 Vlanif1 IP 和物理链路 |
| IP 被封锁 | 连续输错密码 | `activate ssh server ip-block ip-address x.x.x.x` |
| 登录后立刻断开 | VTY 未配 `protocol inbound ssh` | 检查 VTY 配置 |

---

## 文档信息

- 文档生成时间：2026-08-17
- 适用设备：Quidway S3328TP-EI（VRP 5.30 V100R003C00SPC301）

