# Linux 文件权限管理

## chown

chown 属主：属组 文件

```shell
[root@localhost ~]# chown user01:it file.txt
```

chown -R 修改目录

```shell
[root@localhost ~]# chown -R user01:it dir/*
```

## chmod

### 八进制语法

| #    | 权限           | rwx  | 二进制 |
| :--- | :------------- | :--- | :----- |
| 7    | 读 + 写 + 执行 | rwx  | 111    |
| 6    | 读 + 写        | rw-  | 110    |
| 5    | 读 + 执行      | r-x  | 101    |
| 4    | 只读           | r--  | 100    |
| 3    | 写 + 执行      | -wx  | 011    |
| 2    | 只写           | -w-  | 010    |
| 1    | 只执行         | --x  | 001    |
| 0    | 无             | ---  | 000    |

chmod 【u g a o】 【+ - =】 【rwx】文件路径

- `u`表示该文件的拥有者，`g`表示与该文件的拥有者属于同一个群体(group)者，`o`表示其他以外的人，`a`表示这三者皆是。

- `+`表示增加权限、`-`表示取消权限、`=`表示唯一设定权限。

- `r`表示可读取，`w`表示可写入，`x`表示可执行

  

- **-f**: 若该文件权限无法被更改也不要显示错误讯息
- **-v**: 显示权限变更的详细资料
- **-R**: 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)

## getfacl

文件访问控制列表（Access Control Lists，ACL）

-c 不显示注释表头

## setfacl

```shell
setfacl [-bkndRLP] { -m|-M|-x|-X ... } file ...
```

### 短选项[-bkndRLP]

##### -b 清除所有自定义ACL规则

setfacl  -b  文件或目录

##### -k 删除默认ACL规则

##### -d 设置默认ACL规则，新创建的目录或文件会继承

```shell
[root@localhost ~]# setfacl -m d:u:user1:rx /workdir
```

##### -R 递归处理

### 主要操作模式（{ -m|-M|-x|-X ... } 部分）

##### -m 修改或添加ACL 

setfacl    -m    u或g或o：名称：rwx     文件或目录

##### -M 批量修改ACL条目

##### -x 删除ACL

setfacl   -x   u或g或o：名称    文件或目录

##### -X批量删除ACL条目

### ACL 条目格式

- u:用户名:权限 或 u::权限（owner）
- g:组名:权限 或 g::权限（group）
- o:权限（other）
- m:权限（mask，限制最大权限）
- d: 前缀表示 default（默认 ACL）

权限用 rwx 或八进制

#### 

#### 

## mask有效权限

mask 权限，指的是用户或群组能拥有的最大 ACL 权限，也就是说，给用户或群组设定的 ACL 权限不能超过 mask 规定的权限范围，超出部分做无效处理。

```shell
[root@localhost ~]# setfacl -m m::rwx /workdir/newfile 
```

## 特殊权限

### suid sgid sticky

| 权限名称       | 数字位 | 符号标记 | 主要作用对象 | 对**文件**的效果                             | 对**目录**的效果                     | 典型系统示例                   | 常见权限数字 | ls -l 显示示例           |
| -------------- | ------ | -------- | ------------ | -------------------------------------------- | ------------------------------------ | ------------------------------ | ------------ | ------------------------ |
| **SUID**       | 4xxx   | s (小写) | 用户 (owner) | 执行时**有效用户** = 文件所有者（通常 root） | 无特殊效果                           | /usr/bin/passwd, /usr/bin/sudo | 4755         | -rwsr-xr-x               |
| **SGID**       | 2xxx   | s (小写) | 组 (group)   | 执行时**有效组** = 文件所属组                | 新文件/子目录**继承父目录所属组**    | /project/code (团队目录)       | 2775         | -rwxr-sr-x 或 drwxrwsr-x |
| **Sticky Bit** | 1xxx   | t (小写) | other        | 无实际效果（对文件无效）                     | 限制删除：只能删除**自己创建的文件** | /tmp, /var/tmp                 | 1777         | drwxrwxrwt               |

**SUID (Set User ID) – “借用户权限”**

- 口诀：**“SUID 借 owner 的身份证”**

- 当普通用户执行带有 SUID 的可执行文件时，进程的**有效用户 ID (EUID)** 临时变成**文件所有者**（通常是 root）。
- 经典例子：passwd 命令
  - 文件权限：-rwsr-xr-x root root
  - 普通用户执行 passwd → 进程以 root 身份运行 → 能修改 /etc/shadow（只有 root 可写）
- 注意：**只能用于可执行文件**，非可执行文件设置 SUID 会显示大写 **S**（无效）。

**SGID (Set Group ID) – “借组权限 + 组继承”**

- 口诀：**“SGID 借 group 的身份证 + 目录传家产”**
- 文件执行时：进程**有效组 ID (EGID)** 变成**文件所属组**。
- 目录上最常用：新创建的文件/子目录**自动继承父目录的所属组**（而不是创建者的主组）。
- 经典例子：团队共享目录
  - chmod 2775 /project/
  - user1（主组 user1）创建文件 → 文件组自动变成 project 组（继承 SGID）
- 注意：目录上 SGID 非常实用，文件上较少用（风险高）。

**Sticky Bit – “粘住文件不让删”**

- 口诀：**“Sticky Bit 粘住别人的文件”**
- 只对**目录**有效：即使目录是 777（全可写），也**只能删除自己创建的文件**。
- root 和文件所有者不受限制。
- 经典例子：/tmp 目录
  - drwxrwxrwt
  - 任何人可创建文件，但不能 rm 别人放的文件（防止恶意删除）。

## lsattr

查看文件或目录的扩展属性

lsattr file.txt
lsattr -d /dir/     # 只看目录本身（不递归）
lsattr -R /dir/     # 递归查看目录下所有文件

## chattr

```shell
chattr [-RV][+/-/=<属性>][文件或目录...]
```

-R 递归处理

-V 显示指令过程

### chattr属性

- **a**：让文件或目录仅供追加用途
- **i**：( immutable（不可变）)不得任意更动文件或目录
- **s**：安全删除文件或目录，删除时覆盖数据块（擦除内容，防恢复）
- **S**：即时更新文件或目录
- **u**：预防意外删除

chattr +i file      → 设置 immutable（不可变，root 也删不了） 

chattr +a logfile   → 设置 append only（只能追加，不能覆盖/删除） 

chattr -R +i dir/   → 递归锁死目录

恢复： chattr -i / -a / -R -i

常见组合： chattr +i /etc/passwd     # 防篡改 chattr +a /var/log/secure # 防日志清空



实战案例：

```shell
chattr +i /etc/resolv.conf
lsattr /etc/resolv.conf
# 输出：----i--------e-- /etc/resolv.conf

rm /etc/resolv.conf
# rm: cannot remove '/etc/resolv.conf': Operation not permitted
# 测试：即使 root 也无法删除或修改
```

```shell
touch /var/log/audit/myaudit.log
chattr +a /var/log/audit/myaudit.log
lsattr /var/log/audit/myaudit.log
# -----a-------e--

echo "new log" >> /var/log/audit/myaudit.log   # 成功
# 可以追加

echo "overwrite" > /var/log/audit/myaudit.log
# bash: /var/log/audit/myaudit.log: Operation not permitted
# 但不能覆盖或清空
```

## umask

```shell
umask
# 输出示例：0022（八进制）

umask -S   # 符号形式，更易读
# 输出：u=rwx,g=rx,o=rx
```

**计算公式**（按位减法）：

- 文件：666 - umask = 最终权限
- 目录：777 - umask = 最终权限

### 临时修改umask

```
[root@localhost ~]# umask 0000
[root@localhost ~]# touch file
[root@localhost ~]# mkdir dir
[root@localhost ~]# ll
总用量 4
drwxrwxrwx. 2 root root    6 4月  14 11:25 dir
-rw-rw-rw-. 1 root root    0 4月  14 11:25 file
```

### 当前shell永久

```shell
vim /etc/profile
```

```shell
# 示例：普通用户 umask 002，root 或系统用户 umask 022
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
    umask 002
else
    umask 022
fi
```

使当前 shell 立即生效（非常重要！）：

```shell
source /etc/profile
# 或
. /etc/profile
```

### 全系统新用户默认

```shell
vim /etc/login.defs
```

在最下面找到UMASK并修改

```shell
UMASK           077
```

**效果**：

- UMASK = 077 → 新用户家目录默认权限 = 777 - 077 = **700**（drwx------）
- UMASK = 022 → 默认 755（drwxr-xr-x）
- UMASK = 027 → 默认 750（drwxr-x---）

### 常用 umask 值推荐总结

| umask 值 | 新文件默认 | 新目录默认 | 家目录默认 | 推荐场景                     |
| -------- | ---------- | ---------- | ---------- | ---------------------------- |
| 0022     | 644        | 755        | 755        | 普通服务器，默认值           |
| 0027     | 640        | 750        | 750        | 生产环境，推荐（其他无权限） |
| 0077     | 600        | 700        | 700        | 高安全需求，个人服务器       |
| 0002     | 664        | 775        | 775        | 开发团队，组内可写           |
| 0000     | 666        | 777        | 777        | 只用于测试，极不安全         |
