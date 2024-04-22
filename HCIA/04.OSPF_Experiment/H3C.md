# OSPF实验

![Comware_OSPF基本实验](https://www.z4a.net/images/2024/03/23/Comware_OSPF.png)

## 参考配置

```Comware
#----------------AR1---------------
<H3C>system-view
[H3C]sysname R1
[R1]interface g0/1
[R1-GigabitEthernet0/1]ip address 14.0.0.1 24
[R1-GigabitEthernet0/1]interface g0/0
[R1-GigabitEthernet0/0]ip address 12.0.0.1 24
[R1-GigabitEthernet0/0]interface loopback 0
[R1-LoopBack0]ip address 1.1.1.1 32
[R1-LoopBack0]ospf 1 router-id 1.1.1.1
[R1-ospf-1]area 0
[R1-ospf-1-area-0.0.0.0]network 1.1.1.1 0.0.0.0
[R1-ospf-1-area-0.0.0.0]network 12.0.0.1 0.0.0.0
[R1-ospf-1-area-0.0.0.0]area 1
[R1-ospf-1-area-0.0.0.1]network 14.0.0.1 0.0.0.0

#----------------AR2---------------
<H3C>system-view
[H3C]sysname R2
[R2]interface g0/0
[R2-GigabitEthernet0/0]ip address 12.0.0.2 24
[R2-GigabitEthernet0/0]interface g0/1
[R2-GigabitEthernet0/1]ip address 23.0.0.2 24
[R2-GigabitEthernet0/1]interface loopback 0
[R2-LoopBack0]ip address 2.2.2.2 32
[R2-LoopBack0]ospf 1 router-id 2.2.2.2
[R2-ospf-1]area 0
[R2-ospf-1-area-0.0.0.0]network 12.0.0.2 0.0.0.0
[R2-ospf-1-area-0.0.0.0]network 23.0.0.2 0.0.0.0
[R2-ospf-1-area-0.0.0.0]network 2.2.2.2 0.0.0.0

#----------------AR3---------------
<H3C>system-view
[H3C]sysname R3
[R3]interface g0/0
[R3-GigabitEthernet0/0]ip address 23.0.0.3 24
[R3-GigabitEthernet0/0]interface g0/1
[R3-GigabitEthernet0/1]ip address 35.0.0.3 24
[R3-GigabitEthernet0/1]interface loopback 0
[R3-LoopBack0]ip address 3.3.3.3 32
[R3-LoopBack0]ospf 1 router-id 3.3.3.3
[R3-ospf-1]area 0
[R3-ospf-1-area-0.0.0.0]network 23.0.0.3 0.0.0.0
[R3-ospf-1-area-0.0.0.0]network 3.3.3.3 0.0.0.0
[R3-ospf-1-area-0.0.0.0]area 2
[R3-ospf-1-area-0.0.0.2]network 35.0.0.3 0.0.0.0

#----------------AR4---------------
<H3C>system-view
[H3C]sysname R4
[R4]interface loopback 0
[R4-LoopBack0]ip address 4.4.4.4 32
[R4-LoopBack0]interface g0/0
[R4-GigabitEthernet0/0]ip address 14.0.0.4 24
[R4-GigabitEthernet0/0]ospf 1 router-id 4.4.4.4 
[R4-ospf-1]area 1
[R4-ospf-1-area-0.0.0.1]network 4.4.4.4 0.0.0.0
[R4-ospf-1-area-0.0.0.1]network 14.0.0.4 0.0.0.0

#----------------AR5---------------
<H3C>system-view
[H3C]sysname R5
[R5]interface loopback 0
[R5-LoopBack0]ip address 5.5.5.5 32
[R5-LoopBack0]interface g0/0
[R5-GigabitEthernet0/0]ip address 35.0.0.5 24
[R5-GigabitEthernet0/0]ospf 1 router-id 5.5.5.5
[R5-ospf-1]area 2
[R5-ospf-1-area-0.0.0.2]network 5.5.5.5 0.0.0.0
[R5-ospf-1-area-0.0.0.2]network 35.0.0.5 0.0.0.0
```