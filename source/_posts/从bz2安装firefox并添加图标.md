---
title: 从bz2安装firefox并添加图标
date: 2017-11-20 15:11:43
categories: 其他
tags: 
    - linux
---

> 首先下载最新版，地址：http://www.firefox.com.cn/download

1. > sudo apt remove firefox  [--purge 这样会删除旧版的配置信息]

2. > sudo tar -jxvf ~/Firefox-latest-x86_64.tar.bz2 -C /usr/lib/  

3. > sudo ln -s /usr/lib/firefox/firefox /usr/bin/  

4. > cd /usr/share/applications  

5. > sudo vi firefox.desktop  

```
[Desktop Entry]  
Name=Firefox 57.0
Comment=firefox
Exec=/usr/lib/firefox/firefox  
Icon=/usr/lib/firefox/browser/icons/mozicon128.png  
Terminal=false  
Type=Application  
Categories=Application
```
