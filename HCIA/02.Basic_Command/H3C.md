# Comware

Comware是H3C网络设备的操作系统，也是H3C产品的核心软件平台。Comware基于模块化的方式对硬件驱动和底层操作系统进行封装，集成丰富的链路层协议、以太网交换、IP路由转发、安全等功能模块，制定了软硬件接口标准规范，对第三方厂商提供开放平台和接口

**命令行视图**

| 命令行 | 视图 | 作用 |
| :-- | :-- | :-- |
| \<H3C\> | 用户视图 | 设备启动后的缺省视图，可查看启动后基本运行状态和统计信息 |
| [H3C] | 系统视图 | 配置系统全局通用参数 |
| | 协议视图 | 配置路由协议参数 |
| [H3C-GigabitEthernet0/0] | 接口视图 | 配置接口参数 |
| [H3C-line-vty0-63] | 用户线视图 | 配置登录设备ide各个用户属性 |

## Comware基础命令

Comware基础命令与VRP基础命令重复度比较高，此处不会完全将两者不同出记录，鼓励查看H3C官方的命令手册

```Comware
<H3C>system-view	#进入系统视图
[H3C]interface GE0/0	#进入接口视图
[H3C]line vty 0 63		#进入用户线视图
[H3C]undo info-center enable	#关闭信息中心通知功能

# H3C设备开启路由追踪功能需要的前置命令
[H3C]ip ttl-expires enable
[H3C]ip unreachables enable

# H3C设备配置DNS代理
[H3C]dns proxy enable
[H3C]display dns host ip    #查看域名解析表
[H3C]ip host HOSTNAME IP-ADDRESS    #配置静态域名解析表中的主机名与对应地址
[H3C]dns server IP-ADDRESS    #配置指定域名服务器的地址，交换机收到内网的域名解析请求后，会向此DNS服务器发起解析请求
[H3C]display dns server    #查看DNS服务器的地址信息
[H3C]dns domain DOMAIN-NAME    #配置域名后缀
[H3C]display dns domain    #查看域名后缀信息
```

DNS代理能够节省内网上公网解析域名的流量，内网所有终端的DNS服务器指向交换机的IP，交换机收到终端的解析请求后再由交换机向公网请求域名解析，如果内网多个终端请求的同一个域名，交换机则会直接返回解析结果，而不用多次重复发起相同的解析请求

### Comware远程管理

**用户角色**

| 用户角色 | 用户权限 |
| :-: | :-: |
| network-admin | 最高权限，可操作系统所有的功能和资源 |
| network-operator | 可查看系统所有的功能和资源相关的display命令（display history-command all除外） |
| level-n（n=0~15） | level-0 ~ level-14可以由管理员为其配置权限，其中level-0、level-1和level-9有缺省用户权限，level-15的用户权限与network-admin几乎相同（日志统计信息可能不一致），管理员无法对其进行配置 |

**Telnet远程登录**

```Comware
# Password认证模式
[R1]inte g0/0
[R1-GigabitEthernet0/0]ip add 192.168.56.2 24	#配置与网络接口IP
[R1-GigabitEthernet0/0]quit
[R1]telnet server enable	#启用Telnet服务端
[R1]line vty 0 63	#进入vty用户界面视图
[R1-line-vty0-63]authentication-mode password	#设置验证方式
[R1-line-vty0-63]set authentication password simple Huawei@123.com	设置登录密码
[R1-line-vty0-63]user-role network-adminr		#设置用户级别

# AAA认证模式
[R1]local-user hebor class manage	#创建一个管理类型账户
[R1-luser-manage-hebor]password simple Admin@huawei.com		#设置用户密码
[R1-luser-manage-hebor]service-type telnet		#设置用户服务类型
[R1-luser-manage-hebor]authorization-attribute user-role network-admin	#设置用户级别
[R1-luser-manage-hebor]line vty 0 63
[R1-line-vty0-63]authentication-mode scheme		#修改认证方式
```

Comware操作系统的line命令和user-interface命令都是用于用户线视图配置；不指定新建账户类型时，默认是manage类型账户

**SSH远程登录**

```Comware
[R1]dis ssh server status	#查看SSH服务状态
[R1]ssh server enable	#开启SSH服务
[R1]line vty 0 63
[R1-line-vty0-63]user-role network-admin
[R1-line-vty0-63]authentication-mode scheme
[R1-line-vty0-63]protocol inbound ssh	#使用vty登录时仅允许ssh协议登录。此参数不是必选项，不设置此选项时缺省是允许所有方式登录
[R1-line-vty0-63]local-user hebor class manage
[R1-luser-manage-hebor]service-type telnet ssh		#增加支持ssh协议
[R1-luser-manage-hebor]password simple Admin@9000
[R1-luser-manage-hebor]authorization-attribute user-role level-1
[R1-luser-manage-hebor]quit
[R1]super authentication-mode local scheme    #设置super提权
[R1]super password role network-admin simple Admin@huawei.com
[R1]public-key local create rsa		#生成RSA密钥
[R1]public-key local export rsa ssh2	#导出RSA密钥，此命令会直接将公钥内容标准输出，也可以选择将公钥内容导入指定文件
[SW1]public-key local destroy rsa	#销毁RSA密钥
```

### Comware文件系统管理

Comware的配置文件通过文本文件的形式保存，每一个单独视图下的配置文件保存在一起，各个视图下的配置通过“#”号分隔

```Comware
<H3C>dis current-configuration 
#
 version 7.1.064, Release 0427P22
#
 sysname H3C
#
 system-working-mode standard
 xbar load-single
 password-recovery enable
 lpu-type f-series
......
```

**配置文件**

Comware的主配置文件默认命名为startup.cfg，初始状态下startup.cfg文件是不存在的，需要手动执行save生成startup.cfg文件，如果用户指定了启动主配置文件，且主配置文件存在，则以主配置文件进行初始化，如果用户指定的主配置文件不存在，则以空配置进行初始化；配置文件又分为主配置文件和备用配置文件，主配置文件出现问题时，可以通过启用备用配置文件进行初始化

```Comware
<H3C>dis startup
 Current startup saved-configuration file: NULL 
 Next main startup saved-configuration file: flash:/startup.cfg
 Next backup startup saved-configuration file: NULL		#备用配置文件
 
<H3C>reset saved-configuration	#擦除配置，已保存的主配置文件会被删除
<H3C>startup saved-configuration test.cfg	#设置下次启动的配置文件
<H3C>save force		#强制保存，默认会保存到下一次启动的配置文件，如果下一次启动的配置文件未指定，默认保存到startup.cfg配置文件

# 备份/恢复下次启动配置文件
<H3C>backup startup-configuration to <tftp-server> [dest-filename]	#将下次启动的配置文件备份到远程ftp服务器
<H3C>restore startup-configuration from <tftp-server> src-filename	#从远程ftp服务器拉取配置文件到本地，并将该配置文件作为下次启动文件
```

**启动文件**

```Comware
<H3C>boot-loader file <file-path>	#指定下次启动加载的系统引导文件
<H3C>display boot-loader
```

**FTP**

```Comware
[H3C]dis ftp-server		#查看FTP服务状态
[H3C]ftp server enable	#启动FTP服务
[H3C]local-user hebor	#创建本地用户，默认类型为manage
[H3C-luser-manage-hebor]service-type ftp	#设置用户服务类型
[H3C-luser-manage-hebor]password simple Admin@huawei.com
[H3C-luser-manage-hebor]authorization-attribute user-role level-15
```

**网络设备的一般引导过程**

![网络设备的一般引导过程](https://www.z4a.net/images/2023/11/28/d87a29d90b369e750f133c78993de7b4.png)

### 系统调试

对网络设备所支持的绝大部分协议和功能，Comware都提供了相应的调试功能，协助管理员对错误进行诊断和定位，调试信息的输出由两个开关控制：协议调试开关、标准输出开关

![系统调试介绍](https://www.z4a.net/images/2023/11/28/e93961da048501dcda48597e28f20aef.png)

```Comware
<H3C>terminal monitor    #开启控制台对系统信息的监视功能，设置当前终端可以显示操作日志。它属于系统信息的一种，默认就是开启的
<H3C>terminal debugging    #打开调试信息的屏幕输出开关，设置当前终端可以显示调试日志
<H3C>debugging ip icmp    #打开IP模块的调试开关
<H3C>display debugging    #查看已打开的调试开关
<H3C>undo debugging all    #关闭所有调试开关
```

在设备正常工作的情况下不建议使用debugging，debugging工具本身比较占用设备资源，在调试开关打开的情况下可能会有大量的日志信息输出到屏幕，此时可能无法通过undo命令关闭调试日志，可以使用组合键`Ctrl+O`