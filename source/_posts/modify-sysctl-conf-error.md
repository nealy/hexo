title: CentOS 6.x 下修改sysctl.conf报错问题解决
date: 2016-01-22 16:23:18
updated: 2016-01-22 16:23:18
tags: [linux]
categories: Others
---

sysctl.conf在涉及内核、VPN等配置时经常要修改，而在CentOS 6.x VPS主机下修改sysctl.conf经常出现错误导致更新失败。常见错误：    
```bash
error: "net.bridge.bridge-nf-call-ip6tables" is an unknown key
error: "net.bridge.bridge-nf-call-iptables" is an unknown key
error: "net.bridge.bridge-nf-call-arptables" is an unknown key
```
解决方案
<!-- more -->

#### Xen架构的VPS

```bash
modprobe bridge
lsmod|grep bridge
```

#### OpenVZ架构的VPS

OpenVZ架构下直接执行上述命令会抛出错误：    
`FATAL: Module bridge not found.`

这是因为 OpenVZ 架构的 VPS 无法加载内核模块，最好的解决办法当然是……注释掉 sysctl.conf 报错的设置项。    
这里提供另一种解决办法，让 modprobe 或者 sysctl 直接返回 true，当然，这并不是真正的修改 modprobe 或者 sysctl。

modprobe
```bash
rm -f /sbin/modprobe
ln -s /bin/true /sbin/modprobe
```

sysctl
```bash
rm -f /sbin/sysctl
ln -s /bin/true /sbin/sysctl
```
