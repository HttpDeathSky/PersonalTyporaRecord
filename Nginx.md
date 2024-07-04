# Nginx

## 安装

参考链接：

http://t.csdn.cn/xbhF9

nginx官网：

https://nginx.org/

1. Linux安装

   - 下载nginx-1.xx.xx.tar.gz代码包|解压，标准存放路径`/usr/local/nginx`没有可以自己创建

   - 进入nginx-1.xx.xx目录

   - 添加3个模块

     - ```shell
       #
       ./configure --with-http_ssl_module --with-http_v2_module --with-stream
       #解释
       --with-http_ssl_module	# 配置HTTPS时使用
       --with-http_v2_module	# 配置GOLANG语言时使用
       --with-stream			# 启用TCP/UDP代理服务
       ```

   - 安装Nginx依赖模块1

     - ```shell
       yum install pcre pcre-devel -y
       ```

   - 安装依赖2

     - ```shell
       yum -y install make zlib zlib-devel gcc gcc-c++ libtool  openssl openssl-devel
       ```

   - 编译make

   - 安装make install

2. Docker安装

   - 下载docker镜像

     - ```shell
       docker pull nginx:1.20.1
       ```

   - 先启用一个测试容器

     - ```shell
       docker run -d  -p 80:80 --name nginx-test nginx:1.20.1
       ```

   - 创建需要挂载的宿主机文件夹data

     - ```shell
       mkdir -p /data
       ```

   - 复制容器内的nginx文件夹到宿主机的 /data 文件夹下，覆盖/data/nginx文件夹

     - ```
       docker cp nginx-test:/etc/nginx /data
       ```

   - 创建需要挂载的宿主机文件夹,这里的www文件夹是存放静态文件的，logs是存放日志文件，这两个文件夹后面会用到

     - ```shell
       mkdir -p /data/nginx/logs
       mkdir -p /data/nginx/www
       ```

   - 停止之前测试的nginx容器，要不重新启动新容器端口80会被占用

     - ```shell
       docker stop nginx-test
       ```

   - 启动新的nginx容器，并挂载宿主机配置文件以及日志和静态文件夹，如果不想以80端口启动的话需要修改/data/nginx/conf.d/default.conf下的默认80改成想要的端口即可

     - ```shell
       docker run -d -p 80:80 --name nginx -v /data/nginx/www:/usr/share/nginx/html -v /data/nginx/nginx.conf:/etc/nginx/nginx.conf -v /data/nginx/logs:/var/log/nginx nginx:1.20.1
       ```

     - ```shell
       #参数说明
       #挂载容器内的静态文件夹到宿主机/data/nginx/www下
       -v /data/nginx/www:/usr/share/nginx/html
       #挂载容器内的配置文件到宿主机/data/nginx/nginx.conf下
       -v /data/nginx/nginx.conf:/etc/nginx/nginx.conf
       #挂载容器内logs文件到宿主机/data/nginx/logs下
       -v /data/nginx/logs:/var/log/nginx
       ```

   - 下面给出我的default.conf配置内容跟如下

     - ```shell
       
       server {
           listen       80;
           listen  [::]:80;
           server_name  localhost;
       
           #access_log  /var/log/nginx/host.access.log  main;
           location /upload/ {
               root   /usr/share/nginx/html/;
       	autoindex on;
           }
           location / {
               root   /usr/share/nginx/html;
               index  index.html index.htm;
       	#如果vue-router使用的是history模式，try_files $uri $uri/ /index.html; 必须配置
       	try_files $uri $uri/ /index.html;
           }
       
           #error_page  404              /404.html;
       
           # redirect server error pages to the static page /50x.html
           #
           error_page   500 502 503 504  /50x.html;
           location = /50x.html {
               root   /usr/share/nginx/html;
           }
       
           # proxy the PHP scripts to Apache listening on 127.0.0.1:80
           #
           #location ~ \.php$ {
           #    proxy_pass   http://127.0.0.1;
           #}
       
           # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
           #
           #location ~ \.php$ {
           #    root           html;
           #    fastcgi_pass   127.0.0.1:9000;
           #    fastcgi_index  index.php;
           #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
           #    include        fastcgi_params;
           #}
       
           # deny access to .htaccess files, if Apache's document root
           # concurs with nginx's one
           #
           #location ~ /\.ht {
           #    deny  all;
           #}
       }
       ```

     - ```shell
       #参数说明：
       
       location /upload/ {
       root /usr/share/nginx/html/;
       autoindex on;
       }
       
       这段配置是图片代理：
       指定静态文件带http://ip/upoad/a.jpg访问的图片，比如说访问http://119.3.83.230/upload/test.jpeg 这就是访问宿主机/data/nginx/www/upload 文件夹下的test.jpeg图片
       location / {
       root /usr/share/nginx/html;
       index index.html index.htm;
       #如果vue-router使用的是history模式，try_files $uri $uri/ /index.html; 必须配置
       try_files $uri $uri/ /index.html;
       }
       这段配置用来部署前后端分离的vue项目：
       访问http://ip，会显示/data/nginx/www下的index.html，即这个www下文件夹里是vue打包后dist里的全部文件，特别注意的是不能直接放入整个dist文件夹，而是dist里的全部文件
       ```

   - 需要使用和我一样的default.conf,复制上面default.conf内容修改改/data/nginx/conf.d/default.conf即可

     - ```shell
       vim /data/nginx/conf.d/default.conf
       ```

   - 重启nginx容器就可以了

     - ```shell
       docker restart nginx
       ```

3. Windows安装

   - 下载nginx/Windows-1.xx.xx.zip后直接解压即可

## 概述

参考链接：

http://t.csdn.cn/UbLkE

1. 组成部分：

   - 全局块

     - 就是配置文件从头开始到events块之间的内容，`主要设置的是影响nginx服务器整体运行的配置指令`比如worker_process, 值越大，可以支持的并发处理量也越多，但是还是和服务器的硬件相关

   - events块

     - events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

       下述例子就表示每个 work process 支持的最大连接数为 1024
       这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置

   - http块

     - http全局块

       - http 全局块配置的指令包括文件引入、 MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

     - server块

       - 这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本

       - 每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机

       - 而每个 server 块也分为全局 server 块，以及可以同时包含多个 location 块

       - server全局块

         - 最常见的配置是本虚拟机主机的`监听配置和本虚拟主机的名称或 IP 配置`

         - ```nginx
           #这一行表示这个server块监听的端口是80，只要有请求访问了80端口，此server块就处理请求
           listen       80;
           #  表示这个server块代表的虚拟主机的名字
           server_name  localhost;
           ```

       - location块

         - 一个 server 块可以配置多个 location 块

         - `主要作用是根据请求地址路径的匹配，匹配成功进行特定的处理`

         - 这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行

         - ```nginx
           # 表示如果请求路径是/就是用这个location块进行处理
           location / {
                       root   html;
                       index  index.html index.htm;
                   }
           ```

2. nginx示例参考：

   - ```nginx
     # 全局快
     ------------------------------------------------------------------------------
     #user  nobody;
     worker_processes  1;
     
     #error_log  logs/error.log;
     #error_log  logs/error.log  notice;
     #error_log  logs/error.log  info;
     
     #pid        logs/nginx.pid;
     
     ------------------------------------------------------------------------------
     
     # events块
     events {
         worker_connections  1024;
     }
     
     # http块 
     http {
     ------------------------------------------------------------------------------# http全局块
         include       mime.types;
         default_type  application/octet-stream;
     
         #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
         #                  '$status $body_bytes_sent "$http_referer" '
         #                  '"$http_user_agent" "$http_x_forwarded_for"';
     
         #access_log  logs/access.log  main;
     
         sendfile        on;
         #tcp_nopush     on;
     
         #keepalive_timeout  0;
         keepalive_timeout  65;
     
         #gzip  on;        
     ------------------------------------------------------------------------------    
     # server块
     server {
     # server全局块
             listen       80;
             server_name  localhost;
     
             #charset koi8-r;
     
             #access_log  logs/host.access.log  main;
     
     # location块 可以配置多个location
             location / {
                 root   html;
                 index  index.html index.htm;
             }
     
             #error_page  404              /404.html;
     
             # redirect server error pages to the static page /50x.html
             #
             error_page   500 502 503 504  /50x.html;
             location = /50x.html {
                 root   html;
             }
     
             # proxy the PHP scripts to Apache listening on 127.0.0.1:80
             #
             location ~ \.php$ {
                 #proxy_pass 转发到指定地址
                 proxy_pass   http://127.0.0.1;
             }
     
             # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
             #
             #location ~ \.php$ {
             #    root           html;
             #    fastcgi_pass   127.0.0.1:9000;
             #    fastcgi_index  index.php;
             #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
             #    include        fastcgi_params;
             #}
     
             # deny access to .htaccess files, if Apache's document root
             # concurs with nginx's one
             #
             #location ~ /\.ht {
             #    deny  all;
             #}
     }
     	
     # 可以配置多个server块	
     
     }
     ```

3. 正向代理 | 反向代理

   - 正向代理代理的是`客户端`，需要为每一个客户端都做一个代理服务器，`客户端访问的路径是目标服务器`
   - 反向代理代理的是`真实服务器`，客户端不需要做任何的配置，`访问的路径是代理服务器`，由代理服务器将请求转发到真实服务器

4. 文件路径

   - nginx.html文件：/usr/share/nginx/html
   - nginx.conf文件：/etc/nginx/nginx.conf

5. root和alias区别

   - root:在匹配命中时会将浏览器访问上下文路径追加到root配置的文件地址后面（包含匹配路径）
   - alias:在匹配命中时会将匹配的路径之后的路径追加到alias配置的文件地址后面（不包含匹配路径）

6. 负载均衡

   - http模块下，server模块外配置，负载均衡服务

   - ```shell
     upstream my_server { //配置负载均衡服务
       # ip轮询添加此配置
       # ip_hash; 
       server localhost:8080;
       server localhost:8081;
       # 权重配置
       # server localhost:8082 weight=2;
       # 热备配置
       # server localhost:8083 backup; 
     }
     
     ```

   - 再在反向代理中配置上面的负责均衡服务

   - ```shell
     location /api/ {         
       proxy_pass http://my_server;  #请求转向mysvr 定义的服务器列表         
     }
     ```

7. 

## 指令

```shell
#docker nginx指令
#启动|停止|状态
docker run 'container'
docker stop 'container'
docker status 'container'
#docker run 指令
docker run 'container' --name nginx -p 80:80 -d -v /xxx/xxx/xx:/usr/share/html
#起名
--name 'xx'
#映射宿主端口和虚拟端口
-p 80:80
#后台运行
-d
#文件挂载
-v
#进入容器界面
docker exec -it `nginx-server` /bin/bash

#linux nginx指令
#检查配置文件
./nginx -t `/usr/local`/nginx/conf/nginx.conf
#版本
./nginx -v
#启动
./nginx
#重启
./nginx -s reload
#关闭（不推荐）
./nginx -s stop
#关闭nginx
./nginx -s quit


#test
docker run -d -p 80:80 --name nginx-server \
-v /program/dockerconfig/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /program/dockerconfig/nginx/html:/usr/share/nginx/html \
-v /program/dockerconfig/nginx/logs:/var/log/nginx \
-v /program/files/images:/store/images nginx
```















# 
