---
title: 在终端中使用代理-proxychains
date: 2019-03-12 18:30:48
categories: 编程
tags: 代理
---
## 背景
命令行默认是不走 *代理* 的，而例如：*brew*/*composer* 这些连接很慢。而 `proxychains` 则可以在命令行中走代理

## 安装并使用
1. 安装
    ```bash
    git clone https://github.com/rofl0r/proxychains-ng.git
    cd proxychains-ng
    ./configure
    make && make install
    cp ./src/proxychains.conf /etc/proxychains.conf
    cd .. && rm -rf proxychains-ng
    ```
1. 编辑 proxychains 配置：`vi /etc/proxychains.conf`, 将`socks4 127.0.0.1 9095` 改为 `sock5 127.0.0.1 1080` (这里视 ss/v2ray 客户端配置情况)
    > ps: 默认的socks4 127.0.0.1 9095是tor代理，而socks5 127.0.0.1 1080是shadowsocks的代理。
1. 使用方法：在需要代理的命令前面加上 `proxychains4` 即可：`proxychains4 wget http://xxx.zip`
    > 可以设置 *alias*


