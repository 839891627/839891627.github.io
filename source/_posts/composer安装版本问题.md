---
title: composer安装版本问题
date: 2017-08-07 10:10:35
categories: 编程
tags: 
    -php 
    - composer
---

进行laravel的数据迁移的时候，对修改字段类型，需要安装 `composer install doctrine/dbal`,但是遇到以下问题：
```
 Problem 1
    - Installation request for doctrine/dbal v2.6.1 -> satisfiable by doctrine/dbal[v2.6.1].
    - doctrine/dbal v2.6.1 requires php ^7.1 -> your PHP version (7.0.16) does not satisfy that requirement.
  Problem 2
    - doctrine/instantiator 1.1.0 requires php ^7.1 -> your PHP version (7.0.16) does not satisfy that requirement.
    - doctrine/instantiator 1.1.0 requires php ^7.1 -> your PHP version (7.0.16) does not satisfy that requirement.
    - Installation request for doctrine/instantiator 1.1.0 -> satisfiable by doctrine/instantiator[1.1.0].

```

**解决如下：**  
    `composer install doctrine/dbal --ignore-platform-reqs`  
    
**如果更新，则：**  
    `composer update --ignore-platform-reqs`
