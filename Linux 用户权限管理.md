# Linux用户权限管理

## id

id -g显示用户的群组ID，-G 显示用户的附加群组ID，-n 显示用户的群组或附加群组名称，-r  显示用户真实的uid ， -u 显示用户的有效id

## uid 约定

Linux 操作系统会依据用户的 uid 数值来判定这个用户的角色，分别如下

- **0**：超级管理员，也就是root，在linux系统中拥有所有权力
- **1~999**：系统用户，系统用户往往是用来约束系统中的服务的
- **1000+**：普通用户，可以用来登陆和使用Linux操作系统

```shell
[root@localhost ~]# id
uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

## passwd文件

用于保存用户信息

```shell
[root@localhost ~]# head -n 1 /etc/passwd
root:x:0:0:root:/root:/bin/bash
# 这个格式为用户名:密码:uid:gid:描述:家目录:登陆后执行的命令
```

## shadow

格式中密码占位置太长了，所以使用x来替代，Linux系统会到shadow中查找x部分的的密码内容

## group

```shell
[root@localhost ~]# head -n 1 /etc/group
root:x:0:
# 这个格式是 组名:口令:组标识号:组内用户列表
```

## groupadd

-g 指定新建工作组的id，-r 创建系统工作组（系统工作组的组ID小于 500）

## groupdel

删除一个用户组

## groupmod

修改用户组

-n 修改组名，-g 修改组的GID，-o 允许重复使用GID

- 修改fd组的名字为finance

```shell
[root@localhost ~]# groupmod -n finance fd
[root@localhost ~]# tail -n 1 /etc/group
finance:x:4000:
```

## gpasswd

gpasswd 是 Linux 下工作组文件 /etc/group 和 /etc/gshadow 管理工具，用于将一个用户添加到组或者从组中删除

-a 添加用户到组，-d 从组删除用户

## useradd

添加用户

 -c 指定一段注释，-d 到目录，-g 到组 ，-G 到附加组，-s 指定用户登录的shell ，-r 创建系统用户（没有家目录）

- 添加一般用户

```shell
[root@localhost ~]# useradd user01
```

- 为添加的用户指定相应的用户组

```shell
[root@localhost ~]# useradd -g root user02
```

- 为新添加的用户指定home目录

```shell
[root@localhost ~]# useradd -d /home/test user03
```

- 建立一个不给登录的系统用户

```shell
[root@localhost ~]# useradd -s /sbin/nologin -r user04
```

- 查看用户描述信息

```bash
[root@localhost ~]# useradd -c "this is python_developmer user" python-developmer
[root@localhost ~]# cat /etc/passwd |grep python-developmer
python-developmer:x:1005:1005:this is python_developmer user:/home/python-developmer:/bin/bash
```

## userdel

-r 删除用户登入目录以及目录中所有文件

## usermod

-d 修改用户登录的目录，-u 修改uid， -m 移动旧的家目录到新的位置

#### 移动用户家目录

```shell
[root@localhost ~]# usermod -d /mnt/mountpoint/user1 -m user1
[root@localhost ~]# usermod -d /mnt/mountpoint/user2 -m user2
```

#### 参数说明：

- `-d /mnt/mountpoint/user1`：设置新家目录路径
- `-m`：**移动**旧家目录的内容到新位置
- `user1`：要修改的用户名



## su

切换用户（switch user /substitute user）

su 切换到root用户（保留当前环境），su - 切换到root并加载root环境，su -c 命令 仅执行单条命令不切换用户，su -s 指定shell替代默认shell

## passwd文件的shell

/bin/bash 进入命令模式， /sbin/nologin 禁止登录（有些用户是作为进程权限管理而存在的，不需要登录，如果提供登录的功能反而不安全），/bin/vi 进入vim

## passwd

修改用户密码

**交互式**

```shell
[root@localhost ~]# passwd user1
更改用户 user1 的密码 。
新的密码：           # 需要手动输入
重新输入新的密码：    # 需要再次输入
passwd：所有的身份验证令牌已经成功更新。[root@localhost ~]# echo "123456" | passwd --stdin user1
# 相当于自动输入了密码 "123456"

[root@localhost ~]# echo "P@ssw0rd" | passwd --stdin user2
# 自动设置密码为 P@ssw0rd
```

**非交互式**（用 --stdin）

```shell
[root@localhost ~]# echo 123456 | passwd --stdin test01
更改用户 test01 的密码 。
passwd：所有的身份验证令牌已经成功更新。
```



## login.defs文件

```shell
[root@localhost ~]# egrep -v '^[ ]*$|^#' /etc/login.defs
MAIL_DIR    /var/spool/mail  # 系统消息(邮件)文件夹
PASS_MAX_DAYS    99999        # 密码有效最大天数 
PASS_MIN_DAYS    0            # 密码有效最小天数
PASS_MIN_LEN    5            # 密码长度
PASS_WARN_AGE    7            # 密码失效警告倒计时
UID_MIN                  1000    # 用户UID最小1000
UID_MAX                 60000    # 用户UID最大60000
SYS_UID_MIN               201    # 系统用户UID最小201
SYS_UID_MAX               999    # 系统用户UID最大999
GID_MIN                  1000    # 用户组GID最小1000
GID_MAX                 60000    # 用户组GID最大60000
SYS_GID_MIN               201
SYS_GID_MAX               999
CREATE_HOME    yes              # 创建家目录
UMASK           077          # 创建文件/目录的权限掩码
USERGROUPS_ENAB yes          # 创建用户时同时生成组是  如果此处是no 创建的用户 会是gid=100(users)groups=100(users) 
ENCRYPT_METHOD SHA512        # 加密  方法  sha 512 这个方法生成的密码在/etc/shadow里面的第二列会以$6$开头
```

## chage( **change** + age)

```shell
[root@localhost ~]# chage -l root
最近一次密码修改时间                              ：从不
密码过期时间                                    ：从不
密码失效时间                                    ：从不
帐户过期时间                                    ：从不
两次改变密码之间相距的最小天数          ：0
两次改变密码之间相距的最大天数          ：99999
在密码过期之前警告的天数        ：7
```

- `-l` 看
- `-d 0` 强制改密
- `-M` 最大有效期
- `-m` 最小间隔
- `-W` 警告(W大写)
- `-I` 宽限（inactive）
- `-E` 账号过期

## sudo

通过useradd添加的用户，并不具备sudo权限。需要将用户加入wheel组

usermod -a -G wheel <用户名> （-a是append，追加）



sudo 的权限控制在 `/etc/sudoers` 文件

```shell
[root@localhost ~]# egrep -v '^[ ]*$|^#' /etc/sudoers
=====省略=====
root    ALL=(ALL)     ALL
%wheel    ALL=(ALL)    ALL
```



通过visudo来修改文件

```shell
# 用户 papi 在所有可能出现的主机上, 能够提权到 root 下执行 /bin/chown 和 /usr/sbin/useradd, 都不必输入密码
# 在具有 sudo 操作的用户下, 执行`sudo -l`可以查看到该用户被允许和被禁止运行的命令
papi ALL=(root) NOPASSWD: /bin/chown,/usr/sbin/useradd
```

```shell
# !/usr/sbin/fdisk 表示取消该命令
# 用户 papi 在所有可能出现的主机上, 能够运行目录 /usr/sbin 和 /sbin 下所有的程序, 但 fdisk 除外
papi ALL=/usr/sbin/,/sbin/,!/usr/sbin/fdisk
```

```shell
# 这个是改成60分钟才会需要再次输入密码，并且输入密码的时候会显示*号
[root@localhost ~]# visudo
Defaults env_reset,pwfeedback,timestamp_timeout=60
```

##  sudo 选项

- **-u**：以指定用户或 ID 运行命令(或编辑文件)，一般将sudo -u root写为sudo
- **-l**：显示出自己（执行 sudo 的使用者）的权限
- **-b**：将要执行的指令放在后台执行（复制大文件时避免阻塞终端）
- **-i**：切换到root的登录Shell，环境变量也同步为root。如果需要连续执行多条 root 命令，不用每条都加 sudo，可切换到 root 环境