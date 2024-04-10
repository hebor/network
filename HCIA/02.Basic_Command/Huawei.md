# VRP基础

Versatile Routing Platform，通用路由平台是华为数据通信产品通用的操作系统平台，在多种硬件产品上拥有一致的网络界面、用户界面和管理界面，集成了路由交换技术、QoS技术、安全技术和IP语音技术等数据通信功能

| 管理形式 | 备注 |
| :-: | :-: |
| 本地管理Local | 通过Colsole线连接Console或mini USB口，终端使用Serial协议适合初始化、故障恢复、系统升级，同时只能支持一个会话 |
| 远程管理Remote | 通过IP地址或域名连接虚拟VTY口，使用Telnet或SSH协议，适合后期维护、异地管理，支持多会话 |

**命令行视图**

| 命令行 | 视图 | 作用 |
| --- | --- | --- |
| \<Huawei\> | 用户视图 | 查看运行状态或其他参数 |
| [Huawei] | 系统视图 | 配置设备的系统参数 |
| [Huawei-GigabitEthernet0/0/0] | 接口视图 | 配置接口参数 |
|| 协议视图 | 配置路由协议 |

## VRP基础命令

```
<Huawei>dir flash:	        #查看硬盘存储
<Huawei>display version     #查看版本信息
<Huawei>display users       #查看远程在线用户
<Huawei>system-view         #进入系统视图
[Huawei]interface GigabitEthernet0/0/0      #进入接口
[Huawei-GigabitEthernet0/0/0]return     #从任意视图回到用户视图
<Huawei>display history-command     #查看最近10条历史命令
[Huawei]sysname R1      #修改设备名称
[R1]header login information "Befor Login"      #管理员登录前显示的标题信息
[R1]header shell information "After Login"      #管理员登录后显示的标题信息
[R1]user-interface vty 0 4      #CON接口只有1个，VTY接口可用范围0-4


#修改系统时间
<R1>clock timezone BJ add 08:00:00    #设置所在时区
<R1>clock datetime 14:57:00 2023-9-12    #设置当前时间和日期
<R1>display clock    #查看时间

#用户界面配置
[R1]user-interface vty 0 4      #对4个连接都做配置，仅对某一个窗口限制时，修改单个vty号即可
[R1-ui-vty0-4]idle-timeout 10 20    #每个连接超过10分20秒没有任何动作时，自动剔除连接
[R1-ui-vty0-4]idle-timeout 0 0      #永不剔除
[R1-ui-vty0-4]undo idle-timeout     #永不剔除
[R1-ui-vty0-4]screen-length 0       #设置终端临时显示行数，0表示一次显示所有，默认显示24行
[R1-ui-vty0-4]history-command max-size 100  #设置历史命令缓冲区大小

#登录权限
[R1-ui-vty0-4]user privilege level 15		#配置指定用户界面下的用户级别
[R1-ui-vty0-4]set authentication password cipher Admin@huawei.com		#配置本地认证密码
```

### VRP远程管理

**命令级别**

| 用户等级 | 命令等级 | 名称 | 说明 |
| :-: | :-: | :-: | :-: |
| 0 | 0 | 访问级 | 网络诊断工具（ping、tracert）、从本设备访问外部设备（telnet客户端）、部分display命令等 |
| 1 | 0 and 1 | 监控级 | 用于系统维护，包括display等命令，不是所有display命令都是监控级，例如display current-configuration和display saved-configuration都是3级管理级  |
| 2 | 0,1 and 2 | 配置级 | 业务配置命令，包括路由、各个网络层的命令，向用户直接提供网络服务 |
| 3-15 | 0,1,2 and 3 | 管理级 | 用于系统基本运行的命令，对业务提供支撑作用，包括文件系统、FTP、TFTP下载、命令级别设置命令以及用于业务故障诊断的debugging命令等 |

高命令等级的用户能够使用低命令等级的所有命令，默认情况下创建的远程账户命令等级是0，只能查看很有限的配置；除了设置用户等级，也可以设置super权限，使低等级用户能够切换到高等级用户

```
local-user hebor privilege level 1	#提升命令等级

#配置IP地址
[R1]display [ip] interface brief	#查看网络接口的简要信息
[R1]interface GigabitEthernet 0/0/0     #进入端口模式
[R1-GigabitEthernet0/0/0]ip address 192.168.42.254 255.255.255.0    #配置IP

[R1]display diagnostic-information		#显示设备所有状态信息，由于信息量巨大，这条命令会极大占用设备资源
[R1]display diagnostic-information hcna.txt		#一般情况下都只需要将命令结果导出到某个文件中
```

`display interface brief`与`display ip interface brief`的区别在于，不加ip参数查看的是二层信息，其中`PHY/Physical`表示物理接口状态，接线表示up，没接线表示down，管理员手动关闭的端口会以\*down表示，或者某些安全技术也会引发\*down状态；Protocol表示协议状态，协议则表示三层或三层以上状态，接口配置IP后protocol才会显示up状态，反过来也可以通过protocol的状态判断端口是否配置IP；InUti和OutUti分别表示端口的入/出口的数据利用量，inErrors和outErrors表示入/出口的错误包

加ip参数后查看到的都是三层端口，三层端口可以查看配置的IP，这个特性也可以反过来用于验证端口是否属于三层端口，LoopBack环回口的Physical和Protocol状态，无论配不配IP，永远都是up状态

认证模式

| 认证模式 | 描述 |
| :-: | :-: |
| AAA | 登陆时需要用户名和密码 |
| Password | 登录时仅需要密码 |

```
[R1]display telnet server status	#查看telnet服务状态
[R1]telnet server enable	#开启telnet服务
[R1]user-interface vty 0 4	#进入vty配置模式
[R1-ui-vty0-4]authentication-mode password/aaa	#配置认证模式
[R1-ui-vty0-4]set authentication password cipher Admin@huawei.com	#配置认证密码
[R1-ui-vty0-4]user privilege level 15		#配置用户级别
[R1]user-interface maximum-vty 15	#配置最大vty会话数量

[R1]aaa     #进入aaa配置模式
[R1-aaa]local-user hebor password cipher Admin@huawei.com	#创建用户和密码
[R1-aaa]local-user hebor privilege level 15     #配置用户级别
[R1-aaa]local-user hebor service-type telnet	#配置用户可用服务
```

使用password认证模式时，在未设置密码的前提下，如果设备不支持空密码登录，telnet连接后会有连接成功的提示，但会秒退；使用AAA认证模式时，每个账户必须配置可用服务，因为AAA是一个独立的技术模块，使用AAA创建的用户根据可用服务的设置会用于各种服务，service-type下的terminal表示console口

### VRP文件系统

**存储设备**

| 存储设备类型 | 作用 |
| :-: | :-: |
| SDRAM | 内存 |
| Flash | 闪存，持久性存储 |
| NVRAM | 非易失性内存 |
| SD Card | SD卡，就内置存储来说，SD卡的空间一般是比Flash大的 |
| USB | USB外接口 |

**文件系统管理命令**

通过`display version`能够查看当前设备上有多少存储设备

<table>
    <tr align="center">
        <td colspan="2" bgcolor="#B2BEB5">管理存储设备</td>
        <td colspan="4" bgcolor="#B2BEB5">管理目录/文件</td>
    </tr>
    <tr align="center">
        <td bgcolor="#E5E4E2">管理项目</td>
        <td bgcolor="#E5E4E2">命令</td>
        <td bgcolor="#E5E4E2">管理项目</td>
        <td bgcolor="#E5E4E2">命令</td>
        <td bgcolor="#E5E4E2">管理项目</td>
        <td bgcolor="#E5E4E2">命令</td>
    </tr>
    <tr>
        <td rowspan="2">修复文件系统异常的存储设备</td>
        <td rowspan="2">fixdisk</td>
        <td>创建目录</td>
        <td>mkdir</td>
        <td>显示文件内容</td>
        <td>more</td>
    </tr>
    <tr>
        <td>重命名目录</td>
        <td>rename</td>
        <td>拷贝文件</td>
        <td>copy</td>
    </tr>
    <tr>
        <td>格式化存储设备</td>
        <td>format</td>
        <td>查看当前的工作目录</td>
        <td>pwd</td>
        <td>移动文件</td>
        <td>move</td>
    </tr>
    <tr>
        <td align="center" colspan="2" bgcolor="#B2BEB5">管理目录/文件</td>
        <td>改变当前目录</td>
        <td>cd</td>
        <td>重命名文件</td>
        <td>rename</td>
    </tr>
    <tr>
        <td align="center" bgcolor="#E5E4E2">管理项目</td>
        <td align="center" bgcolor="#E5E4E2">命令</td>
        <td>显示目录或文件信息</td>
        <td>dir</td>
        <td>压缩文件</td>
        <td>zip</td>
    </tr>
    <tr>
        <td>恢复删除文件</td>
        <td>undelete</td>
        <td>删除目录</td>
        <td>rmdir</td>
        <td>删除文件</td>
        <td>delete</td>
    </tr>
    <tr>
        <td> </td>
        <td> </td>
        <td>执行批处理文件</td>
        <td>execute</td>
        <td>彻底删除回收站的文件</td>
        <td>reset</td>
    </tr>
</table>

```
<R1>delete test.txt     #临时删除文件
<R1>reset recycle-bin     #清空回收站
<R1>delete /unreserved text.txt    #直接删除文件，不放入回收站
```

**配置文件管理**

华为交换机每次保存设备配置默认会压缩保存到vrpcfg.zip文件，设备配置文件分2种，一种是保存在内存中的`Current-Configuration File`、另一种是保存在Flash或SD卡中的`Saved-Configuration File`，默认所有修改的配置都保存在内存中，如果不手动执行save保存配置命令，那么交换机重启后配置会丢失。未执行save之前，查看`Saved-Configuration File`会提示没有正确的配置文件，设备启动时会加载保存的配置文件到内存，并作为当前配置文件

```
<R1>save    #保存配置
<R1>compare configuration    #比较Current-Configuration和Saved-Configuration文件的不同点
<R1>reset saved-configuration    #重置所有已保存的配置信息，重启后恢复出厂设置
<R1>reboot
<R1>display startup    #查看系统启动配置参数
<R1>dis current-configuration	#查看当前配置文件
<R1>dis saved-configuration		#查看已保存的配置文件

<SW1>display startup
MainBoard:
  Configured startup system software:        NULL   #配置的系统文件
  Startup system software:                   NULL   #当前启动的系统文件
  Next startup system software:              NULL   #下次启动的系统文件
  Startup saved-configuration file:          NULL   #当前启动的配置文件
  Next startup saved-configuration file:     NULL   #下次启动的配置文件
  Startup paf file:                          NULL
  Next startup paf file:                     NULL
  Startup license file:                      NULL
  Next startup license file:                 NULL
  Startup patch package:                     NULL
  Next startup patch package:                NULL   #补丁文件

<SW1>startup saved-configuration $PATH    #修改下次启动的配置文件路径
<SW1>startup saved-configuration flash:/vrpcfg.zip    #修改下次启动的系统文件路径
```

执行reboot时设备会对已保存的配置文件和内存中的配置做对比，如果两个配置不一样会产生提示，也就是说执行reboot时可能会有2条提示，第一条提示会询问内存中配置与保存的配置文件不一致，是否保存内存中的配置，如果需要重置设备，这条提示时不能选Y的

配置文件是指操作交换机的配置，系统文件是指VRP操作系统，执行设备系统升级就需要用到系统文件。eNSP模拟器中没有系统文件，无法执行操作，配置文件也需要保存后才有

### VRP系统管理

VRP命名由VRP自身版本号和关联产品版本号两部分组成，产品版本格式包含Vxxx（产品码）、Rxxx（大版本号）、Cxx（小版本号），如果VRP产品版本有补丁，VRP产品版本号中还会包含SPCxxx

<table>
	<tr>
	<td>Version 5.90 (AR2200 V200R001C00)</td>
	<td>VRP版本为5.90，产品版本号为V200R001C00</td>
	</tr>
	<tr>
	<td>Version 5.120 (AR2200 V200R003C00SPC200)</td>
	<td>VRP版本为5.120，产品版本号为V200R003C00SPC200，此产品版本包含有补丁包</td>
	</tr>
</table>

```
<R1>dis version
Huawei Versatile Routing Platform Software
VRP (R) software, Version 5.130 (AR2200 V200R003C00)    #系统文件版本号
Copyright (C) 2011-2012 HUAWEI TECH CO., LTD
Huawei AR2220 Router uptime is 0 week, 0 day, 2 hours, 36 minutes   #设备系列型号下的细分型号
```

**FTP&TFTP**

系统升级或拷贝配置文件都需要为交换机提供一个FTP服务端，将自身主机作为FTP服务端传输文件，可以使用一些小工具，例如`Xlight FTP`，在主机上启动FTP服务后使用交换机登录传输文件

```
<R1>ftp 192.168.42.1    #交换机登录自身主机
[R1-ftp]get 1.jpg   #下载测试文件
[R1-ftp]put 1.jpg   #FTP上传文件需要可写权限

tftp 192.168.42.1 get 1.jpg     #tftp下载
tftp 192.168.42.1 put 1.jpg     #tftp上传
```

也可以将交换机配置为FTP服务端，主机作为客户端传输文件

```
<LSW1>sys
[LSW1]inte vlanif 1
[LSW1-Vlanif1]ip add 10.1.1.1 24
[LSW1-Vlanif1]quit
[LSW1]ftp server enable
Info: Succeeded in starting the FTP server.
[LSW1]dis ftp-server
[LSW1]aaa
[LSW1-aaa]local-user ftp1 password cipher huawei	#创建用户`ftp1`并设置密码`huawei`
Info: Add a new user.
[LSW1-aaa]local-user ftp1 privilege level 15		#设置用户权限等级，15为最大等级
[LSW1-aaa]local-user ftp1 ftp-directory flash:		#设置共享给用户的目录，`flash`表示设备硬盘
[LSW1-aaa]local-user ftp1 service-type ftp			#设置用户`ftp1`用于`ftp`服务
[LSW1-aaa]Ctrl^Z
<LSW1>save test.zip
```