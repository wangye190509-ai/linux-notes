



# Linux 防火墙与selinux

##  防火墙相关概念

主要作用：保护内部的网络、主机、服务的安全等

我们常听到的防火墙有很多不同的类型，从大的角度来看，主要是分为硬件防火墙和软件防火墙。

##  RHEL/Centos系统

对于RHEL、Centos等红帽系Linux操作系统，我们常用到的防火墙软件主要有**iptables和firewalld**

- iptables:
  - iptables 是 Linux 默认的防火墙管理工具,它是基于命令行的。
  - iptables 使用规则表(tables)和规则链(chains)的方式来控制网络数据包的流向和处理。
  - iptables 提供了强大的灵活性和细粒度的控制,可以根据数据包的各种属性进行复杂的过滤和转发操作。
  - iptables 的配置相对比较复杂,需要对防火墙的工作原理有一定的了解。
- firewalld:
  - firewalld 是 RHEL/CentOS 7 及以后版本中引入的动态防火墙管理工具。
  - firewalld 采用区域(zones)和服务(services)的概念来管理防火墙规则,相比 iptables 更加简单和易用。
  - firewalld 支持动态更新防火墙规则,无需重启服务即可生效,这对于需要经常调整防火墙的场景很有帮助。
  - firewalld 提供了图形化的管理界面,方便管理员进行配置和管理。

**不过，对于Linux而言，其实Iptables和firewalld服务不是真正的防火墙，只是用来定义防火墙规则功能的"防火墙管理工具"，将定义好的规则交由内核中的netfilter即网络过滤器来读取，从而真正实现防火墙功能。**



# iptables

## 五链

iptables命令中设置数据过滤或处理数据包的策略叫做规则，将多个规则合成一个链，叫规则链。 规则链则依据处理数据包的位置不同分类：

- PREROUTING

  - 在进行路由判断之前所要进行的规则(DNAT/REDIRECT)

- INPUT

  - 处理入站的数据包

- FORWARD

  - 处理转发的数据包

- OUTPUT

  - 处理出站的数据包

- POSTROUTING

  - 在进行路由判断之后所要进行的规则(SNAT/MASQUERADE)

  ```tex
                      ┌─────────────────────────────────┐
                      │         网络数据包进入             │
                      └─────────────────┬───────────────┘
                                        ↓
                      ┌─────────────────────────────────┐
                      │    PREROUTING 链 (路由前)         │
                      │    raw → mangle → nat           │
                      └─────────────────┬───────────────┘
                                        ↓
                              ┌─────────┴─────────┐
                              │   路由判断          │
                              │ 目标是本机？         │
                              └─────────┬─────────┘
                                        │
          ┌─────────────────────────────┼─────────────────────────────┐
          │ (是)                        │ (否)                         │ (本机发出)
          ↓                             ↓                             ↓
  ┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
  │   INPUT 链        │     │   FORWARD 链      │     │   OUTPUT 链       │
  │   mangle → filter │     │   mangle → filter │     │   raw → mangle    │
  │                   │     │                   │     │   → nat → filter  │
  └─────────┬─────────┘     └─────────┬─────────┘     └─────────┬─────────┘
            ↓                         ↓                         ↓
  ┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
  │   本地应用程序      │     │   数据包转发         │     │   数据包发出        │
  │   (处理数据)       │     │   (继续路由)         │     │   (准备发出)       │
  └───────────────────┘     └─────────┬─────────┘     └─────────┬─────────┘
                                      ↓                         ↓
                      ┌─────────────────────────────────────────┐
                      │           POSTROUTING 链 (路由后)         │
                      │           mangle → nat                  │
                      └─────────────────────────────────────────┘
                                        ↓
                      ┌─────────────────────────────────────────┐
                      │              网络数据包发出                │
                      └─────────────────────────────────────────┘          
  ```

## 四表

iptables中的规则表是用于容纳规则链，规则表默认是允许状态的，那么规则链就是设置被禁止的规则，而反之如果规则表是禁止状态的，那么规则链就是设置被允许的规则。

- raw表

  - 确定是否对该数据包进行状态跟踪

- mangle表

  - 为数据包设置标记（较少使用）

- nat表

  - 修改数据包中的源、目标IP地址或端口

- filter表

  - 确定是否放行该数据包（过滤）

  

##  iptables规则说明

规则是**从上到下**匹配的，一旦匹配成功，就不再继续向下匹配

- 规则：根据指定的匹配条件来尝试匹配每个经流“关卡”的报文，一旦匹配成功，则由规则后面指定的处理动作进行处理

- 匹配条件

  - 基本匹配条件：sip、dip

    -s（source）和 -d（destination）

  - 扩展匹配条件：sport（Source Port源端口）、dport（Destination Port目标端口）

    --sport 和 --dport

  - 扩展条件也是条件的一部分，只不过使用的时候需要用`-m`参数声明对应的模块

- 处理动作 ( - j )

  - accept：接受
  - drop：丢弃
  - reject：拒绝
  - snat：源地址转换，解决内网用户同一个公网地址上上网的问题
  - masquerade：是snat的一种特殊形式，使用动态的、临时会变的ip上
  - dnat：目标地址转换
  - redirect：在本机作端口映射
  - log：记录日志，/var/log/messages文件记录日志信息，然后将数据包传递给下一条规则

##  iptables高级用法

![image-20260323154422433](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260323154422433.png)

- ```
  -t table
  ```

  指定表

  - raw
  - mangle
  - nat
  - filter 默认

- `-A`：添加规则，后面更上你要添加到哪个链上

- ```
  -j
  ```

  ：设置处理策略

  - accept：接受
  - drop：丢弃
  - reject：拒绝

- `-s`：指定数据包的来源地址

- ```
  -p
  ```

  ：指定数据包协议类型

  - tcp
  - udp
  - icmp
  - ......

### 增加规则

**案例一：**屏蔽来自其他主机上的ping包

```bash
# 正常通过cmd去ping我们的虚拟机的IP地址
C:\Users\Atopos>ping 192.168.179.129

正在 Ping 192.168.88.136 具有 32 字节的数据:
来自 192.168.88.136 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.88.136 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.88.136 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.88.136 的回复: 字节=32 时间<1ms TTL=64

192.168.88.136 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 0ms，最长 = 0ms，平均 = 0ms


# 在虚拟机上屏蔽来自我们物理机的ping包，注意物理机的IP的地址为：192.168.xxx.1
# ping命令是基于icmp协议的，所以这里对类型为icmp协议的数据包进行丢弃
[root@localhost ~]# iptables -A INPUT -s 192.168.179.1 -p icmp -j DROP

# 丢弃以后ping测试：
C:\Users\Atopos>ping 192.168.179.129

正在 Ping 192.168.179.129 具有 32 字节的数据:
请求超时。

# 注意，DROP和REJECT有一定的区别，DROP为丢弃数据包，不给予对方回应，所以对方就会一直等待。REJECT是拒绝对方，会将拒绝的数据包发给对方，对方就会知道我们拒绝的访问。
# 如果改成REJECT现象是：
C:\Users\Atopos>ping 192.168.179.129

正在 Ping 192.168.179.129 具有 32 字节的数据:
来自 192.168.179.129 的回复: 无法连到端口。
来自 192.168.179.129 的回复: 无法连到端口。
来自 192.168.179.129 的回复: 无法连到端口。
来自 192.168.179.129 的回复: 无法连到端口。

192.168.179.129 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，


`其他用法`
[root@localhost ~]# iptables -A INPUT -s 192.168.88.100  -j ACCEPT
# 目标来自 192.168.88.100 这个 IP 的封包都予以接受
[root@localhost ~]# iptables -A INPUT -s 192.168.88.0/24  -j ACCEPT
[root@localhost ~]# iptables -A INPUT -s 192.168.88.10  -j DROP
# 192.168.88.0/24 可接受，但 192.168.88.10 丢弃
```

**案例二：**屏蔽Linux系统中的远程链接服务（sshd），使得我们无法通过MobaXterm连接到虚拟机上

```bash
[root@localhost ~]# iptables -A INPUT -p tcp --dport 22 -j REJECT
# ssh服务所用的协议类型为tcp协议，端口号为22
# 所以我们将来自22端口上的tcp协议数据包给拒绝
```

当敲完这个命令以后，xshell远程连接就会中断了，可以在VMware中的虚拟机终端上删除这个规则，就恢复了

删除命令：`iptables -D INPUT -p tcp --dport 22 -j REJECT`

###  查看规则

```bash
[root@localhost ~]# iptables [-t tables] [-L] [-nv]
参数：
-t    后面接table,例如nat或filter，如果省略，默认显示filter
-L    列出目前的table的规则
-n    不进行IP与主机名的反查，显示信息的速度会快很多
-v    列出更多的信息，包括封包数，相关网络接口等
--line-numbers 显示规则的序号
[root@server1 ~]# iptables -nvL INPUT --line-numbers
Chain INPUT (policy ACCEPT 1306 packets, 89617 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        6   400 ACCEPT     6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80
2        4   240 DROP       1    --  *      *       192.168.179.1        0.0.0.0/0           
```

- 查看具体每个表的规则
  - policy：当前链的默认策略，当所有规则都没有匹配成功时执行的策略
  - packets：当前链默认策略匹配到的包的数量
  - bytes：当前链默认策略匹配到的包的大小
  - pkts：对应规则匹配到的包数量
  - bytes：对应规则匹配到的包大小
  - target：对应规则执行的动作
  - prot：对应的协议，是否针对某些协议应用此规则
  - opt：规则对应的选项
  - in：数据包由哪个接口流入
  - out：数据包由哪个接口流出
  - source：源ip地址
  - destination：目的ip地址

```bash
[root@localhost ~]# iptables -vnL INPUT -t filter --line-numbers
[root@localhost ~]# iptables -nL -t nat
[root@localhost ~]# iptables -nL -t mangle
[root@localhost ~]# iptables -nL -t raw
```

###  删除规则

- 通过num序号进行删除

```bash
删除input链上num为1的规则
[root@localhost ~]# iptables -D INPUT 1
```

- 通过规则匹配删除

```bash
[root@localhost ~]# iptables -D INPUT -p tcp --dport 80 -j DROP
```

- 清空所有的规则

```bash
[root@localhost ~]# iptables -F
```

###  修改规则

方案一：通过`iptables -D`删除原有的规则后添加新的规则

方案二：通过`iptables -R`可以对具体某一个num的规则进行修改

```bash
# 修改端口的为8080
[root@localhost ~]# iptables -R INPUT 1 -p tcp --dport 8080 -j ACCEPT

# 针对具体的某一个表和链进行修改
# 将8080修改回80
[root@localhost ~]# iptables -R INPUT 1 -p tcp --dport 80 -j ACCEPT
```

### 自定义链

iptables中除了系统自带的五个链之外，还可以自定义链，来实现将规则进行分组，重复调用的目的



**案例一**：我们自定义一个WEB_CHAIN链，专门管理跟web网站相关的规则策略。

```bash
# 1. 先查看我们现有的系统链
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

# 2. 添加WEB_CHAIN自定义链，并且查看
[root@localhost ~]# iptables -t filter -N web_chain
[root@localhost ~]# iptables -L
..........
Chain web_chain (0 references)
target     prot opt source               destination

# 3. 可以通过-E参数修改自定义链
[root@localhost ~]# iptables -t filter -E web_chain WEB_CHAIN
[root@localhost ~]# iptables -L
.........
Chain WEB_CHAIN (0 references)
target     prot opt source               destination
# 可以看到名字已经发生变化了

# 4. 向我们创建的自定义链中添加规则，开放80和443端口上的服务（即http和https）
[root@localhost ~]# iptables -t filter -A WEB_CHAIN -p tcp -m multiport --dports 80,443 -j ACCEPT

# 5. 将自定义链关联到系统链上才能使用
# 因为数据包只会经过上面讲过的五个系统链，不会经过我们的自定义链，所以需要把自定义链关联到某个系统链上

# 我们允许来自IP：192.168.179.1的访问，随后拒绝其他所有的访问，这样，只有192.168.179.1的主机可以访问这个网站
[root@localhost ~]# iptables -t filter -A INPUT -s 192.168.129.1 -j WEB_CHAIN
[root@localhost ~]# iptables -t filter -A INPUT -p tcp -m multiport --dports 80,443 -j DROP
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
WEB_CHAIN  all  --  192.168.179.1        anywhere            
REJECT     tcp  --  anywhere             anywhere             tcp dpt:http reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain WEB_CHAIN (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             multiport dports http,https
```

访问测试：

```shell
# windows本身可以访问我们的网站，但是其他主机无法访问
[root@server2 ~]# curl -I 192.168.179.129
curl: (7) Failed to connect to 192.168.179.129 port 80: Connection refused
```

删除自定义链

```shell
# 先清空自定义链上的规则
[root@server1 ~]# iptables -t filter -F WEB_CHAIN 
# 再删除挂在INPUT上的链
[root@server1 ~]# iptables -nvL INPUT --line-numbers
Chain INPUT (policy ACCEPT 3683 packets, 289K bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1      870 60686 WEB_CHAIN  0    --  *      *       192.168.179.1        0.0.0.0/0           
2        1    60 REJECT     6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 reject-with icmp-port-unreachable
[root@server1 ~]# iptables -D INPUT 1
# 然后通过-X选项删除自定义链
[root@server1 ~]# iptables -t filter -X WEB_CHAIN
```

### 其他用法（模块）

- tcp/udp
  - --dport：指定目的端口
  - --sport：指定源端口
- iprange：匹配报文的源/目的地址所在范围
  - --src-range
  - --dst-range

```bash
[root@localhost ~]# iptables -A INPUT -d 172.16.1.100 -p tcp --dport 80 -m iprange --src-range 172.16.1.5-172.16.1.10 -j DROP
```

- string：指定匹配报文中的字符串
  - --algo：指定匹配算法，可以是bm/kmp
    - bm：Boyer-Moore
    - kmp：Knuth-Pratt-Morris
  - --string：指定需要匹配的字符串
  - --from offset：开始偏移
  - --to offset：结束偏移

```bash
[root@localhost ~]# iptables -A OUTPUT -p tcp --sport 80 -m string --algo bm --from 62  --string "google" -j REJECT
```

#### string模块

**案例：**

server1上：

```shell
[root@server1 ~]# yum install -y httpd
[root@server1 ~]# systemctl stop nginx
[root@server1 ~]# systemctl start httpd
[root@server1 ~]# systemctl enable httpd
[root@localhost ~]# echo "<h1>hello world</h1>" > /var/www/html/index.html
[root@server1 ~]# echo 'hello world' > /var/www/html/index.html
[root@server1 ~]# iptables -A OUTPUT -p tcp --sport 80 -m string --algo bm --string "world" -j REJECT
[root@server1 ~]# iptables -nvL
Chain INPUT (policy ACCEPT 6944 packets, 2635K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 29 packets, 4838 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    9  2790 REJECT     6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp spt:80 STRING match  "world" ALGO name bm reject-with icmp-port-unreachable
    
[root@server1 ~]# echo "hello linux" > /var/www/html/index.html
# 重新输入覆盖index.html
```

server2上：

```shell
[root@server2 ~]# curl  192.168.179.129
hello world
[root@server2 ~]# curl  192.168.179.129
^C
# 一开始能连接上，添加规则后连接失败
[root@server2 ~]# curl  192.168.179.129
hello linux
# 修改后没有“world”，可以连接上
```

#### time模块

```shell
[root@server1 ~]# iptables -A INPUT -s 192.168.0.0/16 -d 192.168.129.1 -p tcp --dport 80 -m time --timestart 22:00 --timestop 23:59 --weekdays Mon,Tue -j DROP

# iptables -A INPUT \
#    -s 192.168.0.0/16 \              # 源IP：整个192.168网段
#    -d 192.168.129.1 \               # 目标IP：特定的服务器
#    -p tcp \                          # 协议：TCP
#   --dport 80 \                      # 目标端口：80（HTTP）
#    -m time \                         # 使用时间模块
#    --timestart 22:00 \               # 开始时间：晚上10点
#    --timestop 23:59 \                # 结束时间：午夜23:59
#    --weekdays Mon,Tue \              # 星期：周一和周二
#    -j DROP                           # 动作：丢弃
```

- 在**周一和周二**的 **22:00 到 23:59**（晚上10点到午夜23:59点）
- 禁止 **192.168.0.0/16 网段** 的所有主机
- 访问 **192.168.129.1** 这台服务器的 **Web 服务（80 端口）**

**通俗理解**：周一、周二晚上10点到23:59点，禁止内网用户访问特定服务器上的网站。

#### connlimit模块

**防止 SSH 暴力破解**的规则，限制同一 IP 的连接数

```shell
[root@server1 ~]# iptables -A INPUT -p tcp -d 192.168.179.1 --dport 22 -m connlimit --connlimit-above 2 -j REJECT
```

- 限制访问 **192.168.179.1:22**（SSH 服务）的连接数
- 当**同一个源 IP** 的连接数 **超过 2 个**时
- 新连接被**拒绝**

**通俗理解**：同一台机器最多只能同时建立 2 个 SSH 连接，超过就会被拒绝。

#### limit模块

- limit：对报文到达的速率继续限制，限制包数量
  - 10/second
  - 10/minute
  - 10/hour
  - 10/day
  - --limit-burst：空闲时可放行的包数量，默认为5，前多少个包不限制

```shell
# --limit-burst 10：设置允许的初始突发量为10个数据包。--limit 20/minute：设置平均速率为20个数据包/分钟，即每分钟允许通过的ICMP数据包数量为20个。
[root@server1 ~]# iptables -A INPUT -p icmp -m limit --limit-burst 10 --limit 20/min -j ACCEPT
[root@server1 ~]# iptables -A INPUT -p icmp -j REJECT 
```

**这两个规则的整体效果**：ICMP 包被限速，每分钟最多 20 个（前10个可以立即发送），超出部分被拒绝 

###  规则的保存与恢复

```shell
# 保存到文件中
[root@server1 ~]# iptables-save > /etc/sysconfig/iptables-config
# 从文件中导入
[root@server1 ~]# iptables-restore < /etc/sysconfig/iptables-config
```

- 安装`iptables-services`，在centos7和centos8上，通过该服务来帮助我们自动管理iptables规则。

```bash
[root@server1 ~]# yum install -y iptables-services
[root@server1 ~]# systemctl start iptables.service 
[root@server1 ~]# systemctl enable iptables.service 

```

##  NAT（地址转换协议）

### SNAT（源地址转换） 

#### 案例一：内网数据代理及服务映射

背景：现在有两个服务器server1和server2，server1是可以上网的，server2是不可以上网的，我们需要在server1上配置NAT来让server2通过server1去访问外网，同样我们也可以通过server1来访问位于server2服务器上的内部网站

**环境准备：** 两台虚拟机，可以克隆一个出来，然后server1添加一张仅主机的网卡。为了模拟server2不能上网的情况，将server2上原有的网卡改成仅主机模式。

```shell
# server2上先联网安装httpd
[root@server2 ~]# yum install -y httpd
[root@server2 ~]# systemctl stop firewalld
[root@server2 ~]# systemctl start httpd
# 安装完成后server2设置成仅主机模式
```

在server1的Linux内核中开启数据转发功能

```bash
# 开启server1数据转发
[root@server1 ~]# echo 1 > /proc/sys/net/ipv4/ip_forward
# 查看
[root@server1 ~]# cat /proc/sys/net/ipv4/ip_forward
1
```

查看server1的网络配置

```shell
[root@server1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:e8:c6:66 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.179.129/24 brd 192.168.179.255 scope global dynamic noprefixroute ens160
       valid_lft 1772sec preferred_lft 1772sec
    inet6 fe80::20c:29ff:fee8:c666/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
# ens160 为NAT网卡可以上网
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:e8:c6:70 brd ff:ff:ff:ff:ff:ff
    altname enp19s0
    inet 192.168.64.128/24 brd 192.168.64.255 scope global dynamic noprefixroute ens224
       valid_lft 1772sec preferred_lft 1772sec
    inet6 fe80::2f04:6fd0:bc5e:c/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
# ens224 为仅主机模式网卡，只能本地使用
```

server1上配置SNAT让server2能够网上冲浪

```shell
# 将来自192.168.64.0/24网段的ip来源转换为server1上的ens160上网ip地址192.168.179.129
[root@server1 ~]# iptables -t nat -A POSTROUTING -s 192.168.64.0/24 -j SNAT --to-source 192.168.179.129
```

配置网络让server2的网关指向server1的本地网卡ens224

```shell
[root@server2 ~]# nmcli connection modify ens160 ipv4.addresses 192.168.64.129 ipv4.gateway 192.168.64.128 ipv4.dns 114.114.114.114 ipv4.method manual
# 重启让配置生效
[root@server2 ~]# nmcli connection down ens160
[root@server2 ~]# nmcli connection up ens160
# 此时在server2上也可以访问www.baidu.com了
```

### DNAT(目标地址转换)

####  配置DNAT使得可以访问server2中的内部网站

```shell
# 在server1上添加规则
[root@server1 ~]# iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.64.129：80
# 在PREROUTING链上将来自8080端口上的访问转发到内网中的192.168.64.129上的80端口
```

####  案例二：转发本地端口

```shell
[root@server1 ~]# iptables -t nat -A PREROUTING -p tcp --dport 666 -j REDIRECT --to-port 22
# 此时连接666端口，iptables会帮我们转发到22端口
```

##  Firewalld

- firewalld中常用的区域名称及策略规则

| 区域     | 默认策略规则                                                 |
| -------- | ------------------------------------------------------------ |
| public   | 默认区域,对应普通网络环境。允许选定的连接,拒绝其他连接       |
| drop     | 所有传入的网络包都会被直接丢弃,不会发送任何响应              |
| block    | 所有传入的网络包会被拒绝,并发送拒绝信息                      |
| external | 用于路由/NAT流量的外部网络环境。与public类似,更适用于网关、路由器等处理外部网络流量的设备 |
| dmz      | 隔离区域,只允许选定的连接，适用于部署公开服务的网络区域,如 Web 服务器，可以最大限度地降低内部网络的风险 |
| work     | 适用于工作环境，开放更多服务,如远程桌面、文件共享等。比 public 区域更加信任 |
| home     | 适用于家庭环境，开放更多服务，比如默认情况下会开放一些如:3074端口(Xbox)、媒体、游戏数据等待 |
| trusted  | 信任区域,允许所有连接                                        |

### firewall-cmd 管理工具

| 参数                          | 作用                                                 |
| ----------------------------- | ---------------------------------------------------- |
| --get-default-zone            | 查询默认的区域名称                                   |
| --set-default-zone=<区域名称> | 设置默认的区域，使其永久生效                         |
| --get-zones                   | 显示可用的区域                                       |
| --get-services                | 显示预先定义的服务                                   |
| --get-active-zones            | 显示当前正在使用的区域与网卡名称                     |
| --add-source=                 | 将源自此IP或子网的流量导向指定的区域                 |
| --remove-source=              | 不再将源自此IP或子网的流量导向某个指定区域           |
| --add-interface=<网卡名称>    | 将源自该网卡的所有流量都导向某个指定区域             |
| --change-interface=<网卡名称> | 将某个网卡与区域进行关联                             |
| --list-all                    | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
| --list-all-zones              | 显示所有区域的网卡配置参数、资源、端口以及服务等信息 |
| --add-service=<服务名>        | 设置默认区域允许该服务的流量                         |
| --add-port=<端口号/协议>      | 设置默认区域允许该端口的流量                         |
| --remove-service=<服务名>     | 设置默认区域不再允许该服务的流量                     |
| --remove-port=<端口号/协议>   | 设置默认区域不再允许该端口的流量                     |
| --reload                      | 让“永久生效”的配置规则立即生效，并覆盖当前的配置规则 |
| --panic-on                    | 开启应急状况模式                                     |
| --panic-off                   | 关闭应急状况模式                                     |
| --permanent                   | 设定的当前规则保存到本地，下次重启生效               |

####  **简单用法**

```bash
# 查看firewalld是否启用，可以看到当前是出于actice的状态
[root@localhost ~]# systemctl status firewalld
# 查看当前所在的区域
[root@localhost ~]# firewall-cmd --get-default-zone
public
# 查看当前网卡所在的区域
[root@localhost ~]# firewall-cmd --get-zone-of-interface=ens160
public
# 查询public区域是否允许请求SSH或者HTTP协议的流量
[root@localhost ~]# firewall-cmd --zone=public --query-service=ssh
yes
[root@localhost ~]# firewall-cmd --zone=public --query-service=http
no
```

#### **修改firewalld策略放行**

```bash
- 方法一：针对服务协议类型进行放行
# 临时放行
[root@server1 ~]# firewall-cmd --zone=public --add-service=http
# 可以加上--permanent实现永久放行
[root@server1 ~]# firewall-cmd --permanent --zone=public --add-service=http

- 方法二：针对服务具体访问请求的端口号进行放行
[root@server1 ~]# firewall-cmd --zone=public --add-port=80/tcp
success
# 同样也可以加上--permanent实现永久放行

[root@server1 ~]# firewall-cmd --reload
success
# 加上--permanent的需要重载才能生效
```

#### **取消放行**

```bash
[root@server1 ~]# firewall-cmd --zone=public --remove-service=http --permanent
success
[root@server1 ~]# firewall-cmd --reload
success
# 修改后重新加载一下使得策略生效
```

#### **查看放行的服务**

```shell
[root@server1 ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

#### **修改firewall服务的默认区域设置**

```shell
[root@server1 ~]# firewall-cmd --set-default-zone=public
Warning: ZONE_ALREADY_SET: public
success
[root@server1 ~]# firewall-cmd --get-default-zone
public
```

#### **firewalld 切换到紧急模式(panic mode)**

#### 这种模式下,firewalld 会采取一种非常严格的防护策略:

1. 所有传入的网络连接都会被拒绝。
2. 所有区域的防火墙规则都会被重置为默认的严格策略。
3. 所有区域的默认策略都会被设置为 DROP

```shell
[root@server1 ~]# firewall-cmd --panic-on
success
[root@server1 ~]# firewall-cmd --panic-off
success
```

#### 在firewalld服务中访问8080和8081端口的流量策略设置为允许，当前生效

```bash
# 可以指定要开放端口的范围
[root@server1 ~]# firewall-cmd --zone=public --add-port=8080-8081/tcp
success
[root@server1 ~]# firewall-cmd --zone=public --list-ports
8080-8081/tcp
```

### 端口转发

```shell
[root@server1 ~]# firewall-cmd --permanent --add-forward-port=port=6666:proto=tcp:toport=22
success
[root@server1 ~]# firewall-cmd --reload
success
[root@server1 ~]# 
logout
Connection closing...Socket close.

Connection closed by foreign host.

Disconnected from remote host(192.168.179.129:22) at 14:14:25.

Type `help' to learn how to use Xshell prompt.
[G:\~]$ ssh root@192.168.179.129 6666


Connecting to 192.168.179.129:6666...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Tue Mar 24 14:02:08 2026
[root@server1 ~]# firewall-cmd --remove-forward-port=port=6666:proto=tcp:toport=22
success
```

###  firewalld富规则

各个选项的含义如下:

- `family`: 指定地址族,可以是 `ipv4` 或 `ipv6`
- `source`/`destination`: 指定源/目标地址
- `port`: 指定端口号
- `protocol`: 指定协议,如 `tcp`、`udp` 等
- `icmp-block-inversion`: 反转 ICMP 阻止规则
- `forward-port`: 端口转发规则
- `masquerade`: 启用地址伪装
- `log`: 日志记录规则,可指定前缀、日志级别、限速
- `accept`/`reject`/`drop`: 动作,分别表示允许、拒绝、丢弃

**案例一：**允许某个IP地址访问系统中的Web网站服务

```bash
# 网站准备
[root@server1 ~]# yum install -y httpd
[root@server1 ~]# systemctl start httpd
[root@server1 ~]# setenforce 0
[root@server1 ~]# echo 'hello world' >> /var/www/html/index.html
# 添加富规则
[root@server1 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.179.130" port port="80" protocol="tcp" accept'
success
[root@server1 ~]# firewall-cmd --reload
success
[root@server1 ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.179.130" port port="80" protocol="tcp" accept
# 可见rich rule已被添加

# 此时在server2上可以访问server1
[root@server2 ~]# curl 192.168.179.129 -I
HTTP/1.1 200 OK
Date: Tue, 24 Mar 2026 23:56:37 GMT
Server: Apache/2.4.62 (Rocky Linux)
Last-Modified: Tue, 24 Mar 2026 23:56:29 GMT
ETag: "c-64dcde66e4635"
Accept-Ranges: bytes
Content-Length: 12
Content-Type: text/html; charset=UTF-8
```

**案例二：**限制某个区域内的 SSH 连接次数

```bash
[root@server1 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.179.0/24"  port port="22" protocol="tcp"  limit value="3/m" accept'
success
[root@server1 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.179.0/24"  port port="22" protocol="tcp"  limit value="3/m" accept'
success
[root@server1 ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.179.130" port port="80" protocol="tcp" accept
	rule family="ipv4" source address="192.168.179.0/24" port port="22" protocol="tcp" accept limit value="3/m"
# 删除富规则
[root@server1 ~]# firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.179.0/24" port port="22" protocol="tcp" limit value="3/m" accept' 
success
[root@server1 ~]# firewall-cmd --reload
success
[root@server1 ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.179.130" port port="80" protocol="tcp" accept
```

**案例三**：使用富规则配置将6666端口上的访问转发到22端口上

```shell
# 这条规则将 6666 端口的 TCP 流量转发到 22 端口
[root@server1 ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" forward-port port="6666" protocol="tcp" to-port="22"'
success
[root@server1 ~]# firewall-cmd --reload
success
[root@server1 ~]# 
logout
Connection closing...Socket close.

Connection closed by foreign host.

Disconnected from remote host(192.168.179.129:22) at 08:17:46.

Type `help' to learn how to use Xshell prompt.
[G:\~]$ ssh root@192.168.179.129 6666


Connecting to 192.168.179.129:6666...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Wed Mar 25 08:17:36 2026 from 192.168.179.1
[root@server1 ~]# 
```

## SELinux管理

SELinux 的主要工作模式包括:

- Enforcing 模式:完全执行 SELinux 策略,阻止任何未经授权的访问行为
- Permissive 模式:只记录违反 SELinux 策略的行为,但不会阻止它们
- Disabled 模式:完全关闭 SELinux 功能

```shell
# 查看当前SELinux的模式
[root@server1 ~]# getenforce
Enforcing
# 临时调整为Permissive
[root@server1 ~]# setenforce 0
[root@server1 ~]# getenforce
Permissive
# 设置回Enforcing
[root@server1 ~]# setenforce 1
[root@server1 ~]# getenforce
Enforcing

# [0]为Permissive模式，只记录行为，不阻止
# [1]为Enforcing模式

# 永久关闭
[root@server1 ~]# vim /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled     #设置为diaabled  
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

