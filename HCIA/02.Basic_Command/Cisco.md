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
Switch1(config)#interface e0/0
Switch1(config-if)#no switchport    #交换机端口缺省状态下是2层端口，此命令将端口转换成3层端口
Switch1(config-if)#ip address 192.168.0.1 255.255.255.0
Switch1(config-if)#no shutdown    #Cisco路由器所有端口默认都是shutdown状态，不确定交换机是否也是如此，建议统一习惯
Switch2#ping 192.168.0.1 repeat 10    #发起10次ping包

Switch1(config)#enable password cisco    #设置特权模式的明文密码，密码字符可以通过running-config直接查看到
Switch1(config)#enable secret cisco    #设置特权模式的密文密码，通过running-config查看密码时显示加密字符
```

> **输入 ? 号**
>
> 华为VRP操作系统中无法输入?字符，IOS中先输入组合键ctrl+v，再通过shift+?即可在命令行中输入?号，在IOS系统中，?号能够作为密码使用
>
> **特权模式密码字符**
>
> `enable password <密码字符>`命令中的密码字符，空格也会被认为是一个密码字符，并且在查看`running-config`配置时，密码字符中的空格不易观察，因此，在设置特权模式密码时切忌多敲一个空格

### **可选性配置**

1. **关闭域名解析**

   一些比较常见的情况是，在CLI命令行下容易敲快键盘时可能会敲错命令，输入错误的命令不属于IOS的命令集时，IOS会认为管理员此时是在寻找一个域名，从而IOS会帮助管理员做域名解析，向网络中广播寻找DNS服务器，如果网络中没有DNS服务器，则这个解析的过程大概会维持半分钟左右，直到解析超时才会回到CLI命令行

   ```IOS
   Switch1#shou
   Translating "shou"...domain server (255.255.255.255)
   % Unknown command or computer name, or unable to find computer address
   ```

   为了防止这种情况可以通过`no ip domain lookup`命令关闭域名解析，但关闭域名解析的前提是 *当前网络设备* 没有任何需要访问域名的服务，在网络设备上关闭域名解析不会影响到下游的PC的域名解析，只要PC自身配置有DNS服务器的IP

2. **关闭会话超时**

   关闭会话超时的场景一般用于调试多个设备的场景，如果设备上都配置有`enable password <密码字符>`，则每次从用户模式进入特权模式时都需要输入密码，一个会话如果长时间无响应则会自动退出会话，而再次连接会话时则进入用户模式

   ```IOS
   Switch1(config)#line console 0
   Switch1(config-line)#no exec-timeout    #关闭console接口的会话超时
   Switch1(config-line)#exec-timeout 5 10    #超过5分10秒无响应则退出会话
   ```

   关闭会话超时是不安全的，如果忘记恢复会话超时配置，则下一位管理员接入设备时仍是无需密码即可直连设备的

3. **开启信息同步**

   在设备配置过程中，设备本身的提醒信息非常多，例如开关端口、开关协议等操作都会导致设备生成日志或会话信息，并输出到终端，默认情况下这些设备的提示信息会切断管理员在当前CLI命令行下正在输入的配置命令

   ```IOS
   Switch1(config)#line console 0
   Switch1(config-line)#logging synchronous    #开启信息同步功能
   ```

   开启信息同步配置后，设备输出调试信息时会单独保留一行管理员的CLI命令行输入。开启信息同步需要区别于关闭信息回显，关闭信息回显是直接将设备的调试信息全部关闭，设备不会再在终端输出信息，而开启信息同步只是让设备的调试信息不再切断管理员的CLI命令行输入，并不关闭信息输出

### 配置文件

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
Switch1#erase startup-config    #删除启动配置文件
Switch1#erase flash:    #删除磁盘文件
```

删除启动配置文件时通过`write erase`命令可以实现同样的效果。在EVE-NG模拟器下，即便没有执行过`write`保存启动配置文件，在`nvram:`中也会存在一个`startup-config`文件，只不过该文件字节大小为0，执行`write`保存启动配置文件后，`nvram:/startup-config`文件的字节大小会发生改变，可以通过这种方式判断启动配置文件是否保存成功

更加直观的观察启动配置文件是否保存成功的方式是使用`show startup-config`命令，如果保存成功则会直接输出配置文件内容，如果启动配置文件未生效则会出现提示`startup-config is not present`

### 远程登陆

```IOS
Switch1(config)#line vty 0 4    #同时允许5个用户在线
Switch1(config-line)#password Admin@9000    #设置远程登陆密码
Switch1(config-line)#login    #开启远程登陆功能

Switch1(config)#username ccna privilege 15 secret cisco    #设置多用户登陆
```

**踢出远程在线用户**

```IOS
Switch1>show user all    #查看远程在线用户信息
Switch1#show line    #查看VTY进程号和对应在线用户数
*Apr 11 03:35:03.563: %SYS-5-CONFIG_I: Configured from console by console
Switch1#show line 
   Tty Typ     Tx/Rx    A Modem  Roty AccO AccI   Uses   Noise  Overruns   Int
*    0 CTY              -    -      -    -    -      0       0     0/0       -
     1 AUX   9600/9600  -    -      -    -    -      0       0     0/0       -
*    2 VTY              -    -      -    -    -      3       0     0/0       -    #3个在线用户
     3 VTY              -    -      -    -    -      0       0     0/0       -
     4 VTY              -    -      -    -    -      0       0     0/0       -
     5 VTY              -    -      -    -    -      0       0     0/0       -
     6 VTY              -    -      -    -    -      0       0     0/0       -
     
Switch1#clear line 2    #输入tty进程号，清理该进程在线用户
```

