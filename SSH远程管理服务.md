# SSH远程管理服务

##  SSH 客户端使用

OpenSSH 服务提供我们 SSH 工具，该工具采用 SSH 协议来连接到远程主机上

## SSH 常用操作

通过 SSH 协议登录远程主机

```shell
[G:\~]$ ssh root@192.168.179.129


Connecting to 192.168.179.129:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Wed Mar 25 13:29:18 2026 from 192.168.179.1
[root@server1 ~]# 
```

指定连接远程主机的端口号

```shell
# -P 参数指定远程主机的端口号
[root@server1 ~]# ssh root@192.168.179.130 -P22
The authenticity of host '192.168.179.130 (192.168.179.130)' can't be established.
ED25519 key fingerprint is SHA256:KShYTlQRdtU3nPgi3aIes7zXzvzL2h/TBWvOPDisl08.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '192.168.179.130' (ED25519) to the list of known hosts.
root@192.168.179.130's password: 
Last login: Wed Mar 25 11:24:36 2026 from 192.168.179.1
[root@server2 ~]# 
```

不登陆到远程主机中，仅仅执行某个命令并返回结果

```shell
[root@server1 ~]# ssh root@192.168.179.130 cat /etc/hosts
root@192.168.179.130's password: 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

## SCP 远程文件传输

| 选项 | 说明                             |
| :--- | :------------------------------- |
| `-r` | 递归复制整个目录                 |
| `-P` | 指定端口（大写 P）               |
| `-p` | 保留文件属性（修改时间、权限等） |
| `-C` | 压缩传输                         |
| `-i` | 指定私钥文件                     |
| `-v` | 显示详细过程（调试用）           |

### 1. 上传文件到远程服务器

```shell
# server1上
[root@server1 ~]# echo "I am server1" > file
[root@server1 ~]# scp -P22 -r -p /root/file root@192.168.179.130:/tmp
root@192.168.179.130's password: 
file   
# server2上
[root@server2 ~]# cat /tmp/file
I am server1
```

### 2. 从远程服务器下载文件

```shell
# 先把文件都删了
[root@server1 ~]# rm -f fi*
[root@server1 ~]# ll
total 24
-rw-------. 1 root root 956 Mar  5 17:17 anaconda-ks.cfg
-rw-r--r--  1 root root  20 Mar 25 14:08 at.txt
-rwxr-xr-x  1 root root 336 Mar 25 14:28 config_network.sh
-rw-r--r--  1 root root 116 Mar 25 13:41 date.txt
-rw-r--r--  1 root root 309 Mar 25 14:28 ens160.txt
-rw-r--r--  1 root root   8 Mar 25 13:58 log.txt
# 从server2上下载file
[root@server1 ~]# scp -P22 -r -p root@192.168.179.130:/tmp/file /root/
root@192.168.179.130's password: 
file                                                                 100%   13     9.8KB/s   00:00    
[root@server1 ~]# ll
total 28
-rw-------. 1 root root 956 Mar  5 17:17 anaconda-ks.cfg
-rw-r--r--  1 root root  20 Mar 25 14:08 at.txt
-rwxr-xr-x  1 root root 336 Mar 25 14:28 config_network.sh
-rw-r--r--  1 root root 116 Mar 25 13:41 date.txt
-rw-r--r--  1 root root 309 Mar 25 14:28 ens160.txt
-rw-r--r--  1 root root  13 Mar 25 16:20 file  ##
-rw-r--r--  1 root root   8 Mar 25 13:58 log.txt

```

### SFTP

SSH客户端自带SFTP功能，可以直接通过FTP协议进行文件传递

```shell
[root@server1 ~]# sftp -oPort=22 root@192.168.179.130
root@192.168.179.130's password: 
Connected to 192.168.179.130.
sftp> pwd                        # 在根目录下
Remote working directory: /root
sftp> cd /tmp/
sftp> ls
file                                                                                                   
systemd-private-eaa190af24a44e1cb9bfa8e3eb3c025b-chronyd.service-9e1HY1                                
systemd-private-eaa190af24a44e1cb9bfa8e3eb3c025b-dbus-broker.service-vTOx8l                            
systemd-private-eaa190af24a44e1cb9bfa8e3eb3c025b-kdump.service-F28NXw                                  
systemd-private-eaa190af24a44e1cb9bfa8e3eb3c025b-systemd-logind.service-jJmz2E                         
sftp> get file                  # 下载文件
Fetching /tmp/file to file
file                                                                 100%   13    10.1KB/s   00:00    
sftp> 
```

| 命令             | 说明             |
| :--------------- | :--------------- |
| `ls`             | 列出远程目录内容 |
| `lls`            | 列出本地目录内容 |
| `cd`             | 切换远程目录     |
| `lcd`            | 切换本地目录     |
| `pwd`            | 显示远程当前目录 |
| `lpwd`           | 显示本地当前目录 |
| `get file`       | 下载文件         |
| `put file`       | 上传文件         |
| `get -r dir`     | 下载整个目录     |
| `put -r dir`     | 上传整个目录     |
| `rm file`        | 删除远程文件     |
| `mkdir dir`      | 创建远程目录     |
| `exit` 或 `quit` | 退出 SFTP        |

## 服务端配置文件

**常见配置项**

| 配置项                            | 说明                                |
| :-------------------------------- | :---------------------------------- |
| Port 22                           | 默认的sshd服务端口                  |
| ListenAddress 0.0.0.0             | 设定sshd服务器监听的IP地址          |
| Protocol 2                        | SSH协议的版本号                     |
| HostKey /etc/ssh/ssh_host_key     | SSH协议版本为1时，DES私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_rsa_key | SSH协议版本为2时，RSA私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_dsa_key | SSH协议版本为2时，DSA私钥存放的位置 |
| PermitRootLogin yes               | 设定是否允许root管理员直接登录      |
| StrictModes yes                   | 当远程用户的私钥改变时直接拒绝连接  |
| MaxAuthTries 6                    | 最大密码尝试次数                    |
| MaxSessions 10                    | 最大终端数                          |
| PasswordAuthentication yes        | 是否允许密码验证                    |
| PubkeyAuthentication yes          | 是否允许使用公钥进行身份验证        |

### SSH登录Linux虚拟机时密码被拒绝

1. 登录到虚拟机后，编辑SSH服务的主配置文件`/etc/ssh/sshd_config`：

   ```bash
   [root@localhost ~]# vim /etc/ssh/sshd_config
   ```

   

2. **检查并修改关键参数**
   在文件中找到以下几行，确保它们被设置成下面的样子（如果行首有`#`，需要去掉`#`来启用该配置）：

   - **允许密码登录**：找到 `#PasswordAuthentication yes`，确保它是 `PasswordAuthentication yes`
   - **允许root用户登录**：如果想直接用root登录，找到 `#PermitRootLogin prohibit-password` ，确保它是 `PermitRootLogin yes`。
   - **检查PAM认证**：找到 `UsePAM`，确保它是 `UsePAM yes`。这个选项与密码认证协同工作。

3. **重启SSH服务使配置生效**
   保存文件并退出编辑器后，必须重启SSH服务。根据你的Linux系统不同，命令略有差异：

   ```bash
   [root@localhost ~]# systemctl restart sshd
   ```



# 安全密钥验证

## 非对称加密

- **密钥对**：公钥（公开）+ 私钥（保密）
- **数学关联**：从私钥可推导公钥，反向几乎不可能
- **加密**：发送方使用接收方的公钥来加密信息，公钥加密的数据只能由私钥解密。
- **解密**：接收方使用自己的私钥来解密信息，私钥加密的数据只能被公钥解密。
- **两种模式：**

| 模式         | 操作                | 作用                                       |
| :----------- | :------------------ | :----------------------------------------- |
| **加密通信** | 公钥加密 → 私钥解密 | 保证机密性（只有持有私钥者能读）           |
| **数字签名** | 私钥签名 → 公钥验证 | 保证身份认证和完整性（持有私钥者才能生成） |

## SSH密钥对口令验证

### 通过密钥免密登录

#### 在linux里

在server2上免密登录server1

```shell
# server2上生成密钥对
[root@server2 ~]# ssh-keygen -t rsa   # 指定使用rsa算法生成密钥对文件
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:ZyWm//9t04JfFV0tlRqngaQEqmqBp8vdwpU2/nLlUP0 root@server2
The key's randomart image is:
+---[RSA 3072]----+
|       ......  .=|
|      . . .. o.o+|
|     .   .+ . *o.|
|.   .    + + o  .|
|....  . S o .   .|
| oo  = . =   E  .|
|.o. + . + .  .  o|
|o..o.o . . .. .o+|
|.. ...+.    .oo++|
+----[SHA256]-----+
[root@server2 ~]# cd /root/.ssh/
[root@server2 .ssh]# ll
total 8
-rw-------. 1 root root 2602 Mar 25 16:52 id_rsa
-rw-r--r--. 1 root root  566 Mar 25 16:52 id_rsa.pub
# 将公钥发给server1
[root@server2 .ssh]# ssh-copy-id -i id_rsa.pub 192.168.179.129
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "id_rsa.pub"
The authenticity of host '192.168.179.129 (192.168.179.129)' can't be established.
ED25519 key fingerprint is SHA256:KShYTlQRdtU3nPgi3aIes7zXzvzL2h/TBWvOPDisl08.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.179.129's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.179.129'"
and check to make sure that only the key(s) you wanted were added.
# server1上出现了一个存放密钥的文件 authorized_keys
[root@server1 ~]# cd /root/.ssh/
[root@server1 .ssh]# ll
total 12
-rw------- 1 root root 566 Mar 25 16:55 authorized_keys
-rw------- 1 root root 843 Mar 25 16:14 known_hosts
-rw-r--r-- 1 root root  97 Mar 25 16:14 known_hosts.old
# 修改ssh登陆文件
[root@server1 .ssh]# vim /etc/ssh/sshd_config

PasswordAuthentication no       # 禁止密码登录
PubkeyAuthentication yes        # 允许密钥登录
# 重启sshd服务
[root@server1 .ssh]# systemctl restart sshd
# 我们重新尝试ssh登录，发现只能密钥登录
[root@server2 .ssh]# ssh root@192.168.179.129
Last login: Wed Mar 25 16:08:11 2026 from 192.168.179.1
```

![image-20260325170301695](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260325170301695.png)



#### 在windows上

在cmd里免密登录server1

![image-20260325171356440](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260325171356440.png)

```shell
# 我们打开id_rsa.pub 将里面的公钥复制到server1的 /root/.ssh/authorized_keys 文件里
[root@server1 ~]# vim /root/.ssh/authorized_keys 

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDo9lVRTUBtf+DTdRZhHfEstVBaaRyQuaJOGEKybLSV566WxOtGGBnkVfhllrYRxOJeTYrWK9HKBgtBhXjt03xrq5zi23KrJDZt/LfU3QyBX/N2aqIhIwAPrdqfLUjRte+7DII+1VgjcckmoRkwyCz1X6kSJjeXyQRPv0MFJmsxC88nwbZLFjhaDrXjXDE4fSlPwrLyukGI8WQNugQm9N+Yoi2u7G6z32txH1O+XF2nbO5RGm4zgmaIZV1xyZ/SIM82FLHfZMCtaejsXcCnnA0y27wVok28NaqqQIbmu2fXGzH7PzRsN2U/FoNiwnz4c1odH3Atrb7HJd8s5bq6UVoXCO5RqKPZOt2cifvEdasF/covTvuZlKMLMNK9WxarmosbnXWDEu1Uv7+waChlR1735COo3UvAEz7eJStSqqvzoLgNtTJMls6Cdn7t81jvc5Fgcya/aKUGX17pcc1E1LS6AqzacVrQDyrrfKlrDPgPVcvfa0ipNAe9+N74u/pbIZE= w1664@LAPTOP-IDB0CO53
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCL2dgcQFqSlbKrlKzxeZKQZJ76Dvzv3KuTHxmIUQO93JolmJdhfkshYTJlFt10h77kEhURjP7nGaeezDYyfwv+Fizuy8xpaYP5fxxPG06Rx0l7qz1n2UFe3P2VO8K6J7v2nrs435i1e562VfvQMJZ0XlQAGzzbF/0zEyR30B0uwOjW/+ibm3s5pl0GhsnuFNzx2qSMl3k/OLb/OTReajFKsPQst1qx2i0yY+RSEtbl3ppSfgfh62QK7Qd68KBjxsWAmUOSWUo5EnwKx/LxUkJluJvu98n+bQDl3AxAW0GMtAGw0dF+okLmDYGhdpHDi0N3v80gZp+RQJvyrFeYGykamqL68/5PKsLoOYoYQ//kageZjzg35tjWdn7KuK+4Pq9Oj+0/yodSVuDKBOvrjHgKjJOcH+dQiKGi4u9C/IkPIB9mVxzG3zfqlqf0bLjdFANU7O9/2RCWWxbG7SIiXs7wvYW9zBcGJz1Q/nQeCX47djwjJmZf9TdVMxAz7WitQS0= root@server2
```

我们可以从 windows上连接ssh到server1了

```bash
C:\Users\w1664>ssh root@192.168.179.129
The authenticity of host '192.168.179.129 (192.168.179.129)' can't be established.
ED25519 key fingerprint is SHA256:KShYTlQRdtU3nPgi3aIes7zXzvzL2h/TBWvOPDisl08.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.179.129' (ED25519) to the list of known hosts.
Last login: Wed Mar 25 17:03:36 2026 from 192.168.179.130
[root@server1 ~]#
```

