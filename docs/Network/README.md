+++
title = "Network"
+++

## ip

1. 显示网络接口信息：

`ip addr show`

2. 启用/禁用网络接口：

`ip link set <接口名称> up`  
`ip link set <接口名称> down`

3. 设置接口IP地址：

`ip addr add <IP地址>/<子网掩码位数> dev <接口名称>`

4. 删除接口IP地址：

`ip addr del <IP地址>/<子网掩码位数> dev <接口名称>`

5. 显示ARP缓存表：

`ip neigh show`

6. 添加ARP静态映射：

`ip neigh add <目标IP地址> lladdr <目标MAC地址> dev <接口名称>`

7. 删除ARP静态映射：

`ip neigh del <目标IP地址> dev <接口名称>`

8. 显示网络统计信息：

`ip -s link show <接口名称>`

9. 修改接口MTU值：

`ip link set <接口名称> mtu <MTU值>`

10. 开启IP转发：

`sysctl net.ipv4.ip_forward=1`

11. 显示IP转发状态

`sysctl net.ipv4.ip_forward`

## ip route

- dev 指定数据包将从哪个网络设备出发或进入。例如，dev eth0 表示数据包将通过网卡 eth0 进出。
- via 指定下一跳路由器的 IP 地址。例如，via 192.168.1.1 表示下一跳路由器的 IP 地址为 192.168.1.1。
- src 指定数据包的源 IP 地址。例如，src 192.168.1.100 表示数据包的源 IP 地址为 192.168.1.100。
- table 指定路由表的编号。默认情况下，Linux 系统使用主路由表，但也可以创建额外的路由表来管理网络流量。例如，table 10 表示将路由规则添加到编号为 10 的路由表中。
- metric 指定路由规则的优先级。当有多条路由规则匹配目标 IP 地址时，系统会根据每条规则的优先级进行选择。例如，metric 100 表示将该规则的优先级设置为 100。

### 显示路由表

`ip route show`

### 添加静态路由规则

`ip route add <目标网络> via <下一跳地址>`

### 更新静态路由规则

假设我们要替换默认路由（default route）的目标IP和网关。假设默认路由的目标IP为`0.0.0.0/0`，网关为`192.168.1.1`。我们可以使用`ip route replace`命令来进行替换。

```
ip route replace 0.0.0.0/0 via 192.168.1.1
```

上述命令将会替换默认路由的目标IP为`0.0.0.0/0`，并将网关设置为`192.168.1.1`。

### 删除静态路由

`ip route del <目标网络>`

### 删除路由表中所有路由规则

`ip route flush`

# iptables

1. 显示防火墙规则：

`iptables -L`

2. 清除所有防火墙规则：

`iptables -F`

3. 允许特定IP地址或IP范围的入站流量：

`iptables -A INPUT -s <IP地址/范围> -j ACCEPT`

4. 允许特定端口的入站流量：

`iptables -A INPUT -p <协议> --dport <端口号> -j ACCEPT`

5. 允许特定IP地址的出站流量：

`iptables -A OUTPUT -d <IP地址> -j ACCEPT`

6. 允许特定端口的出站流量：

`iptables -A OUTPUT -p <协议> --dport <端口号> -j ACCEPT`

7. 阻止特定IP地址或IP范围的入站流量：

`iptables -A INPUT -s <IP地址/范围> -j DROP`

8. 阻止特定端口的入站流量：

`iptables -A INPUT -p <协议> --dport <端口号> -j DROP`

9. 阻止特定IP地址的出站流量：

`iptables -A OUTPUT -d <IP地址> -j DROP`

10. 阻止特定端口的出站流量：

`iptables -A OUTPUT -p <协议> --dport <端口号> -j DROP`

11. 允许已建立连接和相关的入站流量：

`iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`

12. 允许已建立连接和相关的出站流量：

`iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`

13. 允许所有本地主机的回环流量：

`iptables -A INPUT -i lo -j ACCEPT`

14. 允许所有出站流量：

`iptables -A OUTPUT -j ACCEPT`

15. 阻止所有其它入站流量：

`iptables -A INPUT -j DROP`

16. 阻止所有其它出站流量：

`iptables -A OUTPUT -j DROP`

# tcpdump

[Tcpdump Examples - 22 Tactical Commands | HackerTarget.com](https://hackertarget.com/tcpdump-examples/)

1. 抓取指定网络接口的所有数据包：

`tcpdump -i <接口名称>`

2. 抓取指定源IP地址的数据包：

`tcpdump src <源IP地址>`

3. 抓取指定目标IP地址的数据包：

`tcpdump dst <目标IP地址>`

4. 抓取指定端口号的数据包：

`tcpdump port <端口号>`

5. 抓取指定源端口号的数据包：

`tcpdump src port <源端口号>`

6. 抓取指定目标端口号的数据包：

`tcpdump dst port <目标端口号>`

7. 抓取指定协议类型的数据包：

`tcpdump <协议类型>`

8. 抓取指定源和目标IP地址的数据包：

`tcpdump src <源IP地址> and dst <目标IP地址>`

9. 抓取指定源和目标端口号的数据包：

`tcpdump src port <源端口号> and dst port <目标端口号>`

10. 抓取指定源IP地址和端口号的数据包：

`tcpdump src <源IP地址> and src port <源端口号>`

11. 抓取指定目标IP地址和端口号的数据包：

`tcpdump dst <目标IP地址> and dst port <目标端口号>`

12. 显示数据包的详细信息（十六进制和ASCII）：

`tcpdump -XX`

13. 保存抓取的数据包到文件中：

`tcpdump -w <文件名>`
