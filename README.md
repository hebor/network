**能提高效率的windows工具**

```
# Windows cmd临时修改IP
netsh interface ip set address name="网卡名" static 192.168.42.211 25.255.255.0 192.168.42.1

netsh interface ip set address name="以太网 3" source=dhcp
netsh interface ip set dns name="以太网 3" static 114.114.114.114
netsh interface ip set dns name="以太网 3" source=dhcp
```

虽然记录更改DNS的方式，但如果真的需要同时修改IP和DNS，还是建议使用windows控制面板改，如果只是为了测试临时修改IP，CMD确实能够提高效率

**关于VirtualBox的虚拟网卡**

如果不是没得选，我绝对不会舍弃VMware用VirtualBox。VirtualBox的虚拟网卡经常性抽风，比如Windows的某个小更新网卡就没了，通过VirtualBox重新创建还会有各种问题，懂得都懂

```
cd D:\Oracle\VirtualBox\    #进入VirtualBox安装目录
.\vboxmanage list hostonlyifs    #列出所有的仅主机网卡
.\vboxmanage hostonlyif remove "VirtualBox Host-Only Ethernet Adapter #2"    #删除指定的仅主机网卡
```

这种方式并不适用任何情况，如果不想修改注册表，可以先尝试用这种方式解决