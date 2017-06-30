---
title: 多PC同步hexo
date: 2017-06-30 15:48:10
categories: 其他
tags: hexo
---

## 背景
由于可能同时在公司和家中写文章，所以hexo同步是个问题，现记录解决方案

## 解决
### 首次操作
1. 创建 repo: `username.github.io`
<!--more-->
2. 创建两个分之
    - hexo: **设置为默认分支** （存 源文件）
    - master: 存生成的静态页面
3. 本地安装hexo:
    > 1. `npm install hexo -g`
    > 2. `hexo init`
    > 3. `npm install`
    > 4. `npm install hexo-deployer-git -S`
    > 5. 修改 *_config.yml* *deploy* `branch: master`
    
### 后期同步操作
> 在仓库已经建好后
1. `git clone git@github.com:username/username.github.io.git`
2. `cd username.github.io`
3. `npm install`

### 写文章
> 以下操作在 **hexo** 分支下
1. `git add .`
2. `git commit -m 'message'`
3. `git push origin hexo` 直到这里，是同步的源文件
4. `sudo hexo g -d` 部署静态文件
5. *done*
 
    

