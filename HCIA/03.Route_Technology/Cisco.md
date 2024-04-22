# 静态路由

Cisco的管理距离与Huawei管理距离存在差异

|        | 直连 | 静态 | RIP  | EIGRP | OSPF | IGRP | IBGP | EBGP | ODR  | ISIS |
| ------ | ---- | ---- | ---- | ----- | ---- | ---- | ---- | ---- | ---- | ---- |
| Huawei | 0    | 60   | 100  |       | 10   |      |      |      |      | 15   |
| Cisco  | 0    | 1    | 120  | 90    | 110  | 100  | 200  | 20   | 160  | 115  |

ODR属于思科私有协议，ODR协议用于动态生成缺省路由，EIGRP原本也属于思科私有协议，后被公有化，但用的不多

**配置命令**

| 命令                                               | 说明           |
| -------------------------------------------------- | -------------- |
| show ip route                                      | 查看路由表     |
| ip route {目标IP/目标IP段} {子网掩码} {下一跳地址} | 添加静态路由   |
| show ip route static                               | 查看静态路由表 |

**静态路由解析**

![Cisco_静态路由表解析](https://www.z4a.net/images/2024/04/17/Cisco_.png)

## 静态路由实验

![Cisco_eeb6623fc82482a2.png](https://www.z4a.net/images/2024/04/17/Cisco_eeb6623fc82482a2.png)

**实现全网互通参考配置**

```IOS
--------------------------------------SW1--------------------------------------
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1
SW1(config)#no ip domain lookup
SW1(config)#line console 0
SW1(config-line)#no exec-timeout
SW1(config-line)#logging synchronous
SW1(config-line)#interface e0/0
SW1(config-if)#no switchport
SW1(config-if)#ip address 192.168.1.1 255.255.255.0
SW1(config-if)#ip route 0.0.0.0 0.0.0.0 192.168.1.2

--------------------------------------SW2--------------------------------------
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW2
SW2(config)#no ip domain lookup
SW2(config)#line console 0
SW2(config-line)#no exec-timeout
SW2(config-line)#logging synchronous
SW2(config-line)#interface e0/0
SW2(config-if)#no switchport
SW2(config-if)#ip address 192.168.1.2 255.255.255.0
SW2(config-if)#ip route 192.168.3.0 255.255.255.0 192.168.2.3
SW2(config)#ip route 4.4.4.4 255.255.255.255 192.168.2.3
SW2(config)#interface e0/1
SW2(config-if)#no switchport
SW2(config-if)#ip address 192.168.2.2 255.255.255.0

--------------------------------------SW3--------------------------------------
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW3
SW3(config)#no ip domain lookup
SW3(config)#line console 0
SW3(config-line)#no exec-timeout
SW3(config-line)#logging synchronous
SW3(config-line)#interface e0/0
SW3(config-if)#no switchport
SW3(config-if)#ip address 192.168.2.3 255.255.255.0
SW3(config-if)#interface e0/1
SW3(config-if)#no switchport
SW3(config-if)#ip address 192.168.3.3 255.255.255.0 
SW3(config-if)#ip route 192.168.1.0 255.255.255.0 192.168.2.2
SW3(config)#ip route 4.4.4.4 255.255.255.255 192.168.3.4

--------------------------------------SW4--------------------------------------
Switch>enable 
Switch#configure terminal
Switch(config)#hostname SW4
SW4(config)#no ip domain lookup
SW4(config)#line console 0
SW4(config-line)#no exec-timeout
SW4(config-line)#logging synchronous
SW4(config-line)#interface e0/0
SW4(config-if)#no switchport
SW4(config-if)#ip address 192.168.3.4 255.255.255.0
SW4(config-if)#interface loopback 0
SW4(config-if)#ip address 4.4.4.4 255.255.255.255
SW4(config-if)#ip route 0.0.0.0 0.0.0.0 192.168.3.3
```

## RIP路由实验

![Cisco_RIP路由实验](https://www.z4a.net/images/2024/04/20/Cisco_RIP.png)

```IOS
--------------------------------------R1--------------------------------------
Router>enable 
Router#configure terminal
Router(config)#hostname R1
R1(config)#no ip domain lookup
R1(config)#line console 0
R1(config-line)#no exec-timeout
R1(config-line)#logging synchronous
R1(config-line)#interface loopback 0
R1(config-if)#ip address 1.1.1.1 255.255.255.255
R1(config-if)#interface g0/0
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#router rip 
R1(config-router)#version 2
R1(config-router)#no auto-summary
R1(config-router)#network 1.0.0.0
R1(config-router)#network 192.168.1.0

--------------------------------------R2--------------------------------------
Router>enable
Router#configure terminal
Router(config)#hostname R2
R2(config)#no ip domain lookup
R2(config)#line console 0
R2(config-line)#no exec-timeout
R2(config-line)#logging synchronous
R2(config-line)#interface g0/0
R2(config-if)#ip address 192.168.1.2 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#interface g0/1
R2(config-if)#ip address 192.168.2.2 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#router rip
R2(config-router)#version 2
R2(config-router)#no auto-summary
R2(config-router)#network 192.168.1.0
R2(config-router)#network 192.168.2.0

--------------------------------------R3--------------------------------------
Router>enable
Router#configure terminal
Router(config)#hostname R3
R3(config)#no ip domain lookup
R3(config)#line console 0
R3(config-line)#no exec-timeout
R3(config-line)#logging synchronous
R3(config-line)#interface g0/1
R3(config-if)#ip address 192.168.2.3 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#interface loopback 0
R3(config-if)#ip address 3.3.3.3 255.255.255.255
R3(config-if)#router rip 
R3(config-router)#version 2
R3(config-router)#no auto-summary
R3(config-router)#network 3.0.0.0
R3(config-router)#network 192.168.2.0
R3(config-router)#end
R3#show ip rip database
```

![Cisco_RIP路由表解析](https://www.z4a.net/images/2024/04/20/Cisco_RIP64c1dd5663483f85.png)
