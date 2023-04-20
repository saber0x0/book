---
description: author:huahua
---

# 😘 ip命令好用捏

不会有人还在ifconfig吧😊

## ip 命令

```sh
ip [ OPTIONS ] OBJECT { COMMAND | help }
```

常用对象的取值含义如下：

* link：网络设备
* address：设备上的协议（IP或IPv6）地址
* addrlabel：协议地址选择的标签配置
* route：路由表条目 (常用，作为路由走向配置)
* rule：路由策略数据库中的规则(查看table，这个表决定了route应该配在哪shell)

OPTIONS 为常用选项，值可以是以下几种：

```sh
OPTIONS={ -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] | -h[uman-readable] | -iec | -f[amily] { inet | inet6 | ipx | dnet | link } | -o[neline] | -t[imestamp] | -b[atch] [filename] | -rc[vbuf] [size] }
```

常用选项的取值含义如下：

* \-V：显示命令的版本信息；
* \-s：输出更详细的信息；
* \-f：强制使用指定的协议族；
* \-4：指定使用的网络层协议是IPv4协议；
* \-6：指定使用的网络层协议是IPv6协议；
* \-0：输出信息每条记录输出一行，即使内容较多也不换行显示；
* \-r：显示主机时，不使用IP地址，而使用主机的域名。
* help 为该命令的帮助信息。

### 例子🌰

{% code lineNumbers="true" %}
```
比如ap(192.168.2.1)做了wlan隔离，我连上热点后(分配地址192.168.2.99)
访问不了一个目标机(192.168.2.44)，怎么让他访问？

在ap上，或者过流量的设备上
ip route add 192.168.2.44 via 192.168.2.99 dev eth0
添加一条静态路由指定下一跳地址(via),dev指定网卡
删除的话add换成del


下边情况来自网络
ip link show                     # 显示网络接口信息
ip link set eth0 up              # 开启网卡（这不比ifconfig up香？）
ip link set eth0 down            # 关闭网卡
ip link set eth0 promisc on      # 开启网卡的混合模式
ip link set eth0 promisc offi    # 关闭网卡的混个模式
ip link set eth0 txqueuelen 1200 # 设置网卡队列长度
ip link set eth0 mtu 1400        # 设置网卡最大传输单元
ip addr show                     # 显示网卡IP信息（我喜欢ip add show！）
ip addr add 192.168.0.1/24 dev eth0 # 设置eth0网卡IP地址192.168.0.1
ip addr del 192.168.0.1/24 dev eth0 # 删除eth0网卡IP地址

ip route show                     # 显示系统路由
ip route add default via 192.168.1.254   # 设置系统默认路由
ip route list                 # 查看路由信息（核心路由表）
ip route add 192.168.4.0/24  via  192.168.0.254 dev eth0 # 设置192.168.4.0网段的网关为192.168.0.254,数据走eth0接口
ip route add default via  192.168.0.254  dev eth0        # 设置默认网关为192.168.0.254
ip route del 192.168.4.0/24   # 删除192.168.4.0网段的网关
ip route del default          # 删除默认路由
ip route delete 192.168.1.0/24 dev eth0 # 删除路由
ip link list                   #显示网络设备的运行状态
ip -s link list                #详细设备信息
ip neigh list                  #邻表
ip link | grep -E '^[0-9]' | awk -F: '{print $2}' #获取主机所有网络接口
```
{% endcode %}

这里要补充的是，有时候路由信息不在default表里，所以要看ip rule，看流量是怎么走的捏





