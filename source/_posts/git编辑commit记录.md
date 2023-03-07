---
title: git编辑commit记录
date: 2023-03-07 09:39:55
tags: git
---

# 修改Git提交记录

## 1. 修改最近一次commit的信息

查看当前的commit信息

```bash
$ git log
```

修改commit信息

```bash
# 进入命令模式编辑提交信息
$ git commit --amend
# 或使用命令修改author信息
$ git commit --amend --author="xxxx <xxx#xx.cn>"
```

## 2. 修改近期的提交

```bash
# git rebase 最近的n次提交，例如2次
$ git rebase -i HEAD~2
# 按vim编辑器的模式，a\i\o进入编辑模式
# 将要修改的commit记录的 pick 修改为edit，保存退出
# 修改提交信息
$ git commit --amend

$ git commit --amend --author="xxxx <xxx#xx.cn>"

# 继续修改其他commit记录
$ git rebase --continue

# 提交修改的commit记录
$ git push --force

```

