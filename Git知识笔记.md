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

### 为什么要有暂存区

- **`commit` 提交的是暂存区的快照**，不是工作区里文件的当前状态。`git add` 记录的是文件**在那一刻**的内容，之后再改的部分，要重新 `add` 才会被提交。
- **重新 `add` 会用文件的最新内容覆盖暂存区里的旧版本**。暂存区存的是整个文件的快照，不是一条一条的修改记录。
- 暂存区是修改和提交之间的缓冲区，作用是**挑选**每次提交放哪些内容，让一次提交只做一件事。
- VS Code 里还能只暂存**一个文件中的部分行**：在 diff 视图里选中几行，右键选“暂存所选范围”。

### 保存和提交的区别

| | `Ctrl + S` 保存 | `git commit` 提交 |
|---|---|---|
| 保存的是什么 | 文件的最新内容 | 当时那一刻的快照 |
| 旧版本 | 被覆盖，没了 | 全都留着，随时能回去看 |
| 记录了什么 | 只有内容 | 内容、时间、作者、为什么改 |

- 保存像在同一页纸上擦了重写；提交像游戏里的**存档点**。
- 不需要每次保存都提交，**在告一段落的时候提交**（比如完成一个功能、学完一课）。

### 什么时候提交：一次提交 = 一件完整的事

- 衡量标准是**改动是不是一件完整的事**，不是改动量的大小。改一个错别字可以单独提交，写一个新功能可能要改十几个文件，也是一次提交。
- 自查：能不能用**一句话**写出提交信息？需要用“和”连接好几件不相关的事，就说明应该拆开提交。
- 提交时项目最好处于**能正常工作**的状态，不要提交改到一半、跑不起来的代码。
- 宁可小而多，也别大而少：大提交以后很难看懂，也很难只撤销其中一部分。

### Git 把历史存在哪：`.git` 文件夹

- 仓库根目录下的隐藏文件夹 `.git` 就是整个仓库：所有提交、暂存区（`.git/index`）、分支都在里面。⚠️ **删掉它，所有历史就没了**。不要手动修改里面的文件。
- 每个提交都记录了：**整个项目的快照（tree）**、**上一个提交（parent）**、作者、时间和提交信息。
- 提交之间通过 parent 连成一条链：

```
HEAD → master → 最新提交 → parent → parent → … → 第一个提交（没有 parent）
```

- `git log` 就是从 HEAD 出发，沿着 parent 一路往回走，把经过的提交列出来。
- 提交是**整个项目**的快照，不是某个文件的快照，所以 `git log` 显示的是所有文件的历史。只想看某个文件，用 `git log -- <文件>`。
- 查看一个提交对象的原始内容：`git cat-file -p HEAD`

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
| `git commit --amend --no-edit` | 把暂存区的内容补进上一次提交，提交信息不变（只用于**还没推送**的提交） | 提交按钮旁的 “⋯” → 提交 → 提交（修改） |
| `git log --oneline` | 查看提交历史 | 源代码管理面板里的“图形”视图 |
| `git show` | 查看最后一次提交改了什么 | 点击历史里的某次提交 |

## git diff：看具体改了哪几行

### 比较的是哪两个区域

```
           git diff               git diff --staged
工作区  <────────────>  暂存区  <────────────────>  最新提交 (HEAD)
   └──────────────────── git diff HEAD ─────────────────────┘
```

| 命令 | 比较 | 能看到什么 |
|---|---|---|
| `git diff` | 工作区 ↔ 暂存区 | 还**没暂存**的改动 |
| `git diff --staged` | 暂存区 ↔ 最新提交 | **将要提交**的改动（`--cached` 是同一个意思） |
| `git diff HEAD` | 工作区 ↔ 最新提交 | 自上次提交以来的**所有**改动 |

- 不写文件名时，显示**整个仓库**里所有有差异的文件；写了文件名，就只显示这些文件。
- ⚠️ **未跟踪（Untracked）的新文件不会出现在 `git diff` 里**，因为 Git 还没管它。`add` 之后，它才会出现在 `git diff --staged` 里。

### 常用参数

| 写法 | 作用 |
|---|---|
| `git diff <文件>` | 只看某个文件 |
| `git diff --stat` | 只看哪些文件改了多少行，不显示具体内容 |
| `git diff --name-only` | 只列出改过的文件名 |
| `git diff --color-words=.` | 在行内按字高亮改动，适合中文 |
| `git diff <提交A> <提交B>` | 比较两次提交 |

### 怎么读输出

- `-` 开头（红色）表示删掉的行，`+` 开头（绿色）表示新增的行，空格开头表示上下文。
- 修改一行 = 删掉旧行 + 新增新行。
- `@@ -7,4 +7,4 @@`：旧版本从第 7 行开始的 4 行，对应新版本从第 7 行开始的 4 行。
- `\ No newline at end of file` 只是提示文件末尾没有换行符，不是改动。

## 图形界面和命令行怎么分工

- **浏览历史、看某次提交改了什么**：用 VS Code 的“图形”视图和“时间线”视图，更直观、更快。
- **搜索和精确查询**：用命令行。例如按内容搜索（`git log -S`）、按提交信息搜索（`--grep`）、按时间或作者筛选、比较任意两次提交（`git diff A B`）。

## log 和 show 的区别

- **`git log` 是目录**：列出很多个提交，默认只显示提交信息。加 `-p` 可以显示改动，加 `--stat` 可以显示改了哪些文件。
- **`git show` 是详情页**：显示某一个提交的信息，以及它改了什么（diff）。写成 `git show <提交>:<文件>` 时，显示的是这个文件在那个版本的完整内容，没有 diff。
- **show 的 diff 是和它的 parent（上一个 commit）比**，只显示那次 commit 本身改了什么，不是和现在的版本比。

| 想看 | 命令 |
|---|---|
| 这个 commit 改了什么（和 parent 比） | `git show 75c91c4` |
| 这个 commit 里文件的完整内容 | `git show 75c91c4:fruits.txt` |
| 那个版本和我现在的文件差在哪 | `git diff 75c91c4 -- fruits.txt` |
| 那个版本和最新 commit 差在哪 | `git diff 75c91c4 HEAD -- fruits.txt` |

记法：show 看“一次 commit 本身”，diff 看“任意两个版本之间”。

## 在历史里搜索（命令行专长）

| 我想知道… | 命令 |
|---|---|
| 某段内容是哪次提交加进来或删掉的 | `git log --oneline -S "内容"` |
| 哪些提交改过包含某内容的行 | `git log --oneline -G "内容"` |
| 提交信息里提到某个词的提交 | `git log --oneline --grep="词"`（加 `-i` 不区分大小写） |
| 某个文件的所有改动历史 | `git log --oneline -- <文件>` |
| 某段时间内的提交 | `git log --oneline --since="2026-09-10 00:00" --until="2026-09-10 23:59"` |
| 最近一段时间的提交 | `git log --oneline --since="3 days ago"` |
| 某人的提交 | `git log --oneline --author="名字"` |
| 两个版本之间改了哪些文件 | `git diff <旧> <新> --stat`，后面加 `-- <文件>` 只看一个文件 |
| 把文件的旧版本另存出来 | `git show <提交>:<文件> > 旧版.txt` |

经验：

- **`-S` 统计的是内容出现的次数有没有变**。“苹果”改成“红苹果”，“苹果”两个字出现的次数没变，所以 `-S "苹果"` 搜不到这次提交。想找“改过含有某内容的行”的提交，用 `-G`。
- **`--since` / `--until` 只写日期不写时间时，会默认用当前时刻**。比如现在是下午 4 点，`--since="2026-09-10"` 实际表示 9 月 10 日下午 4 点之后。要按整天查，就把时间写上（`00:00`、`23:59`）。
- 这些参数**可以组合**：`git log --oneline -S "香蕉" -- fruits.txt`
- 找到提交号以后，具体改了什么，去 VS Code 的“图形”视图里点开看更直观。

## .gitignore：让 Git 忽略文件

项目根目录下的文本文件，写明哪些文件不该提交：日志、临时文件、编译产物、存密码的配置（`.env`）等。**新项目一开始就写好**，GitHub 建仓库时可以选语言模板。

### 常用写法

| 写法 | 意思 |
|---|---|
| `*.log` | 所有 .log 文件，不管在哪个文件夹 |
| `build/` | 整个 build 文件夹（结尾 `/` 表示只匹配文件夹） |
| `/todo.txt` | 只匹配根目录下的 todo.txt |
| `!keep.log` | 例外：前面忽略了，这个要保留 |
| `#` 开头 | 注释 |

### ⚠️ 核心规则：.gitignore 只管还没被跟踪的文件

文件只要提交过一次，再写进 .gitignore 也没用，修改后照样出现在“更改”里。

**文件还没提交过**：写规则（或在“更改”里右键 → Add to .gitignore）→ 文件变灰 → 提交 .gitignore。

**文件已经提交过**（只能用命令行，VS Code 没有对应按钮）：

```
1. 在 .gitignore 里写规则
2. git rm --cached <文件>        ← 停止跟踪，硬盘上的文件保留；文件夹加 -r
3. 把 .gitignore 和这次“删除”一起提交
```

- 第 3 步之前反悔：`git restore --staged <文件>`，或者在“暂存的更改”里点 “−”。
- `--cached` 不能漏：不带它，`git rm` 会把硬盘上的文件一起删掉。命令面板里的 **Git: Delete** 也是这个效果。
- 误提交的是**密码**：`rm --cached` 只管以后，旧提交里还留着它，推送后别人翻得到。**马上改密码**。

### 排查

| 遇到的问题 | 命令 |
|---|---|
| 写了规则，文件还在“更改”里 | `git ls-files <文件>`：有输出，说明它已被跟踪 |
| 文件不见了，想知道被哪条规则忽略 | `git check-ignore -v <文件>`：显示是哪个文件第几行的规则 |
| 想看所有被忽略的文件 | `git status -s --ignored`（`!!` 开头的就是；加 `-uall` 会展开文件夹） |

- 只看 .gitignore 不一定准：它写的是规则，不是文件清单。子文件夹里也可以有 .gitignore，规则会叠加。日常看资源管理器里的**灰色文件**就够了。
- 是 `--ignored`，不是 `--ignore`，写错会报 ambiguous option。

## 撤销和后悔药（阶段 2）

### 先选对工具

| 情况 | 用什么 |
|---|---|
| 改坏了还没 commit，想丢掉 | Discard（`git restore 文件`） |
| add 错了，想拿出来 | Unstage（`git restore --staged 文件`） |
| 想要某个文件的旧版本 | `git restore --source=commit号 文件` |
| 刚 commit 完，message 写错 / 漏了文件 | `git commit --amend` |
| 本地还没 push，想撤掉最近几个 commit | `git reset` |
| 已经 push 了，或者只想撤掉中间某一个 | `git revert` |
| commit 被 reset / amend 弄丢了 | `git reflog` 找 hash |

**两条铁律**
1. **从没 commit 过的改动丢了就找不回来**（Discard、`reset --hard`、`restore --source` 覆盖都是）。拿不准就先 commit。
2. **已经 push 的 commit 不要 amend / reset**（会改写历史，和远程分叉），用 revert。

### restore：撤销没 commit 的改动 / 取回旧版本

| 我想… | GUI | 命令 |
|---|---|---|
| 丢掉 working tree 的修改 | 文件旁 ↶（Discard Changes） | `git restore 文件` |
| unstage（修改保留） | 已 stage 文件旁 − | `git restore --staged 文件` |
| 把某个文件换成某个 commit 里的版本 | 不方便 | `git restore --source=08c717c fruits.txt` |
| 只挑旧版本里的某几处 | 不方便 | `git restore -p --source=08c717c fruits.txt`（逐块问，`y` 要 `n` 不要 `q` 结束） |

- `--source` 可以写 hash，也可以写相对位置：`HEAD~1` 是上一个 commit，`HEAD~3` 是往前数 3 个。
- **restore 不合并、不冲突，是整份覆盖**：文件变成那个版本，一字不差。
  - 和已 commit 内容的差别只是显示为改动，Discard 就能回来。
  - 这个文件上**未 commit 的改动会直接丢，没有提示**。
- 只动 working tree：HEAD 和历史不变，文件显示为未 stage 的修改，满意就 commit，不满意就 Discard。文件名写 `.` 就是所有文件。
- 只想要那次 commit **带来的改动**（不是整个文件）：`git cherry-pick commit号`（阶段 5）。
- 文档里的 `<提交>` 表示“换成你自己的值”，`[...]` 表示可选，照抄时括号都要去掉。PowerShell 里 `<` 是特殊符号，带着它敲会报错。

### commit --amend：修改最后一次 commit

- GUI：commit 按钮旁 ▾ → **Commit (Amend)**。
- `git commit --amend -m "新message"` 改 message；`git commit --amend --no-edit` message 不变，把 staging area 的改动补进去。
- 只写 `git commit --amend`：会打开编辑器（这里是 VS Code 的 `COMMIT_EDITMSG` 标签页），旧 message 已预先填好，关掉标签页就沿用。终端会一直等到标签页关闭；把内容删空再关就是取消。
- **amend 是替换，不是修改**：新 commit = 上一个 commit 的内容 + staging area 的新改动 + 新 message，顶替掉旧的。所以 staging area 为空也能 amend（只改 message），而普通 commit 会提示 `nothing to commit`。
- 只改 message，hash 也会变：hash 由内容、parent、作者、时间、message 一起算出。
- 旧 commit 还在 reflog 里（`commit (amend)` 那条的下一条就是被顶替的）。
- 只带 **staging area** 里的改动，没 stage 的不会进去。
- 只能改最后一个 commit；改了之后和上一个 commit 完全相同时，Git 会拒绝（`would make it empty`）。
- **改错用 amend，做新事用新 commit**。别对一个 commit 反复 amend 一整天：commit 就是存档点，多存几个不吃亏。

### reset：把分支退回去

`git reset commit号` 把当前分支（如 master）挪到那个 commit 上，后面的 commit 就不在历史里了。区别在于被退掉的**改动**去哪：

| 模式 | staging area | working tree | 典型用途 |
|---|---|---|---|
| `--soft` | 改动留在这里 | 不动 | **合并（squash）**几个 commit：reset 后直接 commit 一次 |
| `--mixed`（默认） | 清空 | 改动留在这里 | **拆分/重组**：退回后挑着 add |
| `--hard` | 清空 | **清空** | 彻底丢掉这些 commit 和改动 |

- GUI：`…` → 提交 → 撤销上次提交（Undo Last Commit）= `git reset --soft HEAD~1`。退多个 commit 用命令行。
- soft 和 mixed 只差“改动有没有 stage”，GUI 里一键全部 stage / unstage 就能互相转换，选离下一步近的那个。
- **reset 只比较两头的快照**，不会一个个回放中间的 commit。中间加了又删的内容，退回后看不到任何改动。
- `--soft` 不碰 staging area 和 working tree：原来没 stage 的改动依旧没 stage，所以同一个文件可能同时出现在 staged 和 unstaged 里。此时 working tree 才是最新的，直接 add 就行。
- squash 会让中间的 commit 从历史里消失，只压“过程垃圾”（测试、改错字、加了又删）。拿不准就先 `git branch 备份` 留一个分支指着旧历史。
- 目标写 hash 比 `HEAD~4` 稳，不会数错。
- ⚠️ `--hard` 会清掉**未 commit** 的改动，找不回来，用之前先 `git status`。

### revert：用新 commit 抵消旧 commit

`git revert --no-edit commit号`：算出那个 commit 的 diff，反过来应用到**当前 HEAD** 上，再自动 commit（message 为 `Revert "原message"`）。历史不改写，新 commit 接在最后。

| | reset | revert |
|---|---|---|
| 历史 | 改写，commit 消失 | 保留，多一个反向 commit |
| 能撤销哪些 | 只能从最新往回退一段 | 任意一个，前后都不受影响 |
| 已 push 的 commit | ❌ | ✅ |

- VS Code 自带界面没有 revert 按钮，用命令行。不加 `--no-edit` 会打开 COMMIT_EDITMSG，关掉即接受默认 message。
- 要求 staging area 干净，要改的文件也不能有未 commit 的改动，否则会被混进 commit 或者被覆盖。无关文件有改动不影响。
- 撤销一次 revert = 再 revert 它（message 变成 `Reapply "…"`）。
- Git 只机械检查那次 commit 改过的几行（及上下文）现在是否原样：
  - 没被动过 → 自动成功
  - 被改过（或紧挨着的行被改过）→ **冲突**：手动改好、add，`git revert --continue`；或 `git revert --abort` 放弃
  - 已经被改回原样 → 没东西可 commit
- 越早 revert 越顺利；commit 越小越好 revert；改得面目全非时，直接手写一个修正 commit 更省事。

### reflog ⭐：找回“丢掉”的 commit

HEAD 每移动一次（commit、amend、reset、revert、切换分支…），Git 都会在 reflog 这本流水账里记一笔。被 reset / amend 弄丢的 commit，只要记得 hash 就能救回来。VS Code 界面看不到，纯命令行能力。

```
6b3b8d8 HEAD@{0}: commit: 添加榴莲
74c6b59 HEAD@{1}: commit: 2-4
4806492 HEAD@{2}: reset: moving to 4806492...
```
左边是做完这个动作后 HEAD 指向的 commit，`HEAD@{n}` 是往前数第 n 次移动，右边是动作和 message。

- **`git reflog` = `git log -g` 的简写**：普通 log 顺着 parent 往前走，`-g` 顺着流水账往前翻。所以 log 的筛选参数都能用：
  - `git log -g --oneline --grep=榴莲`：在流水账里搜
- 找到 hash 之后：

| 想要 | 命令 |
|---|---|
| 看内容 | `git show 75c91c4` |
| 只取回一个文件 | `git restore --source=75c91c4 fruits.txt` |
| 整个分支退回到它 | `git reset --hard 75c91c4`（它之后的 commit 会丢） |
| 先挂个分支慢慢处理 | `git branch 救回来的 75c91c4` |

- ⚠️ PowerShell 里 `HEAD@{2}` 要加引号写成 `'HEAD@{2}'`，或者直接用 hash。
- ⚠️ 救不了**从未 commit 过**的改动。
- ⚠️ 只在本地，push / clone 都不带；没有分支指向的 commit 约 30 天后可能被清理（有分支指向的记录约 90 天）。
- 救之前先确认 working tree 干净，免得救一个丢一个。

## 分支与合并（阶段 3）

### 分支是什么

- **branch = 一个贴在 commit 上的标签**。`.git/refs/heads/master` 就是一个文本文件，里面只有一个 hash。所以建分支不花时间，删分支也不会删掉 commit。
- commit 只记录自己的 parent，**只能往回找，不能往前找**。所以需要 branch 记住最新的 commit 是哪个，`git log` 就是从 branch 出发顺着 parent 往回走。没有 branch 指着的 commit，log 和 GUI 都看不到，只能去 reflog 里找。
- **HEAD = 当前在哪个分支上**（`.git/HEAD` 里写的是 `ref: refs/heads/master`）。commit 时只有当前分支往前挪。
- commit 本身不记录自己属于哪个分支。说“这个 commit 在 master 上”，意思是从 master 往回走能走到它。共有的历史同时属于好几个分支。
- **working tree 和 staging area 都只有一份，所有分支共用。只有 commit 属于分支。**

### 命令速查

| 想做什么 | 命令 |
|---|---|
| 新建并切过去（最常用） | `git switch -c add-melon` |
| 切到已有分支 | `git switch master`（已有的分支不能加 `-c`，会报 already exists） |
| 切回上一个分支 | `git switch -` |
| 只建不切 / 动手前留存档点 | `git branch backup` |
| 从旧 commit 开分支 | `git switch -c old-version 74c6b59` |
| 列出分支 / 带最新 commit | `git branch` / `git branch -v` |
| 改名 | `git branch -m new-name` |
| 删除（已合并） / 强制删除 | `git branch -d 名字` / `git branch -D 名字` |
| 看所有分支的走向 ⭐ | `git log --oneline --graph --all`（GUI：源代码管理里的 Graph） |
| 比较两个分支的内容 | `git diff master add-melon` |
| B 有、A 没有的 commit | `git log --oneline A..B`（方向别写反） |
| 某个 commit 在哪些分支里 | `git branch --contains 74c6b59` |

- GUI：点左下角的分支名，可以切换或新建分支。
- `--graph` 画出分叉线，`--all` 显示所有分支（不加只显示当前分支），`-5` 只显示最近 5 个 commit。括号里的 `(HEAD -> master)` 表示你现在在 master 上。
- `-d`：分支上有 commit 没合并进**当前分支**时会拒绝（`not fully merged`）。先用 `-d`，被拒绝时想清楚了再用 `-D`。不能删除当前所在的分支。
- 网上常见的 `git checkout 分支` 是旧写法，和 switch 是一回事。
- ⚠️ detached HEAD：HEAD 直接指着一个 commit，没有指着分支。在这种状态下 commit，切走以后新 commit 就丢了。想在旧版本上改东西，用 `git switch -c 新分支 hash`。

### 带着没 commit 的改动切换分支

- 文件在两个分支上**内容一样**：改动跟着你走（stage 过的也仍然是 stage 状态）。
- 文件在两个分支上**内容不一样**：拒绝切换（`Your local changes ... would be overwritten`），因为切过去改动会被覆盖。**stage 了也一样会被拒绝**，staging area 同样会被换成目标分支的版本。
- 被拒绝时：要留着就 commit，不要了就 `git restore 文件`，想先收起来就用 `git stash`（阶段 5）。
- 改到一半才想起应该开分支：直接 `git switch -c 新分支`。新分支和当前分支指着同一个 commit，切换一定成功，改动也会跟过去。
- Git 宁可拒绝切换，也不会悄悄丢掉你的改动。切换前最好先让 working tree 干净。

### merge

**规矩：先切到接收改动的分支，再把别的分支合进来。** 合并前先用 `git log --oneline master..分支` 看看会带进来什么。

| | 快进合并（Fast-forward） | 三方合并 |
|---|---|---|
| 什么时候 | 当前分支在这期间没有新 commit | 两边都有新 commit |
| 做了什么 | 直接把标签往前挪 | 比较分叉点 + 两边最新的 commit，生成 merge commit |
| 新 commit | 没有 | 有，而且有**两个 parent** |
| Graph | 一条直线 | 先分叉再汇合 |

- 三方合并会打开 VS Code 的 `MERGE_MSG` 标签页让你确认信息，关掉标签页就完成了（终端里的 `Waiting for your editor` 不是卡住了）。加 `--no-edit` 会直接用默认信息 `Merge branch 'xxx'`。
- `--no-edit` 只是跳过编辑信息这一步，**不会帮你解决冲突**。
- Graph 里，merge commit 的第一个 parent（合并时你所在的分支）画成主线，合并进来的分支画成支线。
- merge 之前的位置记在 `ORIG_HEAD` 里，合错了可以 `git reset --hard ORIG_HEAD`。
- 合并完的分支用 `-d` 删掉。

### 解决冲突

**冲突的条件：两边都改了同一处，而且改得不一样。** 只有一边改了的地方 Git 会自动合并，哪怕在同一个文件里。

```
<<<<<<< HEAD
蓝莓          ← 当前分支（你站着的那个分支）
=======
番茄          ← 合并进来的分支
>>>>>>> add-tomato
```

- 有一边是空的：说明那一边**删掉了**这段内容（一边删、一边改）。
- VS Code 按钮：Accept Current / Accept Incoming / **Accept Both** / Compare。按钮显示不全时，用 `Ctrl+B` 收起侧边栏。Accept Both 按“当前分支在前、合进来的在后”排列。
- 按钮给不出想要的结果时（比如 `大青苹果`），就手动编辑。**归根结底就是把文件改成正确的样子，再删干净三行标记。**“保留双方”不一定对，改完要看一眼结果。
- 步骤：改好并保存 → `git add 文件`（合并过程中 add = 告诉 Git 这个冲突解决了；GUI 里在 Merge Changes 分组点 `+`）→ `git commit --no-edit`
- 搞乱了：`git merge --abort` 回到合并前，只要还没 commit 都能用。
- 合并没完成时，不能开始下一次 merge（`MERGE_HEAD exists`）。

### reset 共有的 commit

- reset 只挪当前分支的标签。别的分支照样指着原来的 commit，不受影响。
- ⚠️ 坑：在 master 上 reset 掉的 commit，如果别的分支的历史里还有，下次 merge 那个分支时，它会被带回来。想撤销共有历史里的 commit，用 revert。

## 远程仓库与 GitHub（阶段 4）

### 三件互相独立的事

| | 是什么 | 设置在哪 | 决定什么 |
|---|---|---|---|
| ① commit 作者 | commit 里写的名字和邮箱，**不验证**，随便写 | `git config --global user.name` / `user.email` | 只是个签名 |
| ② push 身份 | 连接 GitHub 时证明“我是谁” | SSH 密钥，或 HTTPS 登录（VS Code 登录 GitHub 属于这种） | 能不能 push |
| ③ 网页显示 | GitHub 拿 commit 里的邮箱去找账号 | GitHub 设置 → Emails | commit 显示成谁、算不算进贡献格子 |

- VS Code 登录哪个账号，**不会改** `user.email`。
- `user.email` 要设成 GitHub 账号里已添加的邮箱，③ 才对得上。改邮箱只影响以后的 commit。
- 换了邮箱，旧 commit **不用改写**，把旧邮箱也加到 GitHub 账号里就行（改写历史要强推，还可能把 commit 日期改掉）。
- 绿格子按 **commit 的日期**画，不是 push 的日期；更新可能延迟到 24 小时。
- 因为 ① 不验证，commit 作者可以冒充。可信的证明是 **commit 签名**（网页上显示 Verified），阶段 6 选学。

### SSH 是什么

- **SSH = Secure Shell**：两台电脑之间建立**加密连接**的协议。最初用来远程登录服务器敲命令；GitHub 只把它当传输 Git 数据的通道，不给 shell（所以提示 `does not provide shell access`）。
- 解决两个问题：**防偷看**（全程加密）、**防冒充**（双方都要证明身份）。
- **服务器证明自己**：第一次连接问 `yes/no`，答 yes 后服务器公钥记进 `~/.ssh/known_hosts`；以后对不上就报 WARNING 拒绝连接。
- **你证明自己**：公钥 = 锁（可以随便给），私钥 = 唯一的钥匙。GitHub 出题，你用私钥算答案，GitHub 用公钥检验。**私钥从不发出去**；私钥泄露 = 别人能用你的身份 push。
- `ssh -vT git@github.com`：看连接全过程（`Server host key` → `is known and matches` → `Offering public key` → `Authenticated`）。

### 网络分层：SSH 不负责传信号

| 层 | 负责什么 | 快递比喻 |
|---|---|---|
| 物理连接（Wi-Fi、网线、光纤、运营商） | 传信号 | 公路、货车 |
| IP | 地址和找路 | 收件地址、中转站 |
| TCP | 不丢、不乱序 | 签收、补发 |
| **SSH** | 加密 + 身份验证 | 带锁的保险箱 |
| Git | 要传的 commit | 箱子里的货 |

- 每层只管自己的事；数据沿途经过的设备都能看到“谁发给谁”，但打不开 SSH 的箱子。
- **端口**：SSH 用 22，HTTPS 用 443。`tracert github.com`（cmd/PowerShell）可以看沿途的中转站。

### SSH 的时效

- **连接**：每次 push / fetch 都是“连上 → 验证 → 传数据 → 断开”，没有“登录过期”这回事。
- **密钥**：默认不过期。失效的情况：在 GitHub 删了公钥、本地私钥没了、**一年没用被 GitHub 自动删除**。失效时报 `Permission denied (publickey)`，重做 4-1 即可。
- 每台电脑各生成一把密钥，不要复制私钥。电脑丢了就在 GitHub 上删掉那一把。
- 对比：HTTPS 方式（包括 VS Code 登录 GitHub）用**令牌**，一般有有效期。
- 不是非得用 SSH：远程地址是 `git@github.com:...` 走 SSH，`https://...` 走 HTTPS。VS Code 点推送，背后就是运行 `git push`。

### SSH 密钥

- `ssh -T git@github.com`：`-T` = 不要终端，只测试连接。用户名所有人都是 `git`，GitHub 靠公钥认出你。
- `ssh-keygen -t ed25519 -C "备注"`：生成密钥。`-C` 只是备注，写什么都行。
- `~/.ssh/id_ed25519` 是**私钥**，永远不给任何人；`id_ed25519.pub` 是**公钥**，加到 GitHub → Settings → SSH keys。
- `ssh -T git@github.com`：测试连接，显示 `Hi 用户名! You've successfully authenticated` 就成功了。
- 复制公钥到剪贴板：Git Bash 用 `cat ~/.ssh/id_ed25519.pub | clip`，PowerShell 用 `Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard`。命令执行后终端没有输出是正常的。

### remote + push

```bash
git remote add origin git@github.com:lbc94777/git-practice.git   # 添加远程，起名 origin
git remote -v                                                    # 查看远程地址
git push -u origin master                                        # 第一次推送，-u 记住对应关系
git push                                                         # 以后直接推
git branch -vv                                                   # 看每个本地分支对应哪个远程分支
```

- `git remote add` **不联网**，只是在 `.git/config` 里记一条“名字 → 地址”。其他操作：`git remote rename`、`git remote set-url`、`git remote remove`。
- 地址 `git@github.com:lbc94777/git-practice.git` = `用户@服务器` + `:` + 仓库路径。
- push 只发送远程**缺的** commit，然后挪动远程的 master 和本地的 `origin/master`。
- **`-u` 设置 upstream（上游）**：记住“本地 master 对应 origin/master”，之后 `git push` / `git pull` 不用写参数，status 也能显示领先/落后几个 commit。只需第一次加。
- ⚠️ 远程有你本地没有的 commit 时，push 会**被拒绝**，要先 pull 下来。
- 在 GitHub 新建仓库时，**不要勾选 README / .gitignore / license**，保持空仓库，否则第一次推送会冲突。
- push 推的是 **commit**。被 `.gitignore` 忽略的文件从没进过 commit，不会被推上去。
- **`origin/master`（远程跟踪分支）**：记录“上次和 GitHub 通信时，GitHub 上的 master 在哪”。它和 `master` 指着同一个 commit，就是已同步。
- 私有仓库的贡献默认只有自己能看到；个人主页 Contribution settings 里勾选 Private contributions，别人才能看到（只显示数量）。

### fetch / pull

```bash
git fetch                              # 下载远程的新 commit，只挪 origin/master，不动 master 和文件
git log --oneline master..origin/master   # 远程有、本地没有的 commit
git diff master origin/master          # 具体改了什么
git merge                              # 把 origin/master 合并进当前分支
git pull                               # = fetch + merge
```

- **`git status` 不联网**。`up to date with 'origin/master'` 的意思是“和**上次知道的** GitHub 一致”，想知道最新情况要先 fetch。
- fetch 后 status 显示 `behind ... by 1 commit, and can be fast-forwarded`。
- `origin/master` 本质上就是个分支，合并它和阶段 3 合并普通分支一样；本地没有新 commit 时就是 **Fast-forward**。
- 日常用 `pull`；想先看看远程改了什么再决定，就分开用 `fetch` + `merge`。

### clone

```bash
git clone git@github.com:lbc94777/git-practice.git git-practice-B   # 最后是文件夹名，可省略
git branch -a                                                       # 看本地 + 远程跟踪分支
```

- 下载全部文件和**完整历史**，并自动设置好 origin 和 master 的上游，下载完直接能 push / pull。
- 相当于一口气做了：新建文件夹 → `git init` → `git remote add origin` → `git fetch` → 切到 master 并设好上游。
- **仓库从你这里开始，用 init；仓库已经在 GitHub 上了，用 clone**（换电脑、加入别人的项目、看开源代码）。
- 被 `.gitignore` 忽略的文件（`.env` 等）从没进过 commit，clone 下来也没有。换电脑时这类文件要自己另外拷贝。
- `origin/HEAD -> origin/master`：记录远程仓库的默认分支是哪个。

### 两人改了同一处：push 被拒绝 → pull → 解决冲突 → push

1. 远程有自己没有的 commit 时，push 报 `! [rejected] ... (fetch first)`。Git 不允许覆盖别人的 commit。
   - ⭐ **拒绝和冲突是两回事**：拒绝只看**历史**（你推的历史必须包含远程最新的 commit，即只能快进），和改了什么内容无关，改不同文件也会被拒绝；冲突是 pull 时才检查的**内容**（两边改了同一处）。
   - `git push --force`：强推，远程分支直接挪到你的 commit，别人的 commit 从远程消失。共用分支上禁止使用；只在自己的分支、改写过历史时用，而且用更安全的 `git push --force-with-lease`。
2. `git pull` 的输出分两段：前半是 fetch（`2013482..e3aec9f master -> origin/master`），后半是 merge（`CONFLICT` 或 `Fast-forward`）。
3. 冲突和阶段 3 一样处理：`HEAD` 那半是自己的，另一半是从远程拉下来的。改好后 add → commit → push。
4. 另一边再 `git pull`，就是 Fast-forward。
5. 💡 养成习惯：**开始干活前先 pull，干完马上 push**，能大大减少冲突。

### 不想马上合并别人的修改

- `git pull` 会直接合并到**当前分支**。想先看再决定，用 `git fetch` + `git log master..origin/master` / `git diff`，满意了再 `git merge`。
- 冲突还没解决时后悔：`git merge --abort`，恢复到 pull 之前。
- 合并完成后后悔：`git reset --hard ORIG_HEAD`（`ORIG_HEAD` 是合并前自动记下的位置）。
- 同一个分支上迟早要合并，否则 push 不上去。真正的办法是**各自在自己的分支上干活**，通过 Pull Request 合并进 master（阶段 5）。

### 谁能 push：权限

| 身份 | 能直接 push 吗 | 怎么贡献 |
|---|---|---|
| Owner（仓库是你建的） | ✅ | 直接 push |
| Collaborator（Settings → Collaborators 里被邀请的人） | ✅ | 直接 push |
| 其他人 | ❌ `Permission denied` / `403` | **Fork**（复制一份到自己账号）→ push 到副本 → 发 **Pull Request** → owner 审查后合并 |

- push 被拒绝有两种完全不同的原因：**没权限**（身份问题），或**历史对不上**（缺远程最新的 commit）。
- 公司仓库常给 master 开**分支保护**：有权限也不能直接 push / 强推，必须走 PR 并经同事审查。

## 日常协作（阶段 5）

### stash：把没 commit 的改动先收起来

适用场景：改到一半，要临时切去别的分支干别的事（比如修紧急 bug），现在的改动又不想 commit。

| 命令 | 作用 |
|---|---|
| `git stash` | 收起已跟踪文件的改动，working tree 变干净 |
| `git stash -u` | 连 untracked（新建的）文件一起收 |
| `git stash -m "火龙果茄子"` | 给这条 stash 起名字，存了好几条的时候才分得清 |
| `git stash list` | 查看所有 stash，最新的一条是 `stash@{0}` |
| `git stash pop` | 拿回最新的一条，并从列表里删掉 |
| `git stash apply` | 拿回来，但列表里保留这条 |
| `git stash drop` | 删掉最新的一条 |

- ⚠️ **新建的文件默认不会被 stash**，还会留在 working tree 里。要收就加 `-u`。
- 典型流程：`stash` → `switch -c hotfix` → 修复并 commit → `switch master` → `merge hotfix` → `branch -d hotfix` → `stash pop`。
- 在 hotfix 修的东西和 stash 里的改动可以同时保留，pop 时 Git 会把两边合起来。
- **Fast-forward 的条件**：开分支之后 master 没有新的 commit，Git 只需把 master 的指针往前挪。能不能快进跟有没有冲突无关。

## 要点

- **`git status` 会提示下一步能做什么**，不会的时候先看它括号里的提示。
- **提交信息必须写**，为空时提交会被取消。要说清楚这次做了什么：写 `新增西瓜`，不要写 `update`、`修改`。
- **`restore` 和 `restore --staged` 只差一个参数，后果完全不同**：前者丢掉修改，后者只是把文件从暂存区拿出来。
- **“放弃更改”不是 Ctrl+Z**：它不是撤销一步，而是把文件一次性恢复成暂存区的版本（没暂存过就是上次提交的版本），中间所有修改全部丢掉。
  - 对**已跟踪**的文件（标 M）：Git 层面找不回来。后悔了可以去 VS Code 的“时间线”视图找**本地历史**，每次保存都会记一个版本。
  - 对**未跟踪**的文件（标 U）：等于删除文件。VS Code 默认把它移到**回收站**（设置项 `git.discardUntrackedChangesToTrash`），可以从回收站还原。但用命令行 `git clean` 删除的文件不会进回收站。
- **`git status -s` 开头的两个字符：左边一位是暂存区，右边一位是工作区**。` M` 是改了没暂存，`M ` 是已暂存，`MM` 是暂存后又改了，`A ` 是新文件已暂存，`??` 是未跟踪，`!!` 是被忽略。
- **分页器**：`git log`、`git show` 的输出会进入翻页模式，按 `q` 退出，`空格` 向下翻页，`b` 向上翻页。临时不想翻页可以用 `git --no-pager log`。

## 环境设置（已完成）

- `git config --global core.quotepath false`：让中文文件名正常显示
- 建议在 VS Code 设置里打开 `Files: Insert Final Newline`：自动在文件末尾补换行，否则每次加一行，diff 都会显示改了两行

## 附：让 Claude 看到 VS Code 终端（暂未实施）

Claude 在 VS Code 插件里看不到你终端里的内容。以后需要的话：

- **方案 A**：`Start-Transcript -Path "$HOME\terminal-log.txt" -Append` 录制终端操作。最简单，但 PowerShell 5.1 可能录不到 git 的输出。
- **方案 B**：自制 VS Code 插件（用“终端 shell 集成”接口记录每条命令和输出），再加一个 Claude Code hook，发消息时自动附上终端记录。更可靠，大约要 20–30 分钟。
- 注意：日志会记下终端里敲的所有内容，要放在仓库外面，不能提交到 Git。
