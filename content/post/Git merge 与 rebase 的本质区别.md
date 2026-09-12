---
title: Git merge 与 rebase 的本质区别
description: 同一个分叉状态，git merge 和 git rebase 会产出完全不同的历史——一个保留分叉，一个拉直历史。用同状态对比实验讲透二者的本质区别、方向陷阱和选择标准。
date: 2026-09-12
image: https://image-1258996033.cos.ap-shanghai.myqcloud.com/westlake
tags:
  - Git
  - rebase
  - merge
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
lastmod: 2026-09-12
---
`git merge` 和 `git rebase` 都是把两个分支的成果合到一起的方法，但它们的**历史产出完全不同**——一个保留分叉结构，一个把历史拉直。很多人知道命令怎么用，却不清楚二者动的"对象"方向其实是反的。

这篇文章用一个干净的实验讲透：同一分叉状态，`merge` 和 `rebase` 各自产生什么，本质区别在哪，以及什么时候该用哪个。

# 起点：一个干净的分叉

假设两个分支 A 和 B 从共同祖先 C0 分叉：

```text
* bd3c398 (A) A1: add a
| * 3412a0c (B) B2: add b2
| * 96eeddc B1: add b1
|/
* bf6c99c (main) C0: base
```

A 有一个提交 A1，B 有两个提交 B1、B2，二者互不包含对方。这是 merge 和 rebase 的典型使用场景。

> 注意一个实验细节：要构造"真分叉"，必须用 `git branch A && git branch B`（都指向 C0）再从各自 checkout 提交。如果直接用 `git checkout -b B C0`，B 会包含 A 的提交，A 成为 B 的祖先，merge 会直接 fast-forward，无法演示分叉合并。

# 同一状态下的两种操作

对相同的分叉状态，克隆两份副本，一份做 `git merge B`，一份做 `git rebase B`（都在 A 分支上执行）。

## git merge B：保留分叉，新增汇合点

```text
*   6dd707a (A) Merge branch 'B' into A   ← merge commit，2 个 parent
|\
| * 3412a0c B2: add b2
| * 96eeddc B1: add b1
* | bd3c398 A1: add a
|/
* bf6c99c C0: base
```

merge 的特征：

- 产生一个 **merge commit**（`6dd707a`），它有 **2 个 parent**
- A 和 B 的原始提交 SHA **都不变**（`bd3c398`、`3412a0c` 原样保留）
- 历史**保留分叉结构**（两条线 + 一个汇合点）
- A 的内容 = A1 + B1 + B2

## git rebase B：拉直历史，重放提交

```text
* 3faa28a (A) A1: add a                   ← A1 被重放，SHA 变了！
* 3412a0c (B) B2: add b2                  ← B 原样（是 A 的新基底）
* 96eeddc B1: add b1
* bf6c99c C0: base
```

rebase 的特征：

- **没有 merge commit**
- A 的提交被**重放到 B 之上**：A1 变成 `3faa28a`（新 SHA，parent 从 C0 变成了 B2）
- B 的提交**原样保留**（`3412a0c`、`96eeddc` 不变，它们成为新基底）
- 历史变成**一条直线**，分叉消失

# 本质区别

| 维度 | `git merge B` | `git rebase B` |
|---|---|---|
| 新增提交 | merge commit（2 parent） | 无，纯重放 |
| A 的提交 | 原样保留，SHA 不变 | 重放，SHA 全变 |
| B 的提交 | 原样保留 | 原样保留（成为新基底） |
| 历史形态 | 保留分叉（两条线汇合） | 拉直成一条线 |
| 结果指向 | A 指向 merge commit | A 指向重放后新 tip |

> **最容易搞反的一点：二者动的对象不同**
>
> - `git merge B` = **把 B 并入 A**（B 的成果进入 A）
> - `git rebase B` = **把 A 重放到 B 之上**（A 的提交被搬走，parent 变成 B）
>
> 它们不是"同一件事的两种做法"，而是方向相反：merge 让 B 的提交进 A；rebase 让 A 的提交骑到 B 头上。

rebase 的本质可以用一句话概括：**replay all commits of current branch on B**（把当前分支的所有提交在 B 上重放一遍）。动的对象是**当前分支**，B 只是新基底。merge 则是 bring B's commits into current branch，动的对象是 **B**。

# 为什么 rebase 会改写历史

rebase 不是"移动"旧提交，而是**用同样的改动内容生成一批全新提交**接到新基点上。旧提交变成悬空提交（对象库里还在，但不再有分支指向它）。

所以：

- A1 从 `bd3c398` 变成 `3faa28a`——消息没变，但 **SHA、parent、时间戳全变了**
- 被重放提交的 SHA 变了，是因为它**内容里的 parent 字段变了**（从 C0 变成 B2），而 SHA 由提交全部内容哈希生成

> 顺带一提：这也解释了为什么"改写历史"有风险。如果 A 分支已经 push 到远端、别人基于它工作过，你 rebase 后就等于把别人脚下的地基抽掉了——他们的历史会和新远端历史分叉，pull 时直接报错。所以 rebase 只对**没出过门**的提交用。

# 什么时候用哪个

| 场景 | 选择 |
|---|---|
| 想要线性历史、图干净、分支未共享 | **rebase**（让当前分支建立在最新目标之上） |
| 想保留分支结构和汇合点 | **merge**（不破坏任何历史） |
| 分支已 push、别人可能基于它工作 | **merge**（共享历史不可改写） |

# 小结

一句话记法：

- `git merge B` = bring B's commits into current branch（动的是 B）
- `git rebase B` = replay current branch's commits on B（动的是当前分支）

merge 保留分叉、产生 merge commit、不改写任何历史；rebase 拉直历史、重放当前分支提交、改写 SHA。理解了"动的对象方向相反"这一点，就不会再把两者混为一谈了。

# 参考文献

---

本文原载于 [巴巴变的博客](http://blog.bugxch.top)，遵循CC BY-NC-SA 4.0协议，复制请保留原文出处。