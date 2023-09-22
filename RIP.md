#### RIP

距离矢量协议（DV）：RIP、BGP <br/>链路状态协议（LS）：OSPF、IS-IS

RIP协议构建、维护、更新RIB，并使用UDP 520端口进行路由信息交换，支持水平分割、毒性逆转和触发更新，优先级为100，默认配置RIP协议时是v1。在IGP中`network`有2层含义：1. 在一个范围内运行RIP 2. 该范围内的网络被通告到RIP中

```diff
+ RIP的两种报文：Requiest、Response
```

##### RIP工作原理

设备运行RIP协议，相互发现邻居后发送`Requiest`和`Response`报文，形成RIP的数据库，网络稳定后周期性发送路由更新信息

##### RIPv1

RIPv1																		RIPv2								
 1. 有类路由协议，不支持VLSM和CIDR		       1. 无类路由协议
 2. 广播的形式发送报文									 	2. 支持组播和广播
 3. 不支持认证												  	   3. 支持明文和MD5认证
  4. 不支持TAG														 4. 更新报文携带掩码
  5. 有类路由协议，不支持汇总

```diff
两类汇总：手动汇总、自动汇总
[AR2-rip-1]summary always	#开启自动汇总
[AR2-rip-1]undo summary		#关闭自动汇总
```



示例：RIPv1拓扑

```
		AR1 10.0.1.1/24
		 |
	-----------	10.0.123.0/24
	|		  |
   AR2		 AR3 10.0.3.3/24
```

示例：RIPv1配置

```
[AR1]dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              10.0.123.1/24        up         up        
LoopBack0                         10.0.1.1/24          up         up(s)     

[AR2]dis ip inte brief 
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              10.0.123.2/24        up         up        
LoopBack0                         20.0.2.2/24          up         up(s)     
LoopBack1                         10.0.2.2/24          up         up(s)     

[AR3]dis ip inte brief
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              10.0.123.3/24        up         up        
LoopBack0                         10.0.3.3/24          up         up(s)     

[AR2]rip 1
[AR2-rip-1]dis this
rip 1
 version 1
 network 10.0.0.0
 network 20.0.0.0
 
[AR1]dis rip 1 neighbor 	#查看RIP邻居
---------------------------------------------------------------------
 IP Address      Interface                   Type   Last-Heard-Time
---------------------------------------------------------------------
 10.0.123.2      GigabitEthernet0/0/0        RIP    0:0:5
 Number of RIP routes  : 2
 10.0.123.3      GigabitEthernet0/0/0        RIP    0:0:8
 Number of RIP routes  : 1
 
[AR1]dis rip 1 database 	#查看RIP数据库
 ---------------------------------------------------
 Advertisement State : [A] - Advertised					通告
                       [I] - Not Advertised/Withdraw	将要撤销的路由
 ---------------------------------------------------
   10.0.0.0/8, cost 0, ClassfulSumm
       10.0.1.0/24, cost 0, [A], Rip-interface
       10.0.2.0/24, cost 1, [A], nexthop 10.0.123.2
       10.0.3.0/24, cost 1, [A], nexthop 10.0.123.3
       10.0.123.0/24, cost 0, [A], Rip-interface
   20.0.0.0/8, cost 1, ClassfulSumm
   20.0.0.0/8, cost 1, [A], nexthop 10.0.123.2
   
[AR1]dis ip routing-table pro rip
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       10.0.2.0/24  RIP     100  1           D   10.0.123.2      GigabitEthernet0/0/0
       10.0.3.0/24  RIP     100  1           D   10.0.123.3      GigabitEthernet0/0/0
       20.0.0.0/8   RIP     100  1           D   10.0.123.2      GigabitEthernet0/0/0
```

R1上运行RIPv1，得到的是`10.0.2.0/24`的路由，而不是`10.0.0.0/8`的路由，因为发送路由更新的接口的地址是`10.0.123.0/24`，发送出去的路由是`10.0.2.0/24`，主类前缀相同，那么发送明细路由`10.0.2.0/24`，如果主类前缀不同，发送主类即可

##### RIPv2

更新定时器-Update time-30s：即使网络没有发生任何变化，每隔30s周期性更新<br/>老化计时器-Age time-180s：超过180s没有从邻居收到路由更新则认为路由不可达<br/>垃圾收集时间-Garbage collect-120s：在垃圾收集时间内没有从邻居收到路由更新则从RIP路由中删除此路由条目<br/>抑制定时器-Suppress timer：默认为0，大部分设备无法修改，即使调整也无法生效。当RIP设备从邻居收到cost为16的路由更新时，对应路由进入抑制状态，并启动抑制定时器。为了防止路由震荡，在抑制定时器超时前，再收到邻居cost小于16的更新也不接受。

```diff
- `update time`与`age time`同时开始计时，`garbage collect`在`age time`结束后开始计时
- 处于`garbage collect`时，路由表中已经将路由条目删除，在RIP路由中也处于16条不可达的状态
```

```
<R4>dis curr configuration rip 		#查看全局配置中关于RIP的配置
[V200R003C00]
#
rip 1
 undo summary
 version 2
 network 10.0.0.0
 silent-interface Serial2/0/0		#静默端口。仅接收RIP的更新而不给发送更新
#
return

<R1>dis rip 1 route 				#查看RIP路由
 Route Flags : R - RIP
               A - Aging, G - Garbage-collect
 ----------------------------------------------------------------------------
 Peer 10.0.14.4 on Serial2/0/0
      Destination/Mask        Nexthop     Cost   Tag     Flags   Sec
         10.1.0.0/23         10.0.14.4     16    0        RG      43
         10.0.4.0/24         10.0.14.4     16    0        RG      43
         #此时已经处于垃圾回收时间，开销值为16条，不可达
 Peer 10.0.123.2 on GigabitEthernet0/0/0
      Destination/Mask        Nexthop     Cost   Tag     Flags   Sec
         10.0.2.0/24        10.0.123.2      1    0        RA      19
 Peer 10.0.123.3 on GigabitEthernet0/0/0
      Destination/Mask        Nexthop     Cost   Tag     Flags   Sec
         10.0.3.0/24        10.0.123.3      1    0        RA      32
```

##### RIP的防环机制

1. 水平分割 Split-Horizon：从邻居收到路由就不会再将该路由发回给邻居
2. 毒性逆转 Poison-reverse：从邻居收到路由，将cost改为16后发回给邻居
3. 触发更新：路由信息发生变化时，立即向邻居发送触发更新报文

```diff
- 水平分割时所有DV共有的特征
- 毒性逆转默认不开启，如果开启了毒性逆转，水平分割会失效，查看RIP的接口信息时水平分割依然显示为开启状态
```

示例：开启毒性逆转

```
[R4-Serial2/0/0]rip poison-reverse
```

