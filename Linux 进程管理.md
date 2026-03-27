# 进程管理

## 进程基本概念

进程是一个在系统中运行的程序，是已启动的可执行程序的运行实例。

程序 vs 进程：

程序：二进制文件，是静态的

进程：是程序运行的过程，动态的，有生命周期及运行状态

## 进程类型

**守护进程**：在系统引导过程中启动，跟终端无关的进程（常用于后台服务）

**前台进程**：跟终端相关，通过终端启动的进程

## 进程的生命周期

**创建过程**：父进程复制自己的地址空间（**fork**）创建一个新的子进程结构。

- 每个新进程分配唯一的**进程 ID (PID)**，满足跟踪安全性需求。
- PID 和父进程 ID (**PPID**) 是子进程环境的元素。
- 任何进程都可以创建子进程，所有进程都是第一个系统进程的后代。

- 生命周期关键阶段：
  - fork：父进程复制地址空间
  - 分配 PID/PPID
  - 执行：子进程加载代码运行
  - 终止：子进程结束，向父进程发送信号；父进程用 wait() 回收资源
- 特殊进程：
  - **僵尸进程**：子进程结束但父进程未回收（状态 Z）
  - **孤儿进程**：父进程先结束，子进程被 systemd (PID=1) 收养

**复习要点**：所有进程的祖先是 systemd (PID=1)。记住 fork → PID/PPID → 执行 → 终止。

## systemd

Linux 系统中最新的初始化系统（init）

在 Rocky Linux 中的特点：所有进程都是 systemd 的后代

**复习要点**：systemd 是 PID=1 的进程，现代 RHEL/Rocky 默认用它。目标：加速启动、并行管理。

## systemd unit 类型

systemd 支持多种配置单元类型，用于管理不同功能

主要了解service

| Unit 类型 | 文件后缀   | 描述                   | 复习关键词                   |
| --------- | ---------- | ---------------------- | ---------------------------- |
| service   | .service   | 服务类（最常用）       | 系统服务（如 nginx.service） |
| target    | .target    | 服务组，模拟运行级别   | multi-user.target            |
| automount | .automount | 文件系统自动挂载点     | 动态挂载                     |
| device    | .device    | 内核识别的设备文件     | 硬件设备                     |
| mount     | .mount     | 文件系统挂载点         | /home.mount                  |
| path      | .path      | 文件或目录监控         | 变化触发                     |
| scope     | .scope     | 外部创建的进程组       | cgroup 进程组                |
| slice     | .slice     | 进程层级分组           | 资源限制                     |
| snapshot  | .snapshot  | 系统快照               | 备份恢复                     |
| socket    | .socket    | 套接字（延迟激活服务） | docker.socket                |
| swap      | .swap      | swap 设备              | 虚拟内存                     |
| timer     | .timer     | 定时任务（替代 cron）  | logrotate.timer              |

## unit 文件保存位置

- **优先级从高到低**（systemd 加载时按此顺序覆盖）：

| 目录                     | 描述                              | 复习关键词        |
| ------------------------ | --------------------------------- | ----------------- |
| /etc/systemd/system/     | systemctl enable 创建的 unit 文件 | 管理员自定义/覆盖 |
| /run/systemd/system/     | systemd 运行时创建的文件          | 临时运行时        |
| /usr/lib/systemd/system/ | RPM 包安装时分发的 unit 文件      | 系统默认          |

## systemctl 常用命令

**启动/停止/重启**：

- systemctl start name.service：启动服务
- systemctl stop name.service：停止服务
- systemctl restart name.service：重启服务（未启动会先启动）
- systemctl try-restart name.service：只重启正在运行的服务
- systemctl reload name.service：重载配置文件（不中断服务）

**状态检查**：

- systemctl status name.service：查看服务状态 + 最近日志
- systemctl is-active name.service：检查服务是否启动（返回 active/inactive）
- systemctl list-units --type service --all：显示所有服务状态

**自启管理**：

- systemctl enable name.service：启用开机自启
- systemctl disable name.service：停用自启
- systemctl is-enabled name.service：查看是否自启（返回 enabled/disabled）

**依赖 & 列表**：

- systemctl list-unit-files --type service：查看所有服务
- systemctl list-dependencies --after name.service：列出在指定服务之前启动的服务（依赖）
- systemctl list-dependencies --before name.service：列出在指定服务之后启动的服务（被依赖）

xxxxxxxxxx [root@localhost ~]# systemctl restart sshdbash

## ps

### ps aux

- **`a`**：显示所有用户的进程，包括其他用户的进程（不仅限于当前用户）。
- **`u`**：以用户为中心的格式显示进程信息，提供更多详细信息，如用户名、CPU 和内存使用率等。
- **`x`**：显示没有控制终端（TTY）的后台进程。

```bash
[root@localhost ~]# ps aux | head -n 5
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.3 128144  6656 ?        Ss   5月10   0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root          2  0.0  0.0      0     0 ?        S    5月10   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        S    5月10   0:00 [ksoftirqd/0]
root          5  0.0  0.0      0     0 ?        S<   5月10   0:00 [kworker/0:0H]
```

| **列名** | **说明**           |
| :------- | :----------------- |
| USER     | 进程拥有者         |
| PID      | 进程ID             |
| %CPU     | 占用的 CPU 使用率  |
| %MEM     | 占用的内存使用率   |
| VSZ      | 占用的虚拟内存大小 |
| RSS      | 占用的常驻内存大小 |
| TTY      | 执行的终端编号     |
| STAT     | 该进程的状态*      |
| START    | 进程开始时间       |
| TIME     | CPU使用时间        |
| COMMAND  | 所执行的命令       |

### ps -ef

```bash
-e：显示所有进程
-f：显示完整格式程序信息
[root@localhost ~]# ps -ef |head -n 10
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 5月10 ?       00:00:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root          2      0  0 5月10 ?       00:00:00 [kthreadd]
root          3      2  0 5月10 ?       00:00:00 [ksoftirqd/0]
root          5      2  0 5月10 ?       00:00:00 [kworker/0:0H]
```

| 列名  | 含义                   | 解释示例                                                  |
| ----- | ---------------------- | --------------------------------------------------------- |
| UID   | 进程所有者（真实用户） | root、wangye 等                                           |
| PID   | 进程 ID                | 唯一标识，每个进程不同                                    |
| PPID  | 父进程 ID              | 父进程的 PID，1 表示 systemd（init）                      |
| C     | CPU 使用率（旧式）     | 现在基本不用看（0~99），现代用 %CPU 更准                  |
| STIME | 启动时间               | 进程启动的日期/时间（如果是当天只显示时间）               |
| TTY   | 控制终端               | ? 表示无终端（守护进程），pts/0 表示伪终端（如 SSH 登录） |
| TIME  | 累计 CPU 时间          | 该进程已使用的 CPU 时间（格式：时:分:秒）                 |
| CMD   | 命令行                 | 启动进程的完整命令（可能被截断）                          |

### ps -efH

查看进程以层级形式，类似于pstree

```bash
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Mar05 ?        00:00:10 /usr/lib/systemd/systemd --switched-root --system --deserialize 17
root         2     1  0 Mar05 ?        00:00:00 [kthreadd]
root         3     2  0 Mar05 ?        00:00:00   [rcu_gp]
root       567     1  0 Mar05 ?        00:00:05 /usr/sbin/sshd -D
root      1234   567  0 10:15 pts/0    00:00:00   -bash
root      5678  1234  0 10:20 pts/0    00:00:00     ps -efH
```

### ps -auxf

按照父子进程显示服务的层级关系

```bash
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.5  22480  4800 ?        Ss   Mar05   0:10 /usr/lib/systemd/systemd --switched-root --system --deserialize 17
root           2  0.0  0.0      0     0 ?        S    Mar05   0:00  \_ [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   Mar05   0:00      \_ [rcu_gp]
root         567  0.0  1.2  56789 1234 ?        Ss   Mar05   0:05  \_ /usr/sbin/sshd -D
root        1234  0.0  0.8  34567  890 pts/0    Ss   10:15   0:00      \_ -bash
root        5678  0.0  0.3  12345  345 pts/0    R+   10:20   0:00          \_ ps -auxf
```



##### ps -efH 和 ps -auxf

ps -efH： “父子关系 + 缩进树” → 重点看 PPID 和启动时间，树用缩进显示，更简洁，适合分析进程依赖、启动顺序。

ps -auxf： “资源监控 + 树状” → 重点看 CPU/内存/状态，树用线条画，适合排查高负载、僵尸进程等。

### ps --sort

排序，通常与-aux组合

```shell
[root@localhost ~]# ps -aux --sort %cpu     # 递增
[root@localhost ~]# ps -aux --sort -%cpu    # 递减
```



## ps -o

自定义输出列（output format）



常用用法：

```bash
ps -axo 字段1,字段2,字段3,...
```

-a：显示所有进程（all processes，包括其他用户的）
-x：包括无控制终端的进程（无 TTY 的守护进程）
-o：自定义输出列（output format），后面跟逗号分隔的字段

```shell
[root@localhost ~]# ps -axo user,pid,ppid,%mem,%cpu,command --sort -%cpu
```

## 查看指定进程的pid

先用 systemctl status sshd（最推荐，显示状态 + 日志）

想看主进程 PID → cat /run/sshd.pid

想看所有 sshd 进程 → pgrep -l sshd 或 ps -aux | grep [s]shd

脚本自动化 → pidof sshd

## 查看进程树

pstree

## top

### 选项

-d: 改变显示的更新速度，或是在交互式指令列( interactive command)按 d或s

-n: 更新的次数，完成后将会退出 top

-c: 切换显示模式，共有两种模式，一是只显示程序的名称，另一种是显示完整的路径与名称
-S: 累积模式，会将己完成或消失的子行程 ( dead child process ) 的 CPU time 累积起来
-s: 安全模式，将交互式指令取消, 避免潜在的危机
-i: 不显示任何闲置 (idle) 或无用 (zombie) 的行程
-b: 显示模式，搭配 "n" 参数一起使用，可以用来将 top 的结果输出到文件内
-z：彩色

### 实例

- 显示进程信息，每个1秒钟刷新一次

```shell
[root@localhost ~]# top -d 1
```

- 更新两次后终止更新显示

```shell
[root@localhost ~]# top -n 2
```

- 显示完整命令

```shell
[root@localhost ~]# top -c
```

- 显示指定的进程信息

```shell
[root@localhost ~]# top -p 7097
```

- 查看指定用户的进程

```sh
[root@localhost ~]# top -d 1 -u user01
```

- 将2次top信息写入到文件

```sh
[root@localhost ~]# top -d 1 -b -n 2 > top.txt
```

###  交互模式快捷键

| 快捷键    | 功能                         | 说明                          |
| --------- | ---------------------------- | ----------------------------- |
| q / Esc   | 退出                         |                               |
| 1         | 显示每个 CPU 核心占用        | 多核服务器必看                |
| k         | 杀进程                       | 输入 PID → 选信号（默认 15）  |
| f         | 字段管理（添加/移除/排序列） | 自定义显示列                  |
| o / O     | 添加过滤条件                 | 如 `user=root` 只看 root 进程 |
| z         | 彩色高亮（开/关）            | 更易读                        |
| c         | 显示完整命令行（切换）       | 看参数                        |
| Shift + P | 按 %CPU 排序（默认）         |                               |
| Shift + M | 按 %MEM 排序                 |                               |
| Shift + T | 按 TIME+ 排序                | 累计 CPU 时间                 |
| Shift + N | 按 PID 排序                  |                               |
| d         | 改刷新间隔（默认 3 秒）      | 输入数字如 1                  |
| h / ?     | 帮助                         | 查看所有快捷键                |

```bash
top - 14:30:45 up 2 days,  4:12,  1 user,  load average: 0.15, 0.10, 0.08 
#      当前时间      系统运行时间     登录用户数     1/5/15分钟平均负载

Tasks: 120 total,   1 running, 119 sleeping,   0 stopped,   0 zombie 
#        总进程数       运行中        睡眠中         暂停中     僵尸进程（>0 就有问题）

%Cpu(s):  5.2 us,  1.8 sy,  0.0 ni, 92.8 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st 
# CPU 使用率：us(用户) sy(系统) ni(nice低优先) id(空闲) wa(等IO) hi(硬中断) si(软中断) st(虚拟化偷取)

MiB Mem :  16000.0 total,   9000.0 free,   3000.0 used,   4000.0 buff/cache 
# 物理内存：      总计             空闲            已用         缓存/缓冲（可回收）

MiB Swap:   2048.0 total,   2048.0 free,      0.0 used
# Swap：        总计             空闲        已用（>0 表示内存压力大）

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND            
  12345 root    20   0   567890  123456  7890 S 12.5  0.8   5:10.23 nginx            
  6789 wangye   20   0   345678   56789  2345 R  8.2  0.3   2:45.67 chrome           
  9012 root     20   0   123456   12345   890 S  0.0  0.1   0:15.12 sshd            

 - PID     进程ID - USER    进程拥有者 - PR      优先级（越低越优先） - NI      nice值（-20~19） - VIRT    虚拟内存总量（KB） - RES     实际物理内存占用（KB） - SHR     共享内存大小（KB） - S       进程状态（R运行 S睡眠 D不可中断 Z僵尸 T暂停） - %CPU    CPU占用百分比 - %MEM    内存占用百分比 - TIME+   累计CPU使用时间 - COMMAND 命令名称（按 c 可切换完整命令行）
```

## kill

- **-l** ：列出全部的信号名称
- **-s <信号名称或编号>**：指定要送出的信息
- **[程序]**：[程序]可以是程序的PID或是PGID

```bash
[root@localhost ~]# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX	
```

| **编号** | **信号名** | **作用**                                     |
| :------- | :--------- | :------------------------------------------- |
| 1        | SIGHUP     | 重新加载配置                                 |
| 2        | SIGINT     | 通过键盘 `ctrl+c` 打印捕获信息，程序继续运行 |
| 3        | SIGQUIT    | 通过键盘 `ctrl+\` 打印捕获信息，程序优雅退出 |
| 9        | SIGKILL    | 强制终止                                     |
| 15       | SIGTERM    | 终止(正常结束)                               |
| 18       | SIGCONT    | 继续                                         |
| 19       | SIGSTOP    | 停止                                         |
| 20       | SIGTSTP    | 暂停`ctrl z`                                 |

先 1（重载） → 再 15（温柔） → 最后 9（强制）

## pkill

pkill 用于杀死一个进程，与 kill 不同的是它会杀死指定名字的所有进程

- **name**： 进程名
- **-u**：指定用户名
- **-t**：指定终端



- 结束所有的sshd进程

```shell
[root@localhost ~]# pkill sshd
```

- 终止pts/2上所有进程，并结束pts/2

```shell
[root@localhost ~]# pkill -9 -t pts/2
```

- 查看远程登录用户，并踢出用户

```shell
[root@localhost ~]# w
 15:02:26 up  5:21,  2 users,  load average: 0.05, 0.03, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1                      14:41   20:34   0.02s  0.02s -bash
root     pts/0    192.168.175.1    14:42    2.00s  0.05s  0.01s w
user1    pts/1   192.168.175.1     14:40   20:34   0.02s  0.02s -bash
[root@localhost ~]# pkill -u user1
```

## nice值

**NICE值的范围：**-20到19，数值越小，优先级越高

### 查看进程的nice级别

#### 使用ps查看

```bash
[root@localhost ~]# ps -axo pid,command,nice --sort=nice
[root@localhost ~]# ps -axo pid,command,nice,cls --sort=-nice
```

#### 使用top查看

```sh
[root@localhost ~]# nice -n -20 vim & （&使vim在后台运行，避免阻塞）
[root@localhost ~]# ps -axo command,pid,nice |grep vim
```

##  

## PRI

PR 由 OS 内核动态调整，用户不能调整（PR 值越低，进程执行的优先级越高）。

```
PR(新) = PR(旧) + nice
```

#### 更改现有进程的nice级别

调整进程的优先级(Nice Level) （-20高) - - - 0 - - - (19低)

- 使用shell更改nice级别

```bash
[root@localhost ~]# vim &
[root@localhost ~]# ps -axo command,pid,nice |grep vim
vim                            1855   0
grep --color=auto vim          1888   0
[root@localhost ~]# renice -20 1855
1855 (process ID) old priority 0, new priority -20
[root@localhost ~]# ps -axo command,pid,nice |grep vim
vim                            1855 -20
grep --color=auto vim          1895   0
```

##  jobs

jobs 命令可以用来查看当前终端放入后台的任务

- "+"号代表最近一个放入后台的工作，也是工作恢复时默认恢复的工作
- "-"号代表倒数第二个放入后台的工作

```shell
[root@localhost ~]# jobs
[1]   已停止               top
[2]-  已停止               vi
[3]+  运行中               ping baidu.com > /dev/null &
```

## 将任务放入后台

Linux 命令放入后台的方法有两种：

- 在命令后面加入`空格 &`。使用这种方法放入后台的命令，在后台处于执行状态
- 命令执行过裎中按 Ctrl+Z 快捷键，命令在后台处于暂停状态



## 将任务恢复到前台

```shell
fg %工作号 （%可省略）
```



##  后台任务恢复到后台运行

```shell
bg %工作号
```

## nohup

将程序一直保持在后台运行

```shell
nohup 命令 &
```

