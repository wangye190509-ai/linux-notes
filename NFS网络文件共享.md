# NFS网络文件共享

**NFS（Network File System）** = 网络文件系统，让客户端通过网络访问服务器文件，就像访问本地文件一样。

## NFS 与 RPC 的关系

| 组件    | 角色     | 说明                         |
| :------ | :------- | :--------------------------- |
| **NFS** | 文件系统 | 负责共享文件，本身不传输数据 |
| **RPC** | 传输机制 | 负责网络通信和数据传输       |

> **一句话**：NFS 是“共享什么”，RPC 是“怎么传输”。NFS 依赖 RPC 才能工作。

------

## RPC 核心要点

| 要点     | 说明                                                 |
| :------- | :--------------------------------------------------- |
| **全称** | Remote Procedure Call（远程过程调用）                |
| **作用** | 让客户端调用远程服务器上的程序，就像调用本地程序一样 |
| **端口** | 111（TCP/UDP）                                       |
| **模式** | 客户机/服务器模式                                    |
| **功能** | 为应用层协议提供端口注册服务                         |

------

## RPC 工作流程



```
1. NFS服务启动 → 向 RPC（portmapper）注册自己的端口
2. 客户端请求 → 询问 RPC：“NFS服务在哪个端口？”
3. RPC 响应 → 告知客户端 NFS 端口号
4. 客户端连接 → 直接访问 NFS 服务端口进行数据传输
```



```
┌────────┐                    ┌────────┐
│ 客户端  │                    │ 服务端  │
└───┬────┘                    └───┬────┘
    │  1. 询问NFS端口               │
    │────────────────────────────►│ RPC:111
    │  2. 返回NFS端口(2049)         │
    │◄────────────────────────────│
    │  3. 建立NFS连接               │
    │────────────────────────────►│ NFS:2049
```



------

## NFS 完整工作流程

| 步骤  | 动作                                       |
| :---- | :----------------------------------------- |
| **1** | 服务端启动 RPC 服务，开启 111 端口         |
| **2** | 服务端启动 NFS 服务，向 RPC 注册端口信息   |
| **3** | 客户端启动 RPC，向服务端 RPC 请求 NFS 端口 |
| **4** | 服务端 RPC 返回 NFS 端口号给客户端         |
| **5** | 客户端通过获取的端口连接 NFS 服务          |
| **6** | 数据传输                                   |

------

## 挂载原理

```
NFS服务器                    客户端A                   客户端B
/opt/share   ──挂载──►    /mnt/nfs1           ──挂载──► /data/nfs
            └───────── 远程访问，像本地一样 ─────────┘
```

**核心**：

- 服务器共享一个目录（如 `/opt/share`）
- 客户端挂载到本地任意目录（如 `/mnt/nfs`）
- 挂载后，客户端访问本地挂载点 = 访问服务器共享目录

**总结**

> **NFS 共享文件，RPC 传数据**
> **RPC 端口 111，NFS 端口 2049**
> **先注册，后查询，再连接**

## NFS相关工具

### rpcinfo

rpcinfo 工具可以查看 RPC 相关信息

查看注册在指定主机的 RPC 程序

```shell
rpcinfo -p hostname
```

查看 rpc 注册程序

```shell
rpcinfo -s hostname
```

### exportfs

可用于管理 NFS 导出的文件系统

常见选项：

- **-v**：查看本机所有 NFS 共享
- **-r**：重读配置文件，并共享目录
- **-a**：输出本机所有共享
- **-au**：停止本机所有共享

### showmount

常见用法：

```shell
showmount -e hostname
```

### 配置共享目录

服务端上创建共享目录

```bash
[root@localhost ~]# mkdir -p /myshare
```

### NFS关于用户映射的参数

**root_squash**

  作用：将远程客户端的 root 用户（UID 0）映射为服务器上的匿名用户（默认是 nfsnobody 或 nobody）。

  默认值：NFS 导出默认启用此选项。

  不同系统的匿名用户：

  CentOS 8/RHEL 8：nobody（UID 65534）。

  早期版本：nfsnobody（UID 65534 或 4294967294，取决于系统配置）。

  用途：增强安全性，防止客户端 root 拥有服务器文件的 root 权限。

**no_root_squash**

  作用：禁用 root 映射，客户端 root 用户保持 root 权限（UID 0）。

  风险：允许客户端 root 完全控制导出的文件系统，可能导致安全问题（慎用）。

  典型场景：需要客户端 root 直接管理共享文件（如磁盘less客户端）。

**all_squash**

  作用：将所有远程用户（包括普通用户和 root）映射为匿名用户（默认 nfsnobody/nobody）。

  用途：强制所有访问共享目录的用户以最低权限运行，适用于公共只读共享。

  注意：会忽略客户端用户的原始 UID/GID。

**no_all_squash**

  作用：保留客户端用户的原始 UID 和 GID（默认行为）。

  用途：允许客户端用户以自身身份访问文件，要求服务器和客户端的 UID/GID 一致。

**anonuid 和 anongid**

  作用：自定义匿名用户的 UID 和 GID（覆盖默认的 nfsnobody/nobody）。

  典型用法：配合 all_squash 或 root_squash，将用户映射到特定本地用户。

  示例：

  /shared 192.168.1.0/24(rw,all_squash,anonuid=1001,anongid=1001)

  所有客户端用户会被映射为服务器端的 UID 1001 和 GID 1001。



**samba**（用于windows、Linux之间跨平台的文件共享）

  企业使用samba很少，服务不稳定，不需要重点掌握

## 手动挂载案例

```bash
# 服务端
[root@server1 ~]# systemctl enable --now rpcbind
[root@server1 ~]# systemctl enable --now nfs-server
[root@server1 ~]# vim /etc/exports
/myshare 192.168.179.0/24
[root@server1 ~]# echo hello > /myshare/file

# 客户端：
[root@server2 ~]# yum install -y rpcbind nfs-utils
[root@server2 ~]# showmount -e 192.168.179.10
Export list for 192.168.179.10:
# 虽然我们自己配置共享了，但是没有重读配置文件，所以读不到

# 服务端：
[root@server1 ~]# exportfs -r
exportfs: No options for /myshare 192.168.179.0/24: suggest 192.168.179.0/24(sync) to avoid warning
[root@server1 ~]# exportfs -v
/myshare      	192.168.179.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,ro,secure,root_squash,no_all_squash)

# 客户端：
[root@server2 ~]# showmount -e 192.168.179.10
Export list for 192.168.179.10:
/myshare 192.168.179.0/24
[root@server2 ~]# mkdir /mnt/nfs
[root@server2 ~]# mount -t nfs 192.168.179.10:/myshare /mnt/nfs
[root@server2 ~]# df -Th
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                   tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs                   tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root     xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1          xfs       960M  226M  735M  24% /boot
tmpfs                   tmpfs     363M     0  363M   0% /run/user/0
192.168.179.10:/myshare nfs4       17G  1.4G   16G   9% /mnt/nfs
[root@server2 ~]# cd /mnt/nfs
[root@server2 nfs]# ll
total 4
-rw-r--r--. 1 root root 6 Mar 26 16:53 file
[root@server2 nfs]# cat file
hello
[root@server2 nfs]# rm file
rm: remove regular file 'file'? yes
rm: cannot remove 'file': Read-only file system
[root@server2 nfs]# cd
[root@server2 ~]# umount /mnt/nfs
# 现在是只读模式，想要修改模式要去改配置文件，先卸载挂载

# 服务端：
[root@server1 ~]# vim /etc/exports
/myshare 192.168.179.0/24(rw,sync,root_squash,no_all_squash)
[root@server1 ~]# exportfs -v
/myshare      	192.168.179.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,ro,secure,root_squash,no_all_squash)
[root@server1 ~]# exportfs -r
[root@server1 ~]# exportfs -v
/myshare      	192.168.179.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)


# 客户端：
[root@server2 ~]# mount -t nfs 192.168.179.10:/myshare /mnt/nfs
[root@server2 ~]# cd /mnt/nfs
[root@server2 nfs]# ll
total 4
-rw-r--r--. 1 root root 6 Mar 26 16:53 file
[root@server2 nfs]# rm -f file
rm: cannot remove 'file': Read-only file system
# 虽然给了rw权限，但是目录权限被linux控制

# 服务端：
[root@server1 ~]# ll -d /myshare
drwxr-xr-x 2 root root 18 Mar 26 16:53 /myshare
[root@server1 ~]# chmod a+w /myshare/

# 客户端：
[root@server2 nfs]# rm -f file
[root@server2 nfs]# echo "hello this is server2" > file
[root@server2 nfs]# useradd user01
[root@server2 nfs]# su - user01
[user01@server2 nfs]$ touch file1
[user01@server2 nfs]$ ll
total 4
-rw-r--r--. 1 nobody nobody 22 Mar 26 17:14 file
-rw-r--r--. 1 user01 user01  0 Mar 26 17:16 file1


# 服务端：
[root@server1 myshare]# ll
total 4
-rw-r--r-- 1 nobody nobody 22 Mar 26 17:14 file
-rw-r--r-- 1   1000   1000  0 Mar 26 17:16 file1
[root@server1 myshare]# useradd -u 1000 zhangsan
[root@server1 myshare]# ll
total 4
-rw-r--r-- 1 nobody   nobody   22 Mar 26 17:14 file
-rw-r--r-- 1 zhangsan zhangsan  0 Mar 26 17:16 file1
[root@server1 myshare]# vim /etc/exports
[root@server1 myshare]# exportfs -r
[root@server1 myshare]# vim /etc/exports
[root@server1 myshare]# exportfs -r
[root@localhost myshare]# exportfs -v
/myshare        192.168.88.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,all_squash)

# 客户端：
[root@server2 ~]# umount /mnt/nfs
[root@server2 ~]# mount -t nfs 192.168.179.10:/myshare /mnt/nfs
[root@server2 ~]# cd /mnt/nfs
[root@server2 nfs]# touch file{2,3}
[root@server2 nfs]# ll
total 4
-rw-r--r--. 1 nobody nobody 22 Mar 26 17:14 file
-rw-r--r--. 1 user01 user01  0 Mar 26 17:16 file1
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file2
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file3
[root@server2 nfs]# su - user01
Last login: Thu Mar 26 17:15:54 CST 2026 on pts/0
[user01@server2 ~]$ cd /mnt/nfs
[user01@server2 nfs]$ touch file4
[user01@server2 nfs]$ ll
total 4
-rw-r--r--. 1 nobody nobody 22 Mar 26 17:14 file
-rw-r--r--. 1 user01 user01  0 Mar 26 17:16 file1
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file2
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file3
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file4
[user01@server2 nfs]$ 
logout
[root@server2 nfs]# cd
[root@server2 ~]# umount /mnt/nfs
[root@server2 ~]# mount -t nfs 192.168.179.10:/myshare /mnt/nfs
[root@server2 ~]# cd /mnt/nfs
[root@server2 nfs]# touch file5
[root@server2 nfs]# ll
total 4
-rw-r--r--. 1 nobody nobody 22 Mar 26 17:14 file
-rw-r--r--. 1 user01 user01  0 Mar 26 17:16 file1
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file2
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file3
-rw-r--r--. 1 nobody nobody  0 Mar 26 17:20 file4
```

## 自动挂载案例

```shell
[root@server2 ~]# yum install -y autofs
[root@server2 ~]# rpm -ql autofs | grep etc
/etc/auto.master
/etc/auto.master.d
/etc/auto.misc
/etc/auto.net
/etc/auto.smb
/etc/autofs.conf
/etc/autofs_ldap_auth.conf
/etc/sysconfig/autofs
# /etc/auto.master 是 autofs 的主配置文件，用于定义挂载点及其对应的映射文件。
[root@server2 ~]# vim /etc/auto.master
/mnt/nfs /etc/auto.nfs --timeout=300
```

 配置说明：

- `/mnt/nfs`：挂载点的根目录。
- `/etc/auto.nfs`：挂载点对应的映射文件。
- `--timeout=300`：挂载超时时间（单位为秒），300 秒后未访问的挂载点将自动卸载。

```shell
[root@server2 ~]# vim /etc/auto.nfs
share   -fstype=nfs4,rw,soft    192.168.129.10:/myshare
```

```
用户访问 /mnt/nfs/share
         ↓
autofs 读取 /etc/auto.nfs
         ↓
匹配到 "share" 配置
         ↓
自动执行：mount -t nfs4 -o rw,soft 192.168.129.10:/myshare /mnt/nfs/share
         ↓
用户可以访问文件
         ↓
300秒无访问 → 自动卸载
```

```shell
[root@server2 ~]# systemctl restart autofs
[root@server2 ~]# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs               tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs               tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1      xfs       960M  226M  735M  24% /boot
tmpfs               tmpfs     363M     0  363M   0% /run/user/0
[root@server2 ~]# cd /mnt/nfs
[root@server2 nfs]# ll
total 0
[root@server2 nfs]# ls share   # 一访问share就会自动挂载
file  file1  file2  file3  file4  file5
[root@server2 nfs]# ll
total 0
drwxrwxrwx. 2 root root 83 Mar 26 17:21 share
[root@server2 nfs]# df -Th
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                   tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs                   tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root     xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1          xfs       960M  226M  735M  24% /boot
tmpfs                   tmpfs     363M     0  363M   0% /run/user/0
192.168.179.10:/myshare nfs4       17G  1.4G   16G   9% /mnt/nfs/share
```

### 其他配置选项

在挂载映射文件中可以使用多种选项，以下是常用参数的详细说明：

#### 文件系统类型选项 (`-fstype=`)

- **nfs**：适用于 NFSv3 文件系统。
- **nfs4**：适用于 NFSv4 文件系统。

#### 挂载选项

- **rw**：读写权限。
- **ro**：只读权限。
- **soft**：允许客户端在超时后返回错误。
- **hard**：客户端会一直尝试连接，直到服务器恢复正常。
- **intr**：允许中断挂载操作（NFSv3 使用）。
- **timeo=**：超时时间（默认 600 分钟）。
- **bg**：后台挂载操作。



### 实战案例

一、NFS服务器(server1)创建用户和对应的目录，将用户user01的家目录共享出来

```bash
[root@server1 ~]# mkdir /data
[root@server1 ~]# useradd -d /data/user01 user01
[root@server1 ~]# id user01
uid=1000(user01) gid=1000(user01) groups=1000(user01)
[root@server1 ~]# vim /etc/exports
[root@server1 ~]# exportfs -r
[root@server1 ~]# exportfs -v
/data/user01  	192.168.179.0/24(sync,wdelay,hide,no_subtree_check,anonuid=1000,anongid=1000,sec=sys,rw,secure,root_squash,all_squash)
```

二、在NFS客户端(server2)中配置autofs

```bash
[root@server2 ~]# vim /etc/auto.master
/-  /etc/auto.user
[root@server2 ~]# vim /etc/auto.user
/data/user01  -fstype=nfs4,rw,soft    192.168.179.10:/data/user01
[root@server2 ~]# systemctl restart autofs
```

三、在server2中创建user01用户

```bash
[root@server2 ~]# useradd -d /data/user01 -u 1000 user01
```

四、测试

```bash
# 在server1中，登录到user01用户创建一个文件
[root@server1 ~]# su - user01
[user01@server1 ~]$ echo "hello this is server1" > file
[user01@server1 ~]$ ls
file

# 在server2中，登录到user01用户查看是否共享了该文件
[root@server2 ~]# su - user01
Last login: Thu Mar 26 21:22:02 CST 2026 on pts/0
[user01@server2 ~]$ ls
file
[user01@server2 ~]$ cat file
hello this is server1
[user01@server2 ~]$ pwd
/data/user01

# 检查server2中挂载情况
[user01@server2 ~]$ df -Th
df: /mnt/nfs: Stale file handle
Filesystem                  Type      Size  Used Avail Use% Mounted on
devtmpfs                    devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                       tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs                       tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root         xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1              xfs       960M  226M  735M  24% /boot
tmpfs                       tmpfs     363M     0  363M   0% /run/user/0
192.168.179.10:/data/user01 nfs4       17G  1.4G   16G   9% /data/user01
```



# ss

`ss`（Socket Statistics）是 Linux 中用于查看网络连接状态的命令，是 `netstat` 的现代替代品，**更快、信息更详细**。

## 常用选项

| 选项 | 说明                         | 示例    |
| :--- | :--------------------------- | :------ |
| `-t` | 仅显示 TCP 连接              | `ss -t` |
| `-u` | 仅显示 UDP 连接              | `ss -u` |
| `-l` | 仅显示监听状态的连接         | `ss -l` |
| `-a` | 显示所有连接（监听+非监听）  | `ss -a` |
| `-n` | 显示数字端口（不解析服务名） | `ss -n` |
| `-p` | 显示进程 PID 和名称          | `ss -p` |
| `-e` | 显示详细信息                 | `ss -e` |
| `-i` | 显示 TCP 内部信息            | `ss -i` |
| `-s` | 显示统计摘要                 | `ss -s` |
| `-4` | 仅显示 IPv4                  | `ss -4` |
| `-6` | 仅显示 IPv6                  | `ss -6` |
| `-r` | 尝试解析主机名               | `ss -r` |

## 常用命令

```shell
# 查看所有 TCP 连接
[root@server1 ~]# ss -t
State     Recv-Q     Send-Q           Local Address:Port            Peer Address:Port      Process     
ESTAB     0          68              192.168.179.10:ssh            192.168.179.1:56136                 
# 查看所有 TCP/UDP 连接（数字格式）
[root@server1 ~]# ss -tun
Netid    State    Recv-Q    Send-Q        Local Address:Port          Peer Address:Port     Process    
tcp      ESTAB    0         68           192.168.179.10:22           192.168.179.1:56136               
# 查看所有监听的端口
[root@server1 ~]# ss -tln
State      Recv-Q      Send-Q           Local Address:Port           Peer Address:Port     Process     
LISTEN     0           128                    0.0.0.0:22                  0.0.0.0:*                    
LISTEN     0           4096                   0.0.0.0:111                 0.0.0.0:*                    
LISTEN     0           128                       [::]:22                     [::]:*                    
LISTEN     0           4096                      [::]:111                    [::]:*                    
# 查看监听端口并显示进程
[root@server1 ~]# ss -tlnp
State   Recv-Q  Send-Q     Local Address:Port     Peer Address:Port  Process                           
LISTEN  0       128              0.0.0.0:22            0.0.0.0:*      users:(("sshd",pid=814,fd=3))    
LISTEN  0       4096             0.0.0.0:111           0.0.0.0:*      users:(("systemd",pid=1,fd=87))  
LISTEN  0       128                 [::]:22               [::]:*      users:(("sshd",pid=814,fd=4))    
LISTEN  0       4096                [::]:111              [::]:*      users:(("systemd",pid=1,fd=97))  
```



