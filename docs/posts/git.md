---
date: 2025-09-08
category:
  - Git
tag:
  - Git
---

# Git 妙用

# 创作背景
  在软件开发的江湖中，若是说编程语言是各路高手的武功招式，那么`Git`便是那平平无奇、实则至关重要的“内功心法”。
  相信每位程序员最熟悉的自然是这套“三板斧”：
  ```shell
  git add <.|文件>
  git commit -m <提交记录>
  git push
  ```
  然而，作为一名合格的程序员，对Git的认知当然不能只是止步于此。`Git`就像一个程序员每日驾驶却从未打开过引擎盖的跑车——知道它能跑，却不知道它究竟有多强的性能。当遇到复杂的路况（紧急Bug修复）、需要炫技的场合（优雅地合并代码）、或是倒车失误时（错误代码回滚），你就会发现，仅仅会“踩油门和刹车”是远远不够的。

# 常见情景应对招式

## 关联远程仓库
1、需求：（1）新项目（2）指定主分支（3）关联指定远程仓库

2、解决：
```shell
git status
git init
git add .
git commit -m <提交描述>
git branch -M <指定分支名>
git push -u origin <指定分支名> # --force（可选）首次提交需要带上
```

## 瞒天过海

### 场景-1
1、需求：不小心提交了node_modules，以及推送到远程仓库了，怎么从Git历史记录中移除呢，怎么从远程仓库中移除node_modules呢？

2、解决：
```shell
git status
git ls-files | grep node_modules # 检查是否已经提交node_modules
# 这里需要创建.gitignore文件，并且把文件push
vim .gitignore
git rm -r --cached <node_modules的路径> # 从缓存移除目录
git commit -m <提交描述>
git ls-files | grep node_modules # 再次检查
git push
```

### 场景-2
1、需求：在修复一个bug问题中，由于在本地测试未测出隐藏bug，光修复该bug就提了好几个`commit`，在最后一次提交`commit`时，不想保留前面的提交记录

2、解决：
```shell
# 前提：该项目时个人开发项目或者pull拉取最新代码，确保无他人提交记录
# （1）本地撤销前面的提交记录（可以手动在vscode或其他支持git操作的编译器手动回退）

# （2）使用命令的话 需要确定基于哪个commit变更（例如是 abc123）
git rebase -i abc123

# 重新提交
git add .
git commit -m xxxx
git push origin xxxx --force # 或--force-with-lease
```