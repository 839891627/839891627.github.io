---
title: 在laravel中使用vue——配置篇
date: 2017-06-29 10:53:36
categories: 编程
tags: 
  - laravel
  - vue
---
### 前言
很喜欢`element-ui`，本来想完全前后端独立的，但是考虑到后面的菜单权限不是很好搞，就在 laravel 中混合使用 vue　吧，虽然显得不伦不类。。。
整体框架使用　**admin-lte**，然后里面组件是用 **vue**

<!--more-->

### 安装
#### 安装**admin-lte**
1. 安装： `composer require arvin/adminlte-for-laravel`
2. 配置：  
    2.1. 添加 `Arvin\Admin\Providers\ArvinServiceProvider::class`, 到 `config/app.php`的providers  
    2.2. `php artisan vendor:publish`  
    2.3. `resources/views/vendor/arvin/layouts/default.blade.php`文件，  
    > * `<head>`里面添加
     ```html
       <!-- CSRF Token -->
       <meta name="csrf-token" content="{{ csrf_token() }}">
       <!-- Styles -->
       <link href="{{ asset('css/app.css') }}" rel="stylesheet">
     ```
    > * `<div class="wrapper">` 添加　`id="app"`  
    > * 最后面
    ```html
        <script src="{{ asset('js/app.js') }}"></script>
        <script src="{{ asset('vendor/arvin/js/admin.min.js') }}"></script>
    ```

#### 安装**element-ui**
1. 安装：　`npm i element-ui -D`
2. 配置：
    `resources/assets/js/app.js`里添加：
    ```javascript
      require('element-ui');
      import 'element-ui/lib/theme-default/index.css';
    ```
    
### 使用
1. 例如添加一个自定义**form**组件:
    1. 首先创建文件`resources/assets/js/components/Form.vue`,这里面就是用*element-ui*的组件了  
    2. 然后在`resources/assets/js/app.js`内添加组件`Vue.component('v-form', require('./components/Form.vue'))`;   
    3. 就可以在 blade　里面使用了`<v-form></v-form>`
2. 使用：
    ```blade
    @extends('arvin::layouts.default')
    @section('content')
        <v-form></v-form>
    @stop
    ```
    
