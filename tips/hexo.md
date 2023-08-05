---
title: hexo
date: 2023-07-30 16:55
tags: [hexo]
toc: true
---

https://www.jianshu.com/p/4f21e948953e

# 1、运行

在hexo根目录，有 `.deploy_git` 的文件夹下，使用

```bash
hexo g
hexo s		# 本地运行 http://localhost:4000/
hexo d		# 远程运行 (hexo g -d)
```

<!--more-->

# 2、关键字

## 2.1、标题与 tag 设置

tags：用英文逗号分隔；

toc：是否展示目录；

```bash
---
title: hexo
date: 2023-07-30 16:55
tags: [hexo]
toc: false
---
```

## 2.2、分割线

```bash
<!--more-->		# 分割线，以上为预览展示内容
```

## 