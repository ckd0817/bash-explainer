# Bash 操作符和特殊符号参考

## 管道和重定向

### 管道符 `|`
将前一命令的标准输出连接到后一命令的标准输入。

```bash
command1 | command2
```

**示例：**
```bash
cat file.txt | grep "error"    # 在文件内容中搜索 error
ps aux | grep nginx            # 在进程列表中查找 nginx
ls -la | head -10              # 只显示前10行
```

---

### 输出重定向 `>` `>>`

| 符号 | 含义 | 行为 |
|------|------|------|
| `>` | 覆盖写入 | 文件不存在则创建，存在则清空后写入 |
| `>>` | 追加写入 | 文件不存在则创建，存在则追加到末尾 |

**示例：**
```bash
echo "hello" > file.txt    # 写入（覆盖原有内容）
echo "world" >> file.txt   # 追加到文件末尾
ls > files.txt             # 保存目录列表
```

---

### 输入重定向 `<` `<<` `<<<`

| 符号 | 名称 | 含义 |
|------|------|------|
| `<` | 输入重定向 | 从文件读取输入 |
| `<<` | Here Document | 多行内联输入 |
| `<<<` | Here String | 单行内联输入 |

**示例：**
```bash
# 从文件读取输入
mysql -u root < backup.sql

# Here Document - 多行输入
cat << EOF
这是第一行
这是第二行
EOF

# Here String - 单行输入
grep "pattern" <<< "$variable"
```

---

### 文件描述符重定向

| 文件描述符 | 名称 | 说明 |
|------------|------|------|
| 0 | stdin | 标准输入 |
| 1 | stdout | 标准输出 |
| 2 | stderr | 标准错误 |

**常用组合：**
```bash
# 将标准错误重定向到文件
command 2> error.log

# 将标准输出和标准错误都重定向到同一文件
command > output.log 2>&1
command &> output.log        # 简写形式（bash）

# 丢弃输出
command > /dev/null 2>&1     # 丢弃所有输出
command &> /dev/null         # 简写形式

# 将标准错误重定向到标准输出（用于管道）
command 2>&1 | grep "error"

# 交换 stdout 和 stderr
command 3>&1 1>&2 2>&3
```

---

## 命令连接符

### 逻辑与 `&&`
前一命令**成功**（退出码为 0）才执行后一命令。

```bash
mkdir dir && cd dir
# 只有 mkdir 成功才会执行 cd

test -f file.txt && echo "文件存在"
# 文件存在才输出
```

### 逻辑或 `||`
前一命令**失败**（退出码非 0）才执行后一命令。

```bash
cd /nonexist || echo "目录不存在"
# cd 失败时输出提示

test -f file.txt || echo "文件不存在"
# 文件不存在才输出
```

### 组合使用
```bash
cd /path && ls || echo "无法进入目录"
# 成功则 ls，失败则提示

mkdir dir 2>/dev/null || true
# 创建目录，失败也不报错
```

### 分号 `;`
无论前一命令成败，都执行后一命令。

```bash
echo "first"; echo "second"
# 两个命令都会执行

cd /path; ls
# 无论 cd 是否成功，都会执行 ls
```

### 换行符
换行符等同于分号，但更清晰。

```bash
echo "first"
echo "second"
# 等同于 echo "first"; echo "second"
```

---

## 后台执行

### `&` - 后台运行
命令在后台执行，立即返回终端控制权。

```bash
long_task &
# 任务在后台运行，可以继续使用终端

sleep 100 &
# 后台等待100秒

nohup python script.py &
# 即使关闭终端也继续运行
```

### `wait` - 等待后台任务
```bash
sleep 5 &
sleep 3 &
wait
# 等待所有后台任务完成
```

### 作业控制
| 命令 | 含义 |
|------|------|
| `jobs` | 列出当前作业 |
| `fg %N` | 将作业 N 调到前台 |
| `bg %N` | 将作业 N 调到后台 |
| `Ctrl+Z` | 暂停当前前台作业 |
| `Ctrl+C` | 终止当前前台作业 |

---

## 变量和命令替换

### 变量引用 `$`

| 语法 | 含义 |
|------|------|
| `$VAR` | 变量值 |
| `${VAR}` | 变量值（明确边界） |
| `${VAR:-default}` | 变量为空时使用默认值 |
| `${VAR:=default}` | 变量为空时赋值并使用 |
| `${VAR:+value}` | 变量非空时使用 value |
| `${VAR:?error}` | 变量为空时报错退出 |
| `${#VAR}` | 变量长度（字符数） |
| `${VAR%pattern}` | 删除最短后缀匹配 |
| `${VAR%%pattern}` | 删除最长后缀匹配 |
| `${VAR#pattern}` | 删除最短前缀匹配 |
| `${VAR##pattern}` | 删除最长前缀匹配 |
| `${VAR/old/new}` | 替换第一个匹配 |
| `${VAR//old/new}` | 替换所有匹配 |

**示例：**
```bash
name="hello world"
echo $name           # hello world
echo ${#name}        # 11（长度）
echo ${name/world/bash}  # hello bash

path="/path/to/file.txt"
echo ${path##*/}     # file.txt（删除最长前缀）
echo ${path%/*}      # /path/to（删除最短后缀）
```

### 命令替换 `$()` 或 `` ` ` ``

将命令的输出作为值使用。

```bash
# 推荐语法
files=$(ls)
echo "当前目录: $(pwd)"
date_str=$(date +%Y%m%d)

# 旧语法（不推荐）
count=`ls | wc -l`

# 嵌套
result=$(echo $(date))
```

### 算术扩展 `$(( ))`

```bash
echo $((1 + 2))      # 3
echo $((5 * 3))      # 15
echo $((10 / 3))     # 3（整数除法）
echo $((10 % 3))     # 1（取模）
x=5
echo $((x + 1))      # 6
echo $((x++))        # 5（后增）
echo $x              # 6
```

---

## 引号

### 单引号 `'...'`
**原样保留**所有内容，不进行任何解释（变量、命令替换等都不生效）。

```bash
echo '$HOME'         # 输出: $HOME（不展开）
echo '*.txt'         # 输出: *.txt（不通配）
echo 'Hello\nWorld'  # 输出: Hello\nWorld（不解释转义）
```

### 双引号 `"..."`
保留大部分内容，但会**展开变量和命令替换**。

```bash
name="world"
echo "Hello $name"        # 输出: Hello world
echo "Path: $(pwd)"       # 输出: Path: /current/path
echo "Files: *.txt"       # 输出: Files: *.txt（通配符不展开）
echo "Line 1\nLine 2"     # 输出: Line 1\nLine 2（\n 不解释）
echo -e "Line 1\nLine 2"  # 输出两行（-e 解释转义）
```

### 反引号 `` `...` ``
旧式命令替换语法，等同于 `$(...)`，但不推荐使用（难以嵌套）。

```bash
echo `date`         # 等同于 echo $(date)
```

### 转义符 `\`
取消特殊字符的特殊含义。

```bash
echo "Quote: \"hello\""   # 输出: Quote: "hello"
echo "Path: C:\\Users"    # 输出: Path: C:\Users
echo \$HOME               # 输出: $HOME
```

**转义序列：**
| 序列 | 含义 |
|------|------|
| `\n` | 换行 |
| `\t` | 制表符 |
| `\\` | 反斜杠 |
| `\"` | 双引号 |
| `\'` | 单引号 |
| `\$` | 美元符号 |

---

## 括号

### 圆括号 `()`

| 用法 | 含义 |
|------|------|
| `(command)` | 在子 shell 中执行命令 |
| `$(command)` | 命令替换 |
| `(array)` | 数组定义 |
| `((expr))` | 算术运算 |

**示例：**
```bash
# 子 shell（变量修改不影响当前 shell）
(cd /tmp; pwd)

# 数组
arr=(one two three)

# 算术运算
((x = 5 + 3))
if ((x > 5)); then echo "big"; fi
```

### 方括号 `[]` `[[]]`

| 用法 | 含义 |
|------|------|
| `[ condition ]` | 测试条件（传统 test 命令） |
| `[[ condition ]]` | 测试条件（增强版，bash 特有） |
| `[abc]` | 字符集（glob 模式） |
| `array[index]` | 数组索引 |

**条件测试：**
```bash
# 传统语法
[ -f file.txt ] && echo "exists"
[ "$var" = "value" ]

# 增强语法（推荐）
[[ -f file.txt ]] && echo "exists"
[[ "$var" == "value" ]]
[[ "$var" == *"pattern"* ]]   # 模式匹配
```

### 大括号 `{}`

| 用法 | 含义 |
|------|------|
| `{ cmd; }` | 命令组（在当前 shell 执行） |
| `${var}` | 变量引用 |
| `{a,b,c}` | 大括号展开 |
| `{1..10}` | 范围展开 |

**命令组 vs 子 shell：**
```bash
# 命令组（当前 shell，变量修改保留）
{ x=1; echo $x; }

# 子 shell（变量修改不保留）
(x=1; echo $x;)
```

**大括号展开：**
```bash
echo {a,b,c}.txt         # a.txt b.txt c.txt
echo file{1..3}.log      # file1.log file2.log file3.log
mkdir dir{A,B,C}         # 创建 dirA dirB dirC
mv file.{txt,bak}        # 将 file.txt 重命名为 file.bak
cp file.txt{,.bak}       # 复制为 file.txt.bak
echo {01..10}            # 01 02 03 ... 10（补零）
echo {a..z}              # a b c ... z
```

---

## 其他特殊符号

### `#` - 注释
```bash
# 这是注释
echo "hello"  # 这也是注释
```

### `:` - 空命令
```bash
:                    # 什么都不做，返回成功
: ${var:=default}    # 设置默认值
while :; do          # 无限循环
    ...
done
```

### `!` - 历史扩展/取反
```bash
!!                   # 执行上一条命令
!n                   # 执行历史中第 n 条命令
!string              # 执行最近以 string 开头的命令

# 取反
[ ! -f file ]        # 文件不存在
! command            # 对退出码取反
```

### `~` - 主目录
```bash
~                    # 当前用户主目录
~/Documents          # 主目录下的 Documents
~username            # 指定用户的主目录
```

### `$_` - 上一个命令的最后一个参数
```bash
mkdir /path/to/dir
cd $_                # 进入刚创建的目录
```

### `$?` - 上一个命令的退出码
```bash
command
if [ $? -eq 0 ]; then
    echo "成功"
fi
```

### `$$` - 当前进程 PID
```bash
echo $$              # 输出当前 shell 的 PID
```

### `$!` - 后台进程 PID
```bash
sleep 100 &
echo $!              # 输出后台进程的 PID
```

### `$*` 和 `$@` - 所有位置参数
```bash
# $* - 所有参数作为一个字符串
# $@ - 所有参数作为独立字符串
for arg in "$@"; do
    echo "$arg"
done
```

### `$#` - 位置参数个数
```bash
echo "共有 $# 个参数"
```

### `$0` - 脚本名称
```bash
echo "脚本名: $0"
```
