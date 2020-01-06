---
title: Mac 小技巧
tags: 
  - Mac
categories: 工具
top: 0
copyright: ture
date: 2019-06-30 00:21:33
---
# 解压.rar、.7z文件

>unrar x 1.rar  
>7z e 1.7z

<!--more-->
## before:
> brew install unrar

> brew search 7z  
> brew install p7zip 

## 坑
```Updating Homebrew...```
 - 法1：

替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git 


 - 法2: 

重置brew.git:
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

重置homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git

# 一些快捷键

显示隐藏文件：`command + shift + .`

彻底删除：`command + delete`