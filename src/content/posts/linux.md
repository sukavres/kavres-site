---
title: 'Linux'
published: 2026-08-08
category: '技术'
password: 'kavres'
draft: false
---

TCP/IP协议概述

要学好Linux，对网络协议要有充分的了解和掌握，例如传输控制协议/因特网互联网协议(Transmision Control Protocol/Internet Protocol,TCP/IP)，TCP/IP名为网络通讯协议，是Internet最基本的协议、Internet国际互联网网络的基础，由网络层的IP协议和传输层的TCP协议组成。

TCP/IP定义了电子设备如何连入因特网，以及数据如何在它们之间传输的标准。协议采用了4层的层级结构，每一层都呼叫它的下一层所提供的协议来完成自己的需求。

TCP负责发现传输的问题，一有问题就发出信号，要求重新传输，知道所有数据安全正确地传输到目的地，而IP是给因特网的每台联网设备规定一个地址。

 基于TCP/IP的参考模型将协议分为四个层次，它们分别是网络接口层、网际互联层（IP层）、传输层（TCP层）和应用层。

| OSI 7层    | 每层功能                               | TCP/IP协议簇                             |
| ---------- | -------------------------------------- | ---------------------------------------- |
| 应用层     | 文件传输，电子邮件，文件服务，虚拟终端 | TFTP、HTTP、SNMP、FTP、SMTP、DNS、Telnet |
| 表示层     | 数据格式化，代码转换，数据加密         | 没有协议                                 |
| 会话层     | 解除或建立与别的接点的联系             | 没有协议                                 |
| 传输层     | 提供端对端的接口                       | TCP、UDP                                 |
| 网络层     | 为数据包选择路由                       | IP、ICMP、OSPF、BGP、IGMP、ARP、RARP     |
| 数据链路层 | 传输有地址的帧以及错误检测功能         | SLIP、PPP、MTU                           |
| 物理层     | 以二进制数据的形式在物理媒体上传输数据 | ISO2110、IEEE802、IEEE802.2              |
|            |                                        |                                          |

IP地址及网络常识

互联网协议地址(Internet Protocol Address, IP)，IP地址是IP协议提供的一种统一的地址格式，它为互联网上的每一个网络和每一台主机分配一个逻辑地址，以此来屏蔽物理地址的差异。IP地址被用来给Internet上的每个通信设备的一个编号，每台联网的PC上都需要有IP地址，这样才能正常通信。

IP地址是一个32位的二进制数，通常被分割位4个“8位二进制数”（即4个字节）。IP地址通常用“点分十进制”表示成(a.b.c.d)的形式，其中，a,b,c,d都是0~255之间的十进制数。

常见的IP地址，分为IPv4与IPv6两大类。IP地址编址方案将IP地址空间划分为A、B、C、D、E五类，其中A、B、C是基本类，D、E、作为多播和保留使用。

IPv4有4段数字，每一段最大不超过255。由于互联网的蓬勃发展，IP地址的需求量愈来愈大，使得IP位址发放愈趋严格，各项资料显示 全球IPv4位址在2011年已经全部发放完毕。

地址空间的不足必妨碍互联网的进一步发展。为了扩大地址空间，拟通过IPv6重新定义地址空间。IPv6采用128位地址长度。在IPv6的设计过程中，除了一劳永逸地解决了地址短缺问题外，IPv6的诞生可以给全球每一粒沙子配置一个IP地址，还考虑了在IPv4中解决不好的其它问题。

IP地址分类

1.A类IP地址

一个A类IP地址是指，在IP地址的四段号码中，第一段号码为网络号码，剩下的三段号码为本地计算机的号码。如果用二进制表示IP地址的话，A类IP地址就由1字节的网络地址和3字节的主机地址组成，网络地址的最高位必须是“0”。A类IP地址中，网络的标识长度为8位，主机标识长度为24位，A类网络地址数量较少，有126个网络，每个网络可以容纳主机数达1600万台。

A类IP地址 地址范围1.0.0.0到127.255.255.255 (二进制表示为：00000001 00000000 00000000 00000000 - 01111110 11111111 11111111 11111111)，最后一个为广播地址，A类IP地址的子网掩码为255.0.0.0,每个网络支持的最大主机数为256的3次方-2=16777214台。

2.B类IP地址

一个B类IP地址是指在IP地址的四段号码中，前两段号码为网络号码。如果二进制表示IP地址的话，B类IP地址就由2字节的网络地址和2字节的主机地址组成，网络地址的最高位必须是“10”。

B类IP地址中网络的标识长度为16位，主机标识的长度为16位，B类网络地址适用于中等规模的网络，有16384个网络，每个网络能容纳的计算机数为6万多台。

B类IP地址范围128.0.0.0-191.255.255.255（二进制表示为：10000000 00000000 00000000 00000000 ------10111111 11111111 11111111 11111111）。

最后一个是广播地址，B类IP地址的子网掩码为255.255.0.0，每个网络支持的最大主机数为256的2次方-2=65534台

3.C类IP地址

一个C类IP地址是指在IP地址的四段号码中，前三段号码为网络号码，剩下的一段号码为本地计算机的号码。如果用二进制表示IP地址的话，C类IP地址就由3字节的网络地址和1字节的主机地址组成，网络地址的最高位必须是“110”。C类IP地址中网络的标识长度为24位，主机标识的长度为8位，C类网络地址数量较多，有209万余个网络。适用于小规模的局域网络，每个网络最多包含254台计算机

C类IP地址范围 192.0.0.0-223.255.255.255 （二进制表示为11000000 00000000 00000000 - 11011111 11111111 11111111 11111111）C类IP地址的子网掩码为255.255.255.0，每个网络支持的最大主机数为256-2=254台。

D类IP地址又被成为多播地址（Multicast Address），即组播地址。在以太网中，多播地址命名了一组应该在这个网络中应用接收到一个分组的站点。多播地址的最高位必须是”1110“，范围从224.0.0.0到239.255.255.255.

5.特殊地址

每一个字节都为0的地址（“0.0.0.0”）表示当前主机，IP地址中的每一个字节都为1的IP地址（“255.255.255.255”）是当前子网的广播地址，IP地址凡是以"11110"开头的E类IP地址都保留用于将来和实验使用。

IP地址中不能以十进制"127"开头，而已数字127.0.0.1到127.255.255.255段的IP地址称为环回地址，用于回路测试，如127.0.0.1可以代表本机IP地址，网络ID的第一个8位组也不能全置位“0”，全“0”表示本地网络。

 子网掩码

子网掩码(Subnet Mask)又名网络掩码、地址掩码，它是一种用来指明一个IP地址的哪位标识的是主机所在的子网，以及哪些位标识的是主机位的掩码。

 通常的讲，子网掩码不能单独存在，它必须结合IP地址一起使用。子网掩码只有一个作用，就是将某个IP地址划分成网络地址和主机地址两部分。

子网掩码是一个32位址，用于屏蔽IP地址的一部分以区别网络标识和主机标识，并说明该IP地址是在局域网上，还是在远程网上。

对于A类地址，默认的子网掩码是255.0.0.0，对于B类地址来说默认的子网掩码是255.255.0.0；对于C类地址来说默认的子网掩码是255.255.255.0。

 互联网是由各种小型网络构成的，每个网络上都有许多主机，这样便构成了有层次的结构。IP地址在设计时就考虑到地址分配的层次特点，将每个IP地址都分割成网络号和主机号两部分，以便IP地址的寻址操作。

网关地址

网关(Gateway)是一个网络连接到另一个网络的"关口"，网关本质上是一个网络通向其他网络的IP地址。主要用于不通网络传输数据。

例如我们电脑设备上网，如果是接入到同一个交换机，在交换机内部传输数据是不需要经过网关的，但是如果两台设备不在一个交换机网络，则需要在本机配置网关，内网服务器的数据通过网关，网关把数据转发到其他的网络的网关，直至找到对方的主机网络，然后返回数据。

MAC地址

媒体访问控制(Media Access Control 或者 Medium Access Control，MAC)，也就是物理地址、硬件地址，用来定义网络设备的位置。

在OSI模型中，第三层网络层负责IP地址，第二层数据链路层则负责mac地址。因此一个主机会有一个MAC地址，而每个网络位置会有一个专属于它的IP地址。

IP地址工作在OSI参考模型的第三层网络层。两者之间分工明确，默契合作，完成通信过程。IP地址专注于网络层，将数据包从一个网络转发到另外一个网络；而MAC地址则专注与数据链路层，将一个数据帧从一个节点传送到相同链路的另一个节点。

IP地址和MAC地址一般是成对出现。如果一台计算机要和网络中另一台计算机通信，那么这两台设备必须配置IP地址和MAC地址，而MAC地址是网卡出厂时设定的，这样配置的IP地址和MAC地址形成了一种对应关系。

在数据通信时，IP地址负责表示计算机的网络层地址，网络层设备（如路由器）根据IP地址来进行操作；MAC地址负责表示计算机的数据链路层地址，数据链路层设备根据MAC地址来进行操作。IP和MAC地址这种映射关系是通过地址解析协议(Address Resolution Protocol，ARP)来实现的。

### 常用命令详解

linux命令可以分为内置命令和外部命令，内置命令是直接内置在shell程序之中，随系统启动而自动加载到内存，不受磁盘的影响。外部命令由相应的系统软件提供，用户需要时才从硬盘中读取入内存中。

```
###查看内置命令：
[root@localhost ~]# enable

### 禁用某个内置命令：
[root@localhost ~]# enable -n cd

###启用内置命令（默认是启动的）：
[root@localhost ~]#enable cd

```

### 2.1 ls命令





RPM软件包管理

Linux软件包管理大致可分为二进制包、源码包，使用的工具也各不相同。Linux常见软件包分为两种，分别是源代码包（Source Code）、二进制包（Binary Code），源代码包是没有经过编译的包，需要经过GCC、C++编译器环境编译才能运行，二进制包无需编译，可以直接安装使用。

通常而言，可以通过后缀简单区别源码包和二进制包，例如.tar.gz、zip、.rar结尾的包通常称之为源码包，以.rpm结尾的软件包称之为二进制包。真正区分是否为远吗还是二进制还得基于代码里面的文件来判断，例如包含.h、.c、.cpp、.cc等结尾的源码文件，称之源码包，而代码里面存在bin可执行文件，称之为二进制包。

 Centos操作系统中有一款默认软件管理的工具，红帽包管理工具(Red Hat Package Manager,RPM)。

使用RPM工具可以对软件包实现快速安装、管理及维护。RPM管理工具适用的操作系统包括：Centos，Redhat，Fedora，SUSE等，RPM工具常用于管理.rpm后缀结尾的软件包。

RPM软件包命令规则详解如下：

RPM包命名格式为：

name-version.rpm

name-version-noarch.com

name-version-arch.rpm

如下软件包格式：

epel-rease-6-8.noarch.rpm

perl-Pod-Plainer-1.0.3-1.el6.noarch.rpm

yasm-1.2.0-4.el7.x86_64.rpm

RPM包格式解析如下：

name 软件名称，例如yasm、perl-pod-Plainer;

version 版本号，1.2.0通用格式：“主版本号.次版本号.修正号”；

​                               4表示是发布版本号，该RPM包是第几次编译生成的；

arch 适用的硬件平台，RPM支持的平台有：i386、i586、i686、x86_64、sparc、alpha等。

.rpm 后缀宝表示编译好的二进制包，可用rpm命令直接安装；

.src.rpm  源代码包，源码编译生成的.rpm格式的RPM包方可适用；

el* 软件包发行版本，el6表示该软件包适用于Rehl 6.x/CentOS 6.x；

devel： 开发包；

noarch 软件包可以在任何平台安装。



RPM选项 PACKGE_NAME

-a，--all 查询所有已安装软件包;

-q， --query 表示询问用户，输出信息；

-l ，--list 打印软件包的列表；

-f，--file FILE查询包含FILE的软件包；

-i，--info 显示软件包信息，包括名称，版本，描述；

-v，--verbose 打印输出详细信息；

-U， --upgrade 升级rpm软件包；

-h --hash 软件安装，可以打印安装进度条；

--last 列出软件包时，以安装时间排序，最新的在上面；

-e， --erase 卸载rpm软件包；

--force 表示强制，强制安装或者卸载；

--nodeps RPM包不依赖

-l，--list 列出软件包中的文件；

--provides 列出软件包提供的特性；

-R --requires 列出软件包依赖的其它软件包；

--scripts 列出软件包自定义的小程序。



RPM企业案例演示：
rpm -q httpd 检查httpd包是否安装；

rpm -ql httpd 查看软件安装的路径；

rpm -qi httpd 查看软件安装的版本信息；

rpm -e httpd 卸载httpd软件；

rpm -e --nodeps httpd 强制卸载httpd；

rpm -qalgrep httpd 检查httpd相关的软件包是否安装；

rpm -ivh httpd-2.4.10-el7.x86_64.rpm 安装httpd软件包；

rpm -Uvh httpd-2.4.10-el7.x86_64.rpm 升级httpd软件；

rpm -ivh --nodeps httpd-2.4.10-el7.x86_64.rpm 不依赖其他软件包；



Tar软件包管理

Linux操作系统除了使用RPM管理工具对软件包管理之外，还可以通过tar、zip、jar等工具进行源码包管理。

tar命令参数详解

-A，--catenate ,--concatenate 将存档与已有的存档合并

-c，--create 建立新的存档

-d，--diff，--compare 比较存档与当前文件的不同之处

--delete 从存档中删除

-r，--append 附加到存档结尾

-t，--list 列出存档中的文件的目录

-u，--update 仅将较新的文件附加到存档中

-x，--extract，--get 解压文件

-j，--bzip2，--bunzip2 有bz2属性的软件包；

-z，--gzip，--ungzip 有gz属性的软件包；

-b，--block-size N 指定块大小为Nx512字节（缺省时 N=20）;

-B，--read-full-blocks 读取时重组块；

-C，--directory DIR 指定新的目录

--checkpoint 读取存档时显示目录名；

-f，--file[HOSTNAME:]F  指定存档或设备，后接文件名称；

--force-local 强制使用本地存档，即使存在克隆；

-G，--incremental 建立老GNU格式的备份

-g，--listed-incremental 建立新GNU格式的备份

-h，--dereference 不转储动态链接，转储动态链接指向的文件；

-i，--ignore-zeros 忽略存档中的0字节块（通常意味着文件结束）

--ignore-failed-read 在不可读文件中作0标记后再退出

-k，--keep-old-files 保存现有文件；从存档中展开时不进行覆盖；

-K，--starting-file F 从存档文件F开始；

-l，--one-file-system 在本地文件系统中创建存档；

-L，--tape-length N 在写入 N*1024个字节后暂停，等待更换磁盘；

-m，--modification-time 当从一个档案中恢复文件时，不使用新的时间标签；

-M，--multi-volume 建立多卷存档，以便在几个磁盘中存放；

-O，--to-stdout 将文件展开到标准输出；

-P，--absolute-paths 不要从文件名中去除’/‘；

-v，--verbose 详细显示处理的文件；

--version 显示tar程序的版本号；

--exclude FILE不把指定文件包含在内；

-X，--exclude-from FILE 从指定文件中读入不想包含的文件的列表；



Tar企业案例演示

tar -cvf jfedu.tar.gz jfedu 打包jfedu文件或目录，打包后名称jfedu.tar.gz；

tar -tf jfedu.tar.gz 查看jfedu.tar.gz包中的内容；

tar -rf jfedu.tar.gz jfedu.txt 将jfedu.txt追加到jfedu.tar.gz中

tar -xvf jfedu.tar.gz 解压jfedu.tar.gz 程序包；

tar -czvf jfedu.tar.gz. 使用gzip格式打包并压缩jfedu目录；

tar -cjvf jfedu.tar.bz2 jfedu 使用bzip2格式打包并压缩jfedu目录

tar -czf jfedu.tar.gz.* -X list.txt 使用gzip格式打包并压当前目录所有文件,排除list.txt中记录的文件；

tar -czf jfedu.tar.gz *   --exclude=zabbnix-3.2.4.tar.gz     --exclude=nginx-1.12.0.tar.gz  使用gzip格式打包并压缩当前目录和文件，排除zabbnix-3.2.4.tar.gz和nginx-1.12.0.tar.gz 



TAR实现Linux操作系统备份

Tar命令工具除了用于日常打包、解压源码包或者压缩包外，最大的亮点还可以用于Linux操作系统文件及目录的备份，使用tar -g可以基于GNU格式的增量备份，备份原理是基于检查目录或者文件的atime、mtime、ctime属性是否被修改。文件及目录时间属性详解如下：

文件被访问时间（Access time，atime）

文件内容被改变时间（Modified time，mtime）

文件写入、权限更改的时间（Change time，ctime）

总结，更改文件内容mtime和ctime都会改变，但ctime可以在mtime未发生变化时被更改，例如修改文件权限，文件mtime时间不变，而ctime时间改变。Tar增量备份案例演示步骤如下：

1） /root 目录创建jingfeng文件夹，同时在jingfeng文件夹中，新建jf.txt，jf2.txt。文件，

> 【本地图片未上传】image-20221128162541132

2）使用tar命令第一次完整备份jingfeng文件夹中的内容，-g指定快照snapshot文件，第一次没有该文件会自动创建

cd /root/jingfeng/

tar -g /data/backup/snapshot -czvf /data/backup/2019jingfeng.tar.gz

> 【本地图片未上传】image-20221128162558432

3)使用tar命令第一次完整备份jingfeng文件夹之后，会生成快照文件  /data/backup/snapshot ，后期增量备份会以snapshot文件为参考，在jingfeng文件夹中再创建jf3.txt jf4.txt，然后通过tar命令增量备份jingfeng目录所有内容

cd /root/jingfeng/

touch jf3.txt jf4.txt 

tar -g /data/backup/snapshot -czvf /data/backup/2019jingfeng_add1.tar.gz *

> 【本地图片未上传】image-20221128162612447

增量备份时，需-g指定第一次完整备份的快照snapshot文件，同时增量打包的文件名不能跟第一次备份后的文件名重复，通过tar -tf可以查看打包后的文件内容



6.2.4 shell+tar实现增量备份

企业中日常备份的数据包括/boot、/etc、/root、/data目录等，备份的策略参考

每周1-6执行增量备份，每周日执行全备份。同时，在企业中备份操作系统均使用shell脚本完成，此处auto_backup_system.sh脚本供参考

```shell
#!/bin/bash
#Automatic Backup Linux System Files
#By Author www.jfedu.net
#Define Variables

SOURCE_DIR=(
      $*
)

TARGET_DIR=/data/backup/
YEAR='date +%Y'
MONTH'date +%m'
DAY='date +%d'
WEEK='data +%u'
FILES=system_bakcup.tgz
CODE=$?

if
  [-z $SOURCE_DIR]; then
  echo -e "Please Enter a File or Directory You Need to Backup:\n----------------\nExample %0 /boot/etc .... "
  exit
fi

#Determine Whether the Target Directory Exists

if
  [! -d $TARGET_DIR/$YEAR/$MONTH/$DAY];  then
  mkdir -p $TARGET_DIR/$YEAR/$MONTH/$DAY
  echo "This $TARGET_DIR Created Successfully !"
  
fi

#EXEC Full_Backup Function Command

Full_Backup()
{
if
  ["$WEEK" -eq "7"]; then
  rm -rf $TARGET_DIR/snapshot
  cd $TARGET_DIR/$YEAR/$MONTH/$DAY ; tar -g $TARGET_DIR/snapshot -czvf $FILES 'echo ${SOURCE_DIR[@]}'
  [ "$CODE"  == "0" ]&&echo -e "------------\nFull_Backup System Files Backup Successfully!"

fi
  
}

#Perform incremental BACKUP Function Command

Add_Backup()
{
 cd $TARGET_DIR/$YEAR/$MONTH/$DAY ;
 if
    [-f $TARGET_DIR/$YEAR/$MONTH/$DAY/$FILES]; then
    read -p "$FILES Already Exists, overwrite confirmation yes ro no ?:" SURE
    if[$SURE == "no" -o $SURE == "n"]; then
    sleep 1 ; exit 0
    fi

#Add_Backup Files System

     if
      [$WEEK -ne "7"]; then
      cd $TARGET_DIR/$YEAR/$MONTH/$DAY ; tar -g $TARGET_DIR/snapshot -czvf $$_$FILES 'echo ${SOURCE_DIR[@]'
      [ "$CODE" == "0" ]&&echo -e "--------------------\nAdd_Backup System Files Backup Successfully!"
      fi
    else
      if
      [$WEEK -ne "7"]; then
      cd $TARGET_DIR/$YEAR/$MONTH/$DAY ; tar -g $TARGET_DIR/snapshot -czvf $FILES 'echo ${SOURCE_DIR[@]}'
      [ "$CODE" == "0" ]&&echo -e "-------------------\nAdd_Backup System Files Backup Successfully!"
      fi

fi
}
Full_Backup; Add_Backup
```



6.3 ZIP 软件包管理

ZIP也是计算机文件的压缩算法，原名Deflate（真空），发明者为菲利普·卡兹（Phil Katez）,他于1989年公布了该格式的资料。ZIP通常使用后缀名“.zip”。

 主流的压缩格式包括tar、rar、zip、war、gzip、bz2、iso等。从性能上比较，TAR、WAR、RAR格式较ZIP格式压缩率较高，但压缩时间远远高于ZIP，ZIP命令行工具可以实现对zip属性的包进行管理，也可以将文件及文件包打包成zip格式。如下为ZIP工具打包常见参数详解：

```
-f  freshen :只更改文件；
-u  update :只更改新文件；
-d 从压缩文件删除文件；
-m  移动文件（删除源文件）；
-r  递归到目录；
-j junk（不记录）目录名
-l 将LF转换为CR LF（-11 CR LF至LF）；
-1 压缩更快1-9压缩更好；
-q 安静操作，不输出执行的过程；
-v verbose 操作/打印版本信息；
-c 添加一行注释；
-z 添加zipfile注释；
-o 读取名称使zip文件与最新条目一样旧；
-x 不包括以下名称；
-F 修复zipfile(-FF尝试更难)；
-D 不要添加目录条目；
-T 测试zip文件完整性；
-X exclude extra文件属性；
-e 加密- 不要压缩这些后缀；
-h2 显示更多的帮助；
```



ZIP企业案例演示；

通过zip工具打包jingfeng
