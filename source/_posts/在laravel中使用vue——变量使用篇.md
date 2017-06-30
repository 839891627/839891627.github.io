---
title: 在laravel中使用vue——变量使用篇
date: 2017-06-29 18:14:45
categories: 编程
tags: 
  - laravel
  - vue
---

### 前言
{% post_link Laravel-5-3-下添加自定义-Facade-的步骤 上一篇 %}
 讲了在 *laravel*中配置*vue*,本篇说说变量如何使用
  
### 实战
#### 首先在`controller`里面传递变量到*blade*模板：
    ```php
      $user = User::find(1)->toJson();
      return view('home', compact('user'));
    ```
<!--more-->
#### 然后在模板里面使用：
    ```blade
    <v-form :user="{{ $user }}"></v-form>
    ```
#### 组件里面配置如下：
    ```
        ...
        data() {
            return {
                ruleForm: {
                    name: this.user.name,
                    email: this.user.email
                },
            };
        },
        props: ['user'],
        ...
    ```