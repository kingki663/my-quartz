---
creation date: 2024-11-13 11:22
modification date: 2025-01-23 12:10
tags:
    - Nginx
---
Nginx (engine x) 是一个**高性能的HTTP和反向代理web服务器**，同时也提供了IMAP/POP3/SMTP服务，其主要为了解决C10K问题（Concurrent 10k，也即高并发）而设计，支持高达 50,000 个并发连接数的响应。

Nginx提供反向代理、负载均衡、虚拟主机等功能

## Docker快速体验Nginx
### 快速上手
1. 获取nginx官方镜像：`docker pull nginx`
2. 查看镜像：`docker images`
3. 测试镜像：
	1. 启动容器：`docker run --name nginx -p 9091:80 -d nginx:1.17.8`
	2. 查看容器：`docker ps`
	3. 访问`localhost:9091`，出现欢迎界面则成功

4. 容器部署
	1. 创建三个目录：
		- `mkdir -p ./nginx/html`
		- `mkdir -p ./nginx/logs`
		- `mkdir -p ./nginx/conf`
	2. 拷贝容器内Nginx默认配置文件到本地当前目录下的 conf 目录，容器 ID 可以查看 docker ps 命令输入中的第一列
		- 拷贝配置文件：`docker cp nginx:/etc/nginx/nginx.conf your-path\nginx\conf\nginx.conf`
		- 拷贝完配置文件后需要把测试运行的容器停掉，然后再删除容器 
			- `docker stop nginx` 
			- `docker rm nginx`
	3. 利用-v参数映射容器目录（数据卷）
		1. `docker run -d -p 9091:80 --name nginx -v E:\project\practice\docker-try\nginx\html:/usr/share/nginx/html -v E:\project\practice\docker-try\nginx\conf\nginx.conf:/etc/nginx/nginx.conf -v E:\project\practice\docker-try\nginx\logs:/var/log/nginx --privileged=true nginx:1.17.8`
	4. 映射后，若启动容器，本地主机的html目录中的index.html文件将变成欢迎页面

## Nginx服务常规配置
步骤：
1. 在`./nginx/html`创建html文件
2. 配置域名：修改hosts文件（win平台在：`C:\Windows\System32\drivers\etc\hosts`）
3. 修改nginx.conf文件
添加一个server段：
```conf
    # www.jaychou.com
    server {
        listen 80;
        server_name www.jaychou.com;
        # 注意此处根目录应该为容器内的根目录，由于我们进行了数据卷映射所以可以使用我们主机的目录
        root /usr/share/nginx/html/www.jaychou.com;

        location / {
            root /usr/share/nginx/html/www.jaychou.com;
            index index.html index.htm;
        }
    }
```

4. 重启docker，`docker restart <container-id>`
注意：需要关闭代理再访问网页`www.jaychou.com:9091`

## 代理模式
### 正向代理
正向代理流程：**客户端<->代理->服务端**
客户端知道代理的存在，而服务端不知道


### 反向代理
反向代理流程：**客户端->代理<->服务端**
服务端知道代理的存在而客户端不知道

### 反向代理的配置
主要通过配置`proxy_pass`这一项进行代理
nginx容器的`nginx.conf`文件配置如下：
```
    # 监听4000，然后代理访问3000
    server {
    listen 4000;
    server_name localhost;

    location /{
    # 反向代理的设置
    # 注意此处需要查看本机ip，只用localhost会默认查找容器内部的
        root /usr/share/nginx/html;
        index index.html;
        proxy_pass http://192.168.0.103:3000;
    }
```
1. 启动主服务器容器`cors-problem-cors-server`
2. 启动Nginx容器`cors-nginx`
3. 通过浏览器访问服务
参考：[[../Web-Dev/FrontEnd/其他/跨域及其解决方案#代理服务器(反向代理)|跨域及其解决方案]]

## 负载均衡
通过配置`upstream`选项配置`server`的weight来实现，`ip_hash`可以实现同一ip只访问一台服务器解决`Session`问题
例子：
```conf

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;
    upstream backend {
        ip_hash; 
        server 192.168.0.103:8888 weight=3;
        server 192.168.0.103:9999;
        server 192.168.0.103:11111;
    }

    include /etc/nginx/conf.d/*.conf;

    # www.jaychou.com
    server {
        listen 80;
        server_name www.jaychou.com;
        # 注意此处根目录应该为容器内的根目录，由于我们进行了数据卷映射所以可以使用我们主机的目录
        root /usr/share/nginx/html/www.jaychou.com;

        location / {
            root /usr/share/nginx/html/www.jaychou.com;
            index index.html index.htm;
        }

        location /app {
            proxy_pass http://backend;
        }
    }
    # kingki.com
    server {
        listen 80;
        server_name aloha.kingki.com;
        # 注意此处根目录应该为容器内的根目录，由于我们进行了数据卷映射所以可以使用我们主机的目录
        root /usr/share/nginx/html/aloha.kingki.com;

        location / {
            root /usr/share/nginx/html/aloha.kingki.com;
            index index.html index.htm;
        }
    }
}

```

参考：
[第十二节 Docker安装nginx以及部署 - 参码踪 (shenmazong.com)](https://www.shenmazong.com/blog/1402091109054885888)

[06.反向代理和负载均衡_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1mz4y1n7PQ?p=6&vd_source=dbefe8621d153f1b1aaef0768a993d25)
