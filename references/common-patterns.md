# 常见 Bash 命令模式

## 文件查找和处理

### 按名称查找文件

```bash
# 基本查找
find . -name "*.txt"              # 查找所有 .txt 文件
find . -iname "*.txt"             # 忽略大小写
find /path -name "file*"          # 在指定路径查找

# 组合条件
find . -name "*.js" -type f       # 只找文件
find . -name "*.js" -type d       # 只找目录
find . \( -name "*.js" -o -name "*.ts" \)  # 或条件

# 排除目录
find . -name "*.js" -not -path "./node_modules/*"
find . -name "*.js" -path "./node_modules" -prune -o -name "*.js" -print
```

### 按时间查找

```bash
# 修改时间（天）
find . -mtime -7                  # 7天内修改
find . -mtime +30                 # 超过30天
find . -mtime 1                   # 恰好1天前

# 访问时间
find . -atime -7                  # 7天内访问

# 创建时间
find . -ctime -7                  # 7天内创建

# 分钟级别
find . -mmin -60                  # 60分钟内修改
```

### 按大小查找

```bash
find . -size +100M                # 大于100MB
find . -size -1k                  # 小于1KB
find . -size 0                    # 空文件
find . -empty                     # 空文件或目录
find . -empty -type f             # 只找空文件
find . -empty -type d             # 只找空目录
```

### 查找并执行命令

```bash
# 对每个文件执行（慢但安全）
find . -name "*.txt" -exec cat {} \;

# 批量执行（快）
find . -name "*.txt" -exec cat {} +

# 删除文件
find . -name "*.tmp" -delete
find . -name "*.log" -exec rm {} \;

# 交互式
find . -name "*.tmp" -exec rm -i {} \;

# 使用 xargs（更快）
find . -name "*.txt" -print0 | xargs -0 cat
find . -name "*.txt" | xargs grep "pattern"
```

---

## 文本处理管道

### 日志分析

```bash
# 统计错误数量
grep -c "ERROR" app.log

# 查找唯一 IP
grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" access.log | sort | uniq

# 最频繁的错误
grep "ERROR" app.log | awk '{print $4}' | sort | uniq -c | sort -rn | head -10

# 按时间范围过滤
awk '/2024-01-15 10:00/,/2024-01-15 11:00/' app.log

# 实时监控
tail -f app.log | grep --color "ERROR"

# 多关键词高亮
tail -f app.log | grep -E "ERROR|WARNING|INFO" --color=always | grep -E "ERROR|WARNING|INFO"
```

### 文件内容统计

```bash
# 统计行数
wc -l file.txt
cat file.txt | wc -l

# 统计多个文件
wc -l *.txt

# 统计代码行数（排除空行和注释）
grep -v '^$\|^\s*#' script.sh | wc -l

# 递归统计
find . -name "*.js" -exec wc -l {} \; | awk '{sum+=$1} END {print sum}'

# 按文件统计
find . -name "*.js" -exec wc -l {} + | sort -n
```

### 文本替换

```bash
# 替换并输出
sed 's/old/new/g' file.txt

# 就地替换
sed -i 's/old/new/g' file.txt

# 批量替换
sed -i 's/old/new/g' *.txt

# 替换特定行
sed '10s/old/new/' file.txt          # 第10行
sed '10,20s/old/new/g' file.txt      # 第10-20行

# 使用正则
sed -E 's/[0-9]+/NUMBER/g' file.txt

# 删除行
sed '/pattern/d' file.txt            # 删除匹配行
sed '5d' file.txt                    # 删除第5行
sed '1,5d' file.txt                  # 删除第1-5行
```

### 列处理

```bash
# 提取列
awk '{print $2}' file.txt            # 第2列
cut -d, -f2 file.csv                 # CSV 第2列
cut -d: -f1,3 /etc/passwd            # 第1和第3列

# 按列过滤
awk '$3 > 100' file.txt              # 第3列大于100
awk '$1 == "test"' file.txt          # 第1列等于 test

# 列计算
awk '{sum+=$1} END {print sum}' file.txt
awk '{print $1, $2, $1+$2}' file.txt
```

---

## 批量操作

### 批量重命名

```bash
# 使用 rename（推荐）
rename 's/.txt/.md/' *.txt           # .txt -> .md
rename 's/^/prefix_/' *.txt          # 添加前缀
rename 's/$/_backup/' *.txt          # 添加后缀
rename 'y/a-z/A-Z/' *.txt            # 转大写

# 使用循环
for f in *.txt; do
    mv "$f" "${f%.txt}.md"
done

# 使用 mv 和大括号
mv file.{txt,md}                     # file.txt -> file.md
```

### 批量处理文件

```bash
# 简单循环
for file in *.jpg; do
    convert "$file" "${file%.jpg}_resized.jpg"
done

# 使用 find 和 xargs
find . -name "*.png" -print0 | xargs -0 -I {} convert {} resized/{}

# 并行处理（使用 GNU parallel）
find . -name "*.png" | parallel convert {} resized/{}

# 限制并发数
find . -name "*.png" | xargs -P 4 -I {} convert {} resized/{}
```

### 批量权限修改

```bash
# 目录设为 755，文件设为 644
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;

# 或者
chmod 755 $(find . -type d)
chmod 644 $(find . -type f)
```

---

## 系统监控

### 进程管理

```bash
# 查找进程
ps aux | grep nginx
pgrep nginx

# 查找并杀进程
ps aux | grep nginx | grep -v grep | awk '{print $2}' | xargs kill
pkill nginx
killall nginx

# 按用户查找
ps -u username

# 进程树
ps --forest
pstree

# 监控
top -b -n 1 | head -20              # 快照
htop                                 # 交互式
```

### 资源监控

```bash
# CPU 和内存使用
ps aux --sort=-%cpu | head -10       # CPU 占用最高
ps aux --sort=-%mem | head -10       # 内存占用最高

# 磁盘空间
df -h                                # 文件系统空间
du -sh *                             # 目录大小
du -h --max-depth=1                  # 一层深度

# 查找大文件
du -ah /path | sort -rh | head -20
find / -size +100M 2>/dev/null

# 内存信息
free -h
cat /proc/meminfo
```

### 网络监控

```bash
# 端口监听
netstat -tlnp
ss -tlnp

# 连接数统计
netstat -an | grep :80 | wc -l
ss -s

# 实时流量
iftop
nethogs
```

---

## 安全操作

### 安全删除

```bash
# 先预览
find . -name "*.tmp" -print

# 确认后删除
find . -name "*.tmp" -delete

# 交互式
rm -i *.txt

# 使用 trash-cli（需要安装）
trash-put *.txt
trash-list
trash-restore
```

### 备份操作

```bash
# 带时间戳的备份
cp file.txt "file.txt.$(date +%Y%m%d_%H%M%S).bak"

# 压缩备份
tar -czvf "backup_$(date +%Y%m%d).tar.gz" /path/to/backup

# 增量同步
rsync -av --delete source/ destination/

# 远程备份
rsync -avz -e ssh source/ user@remote:/backup/
```

### 安全别名

```bash
# 在 ~/.bashrc 中添加
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias ln='ln -i'

# 危险命令保护
alias chmod='chmod --preserve-root'
alias chown='chown --preserve-root'
```

---

## 条件判断

### 文件检查

```bash
# 存在性
[ -e file ]           # 存在（文件或目录）
[ -f file ]           # 存在且是普通文件
[ -d dir ]            # 存在且是目录
[ -L link ]           # 存在且是符号链接

# 权限
[ -r file ]           # 可读
[ -w file ]           # 可写
[ -x file ]           # 可执行

# 内容
[ -s file ]           # 非空文件
[ file1 -nt file2 ]   # file1 比 file2 新
[ file1 -ot file2 ]   # file1 比 file2 旧

# 示例
[ -f config.json ] || echo "配置文件不存在"
[ -d backup ] || mkdir backup
```

### 字符串检查

```bash
[ -z "$var" ]         # 为空
[ -n "$var" ]         # 非空
[ "$var" = "value" ]  # 等于
[ "$var" != "value" ] # 不等于

# 使用双括号（推荐）
[[ -z "$var" ]]
[[ "$var" == "value" ]]
[[ "$var" == *"pattern"* ]]  # 模式匹配
[[ "$var" =~ regex ]]        # 正则匹配
```

### 数字比较

```bash
[ $num -eq 10 ]       # 等于
[ $num -ne 10 ]       # 不等于
[ $num -lt 10 ]       # 小于
[ $num -le 10 ]       # 小于等于
[ $num -gt 10 ]       # 大于
[ $num -ge 10 ]       # 大于等于

# 使用双括号（推荐）
(( num == 10 ))
(( num < 10 ))
(( num >= 10 ))
```

---

## 网络操作

### HTTP 请求

```bash
# GET 请求
curl https://api.example.com/data

# POST JSON
curl -X POST -H "Content-Type: application/json" \
     -d '{"key":"value"}' \
     https://api.example.com/create

# 带认证
curl -u username:password https://api.example.com
curl -H "Authorization: Bearer token" https://api.example.com

# 下载文件
curl -O https://example.com/file.zip
wget https://example.com/file.zip

# 检查状态码
curl -s -o /dev/null -w "%{http_code}" https://example.com
```

### 端口和连接

```bash
# 检查端口
nc -zv localhost 8080
telnet localhost 8080

# 查看端口占用
lsof -i :8080
netstat -tlnp | grep 8080

# 扫描端口
nmap localhost
nmap -p 80,443 example.com
```

---

## 脚本常用模式

### 参数处理

```bash
# 位置参数
$0          # 脚本名
$1, $2...   # 第1、2...个参数
$#          # 参数个数
$@          # 所有参数（独立）
$*          # 所有参数（合并）

# 默认值
file=${1:-"default.txt"}          # 第一个参数或默认值
file=${1:?"必须指定文件"}          # 未指定则报错

# 遍历参数
for arg in "$@"; do
    echo "$arg"
done
```

### 错误处理

```bash
# set 选项
set -e          # 命令失败时退出
set -u          # 使用未定义变量时报错
set -o pipefail # 管道中任一命令失败则退出
set -x          # 打印执行的命令

# 组合使用
set -euo pipefail

# 手动检查
command || exit 1
if ! command; then
    echo "失败"
    exit 1
fi
```

### 清理操作

```bash
# 退出时执行清理
cleanup() {
    rm -f "$temp_file"
}
trap cleanup EXIT

# 中断时执行
trap 'echo "中断"; exit 1' INT TERM
```
