# Bash 通配符和模式匹配

## 基本通配符（Glob Patterns）

### 星号 `*`
匹配**任意数量**的任意字符（包括零个字符）。

```bash
ls *.txt           # 所有 .txt 文件
rm temp*           # 所有以 temp 开头的文件
mv * backup/       # 当前目录所有文件
ls .*              # 所有隐藏文件
ls */*.txt         # 子目录中的 .txt 文件
```

### 问号 `?`
匹配**恰好一个**任意字符。

```bash
ls file?.txt       # file1.txt, fileA.txt, file_.txt 等
rm data???         # data001, dataABC 等（恰好3个字符）
ls ?               # 所有单字符文件名
ls ??              # 所有双字符文件名
```

### 方括号 `[...]`
匹配方括号内的**任意一个**字符。

```bash
ls file[123].txt   # file1.txt, file2.txt, file3.txt
ls [abc]*          # 以 a, b, c 开头的文件
ls [!abc]*         # 不以 a, b, c 开头的文件（取反）
ls [^abc]*         # 同上，另一种取反写法
ls [0-9]*          # 以数字开头的文件
ls [a-z]*          # 以小写字母开头的文件
ls [A-Za-z]*       # 以字母开头的文件
ls [0-9a-zA-Z]*    # 以字母或数字开头的文件
```

### 字符类 `[[:class:]]`
预定义的字符类别，在方括号内使用。

| 类别 | 含义 | 等价于 |
|------|------|--------|
| `[:alnum:]` | 字母和数字 | `[A-Za-z0-9]` |
| `[:alpha:]` | 字母 | `[A-Za-z]` |
| `[:digit:]` | 数字 | `[0-9]` |
| `[:lower:]` | 小写字母 | `[a-z]` |
| `[:upper:]` | 大写字母 | `[A-Z]` |
| `[:space:]` | 空白字符 | 空格、制表符、换行等 |
| `[:blank:]` | 空白字符 | 空格、制表符 |
| `[:punct:]` | 标点符号 | `! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ { | } ~` |
| `[:cntrl:]` | 控制字符 | ASCII 0-31 和 127 |
| `[:print:]` | 可打印字符 | 字母、数字、标点、空格 |
| `[:graph:]` | 可见字符 | 字母、数字、标点（不含空格） |

**示例：**
```bash
ls *[[:digit:]]*   # 包含数字的文件
ls *[[:upper:]]*   # 包含大写字母的文件
ls [[:lower:]]*    # 以小写字母开头的文件
```

---

## 扩展通配符（Extended Globs）

需要先启用：`shopt -s extglob`

| 模式 | 含义 |
|------|------|
| `*(pattern)` | 匹配 0 次或多次 |
| `+(pattern)` | 匹配 1 次或多次 |
| `?(pattern)` | 匹配 0 次或 1 次 |
| `@(pattern)` | 匹配恰好 1 次 |
| `!(pattern)` | 匹配不符合模式的 |

**示例：**
```bash
# 启用扩展通配符
shopt -s extglob

ls *.@(txt|md)         # 所有 .txt 或 .md 文件
ls !(backup*)          # 不以 backup 开头的文件
ls +(*.log|*.tmp)      # 所有 .log 或 .tmp 文件（至少一个）
ls ?(*.txt)            # 匹配空或 .txt 文件
rm !(important).txt    # 删除除 important.txt 外的所有 .txt 文件

# 匹配多个模式
ls @(*.jpg|*.png|*.gif)   # 所有图片文件
```

---

## 大括号展开（Brace Expansion）

大括号展开不是通配符匹配，而是在命令执行前进行文本展开。

### 逗号分隔列表

```bash
echo {a,b,c}              # a b c
cp file.txt{,.bak}        # 复制 file.txt 为 file.txt.bak
rm file.{log,tmp}         # 删除 file.log 和 file.tmp
mv {old,new}_name.txt     # old_name.txt -> new_name.txt
mkdir {src,dist}/{js,css} # 创建 src/js, src/css, dist/js, dist/css
```

### 数字范围

```bash
echo {1..5}               # 1 2 3 4 5
echo {01..10}             # 01 02 03 ... 10（补零）
echo {10..1}              # 10 9 8 ... 1（递减）
echo {1..10..2}           # 1 3 5 7 9（步长为2）
mkdir dir{1..5}           # 创建 dir1, dir2, dir3, dir4, dir5
touch file{1..3}.txt      # 创建 file1.txt, file2.txt, file3.txt
```

### 字母范围

```bash
echo {a..e}               # a b c d e
echo {A..Z}               # A B C ... Z
echo {z..a}               # z y x ... a（递减）
```

### 嵌套展开

```bash
echo {a,b}{1,2}           # a1 a2 b1 b2
echo {{a,b}{1,2},c}       # a1 a2 b1 b2 c
mkdir {src,dist}/{js,css,images}  # 创建6个目录
```

### 前缀和后缀

```bash
echo pre{A,B,C}suf        # preAsuf preBsuf preCsuf
cp file.{txt,bak}         # file.txt -> file.bak
```

---

## 常见组合模式

### 文件类型匹配

```bash
ls *.{jpg,jpeg,png,gif}        # 所有图片文件（展开后）
ls *.{{js,ts},map}             # .js, .ts, .map 文件
rm *.{log,tmp,bak}             # 删除所有临时文件
```

### 备份文件操作

```bash
cp file.txt{,.bak}             # 创建备份: file.txt -> file.txt.bak
cp file{,.bak}.txt             # file.txt -> file.bak.txt
mv file{.bak,}                 # 恢复备份: file.bak -> file
mv file.txt{.bak,}             # file.txt.bak -> file.txt
cp config{.default,}           # config.default -> config
```

### 批量重命名

```bash
mv {old,new}_name.txt          # old_name.txt -> new_name.txt
mv image{1,2}.png              # image1.png -> image2.png
mv {foo,bar}_{old,new}.txt     # foo_old.txt -> bar_new.txt
```

### 批量创建

```bash
mkdir -p project/{src,dist,tests}/{js,css}
touch README.md src/{index,app,utils}.js
```

---

## 转义通配符

当需要匹配字面字符而非通配时，需要转义。

### 转义方法

| 方法 | 示例 | 说明 |
|------|------|------|
| 反斜杠 `\` | `file\*.txt` | 转义单个字符 |
| 单引号 `'...'` | `'file*.txt'` | 转义所有内容 |
| 双引号 `"..."` | `"file*.txt"` | 转义通配符（但保留变量展开） |

**示例：**
```bash
# 查找包含 * 字符的文件
ls *'*'*                        # 方法1
ls *\**                         # 方法2

# 处理带问号的文件名
touch 'file?.txt'               # 创建带 ? 的文件
ls 'file?.txt'                  # 匹配该文件
ls file\?.txt                   # 同上

# 处理带方括号的文件名
touch 'file[1].txt'             # 创建文件
ls 'file[1].txt'                # 匹配该文件
ls file\[1\].txt                # 同上
```

---

## 通配符 vs 正则表达式

注意区分 shell 通配符和正则表达式：

| 功能 | Shell 通配符 | 正则表达式 |
|------|-------------|-----------|
| 任意字符 | `*` | `.*` |
| 单个字符 | `?` | `.` |
| 字符集 | `[abc]` | `[abc]` |
| 字符集取反 | `[!abc]` 或 `[^abc]` | `[^abc]` |
| 开头 | N/A | `^` |
| 结尾 | N/A | `$` |
| 分组 | N/A | `(...)` |
| 或 | N/A | `|` |
| 重复0-1次 | N/A | `?` |
| 重复0-n次 | N/A | `*` |
| 重复1-n次 | N/A | `+` |

**示例对比：**
```bash
# 通配符 - 文件名匹配
ls *.txt              # 所有 .txt 文件
ls file?.txt          # file1.txt, fileA.txt 等

# 正则表达式 - 文本搜索
grep ".*\.txt" files.list    # 匹配以 .txt 结尾的行
grep "file.\.txt" files.list # file后接一个字符再.txt
```

---

## 实用技巧

### 安全使用通配符

```bash
# 危险：可能误删
rm *.txt

# 安全：先查看
echo *.txt
# 确认后再删除
rm *.txt

# 使用引号防止空列表问题
rm *.txt 2>/dev/null || echo "没有匹配的文件"
```

### 处理特殊文件名

```bash
# 文件名含空格
ls "file name.txt"
touch "my document.txt"

# 文件名以 - 开头
touch -- -file.txt
rm -- -file.txt

# 文件名含换行符等特殊字符
find . -print0 | xargs -0 command
```

### 结合 find 命令

```bash
# find 使用通配符需要加引号
find . -name "*.txt"

# 忽略大小写
find . -iname "*.txt"

# 使用正则表达式
find . -regex ".*\.\(txt\|md\)"
```
