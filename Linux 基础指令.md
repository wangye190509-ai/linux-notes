# Linux 基础指令



## 1. 基本文件和目录操作

- **ls**：列出目录内容
  - `ls -a`：显示隐藏文件（以 . 开头）
  - `ls -lha`：详细列表 + 人可读大小 + 隐藏文件（最常用组合）
  - `ls -d`：只显示目录本身信息，不列内容
  - `ls /`：查看根目录

- **cd**：切换目录
  - `cd` 或 `cd ~`：回到家目录
  - `cd ..`：返回上一级目录
  - `cd -`：返回上一次目录

- **pwd**：显示当前工作目录（Print Working Directory）

- **clear**：清屏（Ctrl + L 也可以）

## 2. echo 命令（输出文本）

- 基本用法：`echo "hello world"`

- **-e**：启用转义字符解释（\n 换行、\t Tab 等）

  ```bash
  echo -e "hello\nworld"   # 输出两行
  echo -e "hello\tworld"   # 输出带 Tab
  ```

- 重定向：

  - \>

    ：覆盖写入文件

    ```
    echo "hello" > file.txt
    ```

  - \>>

    ：追加写入文件

    ```
    echo "world" >> file.txt
    ```

- 重定向：

  - `>`：覆盖写入文件

    ```bash
    echo "hello" > file.txt
    ```

  - `>>`：追加写入文件

    ```bash
    echo "world" >> file.txt
    ```

## 3. 关机与重启

- **poweroff**：关机
  - `-n`：不缓存，直接关机
  - `-w`：仅记录关机日志，不执行
  - `-d`：不记录日志
  - `-f`：强制关机

- **reboot**：重启系统

## 4. 命令行快捷键（Ctrl 组合）

| `^C` | 终止前台运行的程序       |
| ---- | ------------------------ |
| `^D` | 退出 等价exit            |
| `^L` | 清屏                     |
| `^A` | 光标移动到命令行的最前端 |
| `^E` | 光标移动到命令行的后端   |
| `^U` | 删除光标前所有字符       |
| `^K` | 删除光标后所有字符       |
| `^R` | 搜索历史命令，利用关键词 |

## 5. history 命令（历史记录）

```bash
history [n]        # 显示最近 n 条命令（不加 n 显示全部）
history -c         # 清空当前 shell 的历史记录
history -a         # 把当前新增命令写入 ~/.bash_history
history -r         # 从 ~/.bash_history 读取历史到当前 shell
history -w         # 把当前历史写入 ~/.bash_history
```

## 6. help 和 man（查命令用法）

- **help**：查看 shell 内置命令用法（快速）

  ```bash
  help cd
  ```

- **man**：查看命令手册（最详细）

  ```bash
  man ls
  man 1 ls   # 指定章节 1（用户命令）
  ```

## 7. alias（别名）

- 查看所有别名：

  ```bash
  alias
  ```

- 设置临时别名（当前会话有效）

  ```bash
  alias wl='ip address'
  ```

- 设置永久别名（写入 /etc/bashrc 或 ~/.bashrc）

  ```bash
  echo "alias wl='ip address'" >> /etc/bashrc
  source /etc/bashrc   # 立即生效
  ```

- 删除别名：

  ```bash
  unalias wl
  ```

## 8. type（查看命令类型）

```bash
type -a ls     # 显示 ls 命令的真实类型和路径（别名、函数、内置、外部命令）
```

## 9. 环境变量（PATH）

```bash
echo $PATH     # 查看当前 PATH 环境变量（命令搜索路径）
```

**总结**：

- 文件看：ls -lha /  
- 切换：cd pwd  
- 清屏：clear 或 Ctrl+L  
- 输出：echo -e（转义） >（覆盖） >>（追加）  
- 关机：poweroff -f 强制  
- 快捷：Ctrl + C终止 D退出 L清屏 A前 E后 U删前 K删后 R搜历史  
- 历史：history -c 清空 -a 写入 -r 读取 -w 保存  
- 帮助：help（内置） man（手册）  
- 别名：alias 新='旧' → /etc/bashrc 永久  
- 类型：type -a 命令  
- 路径：echo $PATH


