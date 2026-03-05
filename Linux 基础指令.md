Linux 基础指令

ls：ls -a , ls -lha ,ls -d， ls /

cd

pwd

clear

echo -e "  "

poweroff -n不缓存

poweroff -w仅记录不执行

poweroff -d不记录

poweroff -f强制关机

reboot重启

| `^C` | 终止前台运行的程序       |
| ---- | ------------------------ |
| `^D` | 退出 等价exit            |
| `^L` | 清屏                     |
| `^A` | 光标移动到命令行的最前端 |
| `^E` | 光标移动到命令行的后端   |
| `^U` | 删除光标前所有字符       |
| `^K` | 删除光标后所有字符       |
| `^R` | 搜索历史命令，利用关键词 |

^等于ctrl

```shell
history [n]  n为数字，列出最近的n条命令
```

| `-c` | 将目前shell中的所有history命令消除                       |
| ---- | -------------------------------------------------------- |
| `-a` | 将目前新增的命令写入histfiles, 默认写入`~/.bash_history` |
| `-r` | 将histfiles内容读入到目前shell的history记忆中            |
| `-w` | 将目前history记忆的内容写入到histfiles                   |

help   和  man   可以查指令用法

alias 可以查看系统指令的别名

`alias 新别名='原命令'` 自定义别名（这种方式定义的别名**仅对当前终端会话有效**，关闭终端后别名就会消失）

```bash
# 为了让别名永久生效，可以将修改别名的命令写入 bashrc 文件，这个文件中的命令会在每次登陆命令行的时候执行
[root@localhost ~]# echo "alias wl='ip address'" >> /etc/bashrc
```

用unalias删除别名： unalias 新别名

`type -a 命令` 可查看命令的真实类型

环境变量echo $PATH