# 静态路由实验

![静态路由实验](https://www.z4a.net/images/2024/03/18/b003790878bd2628e5f8237f796c48c0.png)

```Comware
#----------AR1---------
<H3C>sys
[H3C]sysn R1
[R1]inte g0/0
[R1-GigabitEthernet0/0]ip add 10.2.2.1 24
[R1-GigabitEthernet0/0]inte g0/1
[R1-GigabitEthernet0/1]ip add 10.1.1.1 24
[R1-GigabitEthernet0/1]inte g0/2
[R1-GigabitEthernet0/2]ip add 192.168.1.1 24
[R1-GigabitEthernet0/2]ip route-s 192.168.2.0 24 10.2.2.2
[R1]ip route-s 192.168.3.0 24 10.1.1.4

#----------AR2---------
<H3C>sys
[H3C]sysn R2
[R2]inte g0/0
[R2-GigabitEthernet0/0]ip add 10.2.2.2 24
[R2-GigabitEthernet0/0]inte g0/1
[R2-GigabitEthernet0/1]ip add 10.3.3.2 24
[R2-GigabitEthernet0/1]ip route-s 192.168.1.0 24 10.2.2.1
[R2]ip route-s 192.168.2.0 24 10.3.3.3

#----------AR3---------
<H3C>sys
[H3C]sysn R3
[R3]inte g0/0
[R3-GigabitEthernet0/0]ip add 10.3.3.3 24
[R3-GigabitEthernet0/0]inte g0/1
[R3-GigabitEthernet0/1]ip add 10.4.4.3 24
[R3-GigabitEthernet0/1]inte g0/2
[R3-GigabitEthernet0/2]ip add 192.168.2.1 24
[R3-GigabitEthernet0/2]ip route-s 192.168.1.0 24 10.4.4.4
[R3]ip route-s 192.168.3.0 24 10.4.4.4

#----------AR4---------
<H3C>sys
[H3C]sysn R4
[R4]inte g0/0
[R4-GigabitEthernet0/0]ip add 10.1.1.4 24
[R4-GigabitEthernet0/0]inte g0/1
[R4-GigabitEthernet0/1]ip add 10.4.4.4 24
[R4-GigabitEthernet0/1]inte g0/2
[R4-GigabitEthernet0/2]ip add 10.5.5.4 24
[R4-GigabitEthernet0/2]inte g5/0
[R4-GigabitEthernet5/0]ip add 10.6.6.4 24
[R4-GigabitEthernet5/0]ip route-s 192.168.1.0 24 10.1.1.1
[R4]ip route-s 192.168.2.0 24 10.4.4.3
[R4]ip route-s 192.168.3.0 24 10.5.5.5
[R4]ip route-s 192.168.3.0 24 10.6.6.5

#----------AR5---------
<H3C>sys
[H3C]sysn R5
[R5]inte g0/0
[R5-GigabitEthernet0/0]ip add 10.5.5.5 24
[R5-GigabitEthernet0/0]inte g0/1
[R5-GigabitEthernet0/1]ip add 10.6.6.5 24
[R5-GigabitEthernet0/1]inte g0/2
[R5-GigabitEthernet0/2]ip add 192.168.3.1 24
[R5-GigabitEthernet0/2]ip route-s 0.0.0.0 0 10.5.5.4
[R5]ip route-s 0.0.0.0 0 10.6.6.4
```

# RIP实验

![RIP基本实验](https://www.z4a.net/images/2024/03/21/RIP.png)

```Comware
#----------AR1---------
<H3C>system-view
[H3C]sysname R1
[R1]interface loopback 1
[R1-LoopBack1]ip address 172.16.1.1 32
[R1-LoopBack1]interface loopback 2
[R1-LoopBack2]ip address 172.16.2.1 32
[R1-LoopBack2]interface g0/0
[R1-GigabitEthernet0/0]ip address 200.1.1.1 24
[R1-GigabitEthernet0/0]rip 1
[R1-rip-1]network 200.1.1.0
[R1-rip-1]network 172.16.0.0
[R1-rip-1]silent-interface LoopBack 1
[R1-rip-1]silent-interface LoopBack 2

#----------AR2---------
<H3C>system-view
[H3C]sysname R2
[R2]interface g0/0
[R2-GigabitEthernet0/0]ip address 200.1.1.2 24
[R2-GigabitEthernet0/0]interface g0/1
[R2-GigabitEthernet0/1]ip address 200.2.2.2 24
[R2-GigabitEthernet0/1]rip 1 
[R2-rip-1]network 200.1.1.0
[R2-rip-1]network 200.2.2.0

#----------AR3---------
<H3C>system-view
[H3C]sysname R3
[R3]interface loopback 3
[R3-LoopBack3]ip address 172.16.3.1 32
[R3-LoopBack3]interface loopback 4
[R3-LoopBack4]ip address 172.16.4.1 32
[R3-LoopBack4]interface g0/1
[R3-GigabitEthernet0/1]ip address 200.2.2.3 24
[R3-GigabitEthernet0/1]rip 1
[R3-rip-1]network 200.2.2.0
[R3-rip-1]network 172.16.0.0
[R3-rip-1]silent-interface loopback 3
[R3-rip-1]silent-interface loopback 4
```

此时查看R2路由表，去往172.16.0.0/16的路由被认为是等价路由，但实际上R1和R3的环回口属于不同的网段，此时如果R1或R3访问对方的172.16.0.0/16，那就只能看“运气”通不通，等价路由默认是负载均衡工作模式

```Comware
[R1-rip-1]version 2
[R1-rip-1]undo summary    #关闭自动聚合，否则仍会以172.16.0.0/16向邻居发送路由
```

在3个路由器节点上执行此命令，则3个节点都会收到环回口的明细路由，若不在R2上执行`undo summary`，则R1和R3的路由表中去往对方环回口的路由是`172.16.0.0/16`，R2路由表则具备去往R1和R3的环回口的明细路由