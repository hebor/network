# OSPF综合实验

![OSPF综合实验](https://www.z4a.net/images/2023/10/16/OSPF.png)

## 参考配置

```VRP
#----------------AR1---------------
<Huawei>sys
[Huawei]sys AR1
[AR1]inte g0/0/0
[AR1-GigabitEthernet0/0/0]ip add 192.168.0.1 24
[AR1-GigabitEthernet0/0/0]ospf dr-priority 100
[AR1-GigabitEthernet0/0/0]ospf 1 router-id 1.1.1.1
[AR1-ospf-1]area 0
[AR1-ospf-1-area-0.0.0.0]net 192.168.0.0 0.0.0.255

#----------------AR2---------------
<Huawei>sys
[Huawei]sys AR2
[AR2]inte g0/0/0
[AR2-GigabitEthernet0/0/0]ip add 192.168.0.2 24
[AR2-GigabitEthernet0/0/0]ospf dr-priority 90
[AR2-GigabitEthernet0/0/0]ospf 1 router-id 2.2.2.2 
[AR2-ospf-1]area 0 
[AR2-ospf-1-area-0.0.0.0]net 192.168.0.0 0.0.0.255

#----------------AR3---------------
<Huawei>sys
[Huawei]sys AR3
[AR3]inte g0/0/0
[AR3-GigabitEthernet0/0/0]ip add 192.168.0.4 24
[AR3-GigabitEthernet0/0/0]ospf 1 router-id 3.3.3.3
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 192.168.0.0 0.0.0.255

#----------------AR4---------------
<Huawei>sys
[Huawei]sys AR4
[AR4]inte g0/0/0
[AR4-GigabitEthernet0/0/0]ip add 192.168.0.3 24
[AR4-GigabitEthernet0/0/0]inte s4/0/0
[AR4-Serial4/0/0]ip add 34.0.0.3 8
[AR4-Serial4/0/0]ospf authentication-mode md5 1 cipher Admin@huawei.com
[AR4-Serial4/0/0]ospf 1 router-id 4.4.4.4
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]net 192.168.0.0 0.0.0.255
[AR4-ospf-1-area-0.0.0.0]net 34.0.0.0 0.255.255.255

#----------------AR5---------------
<Huawei>sys
[Huawei]sys AR5
[AR5]inte s4/0/0
[AR5-Serial4/0/0]ip add 34.0.0.4 8
[AR5-Serial4/0/0]ospf authentication-mode md5 1 cipher Admin@huawei.com
[AR5-Serial4/0/0]inte g0/0/1
[AR5-GigabitEthernet0/0/1]ip add 48.0.0.1 8
[AR5-GigabitEthernet0/0/1]inte loop 0
[AR5-LoopBack0]ip add 4.4.4.4 32
[AR5-LoopBack0]ospf 1 router-id 5.5.5.5
[AR5-ospf-1]area 0
[AR5-ospf-1-area-0.0.0.0]net 34.0.0.0 0.255.255.255
[AR5-ospf-1-area-0.0.0.0]net 4.4.4.4 0.0.0.0

#----------------AR6---------------
<Huawei>sys
[Huawei]sys AR6
[AR6]inte g0/0/0
[AR6-GigabitEthernet0/0/0]ip add 48.0.0.2 8
[AR6-GigabitEthernet0/0/0]ip route-static 0.0.0.0 0 48.0.0.1	#为AR6指定默认路由，能通往Area0区域所有网络
```

目前的场景下Area 0区域下的多个路由器已经互相宣告路由，且由于手动配置的OSPF的DR优先级，现在AR1是DR、AR2是BDR。然而AR6目前只跟AR5互通，AR6虽然已经指定了默认路由能够通往整个Area 0区域内的任一路由器，但只有AR5设备上有48.0.0.0/8的路由，这条路由并没有在OSPF网络中宣告，所以现在有两种方式解决AR6的通信问题：AR5在OSPF网络中引入48.0.0.0/8的路由、动态发布静态路由。AR5从引入直连路由的时刻，它就成为了ASBR

```VRP
#----------------AR5---------------
[AR5]ospf 1
[AR5-ospf-1]import-route direct		#引入AR5的直连路由

#----------------AR5---------------
[AR5]ip route-static 0.0.0.0 0 48.0.0.2		#动态发布静态路由的前提是出口设备上本身要有这条静态路由
[AR5]ospf 1
[AR5-ospf-1]default-route-advertise		#通过OSPF动态路由协议发布静态路由
```

### 扩展概念

在真实网络环境下，AR6（运营商）是不可能有到Area 0区域的缺省路由的，AR5的出口一般是通过NAT访问外网

```VRP
#----------------AR6---------------
[AR6]undo ip route-static 0.0.0.0 0 48.0.0.1	#此时Area 0区域内的路由器已经不通48.0.0.0/8网段

#----------------AR5---------------
[AR5]acl 2000
[AR5-acl-basic-2000]rule 1 permit	#ACL不需要做任何限制，放行所有
[AR5-acl-basic-2000]inte g0/0/1
[AR5-GigabitEthernet0/0/1]nat outbound 2000		#AR5出接口设置nat规则
```



# 路由综合实验

![路由综合实验](https://www.z4a.net/images/2023/09/20/49cc64edb11d0f42b2ad362cf649e166.png)

## 参考配置

```VRP
#----------------AR2---------------
<Huawei>sys
[Huawei]sys AR2
[AR2]inte loop 1
[AR2-LoopBack1]ip add 2.2.2.2 32
[AR2-LoopBack1]inte g0/0/0
[AR2-GigabitEthernet0/0/0]ip add 10.1.1.2 30
[AR2-GigabitEthernet0/0/0]rip 1
[AR2-rip-1]version 2
[AR2-rip-1]net 2.0.0.0
[AR2-rip-1]net 10.0.0.0
[AR2-rip-1]ip route-static 0.0.0.0 0 10.1.1.1

#----------------AR1---------------
<Huawei>sys
[Huawei]sys AR1
[AR1]inte g0/0/0
[AR1-GigabitEthernet0/0/0]ip add 10.1.1.1 30
[AR1-GigabitEthernet0/0/0]inte g0/0/1
[AR1-GigabitEthernet0/0/1]ip add 192.168.255.2 30
[AR1-GigabitEthernet0/0/1]rip 1
[AR1-rip-1]version 2
[AR1-rip-1]net 10.0.0.0
[AR1-rip-1]ip route-static 0.0.0.0 0 192.168.255.1
[AR1]dis ip routing-table pro rip	#检验rip路由是否正常获取

#----------------AR4---------------
<Huawei>sys
[Huawei]sys AR4
[AR4]inte g0/0/0
[AR4-GigabitEthernet0/0/0]ip add 151.151.1.1 30
[AR4-GigabitEthernet0/0/0]inte g0/0/1
[AR4-GigabitEthernet0/0/1]ip add 131.131.255.13 30
[AR4-GigabitEthernet0/0/1]inte s4/0/0
[AR4-Serial4/0/0]ip add 131.131.255.6 30
[AR4-Serial4/0/0]ospf 1 router-id 4.4.4.4
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]net 131.131.255.12 0.0.0.3		#30位掩码的反掩码值=255-(128+64+32+16+8+4)=3
[AR4-ospf-1-area-0.0.0.0]net 131.131.255.4 0.0.0.3
[AR4-ospf-1-area-0.0.0.0]ip route-static 0.0.0.0 0 151.151.1.2
[AR4]ospf 1
[AR4-ospf-1]default-route-advertise

#----------------AR5---------------
<Huawei>sys
[Huawei]sys AR5
[AR5]inte g0/0/0
[AR5-GigabitEthernet0/0/0]ip add 131.131.255.14 30
[AR5-GigabitEthernet0/0/0]inte g0/0/1
[AR5-GigabitEthernet0/0/1]ip add 131.131.255.10 30
[AR5-GigabitEthernet0/0/1]ospf 1 router-id 5.5.5.5
[AR5-ospf-1]area 0
[AR5-ospf-1-area-0.0.0.0]net 131.131.255.12 0.0.0.3
[AR5-ospf-1-area-0.0.0.0]net 131.131.255.8 0.0.0.3

#----------------AR6---------------
<Huawei>sys
[Huawei]sys AR6
[AR6]inte g0/0/0
[AR6-GigabitEthernet0/0/0]ip add 131.131.255.9 30
[AR6-GigabitEthernet0/0/0]inte g0/0/1
[AR6-GigabitEthernet0/0/1]ip add 131.131.255.1 30
[AR6-GigabitEthernet0/0/1]inte s4/0/0
[AR6-Serial4/0/0]ip add 131.131.255.5 30
[AR6-Serial4/0/0]ospf 1 router-id 6.6.6.6
[AR6-ospf-1]area 0
[AR6-ospf-1-area-0.0.0.0]net 131.131.255.8 0.0.0.3
[AR6-ospf-1-area-0.0.0.0]net 131.131.255.4 0.0.0.3
[AR6-ospf-1-area-0.0.0.0]net 131.131.255.0 0.0.0.3

#----------------AR7---------------
<Huawei>sys
[Huawei]sys AR7
[AR7]inte g0/0/0
[AR7-GigabitEthernet0/0/0]ip add 131.131.255.2 30
[AR7-GigabitEthernet0/0/0]inte loop 1
[AR7-LoopBack1]ip add 7.7.7.7 32
[AR7-LoopBack1]ospf 1 router-id 7.7.7.7
[AR7-ospf-1]area 0
[AR7-ospf-1-area-0.0.0.0]net 131.131.255.0 0.0.0.3
[AR7-ospf-1-area-0.0.0.0]net 7.7.7.7 0.0.0.0

#----------------AR3---------------
<Huawei>sys
[Huawei]sys AR3
[AR3]inte g0/0/0
[AR3-GigabitEthernet0/0/0]ip add 192.168.255.1 30
[AR3-GigabitEthernet0/0/0]inte g0/0/1
[AR3-GigabitEthernet0/0/1]ip add 151.151.1.2 30
[AR3-GigabitEthernet0/0/1]ip route-static 2.2.2.2 32 192.168.255.2
[AR3]ip route-static 10.1.1.0 30 192.168.255.2		#包含有RIP区域的两个IP的网段
[AR3]ip route-static 7.7.7.7 32 151.151.1.1
[AR3]ip route-static 131.131.255.0 28 151.151.1.1	#OSPF区域的子网，汇总后写入静态路由
```

每完成一个区域的路由协议配置后，都应先检查该区域的路由协议是否正常运行，例如OSPF区域配置完成后，应该先检查区域内所有路由器的OSPF邻居表是否正常、OSPF路由获取是否正常、区域内通信是否正常。静态路由的编写应该尽可能精确，通过`tracert -a`能够检测从`7.7.7.7`到`2.2.2.2`的OSPF路由开销，模拟AR5与AR6之间的线路故障，能够感受到动态路由的自动收敛、自动切换其他路由，恢复AR5与AR6之间的线路，OSPF收敛稳定后又会选择开销最小的路由