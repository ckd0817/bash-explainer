# Bash 路径和目录符号

## 特殊目录符号

### `.` - 当前目录

```bash
ls .                       # 列出当前目录内容
./script.sh                # 执行当前目录的脚本
cp file.txt ./backup/      # 复制到当前目录的 backup 子目录
source ./config.sh         # 加载当前目录的配置文件
```

**常见用途：**
- 执行当前目录的脚本（必须加 `./`，因为当前目录通常不在 PATH 中）
- 明确指定当前目录
- 在路径中表示当前位置（如 `./src/../dist`）

### `..` - 上级目录

```bash
cd ..                      # 进入上级目录
ls ../                     # 列出上级目录内容
cp file.txt ../../         # 复制到上两级目录
vim ../config/settings.js  # 编辑上级目录的配置文件
```

**路径计算：**
```bash
# 当前目录: /home/user/projects/app
ls ../lib      # 查看 /home/user/projects/lib
ls ../../docs  # 查看 /home/user/docs
```

### `~` - 主目录（Home）

```bash
cd ~                       # 进入当前用户主目录
cd ~/Documents             # 进入主目录下的 Documents
ls ~                       # 列出主目录内容
vim ~/.bashrc              # 编辑配置文件
```

**指定用户的主目录：**
```bash
cd ~root                   # 进入 root 用户的主目录
ls ~username               # 列出指定用户的主目录
cp file.txt ~username/     # 复制到指定用户的主目录
```

### `/` - 根目录 / 路径分隔符

```bash
cd /                       # 进入根目录
ls /usr/local              # 列出 /usr/local 目录
cat /etc/passwd            # 查看 passwd 文件
```

**作为路径分隔符：**
- `/home/user/file.txt` - 绝对路径
- `dir/subdir/file.txt` - 相对路径

### `-` - 上一个工作目录

```bash
cd /path/to/dir1
cd /path/to/dir2
cd -                       # 返回 dir1

# 常用于在两个目录间快速切换
cd -                       # 再次执行返回 dir2
```

**其他用法：**
```bash
# 配合其他命令
git checkout -             # 切换到上一个分支
git merge -                # 合并上一个分支
```

---

## 路径类型

### 绝对路径
从根目录 `/` 开始的完整路径。

```bash
/home/user/Documents/file.txt
/usr/bin/python
/etc/nginx/nginx.conf
/var/log/syslog
```

**特点：**
- 以 `/` 开头
- 与当前目录无关
- 路径唯一，不会产生歧义

### 相对路径
相对于当前工作目录的路径。

```bash
./script.sh               # 当前目录
../config/settings.json   # 上级目录的子目录
Documents/report.pdf      # 当前目录的子目录
src/../dist/bundle.js     # 包含 . 和 .. 的路径
```

**特点：**
- 不以 `/` 开头
- 依赖于当前目录
- 可以包含 `.` 和 `..`

---

## 路径相关命令

### pwd - 显示当前目录

```bash
pwd                       # 输出当前工作目录的绝对路径
pwd -P                    # 显示物理路径（解析符号链接）
pwd -L                    # 显示逻辑路径（保留符号链接，默认）
```

**示例：**
```bash
cd /usr/local
pwd                       # /usr/local

ln -s /usr/local link
cd link
pwd                       # /usr/local/link（逻辑路径）
pwd -P                    # /usr/local（物理路径）
```

### dirname - 提取目录部分

```bash
dirname /path/to/file.txt    # 输出: /path/to
dirname file.txt             # 输出: .
dirname /path/to/dir/        # 输出: /path/to
dirname /                    # 输出: /
```

### basename - 提取文件名部分

```bash
basename /path/to/file.txt   # 输出: file.txt
basename /path/to/dir/       # 输出: dir
basename file.txt            # 输出: file.txt

# 去掉后缀
basename /path/to/file.txt .txt  # 输出: file
```

### realpath - 获取绝对路径

```bash
realpath file.txt             # 输出绝对路径
realpath ../file.txt          # 解析 .. 并输出绝对路径
realpath -s file.txt          # 不解析符号链接

# 相对路径计算
realpath --relative-to=/home/user /home/user/docs  # 输出: docs
realpath --relative-to=/home/user /var/log         # 输出: ../../var/log
```

### readlink - 读取符号链接

```bash
readlink link                 # 输出链接目标
readlink -f link              # 输出绝对路径（递归解析）
readlink -e link              # 同 -f，但目标必须存在
```

---

## 路径操作技巧

### 获取脚本所在目录

```bash
# 方法1：推荐
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

# 方法2：使用 BASH_SOURCE（支持 source）
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# 方法3：使用 realpath
SCRIPT_DIR="$(dirname "$(realpath "$0")")"
```

### 路径拼接

```bash
# 使用引号处理空格
path="/path/to"
file="my file.txt"
full_path="$path/$file"

# 去除多余的分隔符
full_path="${path%/}/${file#/}"
```

### 规范化路径

```bash
# 使用 realpath 规范化
realpath "./dir/../file.txt"  # 输出规范化的绝对路径

# 手动规范化
cd "./dir/../subdir" && pwd
```

### 安全创建目录

```bash
mkdir -p /path/to/deeply/nested/directory
# -p 选项：
# - 创建多级目录
# - 目录已存在时不报错
```

### 处理带空格的路径

```bash
# 必须使用引号
cd "/path/with spaces"
cp file.txt "/another path/destination"
vim "my document.txt"

# 在脚本中
path="/path/with spaces"
cd "$path"
```

---

## 导航技巧

### 目录栈

```bash
# pushd - 保存当前目录并切换
pushd /path/to/dir     # 切换到 /path/to/dir 并保存当前目录

# popd - 返回之前保存的目录
popd                   # 返回 pushd 之前的目录

# dirs - 显示目录栈
dirs                   # 显示所有保存的目录
dirs -v                # 带索引显示
dirs -c                # 清空目录栈

# 使用索引跳转
pushd +1               # 跳转到栈中第2个目录
```

### 快速导航别名

```bash
# 在 ~/.bashrc 中添加
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias ~='cd ~'
alias -- -='cd -'
```

### CDPATH

设置 cd 命令的搜索路径：

```bash
export CDPATH=/home/user/projects:/home/user/docs

# 现在 cd 可以直接跳转到这些目录下的子目录
cd myproject            # 会搜索 CDPATH 中的路径
```

---

## 环境变量

### PATH - 命令搜索路径

```bash
echo $PATH                    # 显示当前 PATH
echo $PATH | tr ':' '\n'      # 每行显示一个路径

# 添加新路径
export PATH=$PATH:/new/path           # 追加
export PATH=/new/path:$PATH           # 前置（优先级更高）

# 在 ~/.bashrc 中永久设置
echo 'export PATH=$PATH:/my/custom/path' >> ~/.bashrc
```

### 其他路径相关变量

| 变量 | 含义 | 示例 |
|------|------|------|
| `HOME` | 用户主目录 | `/home/user` |
| `PWD` | 当前工作目录 | `/home/user/projects` |
| `OLDPWD` | 上一个工作目录 | `/home/user` |
| `CDPATH` | cd 命令的搜索路径 | `:/home/user/projects` |

---

## 符号链接（Symlinks）

### 创建符号链接

```bash
ln -s target link_name        # 创建符号链接
ln -s /path/to/file link      # 链接到文件
ln -s /path/to/dir link       # 链接到目录
ln -sf target link            # 强制覆盖已存在的链接
```

### 硬链接 vs 符号链接

```bash
# 符号链接（软链接）
ln -s original softlink

# 硬链接
ln original hardlink

# 区别：
# - 符号链接：指向文件路径，可以跨文件系统
# - 硬链接：指向文件数据，必须同一文件系统
# - 删除原文件：符号链接失效，硬链接仍可访问
```

### 查看链接信息

```bash
ls -l link                    # 显示链接目标
readlink link                 # 输出链接目标
readlink -f link              # 输出绝对路径
stat link                     # 显示详细信息
```

---

## 常见路径模式

### 配置文件路径

```bash
~/.bashrc                     # Bash 配置
~/.zshrc                      # Zsh 配置
~/.gitconfig                  # Git 配置
~/.ssh/config                 # SSH 配置
/etc/                         # 系统配置目录
/usr/local/etc/               # 本地软件配置
```

### 常用系统路径

| 路径 | 用途 |
|------|------|
| `/bin` | 基本命令 |
| `/usr/bin` | 用户命令 |
| `/usr/local/bin` | 本地命令 |
| `/sbin` | 系统命令 |
| `/etc` | 配置文件 |
| `/var` | 可变数据（日志等） |
| `/tmp` | 临时文件 |
| `/home` | 用户主目录 |
| `/root` | root 用户主目录 |
| `/dev` | 设备文件 |
| `/proc` | 进程信息 |
| `/sys` | 系统信息 |
