# Samba文件共享

## Samba用途

**文件共享**：允许用户在不同操作系统之间共享文件。

**打印服务**：提供跨平台的打印服务和在线编辑。

**域控制器**：Samba 可以用作 Windows 网络的域控制器。

**认证与授权**：支持用户认证、访问控制和权限管理。

**跨平台互操作性**：让 Linux/Unix 系统与 Windows 系统无缝协作。

Windows计算机网络管理模式：

- 工作组WORKGROUP：计算机对等关系，帐号信息各自管理
- 域DOMAIN：C/S结构，帐号信息集中管理，DC,AD

## Samba相关软件包介绍

在 Rocky Linux 中，Samba 的核心组件包含以下软件包：

- **samba**：Samba 的主包，包括核心服务和工具。
- **samba-client**：提供客户端工具，用于访问远程的 SMB/CIFS 共享。
- **samba-common**：共享的配置文件和库。
- **samba-libs**：Samba 运行所需的库。
- **samba-common-tools**：包含测试和管理工具，例如 `smbstatus`。
- **smbclient**：命令行工具，用于访问 SMB/CIFS 共享。
- **cifs-utils**：提供挂载 SMB 文件系统的工具（如 `mount.cifs`）。

## 相关服务进程

**smbd**：提供文件共享和打印服务，TCP：139、445。

**nmbd**：负责 NetBIOS 名称解析和浏览功能，UDP：137、138。

**winbindd**：用于与 Windows 域集成，支持用户和组的认证。

**samba-ad-dc**：Samba 4 中的域控制器服务。

## Samba主配置文件

主配置文件：/etc/samba/smb.conf 帮助参看：man smb.conf

语法检查： testparm [-v] [/etc/samba/smb.conf]

```bash
[global]
   workgroup = WORKGROUP        # 工作组名称
   server string = Samba Server # 服务器描述
   security = user              # 认证模式
   log file = /var/log/samba/log.%m # 日志文件路径
   max log size = 50            # 最大日志文件大小（KB）
   dns proxy = no               # 禁用 DNS 代理

[shared]
   path = /srv/samba/shared      # 共享路径
   browseable = yes              # 是否可浏览
   writable = yes                # 是否可写
   valid users = @smbgroup       # 允许访问的用户/组
```

### 全局设置（[global]）

- `workgroup`：指定工作组名称，默认是 `WORKGROUP`。
- security：
  - `user`：用户级认证（常用）。
  - `share`：共享级认证（不推荐）。
  - `domain`：域级认证。
  - `ads`：Active Directory 服务。
- `log file`：日志文件路径。
- `max log size`：限制日志文件大小。

### 共享设置（[共享名]）

- `path`：共享目录的路径。
- `browseable`：决定共享是否可被浏览。
- `writable`：是否允许写入。
- `valid users`：指定允许访问的用户或组。



# windows连接案例

```shell
# 安装Samba服务
[root@server1 ~]# yum install -y samba
[root@server1 ~]# ss -nltp
State   Recv-Q  Send-Q    Local Address:Port     Peer Address:Port  Process                            
LISTEN  0       128             0.0.0.0:22            0.0.0.0:*      users:(("sshd",pid=815,fd=3))     
LISTEN  0       50              0.0.0.0:139           0.0.0.0:*      users:(("smbd",pid=12200,fd=30))  
LISTEN  0       50              0.0.0.0:445           0.0.0.0:*      users:(("smbd",pid=12200,fd=29))  
LISTEN  0       128                [::]:22               [::]:*      users:(("sshd",pid=815,fd=4))     
LISTEN  0       50                 [::]:139              [::]:*      users:(("smbd",pid=12200,fd=28))  
LISTEN  0       50                 [::]:445              [::]:*      users:(("smbd",pid=12200,fd=27))

# 配置Samba用户
[root@server1 ~]# useradd -s /sbin/nologin smbuser
[root@server1 ~]# echo 123 | passwd --stdin smbuser
Changing password for user smbuser.
passwd: all authentication tokens updated successfully.
[root@server1 ~]# tail -1 /etc/passwd
smbuser:x:1000:1000::/home/smbuser:/sbin/nologin
[root@server1 ~]# smbpasswd -a smbuser
New SMB password:
Retype new SMB password:
Added user smbuser.
[root@server1 ~]# smbpasswd -e smbuser
[root@server1 ~]# smbstatus

Samba version 4.22.4
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------

/var/lib/samba/lock/locking.tdb not initialised
This is normal if an SMB client has never connected to your server.

#创建共享目录
[root@server1 ~]# mkdir -p /data/samba
[root@server1 ~]# chown -R smbuser:smbuser /data/samba
[root@server1 ~]# chmod -R 2770 /data/samba
[root@server1 ~]# ls -lha /data/samba/
total 0
drwxrws--- 2 smbuser smbuser  6 Mar 27 08:36 .
drwxr-xr-x 3 root    root    19 Mar 27 08:36 ..
# 添加配置文件
[root@server1 ~]# vim /etc/samba/smb.conf
......
[shared]
   path = /data/samba
   browseable = yes
   writable = yes
   valid users = @smbuser
   create mask = 0660
   directory mask = 2770

# 重启smb服务  
[root@server1 ~]# systemctl restart smb
[root@server1 ~]# systemctl restart nmb
```

**windows客户端连接**

在运行窗口中输入：`\\192.168.179.10\`进行连接

![image-20260327090730865](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260327090730865.png)

**用户验证：smbuser/123**

![image-20260327085230417](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260327085230417.png)

**文件创建写入测试**

![image-20260327085237470](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260327085237470.png)

```shell
[root@server1 ~]# cd /data/samba
[root@server1 samba]# ls
file1.txt
[root@server1 samba]# cat file1.txt
this is file1
[root@server1 samba]# echo "this is file2" > file2.txt
[root@server1 samba]# ls
file1.txt  file2.txt
```

# Linux连接案例

一、客户端工具下载

```bash
[root@server2 ~]# yum install -y samba-client
```

二、创建上传测试文件

```bash
[root@server2 ~]# echo "this is server2......" > server2.txt
```

三、使用smbclient连接服务器测试

```bash
[root@server2 ~]# smbclient -L 192.168.179.10 -U smbuser
Password for [SAMBA\smbuser]:

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	shared          Disk      
	IPC$            IPC       IPC Service (Samba 4.22.4)
	smbuser         Disk      Home Directories
SMB1 disabled -- no workgroup available
[root@server2 ~]# smbclient //192.168.179.10/shared -U smbuser%123
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Mar 27 08:56:33 2026
  ..                                  D        0  Fri Mar 27 08:56:33 2026
  file1.txt                           A       13  Fri Mar 27 08:54:26 2026
  file2.txt                           N       14  Fri Mar 27 08:55:34 2026

		17756160 blocks of size 1024. 16260624 blocks available
smb: \> get file1.txt
getting file \file1.txt of size 13 as file1.txt (4.2 KiloBytes/sec) (average 4.2 KiloBytes/sec)
smb: \> put server2.txt
putting file server2.txt as \server2.txt (7.2 kb/s) (average 7.2 kb/s)
smb: \>  ^C
```

**挂载CIFS文件系统**

手动挂载：

```bash
[root@server2 ~]# yum install -y cifs-utils
[root@server2 ~]# mount -t cifs //192.168.179.10/shared /mnt/smb -o username=smbuser,password=123
[root@server2 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 4.0M     0  4.0M   0% /dev
tmpfs                    1.8G     0  1.8G   0% /dev/shm
tmpfs                    726M  9.0M  717M   2% /run
/dev/mapper/rl-root       17G  1.5G   16G   9% /
/dev/nvme0n1p1           960M  226M  735M  24% /boot
tmpfs                    363M     0  363M   0% /run/user/0
//192.168.179.10/shared   17G  1.5G   16G   9% /mnt/smb
```

开机自动挂载：

```bash
[root@server2 ~]# umount /mnt/smb
[root@server2 ~]# vim /etc/fstab
......
//192.168.179.10/shared  /mnt/smb        cifs    defaults,username=smbuser,password=123 0 0

[root@server2 ~]# systemctl daemon-reload
[root@server2 ~]# mount -a
[root@server2 ~]# df -Th
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                   tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs                   tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root     xfs        17G  1.5G   16G   9% /
/dev/nvme0n1p1          xfs       960M  226M  735M  24% /boot
tmpfs                   tmpfs     363M     0  363M   0% /run/user/0
//192.168.179.10/shared cifs       17G  1.5G   16G   9% /mnt/smb
```

# 实战：不同账户访问不同目录

## 服务端

```shell
# ============================================================
# 第一部分：创建系统用户
# ============================================================

[root@server1 ~]# useradd -s /sbin/nologin -r smb1
# -s /sbin/nologin : 设置用户的登录 shell 为 nologin，禁止该用户通过控制台或 SSH 登录系统
# -r               : 创建系统用户（UID 范围 1-999），用于运行服务而非人类用户
# smb1             : 用户名，仅用于 Samba 认证，不能登录系统
# 作用：创建系统账户，作为 Samba 访问的认证用户，同时禁止系统登录，提高安全性

[root@server1 ~]# useradd -s /sbin/nologin -r smb2
[root@server1 ~]# useradd -s /sbin/nologin -r smb3


# ============================================================
# 第二部分：添加用户到 Samba 数据库
# ============================================================

[root@server1 ~]# smbpasswd -a smb1
# -a : 添加用户到 Samba 认证数据库（/var/lib/samba/private/passdb.tdb）
# 作用：为用户设置 SMB 协议访问密码，与系统登录密码独立
# 执行后会交互式提示输入密码，这是 Samba 访问的凭证
New SMB password:
Retype new SMB password:
Added user smb1.

[root@server1 ~]# smbpasswd -a smb2
New SMB password:
Retype new SMB password:
Added user smb2.

[root@server1 ~]# smbpasswd -a smb3
New SMB password:
Retype new SMB password:
Added user smb3.

[root@server1 ~]# smbpasswd -e smb1
# -e : 启用用户账户（enable）
# 作用：激活已添加的用户，允许通过 Samba 访问
# 注意：smbpasswd -a 添加的用户默认就是启用状态，此步骤可选
Enabled user smb1.

[root@server1 ~]# smbpasswd -e smb2
Enabled user smb2.

[root@server1 ~]# smbpasswd -e smb3
Enabled user smb3.


# ============================================================
# 第三部分：查看已添加的 Samba 用户
# ============================================================

[root@server1 ~]# pdbedit -L
# pdbedit : Samba 用户数据库管理工具
# -L      : 列出所有 Samba 用户
# 输出说明：格式为 "用户名:UID:用户SID"
smbuser:1000:
smb2:986:
smb1:987:
smb3:985:


# ============================================================
# 第四部分：修改 Samba 主配置文件
# ============================================================

[root@server1 ~]# vim /etc/samba/smb.conf

# 在 [global] 段添加 config file 参数
[global]
    config file = /etc/samba/conf.d/%U
    # %U : Samba 变量，代表当前连接的用户名（如 smb1、smb2、smb3）
    # 作用：当用户连接时，自动加载 /etc/samba/conf.d/用户名 配置文件
    # 示例：smb1 连接时加载 /etc/samba/conf.d/smb1

# 定义默认共享（当用户没有独立配置文件时使用）
[share]
    path = /data/samba/share
    browseable = yes        # 共享在客户端可见
    writable = yes          # 允许写入
    Guest ok = yes          # 允许匿名访问
    create mask = 0660      # 新建文件权限：rw-rw----
    directory mask = 2770   # 新建目录权限：drwxrws---（2 表示 SGID）


# ============================================================
# 第五部分：创建共享目录和测试文件
# ============================================================

[root@server1 ~]# mkdir -p /data/samba/share
# 创建默认共享目录（smb3 用户访问的目录）

[root@server1 ~]# mkdir -p /data/samba/smb1
# 创建 smb1 用户的私有目录

[root@server1 ~]# mkdir -p /data/samba/smb2
# 创建 smb2 用户的私有目录

[root@server1 ~]# touch /data/samba/share/share.txt
# 创建测试文件，smb3 用户访问时可见

[root@server1 ~]# touch /data/samba/smb1/smb1.txt
# 创建测试文件，smb1 用户访问时可见

[root@server1 ~]# touch /data/samba/smb2/smb2.txt
# 创建测试文件，smb2 用户访问时可见

[root@server1 ~]# tree /data/samba/
# tree : 以树形结构显示目录内容
# 输出目录结构，便于确认目录创建成功
/data/samba/
├── file.txt
├── server2.txt
├── share
│   └── share.txt
├── smb1
│   └── smb1.txt
└── smb2
    └── smb2.txt


# ============================================================
# 第六部分：设置目录权限
# ============================================================

[root@server1 ~]# chmod 777 -R /data/samba
# chmod 777 : 设置所有用户读写执行权限（测试用）
# -R        : 递归修改目录下所有文件和子目录
# 注意：此权限过于宽松，生产环境建议使用更严格的权限（如 2770）


# ============================================================
# 第七部分：为用户创建独立配置文件
# ============================================================

[root@server1 ~]# vim /etc/samba/conf.d/smb1
# 为 smb1 用户创建独立配置文件
# 该文件会覆盖主配置文件中的 [share] 定义

[share]
    path = /data/samba/smb1
    # 重定向：smb1 访问 share 共享时，实际访问 /data/samba/smb1
    writable = yes
    create mask = 0660
    browseable = yes

[root@server1 ~]# vim /etc/samba/conf.d/smb2
# 为 smb2 用户创建独立配置文件

[share]
    path = /data/samba/smb2
    # 重定向：smb2 访问 share 共享时，实际访问 /data/samba/smb2
    writable = yes
    create mask = 0660
    browseable = yes

# smb3 用户没有独立配置文件，将使用主配置中的 [share] 定义
# 即 smb3 访问 share 共享时，实际访问 /data/samba/share


# ============================================================
# 第八部分：重启 Samba 服务
# ============================================================

[root@server1 ~]# systemctl restart smb
# 重启 Samba 服务，使配置生效
# smb : SMB/CIFS 文件共享服务

[root@server1 ~]# systemctl restart nmb
# 重启 NetBIOS 名称解析服务
# nmb : NetBIOS 名称服务，用于在局域网中解析主机名
```

## 客户端

```shell
# ============================================================
# 测试一：smb1 用户访问测试
# ============================================================

[root@server2 ~]# smbclient //192.168.179.10/share -U smb1
# smbclient : Samba 客户端工具
# //192.168.179.10/share : 访问 server1 上的 share 共享
# -U smb1   : 使用 smb1 用户身份认证
# 作用：模拟客户端访问，验证 smb1 用户的访问权限和路径映射

Password for [SAMBA\smb1]:
# 输入 smb1 用户的 Samba 密码（之前设置的密码）
# 认证成功后进入 smbclient 交互界面

Try "help" to get a list of possible commands.
smb: \> ls
# ls : 列出当前共享目录内容

  .                                   D        0  Fri Mar 27 09:51:07 2026
  ..                                  D        0  Fri Mar 27 09:51:07 2026
  smb1.txt                            N        0  Fri Mar 27 09:51:07 2026
# 输出解析：
# smb1.txt : smb1 用户的测试文件
# 说明 smb1 访问 share 共享时，实际映射到 /data/samba/smb1 目录
# 配置文件生效：/etc/samba/conf.d/smb1 中的 path = /data/samba/smb1

		17756160 blocks of size 1024. 16259704 blocks available
smb: \> ^C
# ============================================================
# 测试二：smb2 用户访问测试
# ============================================================

[root@server2 ~]# smbclient //192.168.179.10/share -U smb2
# 使用 smb2 用户身份访问同一个共享名 share

Password for [SAMBA\smb2]:
# 输入 smb2 用户的 Samba 密码

Try "help" to get a list of possible commands.
smb: \> ls
# 列出共享内容

  .                                   D        0  Fri Mar 27 09:51:14 2026
  ..                                  D        0  Fri Mar 27 09:51:14 2026
  smb2.txt                            N        0  Fri Mar 27 09:51:14 2026
# 输出解析：
# smb2.txt : smb2 用户的测试文件
# 说明 smb2 访问 share 共享时，实际映射到 /data/samba/smb2 目录
# 配置文件生效：/etc/samba/conf.d/smb2 中的 path = /data/samba/smb2

		17756160 blocks of size 1024. 16259724 blocks available
smb: \> pwd
# pwd : 显示当前远程目录路径

Current directory is \\192.168.179.10\share\
# 注意：pwd 显示的仍然是 \share，但实际访问的是 /data/samba/smb2
# 这就是透明重定向效果：用户无感知路径映射

smb: \> ^C
# ============================================================
# 测试三：smb3 用户访问测试
# ============================================================

[root@server2 ~]# smbclient //192.168.179.10/share -U smb3
# 使用 smb3 用户身份访问同一个共享名 share
# smb3 没有独立的配置文件（/etc/samba/conf.d/smb3 不存在）

Password for [SAMBA\smb3]:
# 输入 smb3 用户的 Samba 密码

Try "help" to get a list of possible commands.
smb: \> ls
# 列出共享内容

  .                                   D        0  Fri Mar 27 09:50:41 2026
  ..                                  D        0  Fri Mar 27 09:50:41 2026
  share.txt                           N        0  Fri Mar 27 09:50:41 2026
# 输出解析：
# share.txt : 默认共享目录的测试文件
# 说明 smb3 没有独立配置文件时，使用主配置文件中的 [share] 定义
# 实际映射到 /data/samba/share 目录

		17756160 blocks of size 1024. 16259704 blocks available
smb: \> 

# 核心机制验证：
# 1. %U 变量生效：不同用户名加载不同配置文件
# 2. 配置覆盖生效：用户配置中的 [share] 覆盖主配置
# 3. 路径重定向生效：同一共享名指向不同物理目录
# 4. 透明隔离生效：用户只能看到自己的文件，无法看到其他用户目录

# 配置成功标志：
# ✅ smb1 只能看到 smb1.txt
# ✅ smb2 只能看到 smb2.txt
# ✅ smb3 只能看到 share.txt
# ✅ 三个用户访问同一共享名，获得完全隔离的私有空间
```

