---
title: GitHub撤销已提交的commit
date: 2019-01-16 16:46:06
categories: GitHub
tags: [GitHub, ]
---

今天在向 GitHub 提交某个 commit 时，发现不小心将 Key 配置直接写在了配置文件内，为了安全起见，需要撤销这个 commit

```bash
git reset --hard HEAD~1
gut push --force
```

这样即使在GitHub的历史commit里，也看不到上次的改动了