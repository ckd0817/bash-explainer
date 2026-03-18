# Bash Explainer

一个用于 Claude Code 的 skill，自动解释 bash/shell 命令的每个组成部分，帮助用户学习命令行操作。

## 功能特点

- **自动解释**：每次执行 bash 命令后，自动显示命令的目的和每个组成部分的含义
- **中文说明**：所有解释使用中文，便于理解
- **持续学习**：即使是简单的命令也会解释，帮助用户积累命令行知识
- **详细模式**：支持简洁模式和详细模式两种解释风格

## 安装

将此 skill 复制到 Claude Code 的 skills 目录：

```bash
# 克隆仓库
git clone https://github.com/ckd0817/bash-explainer.git

# 复制到 Claude Code skills 目录
cp -r bash-explainer ~/.claude/skills/
```

## 使用方法

### 自动触发

加载 skill 后，每次执行 bash 命令都会自动显示解释：

```
📋 命令解释：`ls -la`

目的：以详细格式列出当前目录的所有文件（包括隐藏文件）

组成部分：
• `ls` - 列出目录内容
• `-l` - 长格式，显示权限、所有者、大小、时间等详细信息
• `-a` - 显示所有文件，包括以 . 开头的隐藏文件
```

### 请求详细解释

询问"详细解释这个命令"或"这个命令是什么意思"获取更多信息。

## 目录结构

```
bash-explainer/
├── SKILL.md                    # Skill 主配置文件，包含行为说明和示例
└── README.md                   # 本文件
```

## 示例

### 管道命令

```bash
cat log.txt | grep "error" | wc -l
```

```
📋 命令解释：`cat log.txt | grep "error" | wc -l`

目的：统计 log.txt 文件中包含 "error" 的行数

组成部分：
• `cat log.txt` - 读取并输出 log.txt 文件的内容
• `|` - 管道，将前一命令的输出传给后一命令
• `grep "error"` - 筛选包含 "error" 的行
• `|` - 管道，继续传递结果
• `wc -l` - 统计行数
```

### 条件执行

```bash
mkdir dir && cd dir
```

```
📋 命令解释：`mkdir dir && cd dir`

目的：创建目录 dir，成功后进入该目录

组成部分：
• `mkdir dir` - 创建名为 dir 的目录
• `&&` - 与连接符，只有前一命令成功才执行下一命令
• `cd dir` - 切换到 dir 目录
```

## 许可证

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request！
