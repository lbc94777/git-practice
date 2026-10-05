# Git 知识笔记

## 核心概念：三个区域

```
工作区 --git add--> 暂存区 --git commit--> 本地仓库 --git push--> 远程仓库
```

- **工作区**：你正在编辑的文件
- **暂存区**：准备放进下一次提交的内容
- **本地仓库**：已经提交的历史记录

文件的几种状态（`git status` 里能看到）：

| 状态 | 含义 | VS Code 标记 |
|---|---|---|
| Untracked | 新文件，Git 还没开始管它 | U |
| modified（红色） | 改了，但还没暂存 | M |
| modified（绿色） | 已暂存，等着提交 | M（在“暂存的更改”里） |

## 命令速查

| 命令 | 作用 | VS Code 对应操作 |
|---|---|---|
| `git status` | 查看当前状态 | 源代码管理面板里的文件列表 |
| `git status -s` | 简洁版状态 | — |
| `git add <文件>` | 放进暂存区 | 点文件旁边的 “+” |
| `git add .` | 暂存当前文件夹下的所有改动 | 点“更改”标题旁边的 “+” |
| `git restore --staged <文件>` | 取消暂存，改的内容**还在** | 点“暂存的更改”里文件旁边的 “−” |
| `git restore <文件>` | ⚠️ **丢掉**工作区的修改，找不回来 | 点“更改”里文件旁边的弯箭头 |
| `git commit -m "信息"` | 提交 | 填写信息后点 ✓ 提交 |
| `git commit` | 在编辑器里写提交信息，适合写长信息 | 弹出 `COMMIT_EDITMSG` 标签页，写完保存并关闭 |
| `git log --oneline` | 查看提交历史 | 源代码管理面板里的“图形”视图 |
| `git show` | 查看最后一次提交改了什么 | 点击历史里的某次提交 |

## 要点

- **`git status` 会提示下一步能做什么**，不会的时候先看它括号里的提示。
- **提交信息必须写**，为空时提交会被取消。要说清楚这次做了什么。
- **`restore` 和 `restore --staged` 只差一个参数，后果完全不同**：前者丢掉修改，后者只是把文件从暂存区拿出来。
- **分页器**：`git log`、`git show` 的输出会进入翻页模式，按 `q` 退出，`空格` 向下翻页，`b` 向上翻页。临时不想翻页可以用 `git --no-pager log`。

## 环境设置（已完成）

- `git config --global core.quotepath false`：让中文文件名正常显示
- 建议在 VS Code 设置里打开 `Files: Insert Final Newline`：自动在文件末尾补换行，否则每次加一行，diff 都会显示改了两行

## 附：让 Claude 看到 VS Code 终端（暂未实施）

Claude 在 VS Code 插件里看不到你终端里的内容。以后需要的话：

- **方案 A**：`Start-Transcript -Path "$HOME\terminal-log.txt" -Append` 录制终端操作。最简单，但 PowerShell 5.1 可能录不到 git 的输出。
- **方案 B**：自制 VS Code 插件（用“终端 shell 集成”接口记录每条命令和输出），再加一个 Claude Code hook，发消息时自动附上终端记录。更可靠，大约要 20–30 分钟。
- 注意：日志会记下终端里敲的所有内容，要放在仓库外面，不能提交到 Git。
