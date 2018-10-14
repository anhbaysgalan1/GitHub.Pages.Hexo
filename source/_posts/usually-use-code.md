---
title: 常用代码
date: 2018-10-14 16:55:26
tags:
---

# Git 命令

## 查看纯提交信息日志

```bash
git log --pretty="%s" --no-merges
```

# ffmpeg 命令

## 嵌入字幕

```bash
ffmpeg -i <input> -vf scale=<width>:<height>,pad=<width>:<height>:<xpos>:<ypos>,subtitles=<ass> <output>
```
