---
title: Git 如何配置 gitignore 生效
date: 2020-04-12 12:15:22
categories: git
tags: [git]
---

在 git 项目仓库中，配置 .gitignore 文件时，需要重新添加所有文件使得增加的规则生效

```
# Remove cached files
git rm -r --cached .

# Add everything back
git add .

# Commit changes
git commit -m "update .gitignore"
```
