---
title: 根据agent学习git实录
description: 一次与 AI Agent 实验驱动、问题驱动的 Git 学习实录：从 Trae 图形界面到 git worktree 多 agent 并行协作。
date: 2026-09-11
image: https://image-1258996033.cos.ap-shanghai.myqcloud.com/westlake
tags:
  - Git
  - worktree
  - 多agent协作
  - 学习方法
categories:
  - 工具
math: true
license:
hidden: false
comments: true
draft: false
toc: true
build:
  list: always
lastmod: 2026-09-11
---
过去学 Git，模式总是"看教程 → 记命令 → 用时想不起来"。这次换了个方式：让 AI Agent 当陪练，我负责提问和质疑，它负责做实验，我们一起看真实输出。整个过程只遵循一条原则——**任何结论都要用命令跑出来，不接受"应该是这样"**。

这篇实录记录了一天内从图形界面到 `git worktree` 多 agent 并行协作的完整过程，以及一个让我印象深刻的结论：**当我可以随时打断、质疑、要求验证时，知识吸收率远超单向听讲**。

# 学习方式：问题驱动 + 实验验证

这套方式有三个特点：

- **问题驱动**：我的疑惑就是课程，没有固定大纲。
- **实验验证**：每个结论都在独立的练习仓库里跑一遍，用真实输出说话。
- **实时记录**：每一步问答、每一次纠错都写进笔记，形成可复习的轨迹。

最有意思的是"质疑"环节。当我发现 Agent 的说法和我的推理对不上时，它不会用"你应该相信我"来回应，而是直接建仓库、敲命令，让事实裁决。下面就有两次它当场被我说服、修正结论的例子。

# 第一站：Trae 源代码管理面板

## 三区模型

Git 的工作流可以概括为三个区（加上远程共四个）：

| 区域 | 含义 | 快递类比 | Trae 面板 |
|---|---|---|---|
| 工作区 | 你正在编辑的文件 | 桌面上摊开的东西 | 更改 |
| 暂存区（index） | 准备提交的清单 | 购物车 | 暂存的更改 |
| 本地仓库 | 已提交的历史 | 已发货订单 | Git Graph 圆点 |
| 远程仓库 | 服务器上的副本 | 快递公司仓库 | — |

## U / A / M 是什么

面板里每个文件旁边有一个字母，起初我以为它表示"文件在哪个区"，其实它表示**文件的改动类型**：

- `U`（Untracked）：新文件，从未被 `git add` 过
- `A`（Added）：新文件已经 `git add`，进入暂存区
- `M`（Modified）：已跟踪文件的内容被修改

## 一次关键修正：字母其实是"两列"

我提出一个疑问：**新建文件 `git add` 后再修改，命令行显示 `AM`，但相对 HEAD 它仍然是新增文件，为什么不是 `A`？**

Agent 当场跑实验：

```bash
$ git add c.txt
A  c.txt
$ echo b >> c.txt
AM c.txt
```

结论：`git status --short` 的输出是**两列 XY**，分别对应两个不同的比较：

- **第一列 X = index 与 HEAD 比较** → 决定「暂存的更改」的字母
- **第二列 Y = 工作区与 index 比较** → 决定「更改」的字母

| 操作 | 输出 | 暂存列表 | 更改列表 |
|---|---|---|---|
| 新文件 add | `A  ` | A | — |
| 新文件 add 后又改 | `AM` | A | M |
| 已跟踪文件修改 | ` M` | — | M |
| 已跟踪文件修改后 add | `M  ` | M | — |
| 已跟踪 add 后又改 | `MM` | M | M |

原来同一个文件可以**同时**拥有 A 和 M 两个字母，它们分属两个不同的比较，互不覆盖。Agent 之前"字母只由 index 与 HEAD 比较决定"的说法，正是在我追问下被实验推翻的。

## index 与 HEAD 到底是什么

- **HEAD** 是一个指针，文件 `.git/HEAD` 里只有一行 `ref: refs/heads/main`，指向当前分支最近一次提交。
- **index（暂存区）** 是一个二进制文件 `.git/index`，记录"路径 → 内容指纹"的清单。`git add` 往清单里写，`git commit` 把清单固化成新快照。

于是有两次比较：

```text
工作区 worktree  ──比较①──>  index  ──比较②──>  HEAD
                 「更改」              「暂存的更改」
```

# 第二站：Git Graph

Git Graph 是 VSCode 生态的插件，本质上是 `git log --oneline --graph --decorate --all` 的可视化。看懂它，要抓住两个元素：

- **圆点 = 一个 commit**（不可变快照，包含整棵树 + 父指针 + 作者 + 消息）
- **线 = parent 指针**，表示提交之间的父子关系

## merge commit 的双 parent

我一度以为 merge commit 的直接父是"历史里更早的那个提交"，实测纠正了这个误解：

```bash
$ git show daa14b7 --format='%h parents=%P' -s
daa14b7 parents=fa3a2f5 aaddc62
```

merge commit 有 **2 个直接 parent**（main 侧和 feature 侧各一个），而 root commit 有 **0 个 parent**（历史的起点）。`parent` 只看"直接父"，不跨层。

## HEAD -> main 是什么意思

图中某次提交旁标注 `HEAD -> main`，表示：当前检出的分支是 `main`，HEAD 指针经由 main 指向该分支的 tip。执行 `git checkout feat` 后，标记会变成 `HEAD -> feat`。

# 第三站：git worktree 多 agent 并行

这是本次学习的重头戏。普通 Git 仓库一次只能在一个目录检出一个分支；多个 agent 同时工作时共用目录会互相覆盖文件。`git worktree` 为同一个仓库创建多个工作目录：**每个目录有独立的工作区、index 和 HEAD，但共享对象数据库**。

```text
                         共享 .git 对象库
                       /        |        \
                 主工作区    worktree-a  worktree-b
                 main        feature/a   feature/b
```

## 共享 vs 复制

我提出了一个自认为很关键的问题：**"新建两个文件夹分别检出分支"是不是等价于 `git worktree`？**

Agent 用实验区分了两种做法。先建一个裸仓库当"远端"，在本地建一个**没 push** 的分支 `feat/a0`：

**做法 A：新建独立文件夹（clone）**

```text
$ git clone remote.git work2 && cd work2 && git branch -a
* main
  remotes/origin/main
$ git checkout feat/a0
error: pathspec 'feat/a0' did not match any file(s) known to git
```

未 push 的分支，新 clone 的文件夹根本看不到。

**做法 B：`git worktree add`**

```text
$ git worktree add ../work1-b feat/a0
Preparing worktree (checking out 'feat/a0')
HEAD is now at e3f692e feat/a0
```

零 push、零 clone，直接检出。

> 结论：**clone = 复制仓库（独立对象库），worktree = 共享仓库（同一对象库多个视图）**。worktree 因为共享本地对象库，本地私有分支在所有 worktree 里天然可见——这正是它能支撑多 agent 本地协作的本质。

## 一个分支只能被一个 worktree 检出

实测报错最能说明问题：

```text
$ git checkout main
fatal: 'main' is already used by worktree at '/private/tmp/git-work'

$ git worktree add ../x feature/a
fatal: 'feature/a' is already used by worktree at '/private/tmp/git-work-a'
```

原因是分支是一个"会移动的指针"，如果两个工作区同时检出同一分支，两边 commit 时指针该听谁的？Git 用"一分支一工作区"保证指针移动的唯一性。

## `git branch -avv` 里的 `+` 和 `*`

- `*` = 当前工作区检出的分支
- `+` = 被**其他** linked worktree 检出的分支（括号里是占用它的路径）

它和"分支从哪来"无关，只和"此刻被哪个工作区占用"有关。

## 一次事故复盘

练习中途，我为了清理分支，直接 `rm -rf` 删掉了练习仓库再重建，结果把前面积累的完整历史（merge 结构、双 worktree）全丢了。Agent 用 `git reflog` 只剩一条初始提交的证据确认了这次"推倒重来"，并强调了操作边界：

- `git branch -d` 只删分支指针
- `git worktree remove` 只删工作区
- `rm -rf` + `git init` 是整个仓库推倒重来

这次事故本身也成了很好的教材——**练习环境也要当成真实环境对待，删除前先确认**。

# 第四站：merge 的三种结局

最后用两个对比实验搞清了 `git merge`：

```bash
# 情形一：有分叉 → 生成 merge commit
$ git merge feature/a
*   1a24e48 Merge branch 'feature/a'   # 双 parent

# 情形二：当前分支是目标祖先 → fast-forward
$ git merge ff-test
6aac506 ff-test: commit 2               # 指针平移，无新 commit
```

| 情形 | 结果 |
|---|---|
| 两个分支指向同一个 commit | `Already up to date`（无操作） |
| 当前分支是目标分支的祖先 | **fast-forward**，指针平移，无新 commit |
| 两条线有分叉 | **merge commit**，生成双 parent 新提交 |

`git merge --no-ff` 则强制生成 merge commit，即使本可以快进——这样可以**保留分支拓扑**，方便日后回溯"这批改动来自哪个分支"。

# 这套方法为什么有效

一天下来，我最大的感受是：**主动提问 + 实验验证 + 当场纠错**，比任何单向讲解都牢固。几个具体的收获：

1. **疑问必须落到命令上**。`AM` 到底该显示 A 还是 M、worktree 和 clone 的差别，这些光靠讲容易含糊，跑一遍就再也不会忘。
2. **允许并欢迎质疑导师**。这次 Agent 有两处说法被我的追问推翻（字母的两列模型、worktree 路径规则），修正后的结论反而更清晰。
3. **共享 vs 复制**是理解 worktree 的钥匙。抓住"共享对象库、独立 HEAD/index"这一条，剩下的行为都能推导出来。

如果把 worktree 用到日常多 agent 协作，记住这条最小协议：

- 一个 agent 一个 worktree + 一个独立分支，绝不共用目录或分支
- worktree 路径放在仓库外
- 交付时报 commit hash + 测试结果
- 主工作区统一 merge（需要保留拓扑就用 `--no-ff`）
- 回收前先 `git status` 确认干净，再 `git worktree remove`

# 附录：worktree 常用命令速查

| 命令 | 作用 |
|---|---|
| `git worktree list` | 列出所有 worktree |
| `git worktree add <path> <branch>` | 新建工作区并检出已存在分支 |
| `git worktree add -b <new> <path> <start>` | 一步建分支并检出 |
| `git switch <branch>` / `git switch -c <branch>` | 切换 / 新建并切换分支 |
| `git merge <branch>` / `git merge --no-ff <branch>` | 合并 / 强制生成 merge commit |
| `git worktree remove <path>` | 删除工作区（会删目录内文件） |
| `git worktree prune` | 清理失效的 worktree 元数据 |
| `git branch -avv` | 查看分支及 `+`/`*` 占用状态 |

官方文档：<https://git-scm.com/docs/git-worktree>

# 参考文献

---

本文原载于 [巴巴变的博客](http://blog.bugxch.top)，遵循CC BY-NC-SA 4.0协议，复制请保留原文出处。
