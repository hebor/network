# IOS

Cisco IOS（Internetwork Operating System，网络操作系统）是Cisco公司用于管理和控制网络设备的操作系统，根据不同的设备和需求，Cisco IOS存在多个版本和种类，应用在企业级路由器和交换机上的是IOS XE版本，其他产品线因为收购的原因各自使用的操作系统不同

| 种类         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| Cisco IOS    | 基于Unix开发，早期最基本的Cisco IOS版本，适用于大多数Cisco路由器和交换机 |
| Cisco IOS XE | 基于Linux开发的IOS版本，用于Cisco网络设备的下一代路由器和交换机，提供了更高的可靠性、可伸缩性、灵活性 |
| Cisco IOS XR | 专为大型并且需要高可靠性的网络环境设计的IOS版本，适用于Cisco CRS-1和Cisco ASR 9000系列路由器 |
| Cisco NS-OS  | 运行在Cisco Nexus系列交换机上的操作系统，具备高可用性、灵活性、可扩展性，并适用于大型数据中心和企业网络 |
| Cisco ASA-OS | Cisco Adaptive Security Appliance（ASA）系列防火墙的操作系统，它提供了网络安全功能，如防火墙、虚拟专用网络（VPN）、入侵防御系统（IDS） |

**命令行系统视图**


| 命令行 | 视图 | 作用 |
| --- | --- | --- |
| Switch> | 用户模式 | 查看运行状态或其他参数 |
| Switch# | 特权模式 | 配置设备的系统参数 |
| Switch(config)# | 全局配置模式 | 具备设备最高操作权限 |

## IOS基础命令

**IOS CLI 快捷键**

| 组合键       | 说明                                                  |
| ------------ | ----------------------------------------------------- |
| Tab          | 补全命令或关键字的剩余部分                            |
| Ctrl+A       | 光标移动到命令行首部                                  |
| Ctrl+E       | 光标移动到命令行末尾                                  |
| Backspace    | 删除光标左侧的一个字符                                |
| Ctrl+U       | 删除整行。Ctrl+X能实现同样的效果                      |
| Ctrl+Shift+6 | 打断Cisco IOS目前正在执行的命令，例如ping或traceroute |
| Ctrl+C       | 中止当前命令并退出配置模式                            |
| Ctrl+Z       | 结束配置模式并返回EXEC提示符                          |

`Ctrl+C`和`Ctrl+Z`实现的效果几乎是一样的，都会退回特权模式，但如果在使用快捷键之前已经在命令行输入过指令，那么`Ctrl+C`不会执行指令并退回到特权模式，而`Ctrl+Z`会执行完指令后退回到特权模式

**基本命令**

```IOS
Switch>show clock    #查看设备时钟
Switch>enable    #从用户模式进入特权模式
Switch#configure terminal    #从特权模式进入全局配置模式
Switch(config)#clock timezone UTC +8    #修改本地时区
Switch(config)#exit    #用于高级权限的配置模式下回退到低一级的配置模式
Switch#clock set 22:12:0 APR 6 2024    #修改本地时间
Switch#disable    #从特权模式回退到用户模式
Switch>quit    #退出用户模式。主要用于远程访问时退出登录
Switch(config)#hostname Switch1     #修改设备名
Switch(config)#end    #可以在任意高级权限的配置模式下，直接回退到特权模式
Switch1#show history    #查看历史命令
Switch1#terminal history size 50    #配置历史命令的缓存数量
Switch1#show ip interface brief    #查看交换机端口状态
Switch1(config)#arp 192.168.0.1 aaaa.bbbb.cccc arpa e0/0    #设置静态ARP绑定
```

> **输入 ? 号**
>
> 华为VRP操作系统中无法输入?字符，IOS中先输入组合键ctrl+v，再通过shift+?即可在命令行中输入?号，在IOS系统中，?号能够作为密码使用

**配置文件**

通常Cisco的存储器分为四种

| 存储器                                    | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| ROM（Read Only Memory）                   | ROM存储路由器加电自检（POST, Power-On Self-Test）、启动程序（Bootstrap Program）和部分或全部IOS。路由器的ROM是可擦写的，所以IOS是可升级的 |
| NVRAM（Nonvolatile Random Access Memory） | 非易失RAN，存储路由器的启动配置文件。NVRAM是可擦写的，可将路由器的配置信息拷贝到NVRAM中 |
| FLASH RAM                                 | 闪存，一种特殊的ROM，可擦写、编程，用于存储Cisco IOS的其他版本，用于对路由器的IOS进行升级，FLASH是可以被格式化的 |
| RAM（Random Access Memory）               | RAM与PC上的内存相似，提供临时信息的存储，同时保存设备当前的路由表和配置信息 |

IOS系统更加强化的NVRAM的存在，易失性存储（RAM）用于保存实时配置文件`running-config`，更改的配置会直接在RAM中生效，非易失性存储（NVRAM）用于保存启动配置文件`startup-config`，`startup-config`会在设备启动时生效。华为的NVRAM存储空间命名为`flash:`，IOS的NVRAM存储空间命名为`nvram:`，IOS的启动配置文件保存在`nvram:`中

```IOS
Switch1#show running-config    #查看正在运行的配置
Switch1#show startup-config    #查看启动配置文件
Switch1#copy running-config startup-config    #将内存中的配置保存到非易失存储
Switch1#write    #保存配置到startup-config，一种快捷保存方式
Switch1#dir nvram:    #查看nvram目录
```