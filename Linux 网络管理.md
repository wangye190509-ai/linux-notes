# Linux 网络管理

## 计算机网络OSI七层模型：

![image-20260319101215318](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260319101215318.png)

## 三种体系结构对比：

![image-20260319102346570](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260319102346570.png)



## IPv4地址

- 在IP网络中，通信节点需要有一个唯一的IP地址
- IP地址用于IP报文的寻址以及标识一个节点
- IPv4地址一共32bits，使用点分十进制的形式表示

- IPv4地址由网络位和主机位组成
  - 网络位一致表示在同一个广播域中，可以直接通信（指某一个区域或者网段）
  - 主机位用于在同一个局域网中标识唯一节点（用于区分每一个机器的地址）



128 64  32  16   8   4   2   1

   7    6   5    4    3    2   1   0  

### 早期IP地址的划分

![image-20260319113842387](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260319113842387.png)

**A类：**网络位前面写0是固定的，网络位是8位，网络地址范围1.0.0.0~126.0.0.0，127开头的地址单独代表自己

  一个A类网络号下面可以有多少个主机？

  比如10.0.0.0/8,它可用的IP地址范围是10.0.0.1~10.255.255.254

  (主机位全部位0的时候是网络地址，不可用作IP地址；主机位全部为1的时候是广播地址，不可用做IP地址)



**B类：**网络位前面写10是固定的，网络位是16位，网络地址范围128.0.0.0~191.255.0.0

  一个B类网络号下面可以有多少个主机？

  比如172.16.0.0/16,它可用的IP地址范围是172.16.0.1~172.16.255.254

  (主机位全部位0的时候是网络地址，不可用作IP地址；主机位全部为1的时候是广播地址，不可用做IP地址)



**C类：**网络位前面写110是固定的，网络位是24位，网络地址范围192.0.0.0~223.255.255.0

  一个C类网络号下面可以有多少个主机？

  比如192.168.80.0/24,它可用的IP地址范围是192.168.80.1~192.168.80.254

  (主机位全部位0的时候是网络地址，不可用作IP地址；主机位全部为1的时候是广播地址，不可用做IP地址)



**D类：**网络位前面写1110是固定的，从224到239之间是组播地址

**E类：**保留使用

### 私有IP地址

A类B类C类分别划出了一些地址，供局域网内使用，这样的地址叫做私有地址，是不能够进行公网访问的。



## 子网掩码（Netmask）

**子网掩码是一个 32 位二进制数，与 IP 地址一一对应**：

- 掩码中 **1** 的位 → 对应 IP 的**网络位**
- 掩码中 **0** 的位 → 对应 IP 的**主机位**

### 常见子网掩码与对应容量：

| CIDR | 子网掩码        | 主机数（可用） | 适用场景              |
| :--- | :-------------- | :------------- | :-------------------- |
| /8   | 255.0.0.0       | 16,777,214     | 大型网络（如 A 类）   |
| /16  | 255.255.0.0     | 65,534         | 中型网络（如 B 类）   |
| /24  | 255.255.255.0   | 254            | 小型局域网（如 C 类） |
| /30  | 255.255.255.252 | 2              | 点对点链路            |
| /32  | 255.255.255.255 | 1              | 主机地址（无子网）    |

主机可用数计算公式：2主机位数−22主机位数−2（减去网络号和广播地址）

**示例**：

 IP：192.168.1.100   

子网掩码：255.255.255.0（/24）   

二进制：11111111.11111111.11111111.00000000   → 前 24 位是网络位，后 8 位是主机位

**子网掩码的三大好处**： 

1. **划分子网**：把一个大网络切成多个小网络，减少广播域 
2. **减少广播报文**：局域网设备少 → 广播风暴风险降低
3. **更高效利用地址**：不再受限于 A/B/C 类固定划分

## IP网络通信类型

- 单播(Unicast)

普通 IP（一对一）

- 广播(Broadcast)

255.255.255.255 或 子网广播地址

- 组播(Multicast)

224.0.0.0 – 239.255.255.255

## 一台计算机如何获取IP地址

1. 进行手动配置

2. 让计算机通过DHCP协议动态的获取

      获取哪些信息**：ip地址，子网掩码，默认网关，DNS 服务器**

## **网关**

一般是一个局域网的出口，当设备不知道将数据包发往何处时就转发给网关



## DHCP

DHCP（动态主机配置协议），主要用于给设备自动的分配IP地址以及网络信息，以便网络设备能够连接到网络并进行通信

DHCP给设备提供的信息如下：

- IP地址
- 子网掩码
- 网关地址
- DNS服务器地址
- 租约时间
- ![image-20260319151819471](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260319151819471.png)

### DHCP的工作过程

1. 客户端启动时发送 DHCP 广播消息(DHCPDISCOVER),寻找 DHCP 服务器。
2. DHCP 服务器收到广播消息后,会提供一个未被使用的 IP 地址以及其他网络配置信息(如子网掩码、默认网关、DNS 服务器等),并发送 DHCP 提供(DHCPOFFER)消息给客户端。
3. 客户端收到一个或多个 DHCP 提供消息后,会选择其中一个 DHCP 服务器提供的 IP 地址（按收到offer先后顺序选）,并发送 DHCP 请求(DHCPREQUEST)消息确认。
4. DHCP 服务器收到客户端的 DHCP 请求消息后,会将该 IP 地址分配给客户端,并发送 DHCP 确认(DHCPACK)消息给客户端。
5. 客户端收到 DHCP 确认消息后,就可以使用分配到的 IP 地址和其他网络配置参数连接到网络了。

![image-20260319152245774](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260319152245774.png)

其中每个数据包的作用如下：

1. **Discover**

   消息是用于客户端向整个内网发送广播，期待DHCP服务器进行回应

   这个数据包中的重要内容就是：消息类型，客户端ID，主机名，请求获得的信息

2. **Offer**

   消息是DHCP服务器对客户的回应

   这个消息中会回复对方所需要的所有信息

3. **Request**

   这个是客户端确认DHCP服务器的消息

   这个消息和第一个消息差不多，但是消息类别变为`Request`，并且会携带请求的IP地址

4. **ACK**

   DHCP服务器给客户端的最终确认

   这个消息和第二个消息差不多，但是消息类型变为`ACK`

### DHCP续租

DHCP分配的信息是有有效期的，再租约时间快到的时候，如果我们想要继续使用这个地址信息，就需要向DHCP服务器续租

```bash
[root@localhost ~]# cat /etc/NetworkManager/system-connections/ens160.nmconnection
[connection]
id=ens160
uuid=dfea55d8-6ddc-3229-8152-cb9e261de181
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1731149277

[ethernet]

[ipv4]
method=auto

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]


# 可以看到配置中的method字段是以dhcp来获得IP地址的。
```

同样从VMware的虚拟网络设置中，也可以看到租约时间相关的内容

![image-20260323104154427](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260323104154427.png)

### 在VMware中DHCP的情况

DHCP服务器的地址都是固定的x.x.x.2

vmnet1和vmnet8的地址都是x.x.x.1

虚拟机获取到的地址一般都是从x.x.x.128开始的

## DNS（域名解析）

最开始人们把IP和主机名的的对应关系记录在本地，这个文件目前在windows系统的`C:\Windows\System32\drivers\etc\hosts`中。但是后面发现并不好用，而且需要经常手动更新这个文件。类似于黄历

随着主机名越来越多，hosts文件不易维护，所以后来改用**域名解析系统DNS**：

- 一个组织的系统管理机构，维护系统内的每个主机的`IP和主机名`的对应关系
- 如果新计算机接入网络，将这个信息注册到`数据库`中
- 用户输入域名的时候，会自动查询`DNS`服务器，由`DNS服务器`检索数据库, 得到对应的IP地址。

### 域名

主域名是用来识别主机名称和主机所属的组织机构的一种分层结构的名称。 例如：[http://www.baidu.com(域名使用.连接](http://www.baidu.xn--com(-et6f94vu7gxx9d.xn--gzu679g/))

- com： 一级域名，表示这是一个企业域名。同级的还有 "net"(网络提供商)，"org"(非盈利组织) 等。
- baidu: 二级域名, 公司名。
- www: 只是一种习惯用法，并不是每个域名都支持。
- http:// : 要使用什么协议来连接这个主机名。

### 常见的域名解析服务器

- 114dns

 114.114.114.114

 114.114.115.115

 这是国内用户量数一数二的 dns 服务器，该 dns 一直标榜高速、稳定、无劫持、防钓鱼。

- Google DNS

 8.8.8.8

 8.8.4.4

### 域名解析的过程

以访问百度为例

1. 浏览器 → 本地 DNS 缓存
   ↓ 未命中
2. 电脑 → 操作系统 hosts 文件
   ↓ 未命中
3. 电脑 → 配置的 DNS 服务器（如 114.114.114.114）
   ↓ 服务器没有记录
4. DNS 服务器 → 根域名服务器（.）→ 找到 .com 的权威 DNS
5. DNS 服务器 → .com 顶级域服务器 → 找到 baidu.com 的权威 DNS
6. DNS 服务器 → baidu.com 权威 DNS → 返回 www 的 IP
7. DNS 服务器 缓存结果，返回给电脑
8. 电脑 缓存结果，浏览器发起请求

# 配置网络服务

## 修改网卡配置文件

```bash
# 修改网卡配置文件
[root@localhost ~]# vi /etc/NetworkManager/system-connections/ens160.nmconnection 
[connection]
id=ens160
uuid=dfea55d8-6ddc-3229-8152-cb9e261de181
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1732264040

[ethernet]

[ipv4]
address1=192.168.88.110/24,192.168.88.2
dns=114.114.114.114;114.114.115.115;
method=manual

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

当修改完Linux系统中的服务配置文件后，并不会对服务程序立即产生效果。要想让服务程序获取到最新的配置文件，需要手动重启相应的服务，之后就可以看到网络畅通了

```bash
[root@localhost ~]# systemctl restart NetworkManager
[root@localhost ~]# ping -c 4 www.baidu.com
PING www.a.shifen.com (180.101.50.242) 56(84) bytes of data.
64 bytes from 180.101.50.242 (180.101.50.242): icmp_seq=1 ttl=128 time=11.3 ms
...
```

## 网卡配置文件参数

| **节**         | **参数**               | **描述**                                                     |
| -------------- | ---------------------- | ------------------------------------------------------------ |
| `[connection]` | `id`                   | 连接的名称，这里是 `ens160`。                                |
|                | `uuid`                 | 连接的唯一标识符（UUID），用于唯一识别此连接。               |
|                | `type`                 | 连接的类型，`ethernet` 表示这是一个以太网连接。              |
|                | `autoconnect-priority` | 自动连接优先级，数值越小优先级越低。这里设置为 `-999`，表示极低的优先级。 |
|                | `interface-name`       | 连接对应的网络接口名称，这里是 `ens160`。                    |
|                | `timestamp`            | 连接的时间戳，表示最后修改的时间。                           |
| `[ethernet]`   | -                      | 此节用于配置以太网特定的设置，当前没有额外参数。             |
| `[ipv4]`       | `address1`             | 静态 IPv4 地址及其子网掩码，格式为 `IP地址/子网掩码` 和网关，例：`192.168.88.110/24,192.168.88.2`。 |
|                | `dns`                  | DNS 服务器地址，多个地址用分号分隔，这里是 `114.114.114.114;114.114.115.115;`。 |
|                | `method`               | IPv4 地址配置方法，这里设置为 `manual`，表示使用手动配置。   |
| `[ipv6]`       | `addr-gen-mode`        | 地址生成模式，`eui64` 表示使用 EUI-64 地址生成方式。         |
|                | `method`               | IPv6 地址配置方法，这里设置为 `auto`，表示自动获取 IPv6 地址。 |
| `[proxy]`      | -                      | 此节用于配置代理设置，当前没有额外参数。                     |

## curl

```shell
# 访问网站
curl x.x.x.x      # 不加参数是直接返回内容
curl -I x.x.x.x   # 加-I是返回报文的头部信息，通常用与返回http状态返回码
```

##  nmcli 工具

nmcli命令是redhat7或者centos7之后的命令，该命令可以完成网卡上所有的配置工作，并且可以写入 配置文件，永久生效

- 查看接口状态

```bash
[root@localhost ~]# nmcli device status
DEVICE  TYPE      STATE                   CONNECTION
ens160  ethernet  connected               ens160
lo      loopback  connected (externally)  lo
```

- 查看链接信息

```bash
[root@localhost ~]# nmcli connection show
NAME    UUID                                  TYPE      DEVICE
ens160  dfea55d8-6ddc-3229-8152-cb9e261de181  ethernet  ens160
lo      529b2eed-2755-4cce-af3c-999beb49882d  loopback  lo
```

- 配置IP等网络信息

```bash
[root@localhost ~]# nmcli connection modify "ens160" ipv4.addresses 192.168.179.100/24 ipv4.gateway 192.168.179.2 ipv4.dns "8.8.8.8,8.8.4.4" ipv4.method manual
```

- 启动/停止接口

```bash
[root@localhost ~]# nmcli connection down ens160
[root@localhost ~]# nmcli connection up ens160
```

- 创建链接

```bash
[root@localhost ~]# nmcli connection add type ethernet ifname ens160 con-name dhcp_ens160
Connection 'dhcp_ens160' (42512e8e-0740-4503-b8f2-9e9f18d73159) successfully added.
[root@localhost ~]# nmcli connection up dhcp_ens160 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
[root@localhost ~]# nmcli device status 
DEVICE  TYPE      STATE                   CONNECTION  
ens160  ethernet  connected               dhcp_ens160 
lo      loopback  connected (externally)  lo          
```

- 删除链接

```bash
[root@localhost ~]# nmcli connection delete dhcp_ens160 
Connection 'dhcp_ens160' (42512e8e-0740-4503-b8f2-9e9f18d73159) successfully deleted.
[root@localhost ~]# nmcli device status 
DEVICE  TYPE      STATE                   CONNECTION 
ens160  ethernet  connected               ens160     
lo      loopback  connected (externally)  lo         
```

##  ifconfig

ifconfig 命令用于显示或设置网络设备。

**需要安装**：**`yum install -y net-tools`

- 显示网络设备信息

```bash
[root@localhost ~]# ifconfig 
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.179.129  netmask 255.255.255.0  broadcast 192.168.179.255
        inet6 fe80::20c:29ff:fee8:c666  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:e8:c6:66  txqueuelen 1000  (Ethernet)
        RX packets 31597  bytes 39948777 (38.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14220  bytes 1278929 (1.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2  bytes 628 (628.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2  bytes 628 (628.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.179.2   0.0.0.0         UG    100    0        0 ens160
192.168.179.0   0.0.0.0         255.255.255.0   U     100    0        0 ens160
# route -n 命令用于显示系统的路由表，-n 参数表示不解析主机名（直接显示 IP 地址，速度更快
```

- 启动关闭指定网卡

```bash
[root@localhost ~]# ifconfig ens160 down
[root@localhost ~]# ifconfig ens160 up
```

- 配置IP地址

```bash
[root@localhost ~]# ifconfig eth0 192.168.1.56 netmask 255.255.255.0 broadcast 192.168.1.255
// 给eth0网卡配置IP地址,加上子掩码,加上个广播地址
```
