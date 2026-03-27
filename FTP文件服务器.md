B/S  架构：网站-服务器模式

C/S  架构：客户端-服务器模式

# FTP (File Transfer Protocol)

FTP（File Transfer Protocol）是用于在计算机网络中进行文件传输的协议。它使用客户端-服务器模式，允许文件在客户端和服务器之间上传或下载。FTP 服务通常用于在不同主机间传输大容量文件，特别是在网络环境中需要频繁进行文件交换时。

## 工作原理

FTP 使用客户端与服务器之间的通信来传输文件。它通过两条连接来工作：一条用于命令传输（控制连接），另一条用于数据传输（数据连接）。

**控制连接端口**：21

- **功能**：FTP 的控制连接使用端口 21。这条连接用于客户端与服务器之间交换命令和响应。当你在 FTP 客户端中输入命令（例如 `LIST`、`RETR`、`STOR`）时，这些命令是通过端口 21 发送的。
- **数据流向**：控制连接是全双工的，客户端和服务器都可以发送消息，但所有数据传输操作（如文件上传、下载）都不会通过控制连接进行。

**数据连接端口**：20

- **功能**：端口 20 在 FTP 协议中用于数据传输连接，特别是在 **主动模式**（PORT 模式）下。它用于通过数据连接传输实际的文件数据。
- **数据流向**：当 FTP 客户端在主动模式下与服务器建立连接时，服务器通过端口 20 向客户端的指定端口发起数据连接。这条连接用于传输文件内容。

## FTP 工作模式

FTP 支持两种不同的模式：主动模式（PORT）和被动模式（PASV）。

### 主动模式（Active Mode）

客户端使用端口 21 与服务器建立控制连接，而数据连接则由服务器从端口 20 发起到客户端的指定端口。

适用于客户端有公网ip无防火墙设置，可以节约服务器的负载。

### 被动模式（Passive Mode）

客户端通过控制连接（端口 21）请求服务器提供一个可用端口，服务器会提供一个端口范围，客户端再通过这个端口与服务器建立数据连接。在被动模式下，服务器不会主动连接客户端，而是客户端主动与服务器建立数据连接。 

适用于客户端在NAT/防火墙，后避免服务器连不上客户端的数据端口。或者服务器限制端口范围。

## FTP 常用服务软件

许多操作系统和第三方软件都提供 FTP 服务的实现。常见的 FTP 服务器软件有：

- **vsftpd**（Very Secure FTP Daemon）：在 Linux 上广泛使用的高安全性 FTP 服务器。
- **ProFTPD**：功能丰富的 FTP 服务，支持多种认证机制。
- **Pure-FTPd**：简单易用的 FTP 服务器，适用于 Unix/Linux 系统。
- **FileZilla Server**：在 Windows 上使用的开源 FTP 服务器。

# vsftpd

- 软件包：vsftpd
- 服务类型：由Systemd启动的守护进程
- 配置单元：`/usr/lib/systemd/system/vsftpd.service`
- 守护进程：`/usr/sbin/vsftpd`
- 端口：`21(ftp)`,`20(ftp‐data)`
- 主配置文件：`/etc/vsftpd/vsftpd.conf`
- 用户访问控制配置文件：`/etc/vsftpd/ftpusers /etc/vsftpd/user_list`
- 日志文件：`/etc/logrotate.d/vsftpd`

## 配置文件参数

| 参数                        | 作用                                             |
| :-------------------------- | :----------------------------------------------- |
| listen=NO                   | 是否以独立运行的方式监听服务                     |
| listen_address=ip地址       | 设置要监听的IP地址                               |
| listen_port=21              | 设置FTP服务的监听端口                            |
| download_enable=YES         | 是否允许下载文件                                 |
| userlist_enable=YES         | 设置用户列表为"允许"                             |
| userlist_deny=YES           | 设置用户列表为"禁止"                             |
| max_clients=0               | 最大客户端连接数，0为不限制                      |
| max_per_ip=0                | 同一IP地址的最大连接数，0为不限制                |
| anonymous_enable=YES        | 是否允许匿名用户访问                             |
| anon_upload_enable=YES      | 是否允许匿名用户上传文件                         |
| anon_umask                  | 匿名用户上传文件的umask                          |
| anon_root=/var/ftp          | 匿名用户的ftp根目录                              |
| anon_mkdir_write_enable=YES | 是否允许匿名用户创建目录                         |
| anon_other_write_enable=YES | 是否开放匿名用户的其他写入权限（重命名、删除等） |
| anon_max_rate=0             | 匿名用户的最大传输速率，0为不限制                |
| local_enable=yes            | 是否允许本地用户登录                             |
| local_umask=022             | 本地用户上传文件的umask值                        |
| local_root=/vat/ftp         | 本地用户的ftp根目录                              |
| chroot_local_user=YES       | 是否将用户权限禁锢在ftp目录，以确保安全          |
| local_max_rate=0            | 本地用户的最大传输速率，0为不限制                |

## 部署和连接

### Linux上

- 第一种（ftp）

```shell
# 在服务端准备好ftp和文件
[root@server1 ~]# yum install -y vsftpd
[root@server1 ~]# touch  /var/ftp/file.txt   # /var/ftp/ 是默认的共享目录
[root@server1 ~]# echo "hello ftp server" > /var/ftp/file.txt
[root@server1 ~]# vim /etc/vsftpd/vsftpd.conf 
......
anonymous_enable=YES                         # 开启匿名用户登录
......
# 启用服务
[root@server1 ~]# systemctl start vsftpd
[root@server1 ~]# systemctl enable vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
# 此时虚拟机的防火墙和selinux已经默认设置关闭并拍摄快照，之后的操作不再提
```

```shell
[root@server2 ~]# yum install -y ftp
[root@server2 ~]# ftp 192.168.179.129
Connected to 192.168.179.129 (192.168.179.129).
220 (vsFTPd 3.0.5)
Name (192.168.179.129:root): anonymous   # 匿名用户anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> !pwd
/root
ftp> pwd
257 "/" is the current directory         # 这里的“/”是指“/var/ftp/”目录
ftp> !ls
anaconda-ks.cfg
ftp> ls
227 Entering Passive Mode (192,168,179,129,185,71).
150 Here comes the directory listing.
-rw-r--r--    1 0        0              17 Mar 26 00:43 file.txt
drwxr-xr-x    2 0        0               6 Jan 14 20:53 pub
226 Directory send OK.
ftp> quit
421 Timeout.
```

#### FTP命令

##### 一、本地操作（客户端侧）

| 命令         | 作用                                         |
| :----------- | :------------------------------------------- |
| `!`          | 临时切换到本地 Shell（输入 `exit` 返回 FTP） |
| `!ls`        | 列出本地当前目录文件（Linux）                |
| `!dir`       | 列出本地当前目录文件（Windows）              |
| `lcd [目录]` | 切换本地工作目录，如 `lcd /home`             |

------

##### 二、远程操作（服务器侧）

| 命令                 | 作用                             |
| :------------------- | :------------------------------- |
| `ls` 或 `dir`        | 列出服务器当前目录文件           |
| `cd [目录]`          | 切换服务器目录，如 `cd /pub`     |
| `pwd`                | 显示服务器当前目录路径           |
| `mkdir [目录名]`     | 在服务器创建目录                 |
| `rmdir [目录名]`     | 删除服务器**空目录**             |
| `delete [文件名]`    | 删除服务器单个文件               |
| `mdelete [文件列表]` | 批量删除文件，如 `mdelete *.txt` |

------

##### 三、文件传输

| 命令              | 作用                                                      |
| :---------------- | :-------------------------------------------------------- |
| `get [远程文件]`  | 下载单个文件，如 `get file.txt`                           |
| `mget [文件列表]` | 批量下载，如 `mget *.zip`                                 |
| `put [本地文件]`  | 上传单个文件，如 `put backup.tar`                         |
| `mput [文件列表]` | 批量上传，如 `mput *.jpg`                                 |
| `binary`          | 切换到二进制模式（传输图片、压缩包、可执行文件等）        |
| `ascii`           | 切换到文本模式（传输 `.txt`、`.html`、`.php` 等文本文件） |

- 第二种（lftp）

```shell
[root@server2 ~]# yum install -y lftp
[root@server2 ~]# lftp 192.168.179.129
lftp 192.168.179.129:~> ls
-rw-r--r--    1 0        0              17 Mar 26 00:43 file.txt
drwxr-xr-x    2 0        0               6 Jan 14 20:53 pub
lftp 192.168.179.129:/> quit
[root@server2 ~]# lftp 192.168.179.111   # 连接一个不存在的ip地址
lftp 192.168.179.111:~> ls
‘ls’ at 0 [connecting...]                # 数据连接无法建立
# lftp不需要登陆能直接连上，但我们只有输入命令才知道连接的对不对
```

**区别：**

- ftp 工具是一定要输入用户名称和密码的，登录成功或者失败会给出提示。
- lftp 不会直接给出登录成功或者失败的提示，需要输入 ls 工具才可以发现是否连接成功，优点在于连接更加方便

### FTP 名单列表

```shell
# 如果 userlist_deny=NO，则仅允许此文件中的用户访问
# 如果 userlist_deny=YES（默认值），则禁止此文件中的用户访问，并且不会提示输入密码
[root@server1 ftp]# cat /etc/vsftpd/user_list
# vsftpd userlist
# If userlist_deny=NO, only allow users in this file
# If userlist_deny=YES (default), never allow users in this file, and
# do not even prompt for a password.
# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
# for users that are denied.
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
```

###  Windows上

**第一种：**

可以在资源管理器或者运行窗口中输入 `ftp://192.168.179.129` 去连接到 ftp 服务器，如果我们想连接共享目录下面的具体某个目录，比如默认存在的pub目录，我们可以通过 `ftp://192.168.88.10/pub` 这样的方式来连接。

![image-20260326092248225](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260326092248225.png)

![image-20260326092257212](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260326092257212.png)

**第二种：**

可以打开 cmd 窗口，在 cmd 中通过 `ftp 192.168.179.129` 访问即可，跟 Linux 中使用 ftp 工具连接时的操作一致。

![image-20260326092409799](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260326092409799.png)

## 主被动切换测试

```shell
[root@server2 ~]# ftp 192.168.179.129
Connected to 192.168.179.129 (192.168.179.129).
220 (vsFTPd 3.0.5)
Name (192.168.179.129:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
227 Entering Passive Mode (192,168,179,129,28,230).
150 Here comes the directory listing.
-rw-r--r--    1 0        0              17 Mar 26 00:43 file.txt
drwxr-xr-x    2 0        0               6 Jan 14 20:53 pub
226 Directory send OK.
ftp> passive
Passive mode off.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0              17 Mar 26 00:43 file.txt
drwxr-xr-x    2 0        0               6 Jan 14 20:53 pub
226 Directory send OK.
ftp> 
221 Goodbye.
```

# 案例分析01

自定义匿名用户访问目录，能够上传和下载文件

```shell
# server1上
[root@server1 ~]# mkdir -p /share/pub
[root@server1 ~]# chmod 777 /share/pub/
[root@server1 ~]# chown ftp:ftp /share/pub/
[root@server1 ~]# ll /share/
total 0
drwxrwxrwx 2 ftp ftp 6 Mar 26 09:41 pub
[root@server1 ~]# echo "this is download..." > /share/pub/download
[root@server1 ~]# vim /etc/vsftpd/vsftpd.conf

anon_root=/share                 # 修改匿名登录目录
anon_upload_enable=YES           # 允许匿名上传
anon_mkdir_write_enable=YES      # 允许匿名用户创建目录
anon_other_write_enable=YES      # 开放匿名用户的其他写入权限（重命名、删除等）

[root@server1 ~]# systemctl restart vsftpd.service
# server2上
[root@server2 ~]# echo "this is upload..." > upload
[root@server2 ~]# ftp 192.168.179.129
Connected to 192.168.179.129 (192.168.179.129).
220 (vsFTPd 3.0.5)
Name (192.168.179.129:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> get download                                                 # 下载server1上准备好的文件
local: download remote: download
227 Entering Passive Mode (192,168,179,129,20,204).
150 Opening BINARY mode data connection for download (20 bytes).
226 Transfer complete.
20 bytes received in 0.000194 secs (103.09 Kbytes/sec)
ftp> put upload                                                   # 上传server2上准备好的文件
local: upload remote: upload
227 Entering Passive Mode (192,168,179,129,49,214).
150 Ok to send data.
226 Transfer complete.
18 bytes sent in 8.5e-05 secs (211.76 Kbytes/sec)
ftp> cd /pub
250 Directory successfully changed.
ftp> ls
227 Entering Passive Mode (192,168,179,129,234,36).
150 Here comes the directory listing.
-rw-r--r--    1 0        0              20 Mar 26 01:48 download
-rw-------    1 14       50             18 Mar 26 01:50 upload
226 Directory send OK.
ftp> delete upload                                                # 删除刚上传好的文件
250 Delete operation successful.
ftp> mkdir dir1                                                   # 创建目录
257 "/pub/dir1" created
ftp> 
221 Goodbye.
```

# 案例分析02

使用本地用户成功登录并下载上传文件

```shell
# 服务器
[root@server1 ~]# useradd user01
[root@server1 ~]# echo 123 | passwd --stdin user01
Changing password for user user01.
passwd: all authentication tokens updated successfully.
[root@server1 ~]# su - user01
[user01@server1 ~]$ echo "user01 test download..." > download.txt
[user01@server1 ~]$ 
logout
# 客户端
[root@server2 ~]# ftp 192.168.179.129 (或者用lftp -u user01，123 192.168.179.129)
Connected to 192.168.179.129 (192.168.179.129).
220 (vsFTPd 3.0.5)
Name (192.168.179.129:root): user01   # 登录user01
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,179,129,119,217).
150 Here comes the directory listing.
-rw-r--r--    1 1000     1000           24 Mar 26 02:15 download.txt
226 Directory send OK. 
ftp> get download.txt                 # 下载文件
local: download.txt remote: download.txt
227 Entering Passive Mode (192,168,179,129,78,172).
150 Opening BINARY mode data connection for download.txt (24 bytes).
226 Transfer complete.
24 bytes received in 3e-05 secs (800.00 Kbytes/sec)
ftp> put upload                       # 上传文件
local: upload remote: upload
227 Entering Passive Mode (192,168,179,129,182,111).
150 Ok to send data.
226 Transfer complete.
18 bytes sent in 3e-05 secs (600.00 Kbytes/sec)
ftp> 
221 Goodbye.
[root@server2 ~]# cat download.txt
user01 test download...

# 服务器
[root@server1 ~]# su - user01
Last login: Thu Mar 26 10:14:27 CST 2026 on pts/0
[user01@server1 ~]$ cat upload
this is upload...
```

# 案例分析03

虚拟用户访问控制：虚拟用户为 eagleslab001 和 eagleslab002，服务端本地代理用户为 vuser；eagleslab001 默认访问 `/home/vsftpd/eagleslab001` 且能够创建/删除/上传/下载文件，eagleslab002 不做额外配置。

```shell
# 创建用于进行FTP认证的用户数据库，其中奇数行为用户名，偶数行为密码
[root@server1 ~]# cd /etc/vsftpd/
[root@server1 vsftpd]# vim vuser.list
eagleslab001
00123456
eagleslab002
00234567

[root@server1 vsftpd]# cd
[root@server1 ~]# yum install -y libdb-utils
# HASH哈希工具(db_load)：将明文信息转为密文
[root@server1 ~]# db_load -T -t hash -f /etc/vsftpd/vuser.list /etc/vsftpd/vuser.db
# 查看文件描述以及修改权限 & 删除明文信息
[root@server1 vsftpd]# file vuser.db
vuser.db: Berkeley DB (Hash, version 9, native byte-order)
[root@server1 vsftpd]# cat vuser.db
a      ɧ 
𨃉§234567eagleslab002 
  𗜼0эh^00123456eagleslab001[root@server1 vsftpd]# 
[root@server1 vsftpd]# chmod 600 vuser.db
[root@server1 vsftpd]# rm -rf vuser.list 
[root@server1 vsftpd]# cd
# 创建虚拟用户的本地代理用户 & 用户目录
[root@server1 ~]# useradd -d /home/vsftpd -s /sbin/nologin vuser
[root@server1 ~]# tail -1 /etc/passwd
vuser:x:1001:1001::/home/vsftpd:/sbin/nologin
[root@server1 ~]# mkdir -pv /home/vsftpd/eagleslab{001,002}
mkdir: created directory '/home/vsftpd/eagleslab001'
mkdir: created directory '/home/vsftpd/eagleslab002'
[root@server1 ~]# chmod -Rf 755 /home/vsftpd/
# 新建用于虚拟用户认证的PAM文件
[root@server1 ~]# vim /etc/pam.d/vsftpd_vuser
auth required pam_userdb.so db=/etc/vsftpd/vuser
account required pam_userdb.so db=/etc/vsftpd/vuser 

# 更新配置文件
[root@server1 ~]# mkdir -p /etc/vsftp/conf.d
[root@server1 ~]# vim /etc/vsftpd/vsftpd.conf
pam_service_name=vsftpd_vuser
userlist_enable=YES
guest_enable=YES
guest_username=vuser
allow_writeable_chroot=YES
chroot_local_user=YES
user_config_dir=/etc/vsftpd/conf.d
anon_root=/share
anon_upload_enable=YES
anon_other_write_enable=YES

# 针对 eagleslabl001 设置不同权限
[root@server1 ~]# chmod  775 /home/vsftpd/eagleslab001
[root@server1 ~]# vim /etc/vsftpd/conf.d/eagleslab001 
local_root=/home/vsftpd/eagleslab001
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES

# 准备文件
[root@server1 ~]# echo "eagleslab001 test download" > /home/vsftpd/eagleslab001/download001.txt
[root@server1 ~]# echo "eagleslab002 test download" > /home/vsftpd/eagleslab002/download002.txt
# server2上连接
[root@server2 ~]# ftp 192.168.179.129
Connected to 192.168.179.129 (192.168.179.129).
220 (vsFTPd 3.0.5)
Name (192.168.179.129:root): eagleslab001
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,179,129,162,145).
150 Here comes the directory listing.
-rw-r--r--    1 0        0              27 Mar 26 03:06 download001.txt
226 Directory send OK.
ftp> get download001.txt
local: download001.txt remote: download001.txt
227 Entering Passive Mode (192,168,179,129,228,96).
150 Opening BINARY mode data connection for download001.txt (27 bytes).
226 Transfer complete.
27 bytes received in 4e-05 secs (675.00 Kbytes/sec)
ftp> put upload
local: upload remote: upload
227 Entering Passive Mode (192,168,179,129,62,249).
150 Ok to send data.
226 Transfer complete.
18 bytes sent in 5.3e-05 secs (339.62 Kbytes/sec)
ftp> 
421 Timeout.
```

