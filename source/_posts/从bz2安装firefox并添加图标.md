---
title: 从bz2安装firefox并添加图标
date: 2017-11-20 15:11:43
categories: 其他
tags: 
    - linux
---

> 首先下载最新版，地址：http://www.firefox.com.cn/download

1. `sudo tar -jxvf ~/Firefox-latest-x86_64.tar.bz2 -C ~/App/`
> 习惯在主目录创建`App`文件夹专门安装软件
2. `vi ~/.local/share/applications/firefox.desktop` 
```
[Desktop Entry]  
Name=Firefox 57.0
Comment=firefox
Exec=/home/arvin/App/firefix/firefox
Icon=/home/arvin/firefox/browser/icons/mozicon128.png  
Terminal=false  
Type=Application  
Categories=Application
```
