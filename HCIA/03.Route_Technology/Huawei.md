# 静态路由实验

```Topology
 <AR2>-G0/0/0-------G0/0/0-<AR1>-G0/0/2-------PC2
   |                         |
 G0/0/1                    G0/0/1
   |                         |  
 <PC4>                     <PC1>
```

```VRP
#---------AR1------------
<Huawei>sys
[Huawei]inte g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 1.0.0.1 24
[Huawei-GigabitEthernet0/0/1]inte g0/0/2
[Huawei-GigabitEthernet0/0/2]ip add 2.0.0.1 24
[Huawei-GigabitEthernet0/0/2]inte g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 12.0.0.1 24

#---------AR2------------
<Huawei>sys
[Huawei]inte g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 12.0.0.2 24
[Huawei-GigabitEthernet0/0/1]inte g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 4.0.0.1 24
```

如果只有直连路由，那么非直连网络就无法达到，如上例，默认情况下AR1上有到PC1、PC2的直连路由，但没有去往PC4的路由，所以PC1、PC2能够互通，但无法连通PC4，如果要所有主机互通则还需要配置默认路由，配置静态路由时，如果出接口是以太网接口，则必须要指定下一跳地址，如果出接口是串口，可以使用下一跳或出接口来配置

以太网是MA类型的网络，如果下一跳指定为出接口，出接口的对端可能是一个或多个设备，数据包无法选择正确的对端接口，因此以太网网络中**建议能写下一跳地址就优先写下一跳地址；切记，通讯是双向的，配置出去的默认路由后，还需要配置回来的路由**

```VRP
#--------------AR1---------------
[Huawei]ip route-static 4.0.0.0 24 12.0.0.2
[Huawei]display ip routing-table protocol static

#--------------AR2---------------
[Huawei]ip route-static 1.0.0.0 24 12.0.0.1
[Huawei]ip route-static 2.0.0.0 24 12.0.0.1
[Huawei]display ip routing-table protocol static
```

## **负载分担**

```Topology
 <AR2>-G0/0/0----------G0/0/0-<AR1>
 <AR2>-G0/0/1----------G0/0/1-<AR1>
   |                     |
 G0/0/2			       G0/0/2
   |                     |
  PC2			        PC1
```

```VRP
#-------------AR1--------------
<Huawei>sys
[Huawei]inte g0/0/2
[Huawei-GigabitEthernet0/0/2]ip add 1.0.0.1 24
[Huawei-GigabitEthernet0/0/2]inte g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 12.0.0.1 24
[Huawei-GigabitEthernet0/0/0]inte g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 34.0.0.1 24
[AR1]ip route-static 4.0.0.0 24 12.0.0.2
[AR1]ip route-static 4.0.0.0 24 34.0.0.2

#-------------AR2--------------
<Huawei>sys
[Huawei]inte g0/0/2
[Huawei-GigabitEthernet0/0/2]ip add 4.0.0.1 24
[Huawei-GigabitEthernet0/0/2]inte g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 12.0.0.2 24
[Huawei-GigabitEthernet0/0/0]inte g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 34.0.0.2 24
[AR2]ip route-static 1.0.0.0 24 12.0.0.1
[AR2]ip route-static 1.0.0.0 24 34.0.0.1
[AR2]dis ip routing-table
```

此时可以在两台路由器上观察到双方去往对方的两条等价路由；基于这个双链路拓扑还可以实现另一种路由策略：路由备份（浮动静态路由），利用优先级的特性配置浮动路由，在主路由失效的情况下，浮动路由会加入到路由表并承担业务数据转发

## **浮动路由**

```VRP
[AR1]ip route-static 4.0.0.0 24 12.0.0.2
[AR1]ip route-static 4.0.0.0 24 34.0.0.2 preference 100	#静态路由优先级默认60,将其修改更大即可作为备用链路
[AR1]dis ip routing-table protocol static		#修改优先级后，34.0.0.2这个下一跳的路由变更为不活越状态
```

静态路由的缺陷就是无法根据拓扑的变化进行动态的响应。所以写错路由或网络环境产生变动时，一定要实时更新静态路由，即便是写错的路由也会生效，必须要删除，一旦路由算法选中了错误路由，无法正常通信

## **缺省路由**

也叫默认路由，是一种目标网络和掩码都是全0的特殊路由，能匹配所有目标网络。**可以通过静态路由配置，也可以通过动态路由协议发布**，在路由表中，以目标网络是0.0.0.0（掩码为0.0.0.0）的形式出现，通常用于末梢网络，例如家庭上网、企业出口、边界设备，如果报文的目标地址无法匹配路由表中的任何一项，路由器将选择依照缺省路由来转发报文

*一个网关就代表着一条默认路由*

### **LoopBack**

环回口，逻辑的虚拟接口。环回口一般用于模拟直连网段，例如在模拟器中不是一定需要通过PC设备才能测试网络通信的，在路由器上创建一个环回口也等同于一个直连网段，实际用途非常广泛

```VRP
[AR1]inte loop 0	#创建环回口
[AR1-LoopBack0]ip add 2.2.2.2 24    #为环回口配置IP
[AR1-LoopBack0]ping -a 2.2.2.2 4.0.0.10		#指定通过环回IP测试通信
```

ping命令默认使用的SIP是出口接口上的IP，如果要其他直连网段的路由是否正常，需要指定SIP测试，指定环回口IP测试连通性明显会失败，因为没有路由。windows通过route命令可以查看配置静态路由的方式