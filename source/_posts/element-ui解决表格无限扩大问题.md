---
title: element-ui解决表格无限扩大问题
date: 2018-01-22 17:55:13
categories: 编程
tags: php
---

## 问题
element-ui　表格使用的时候，出现宽度无线扩大的情况。
![](https://ws1.sinaimg.cn/large/74f12897gy1g0rn7gxsbvj20sg0sg7id.jpg)

## 解决
```css
    .el-table__header, .el-table__body{
        width: 100% !important;
        table-layout: auto !important;
    }
```
