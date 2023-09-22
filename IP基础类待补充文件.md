
##### ARP

基础ARP：从逻辑地址映射到一个物理地址。ARP请求目的MAC是`全F`<br/>代理ARP：不同网络的设备在不配置网关的情况下能够通过ARP代理实现通信<br/>免费ARP：最主要的作用是检测地址冲突

示例：ARP代理

```
[AR1]ip route-static 23.1.1.0 24 g0/0/0
[AR2-GigabitEthernet0/0/0]arp-proxy enable
[AR2-GigabitEthernet0/0/1]arp-proxy enable
[AR3]ip route-static 12.1.1.0 24 g0/0/1
```

```diff
- 实现ARP代理需要满足两个条件：1.没有路由功能的主机，2.有路由功能，目的地指向本地端口
```

##### VRP

用户视图 - 系统视图 - 接口视图 - 协议视图

设备启动时会加载保存的配置文件到内存，并作为当前配置文件

```diff
+ <AR3>dis current-configuration	#显示当前配置文件
+ <AR3>dis saved-configuration		#显示已保存的配置文件
+ <AR3>dis startup					#查看下次启动的配置文件
+ <AR3>startup saved-configuration sslvpn.zip	#修改下次启动的配置文件为sslvpn.zip
```

##### IP路由基础

路由器的两个基本功能

1. 决策 RIB（路由表，从路由表中选出最优路由添加到FIB）
2. 转发 FIB（转发表，此表中的条目才是真正用于转发数据）

路由原理

1. 最长匹配原则

   0.0.0.0/0 默认路由，最不精确的路由

2. 路由协议间的优先级（AD，管理距离）数值小的优先

   直连 0

   静态 60

   OSPF内部 10

   OSPF外部 150

3. 在同一路由协议中**度量值**较小的优先，*度量值*较小的优先放入`fib`表

```diff
+ 路由协议的优先级由设备本身做出抉择，不受其他设备影响
+ 路由负载不等于数据包的均衡转发，而是分为逐流负载和逐包负载，默认为逐流负载，根据五元组的`hash`值决定路径，五元组分别是SIP、DIP、SPORT、DPORT、协议
```

##### 静态路由

静态路由支持等价负载分担（两台设备之间通过两根线路相连）

示例：链路负载拓扑

```diff
        |---- AR2 ----|
AR1 ----              ---- AR4
        |---- AR3 ----|
```

示例：链路负载配置

```
1. 4台设备基本IP配置
[AR1]dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              12.1.1.1/24          up         up        
GigabitEthernet0/0/1              13.1.1.1/24          up         up        

<AR2>dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              12.1.1.2/24          up         up        
GigabitEthernet0/0/1              24.1.1.2/24          up         up        

<AR3>dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              34.1.1.3/24          up         up        
GigabitEthernet0/0/1              13.1.1.3/24          up         up        

<AR4>dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              34.1.1.4/24          up         up        
GigabitEthernet0/0/1              24.1.1.4/24          up         up        
LoopBack0                         4.4.4.4/24           up         up(s)     

2. 静态路由配置
[AR1]dis current-configuration  | include route-static
ip route-static 4.4.4.4 255.255.255.255 GigabitEthernet0/0/0
ip route-static 4.4.4.4 255.255.255.255 13.1.1.3
[AR1]dis ip routing-table pro static
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        4.4.4.4/32  Static  60   0           D   12.1.1.1        GigabitEthernet0/0/0
                    Static  60   0          RD   13.1.1.3        GigabitEthernet0/0/1
	# IP报文是逐跳转发，如果静态路由的下一跳指定为端口，那这条路由在路由表中就被认定为直连路由。这也是`D`与`RD`的区别
	# 而直连路由转发报文会先请求DMAC，如果实际DIP非直连路由，那么会出现目标不可达

<AR2>dis curr | include route-static
ip route-static 4.4.4.4 255.255.255.255 24.1.1.4
<AR3>dis curr | include route-static
ip route-static 4.4.4.4 255.255.255.255 34.1.1.4
ip route-static 12.1.1.0 255.255.255.0 13.1.1.1
<AR4>dis curr | include route-static
ip route-static 12.1.1.0 255.255.255.0 24.1.1.2
ip route-static 12.1.1.0 255.255.255.0 34.1.1.3

3. ARP代理
[AR2-GigabitEthernet0/0/0]arp-proxy enable
	# AR1的出口线路有两条负载，但逐流负载模式根据五元组的`hash`值决定的路径是从端口`G0/0/0`出，而`G0/0/0`由是直接指的端口
	# 所以需要在AR2的`G0/0/0`端口上配置ARP代理，使AR1的ARP表中能够获取到AR4的`loop0`端口的MAC地址

<AR2>debugging ip icmp
<AR2>terminal debugging
```



示例：浮动路由拓扑

```
     ----------
R1 --|        |-- R2
     ----------
```

示例：浮动静态路由

```
[R1]dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              12.1.1.1/24          up         up        
GigabitEthernet0/0/1              12.2.2.1/24          up         up        

[R2]dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              12.1.1.2/24          up         up        
GigabitEthernet0/0/1              12.2.2.2/24          up         up          
LoopBack0                         2.2.2.2/24           up         up(s)     

[R1]dis curr | include route-static
ip route-static 2.2.2.2 255.255.255.255 12.1.1.2
ip route-static 2.2.2.2 255.255.255.255 12.2.2.2 preference 254

[R1]dis ip routing-table pro static
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Public routing table : Static
         Destinations : 1        Routes : 2        Configured Routes : 2

Static routing table status : <Active>
         Destinations : 1        Routes : 1

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        2.2.2.2/32  Static  60   0          RD   12.1.1.2        GigabitEthernet0/0/0

Static routing table status : <Inactive>
         Destinations : 1        Routes : 1

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        2.2.2.2/32  Static  254  0          R    12.2.2.2        GigabitEthernet0/0/1
```

```diff
- 浮动静态路由：在有多条线路的情况下使数据仅从其中一条链路上转发，路由表中也仅存在一个下一条，其他线路作为备份。只有主链路失效时备份链路才被放入路由表
```











```diff
+ OSPF的协议号是89，链路状态协议 LS，OSPF内部优先级是10，由外部引进的外部路由优先级是150
+ VLSM：子网划分
+ CIDR：路由聚合
```

##### ospf基本特点 

在ospf协议中，每个路由器将自身的直连路由分享出去称之为泛洪LSA，由不同路由器传递过来的不同的路由会在每个路由器上形成一个路由数据库称之为LSDB，经过SPF算法，路由器会得到通往任意网段的最短路径树，然后再将路由写入到自身的路由表

```diff
- OSPF协议中每一台路由器收到的LSA都是最原始的，未经任何更改的路由
+ Router ID:用来在自治系统中唯一标识一台运行OSPF的路由器，每个路由器都有一个Router ID
+ 自治系统(Autonomous System):使用同一种路由协议交换路由信息的一组路由器
+ LSA：链路状态通告
+ 泛洪LSA：每个路由器都会分享出自身的直连网段
```



**7种LSA：**

1.router-LSA：路由LSA，每个路由器都会产生，包含的是路由器的端口信息和链路状态（直连	

路由），只能在本区域内洪泛

组成：LINK-ID --> 链路IP

  	 AREA-ID --> 区域ID

  ROUTER-ID：这个LSA是由谁产生的

 SEQU --> 随机数值

 

2.network-LSA：网络LSA，只会出现在广播型网络中，BDR 和 DROther会把自己产生的 1 类

LSA 发送给 DR，DR 再发送 2 类 LSA 给 BDR 和 DROther



 

3.NETWORK-SUMMARY LSA：网络汇总LSA，由ABR产生，将区域内 1 类LSA转换为 3 类

LSA洪泛	到非骨干区域，可以穿透区域

 

4.ASBR-SUMMARY LSA，自治系统汇总LSA，将本区域内的 5 类 LSA 变成 4 类 LSA 洪泛到其他区域。外部路由由 ASBR 转化为 5 类 LSA 洪泛，传到其他区域有 ABR 代发（代发的是 4 类 LSA）

 

5.AS-EXTENEL LSA，自治系统LSA，由 ASBR 产生，用来描述如何去往另外一个 AS（可以在整个 OSPF 区域洪泛，但一般会转换成 4 类 LSA 进行转发）

 

6.6类LSA，用于OSPF的组播

 

7.NSSA LSA，由 ASBR 产生，所有区域洪泛，因为末梢区域不能存在 4，5 类 LSA，所以就用 7 类 LSA 代替

ospf 注入 AS 的 LSA

ospf 1 router-id 1.1.1.1

area 0

import-route [direct]		// 注入其他协议路由 [直连]

dis ospf lsdb router

**四种特殊区域：**

  **末梢区域 (StubNet)**：允许 1，2，3 类 LSA，拒绝 4，5 类 LSA

该区域内 ABR 会下发一条默认路由指向 ABR

[area-0]stub

  **完全的末梢区域**：允许 1，2 类 LSA，拒绝 3，4，5 类 LSA

与末梢区域同理，ABR 会下发一条默认路由指向 ABR

[area-0]stub no-summary

  **非完全的完全末梢区域**：出现在 ASBR 所在区域，在 ASBR 所在的区域内拒绝 4，5 类，出现第 7 

  类 LSA，允许 1，2，3，7 类 LSA

[area-0]nssa

  **完全的非完全的完全末梢区域**：出现在 ASBR 所在区域，在 ASBR 所在区域内拒绝 3，4，5 类  

  LSA，允许 1，2，7 类 LSA，ABR 下发一条默认路由指向 ABR

[area-0]nssa no-summary

**虚拟链路：**

  当 ospf 中某区域因为无法连接到 ABR 上，而无法获得 ospf 所有区域的 LSA 时，该区域就变成了信息孤岛，无法与 ospf 区域中的主机通信

  解决方法：在 ABR 与连接该区域的路由器之间设置一条虚拟链路，逻辑上看起来就像是该区域连接在 ABR 上一样

  [R3-ospf-1-area-2] vlink-peer 6.6.6.6

  [R6-ospf-1-area-2] vlink-peer 3.3.3.3

​    // 在两台路由器之间相互配置虚拟链路（必须在同一 area 中）

ospf 验证：

  区域验证：[R2-ospf-1-area-0] authen md5 1 cipher redhat

// 处于同一区域的设备都需要进行验证

  端口验证：[R2-e0/0/0] ospf authen md5 1 cipher redhat

// 对应端口进行验证

##### OSPF实施

拓扑：简单实施OSPF

```
				R1 --10.0.14.0/24-- R4
  10.0.123.0/24	| 
			R2 --- R3 10.0.3.3/24	
```

示例：接口视图下启用OSPF

```
[R1]ospf 1
[R1-ospf-1]dis this							#在接口视图下启用OSPF时，必须先在协议视图下划分area，否则接口不会运行OSPF
ospf 1 
 area 0.0.0.0 

[R1-ospf-1]inte g0/0/0
[R1-GigabitEthernet0/0/0]dis this
interface GigabitEthernet0/0/0
 ip address 10.0.123.1 255.255.255.0 
 rip version 1
 ospf enable 1 area 0.0.0.0					#该接口启用OSPF协议，并将该接口所在的网段通告到OSPF中
 
[R1]dis ospf interface 						#查看运行了OSPF的接口

         OSPF Process 1 with Router ID 10.0.123.1
                 Interfaces 

 Area: 0.0.0.0          (MPLS TE not enabled)
 IP Address      Type         State    Cost    Pri   DR              BDR 
 10.0.123.1      Broadcast    DR       1       1     10.0.123.1      10.0.123.2
 10.0.1.1        P2P          P-2-P    0       1     0.0.0.0         0.0.0.0
 10.0.14.1       P2P          P-2-P    48      1     0.0.0.0         0.0.0.0

[R1]dis ospf peer brief						#查看OSPF邻居

         OSPF Process 1 with Router ID 10.0.123.1
                  Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id          Interface                        Neighbor id      State    
 0.0.0.0          GigabitEthernet0/0/0             10.0.123.2       Full        
 0.0.0.0          GigabitEthernet0/0/0             10.0.123.3       Full        
 0.0.0.0          Serial2/0/0                      10.1.0.1         Full        
 ----------------------------------------------------------------------------
 
<R1>dis ospf lsdb							#查看OSPF的数据库表

         OSPF Process 1 with Router ID 10.0.123.1
                 Link State Database 

                         Area: 0.0.0.0
 Type      LinkState ID    AdvRouter          Age  Len   Sequence   Metric
 Router    10.1.0.1        10.1.0.1            77  60    80000004       0
 Router    10.0.123.3      10.0.123.3         126  48    80000006       1
 Router    10.0.123.2      10.0.123.2         127  48    80000007       1
 Router    10.0.123.1      10.0.123.1          76  72    8000000B       1
 Network   10.0.123.1      10.0.123.1         126  36    80000005       0
 
<R1>dis ospf routing 						#查看OSPF的路由

         OSPF Process 1 with Router ID 10.0.123.1
                  Routing Tables 

 Routing for Network 
 Destination        Cost  Type       NextHop         AdvRouter       Area
 10.0.1.1/32        0     Stub       10.0.1.1        10.0.123.1      0.0.0.0
 10.0.14.0/24       48    Stub       10.0.14.1       10.0.123.1      0.0.0.0
 10.0.123.0/24      1     Transit    10.0.123.1      10.0.123.1      0.0.0.0
 10.0.2.2/32        1     Stub       10.0.123.2      10.0.123.2      0.0.0.0
 10.0.3.3/32        1     Stub       10.0.123.3      10.0.123.3      0.0.0.0
 10.1.0.1/32        48    Stub       10.0.14.4       10.1.0.1        0.0.0.0
```

OSPF的`cost`值的计算是根据路由每经过一台设备的出接口的开销，将所有开销累加计算后得出的值。例如由R2去往R4，R2的出接口是`g0/0/0`口，此接口开销值为`1`，路由经过R1的出接口是`s2/0/0`，此接口的开销值是`48`，在R2上查看去往`10.0.14.0/24`段的`cost`值是`49`，因为`loop0`口的`cost`值是`0`，所以实际`cost`值等于`48+1`

示例：OSPF单区域基本配置

```
[R1]ospf 1024 router-id 1.1.1.1				#创建进程为1024的ospf，并设置该路由器的Router ID为1.1.1.1
[R1-ospf-1024]area 0
[R1-ospf-1024-area-0.0.0.0]network 12.1.1.0 0.0.0.255	#接口的网段属于什么区域，就在哪个区域分享该接口网段
[R1-GigabitEthernet0/0/1]ospf timer hello 5	#设置OSPF的hello包发送间隔，单位为秒

[R1]dis ospf inte g0/0/0					#查看接口运行ospf的参数

         OSPF Process 1 with Router ID 10.0.123.1
                 Interfaces 


 Interface: 10.0.123.1 (GigabitEthernet0/0/0)
 Cost: 1       State: DR        Type: Broadcast    MTU: 1500  
 Priority: 1
 Designated Router: 10.0.123.1
 Backup Designated Router: 10.0.123.2
 Timers: Hello 10 , Dead 40 , Poll  120 , Retransmit 5 , Transmit Delay 1 
 
[R3-GigabitEthernet0/0/0]ospf dr-priority 0	#修改设备接口的DR选举优先级，设置为0后无法再成为BR或BDR
 
[R1-GigabitEthernet0/0/0]dis ospf peer		#在R1上查看邻居的DR与BDR状态

         OSPF Process 1 with Router ID 10.0.123.1
                 Neighbors 

 Area 0.0.0.0 interface 10.0.123.1(GigabitEthernet0/0/0)'s neighbors
 Router ID: 10.0.123.2       Address: 10.0.123.2      
   State: Full  Mode:Nbr is  Master  Priority: 0
   DR: 10.0.123.1  BDR: None   MTU: 0    
   Dead timer due in 37  sec 
   Retrans timer interval: 5 
   Neighbor is up for 00:02:23     
   Authentication Sequence: [ 0 ] 

<R2>dis ospf peer							#在R2上查看邻居的DR与BDR状态

         OSPF Process 1 with Router ID 10.0.123.2
                 Neighbors 

 Router ID: 10.0.123.3       Address: 10.0.123.3      
   State: 2-Way  Mode:Nbr is  Master  Priority: 0	#非DR之间建立2-Way状态
   DR: 10.0.123.1  BDR: None   MTU: 0    
   Dead timer due in 35  sec 
   Retrans timer interval: 5 
   Neighbor is up for 00:00:00     
   Authentication Sequence: [ 0 ] 
```

