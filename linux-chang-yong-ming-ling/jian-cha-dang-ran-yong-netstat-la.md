# ⚽ 检查当然用netstat啦

nmap扫描有时候是有问题的，去shell验证，或者看对外端口用它

命令简单，但是常用

我常用的，配合grep可以快速定位LISTEN

```
netstat -anltp ｜ grep LISTEN
netstat -nutlp  | grep LISTEN
...
```

分析的时候，像是绑定在0.0.0.0:5555(ipv4)或者::5555(ipv6)这种就是可以下手的，因为对外暴露捏

如果暴露了，一定要看下iptables有没有做策略,看是否drop掉，在iptables里讲

```
sudo iptables-save | grep 5555
```

如果没有就是一个攻击点～

```
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

* \-a或--all 显示所有连线中的Socket。
* \-A<网络类型>或--<网络类型> 列出该网络类型连线中的相关地址。
* \-c或--continuous 持续列出网络状态。
* \-C或--cache 显示路由器配置的快取信息。
* \-e或--extend 显示网络其他相关信息。
* \-F或--fib 显示路由缓存。
* \-g或--groups 显示多重广播功能群组组员名单。
* \-h或--help 在线帮助。
* \-i或--interfaces 显示网络界面信息表单。
* \-l或--listening 显示监控中的服务器的Socket。
* \-M或--masquerade 显示伪装的网络连线。
* \-n或--numeric 直接使用IP地址，而不通过域名服务器。
* \-N或--netlink或--symbolic 显示网络硬件外围设备的符号连接名称。
* \-o或--timers 显示计时器。
* \-p或--programs 显示正在使用Socket的程序识别码和程序名称。
* \-r或--route 显示Routing Table。
* \-s或--statistics 显示网络工作信息统计表。
* \-t或--tcp 显示TCP传输协议的连线状况。
* \-u或--udp 显示UDP传输协议的连线状况。
* \-v或--verbose 显示指令执行过程。
* \-V或--version 显示版本信息。
* \-w或--raw 显示RAW传输协议的连线状况。
* \-x或--unix 此参数的效果和指定"-A unix"参数相同。
* \--ip或--inet 此参数的效果和指定"-A inet"参数相同。







