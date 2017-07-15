---
title: gitlab搭建笔记
date: 2017-07-11 15:20:21
categories: 编程
tags: gitlab
---

### 安装篇(docker安装)
1. `sudo apt install docker`
2. `sudo apt install docker-compose`
3. 新建 docker配置文件 ``docker-compose.yml`,配置如下：  
<!-- more -->
```yml
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com' # 此处填写服务器的 ip
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'https://gitlab.example.com' # 同理，填写 ip
      # Add any other gitlab.rb configuration here, each on its own line
  ports:
    - '80:80'
    - '443:443'
    - '22:22' # 格式为： 本机端口：docker端口， 如果后面运行端口被占用，可以修改此处配置
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
```
  执行`docker-compose up -d`启动 （不加 `-d` 可以查看启动过程日志）
4. 启动完毕即可以登录上面配置的 ip地址访问gitlab web端(默认端口 80)

### 配置篇
#### 去除注册功能
1. 这个可以直接在web端，通过管理员身份进行设置  
关闭`Sign-up enabled` 选项即可

#### 修改docker配置　　
1. 首先运行：`docker ps` 查看容器名称   
2. 修改配置：`docker exec -it 容器名称  vi /etc/gitlab/gitlab.rb`  
3. 重载配置：`docker exec -it 容器名称  gitlab-ctl reconfigure`  

#### 配置通知邮箱,这里采用*smtp*邮箱： 
> smtp相关部分，例如配置qq、163等，参照 [这里](https://docs.gitlab.com.cn/omnibus/settings/smtp.html#outlook)  
    
gmail 如下：  
```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp-mail.outlook.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "username@outlook.com"
gitlab_rails['smtp_password'] = "password"
gitlab_rails['smtp_domain'] = "smtp-mail.outlook.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
```
#### 验证邮箱是否配置成功：
1. 运行：`docker exec -it 容器名称  gitlab-rails console`  
> irb(main):003:0> Notify.test_email('destination_email@address.com', 'Message Subject', 'Message Body').deliver_now

#### 错误
###### docker相关错误
![](docker-compose.png)

###### outlook邮箱配置错误
- unable to send and recivie emails error code: 535 5.0.0 Authentication Failed code(535)  
**解决：** https://answers.microsoft.com/en-us/outlook_com/forum/oemail-osend/unable-to-send-and-recivie-emails-error-code-535/15db149b-403a-4de7-8ef1-6cdeda9327de
