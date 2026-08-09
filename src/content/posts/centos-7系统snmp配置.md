---
title: 'Centos 7系统SNMP配置'
published: 2026-08-09
draft: false
---

# **Centos 7系统配置**

[TOC]



## 1.1**确认主机开放22端口**

先在装置后台直接测试是否可以ssh主机，命令如下：

ssh 当前主机账号@IP

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps2.jpg)

如果ssh失败，说明主机没有开放22端口，需过去当前主机设备面前操作，切换到root账号，编辑ssh_config文件

```
#su或sudo -i
vi /etc/ssh/sshd_config
Port 22
#去掉注释
```



![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps3.jpg)

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps4.jpg)

保存退出，并重启sshd服务

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps5.jpg)

```
systemctl restart sshd.service
```



## 1.2 **确认系统版本信息**

确认装置后台可以ssh主机后，查看当前Linux系统版本是否为Centos 7，命令如下：

cat /etc/os-release

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps6.jpg)



## 1.3 审计员配置

（1）切换到root账号下，创建审计员账号，命令如下：

su   #切换到root用户

useradd -m -g root tsgzAuditor 

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps19.jpg)

（1）设置审计员密码：

passwd tsgzAuditor 
输入密码： 



（1）将审计员账号加入sudoer组中，命令如下：

visudo 

最后一行加上命令：

tsgzAuditor ALL=(ALL) ALL 

> 【本地图片未上传】img

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps20-1786201978608-21.jpg)

## 1.4 SNMP、SNMP trap配置

**1.3.1 先备份SNMP、SNMP trap配置文件，命令如下：**

```
cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.bak
```

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps7.jpg)

**1.3.2编辑snmpd文件，设置SNMP、SNMP trap参数，命令如下：**

vi /etc/snmp/snmpd.conf

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps8.jpg)



按下i键进入修改模式，修改为这样：

```
view all included .1
rocommunity  tsgz@2018  192.168.1.98  -V  systemview
trap2sink  192.168.1.98  tsgz@2018
```

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps9.jpg)

示例：

团体字：tsgz@2018

IP：对应网段的采集装置IP

然后按ESC取消键，输入：wq回车保存退出。

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps10.jpg)

重启snmp服务,命令如下：

```
service snmpd restart 或 systemctl restart snmpd.service
```

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps11.jpg)

**1.3.3检查系统的SNMP进程状态，确认SNMP进程是否起来，命令如下：**

service snmpd status

显示 active (running) 即正常启动

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps12.jpg)

设置开机启动

systemctl enable snmpd.service

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps13.jpg)

## 1.5 syslog配置

**1.4.1先备份syslog配置文件，命令如下：**

cp /etc/rsyslog.conf /etc/rsyslog.conf.bak 

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps14.jpg)

编辑rsyslog文件，设置syslog参数，命令如下：

vim /etc/rsyslog.conf

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps15.jpg)

最后一行加上采集装置IP，命令如下：

```
*.*  @IP:514 
```

IP：为对应网段的采集装置IP

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps16.jpg)

重启syslog服务,命令如下：

service rsyslog restart 或 systemctl restart rsyslog.service

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps17.jpg)

配置完成后，该服务器的的日志就可以发生到采集装置了。可以利用以下命令查看启用状态。

 service rsyslog status

![img](https://gitee.com/kavres/pictureware/raw/master/assets/态感/wps18.jpg)
