---
layout: post
title:  Github推送大文件
tags: git
categories: git
---
Github推出private库后，有许多源码，可以放到上面去。但是有时候会遇到大于100M的问题，无法推送，可以通过以下命令处理后即可，注意备份！

```
git remote remove origin # 删掉原来git rep
git remote add origin [git rep url] # 修改rep地址
git filter-branch -f --tree-filter 'rm -f /path/to/file' HEAD --all # 删除大文件
git push origin --all #推送所有版本
```

