ngnix是一款代理服务器软件，主要托管用户的web端80、443端口
是目前主流最常用的服务器软件
本文通过从简单到复杂的方式，一步步教会读者怎么写nginx配置

# nginx基础语法
## nginx的语法
nginx语法很简单，通常不需要额外的字符，使用空格分割命令和参数
一般来说，只需要记住以下三点：
普通行以 ; 结尾，
块 {} 结尾不需要任何字符,
**#** 是注释符号

## nginx简单代理转发请求
关键字：
server 定义一个服务块
listen 定义服务块监听的端口号
server_name 定义服务块监听的域名
location 匹配路由（即请求参url的拼接地址）
proxy_pass 向后端真实服务发送请求

由这些关键字可以写出一个极简单的配置：
server {
  listen 80;
  server_name a-bluecake.cc; "#这里写请求传入的域名"
  location / {
    proxy_pass http://127.0.0.1:8080;
  }
}
意思是监听a-bluecake.cc域名的80端口收到的请求，并将其转发给本机8080端口服务

## location匹配路由功能
在上面配置的基础上，可以写：
server {
  listen 80;
  server_name a-bluecake.cc; "#这里写请求传入的域名"
  location /users {
    proxy_pass http://127.0.0.1:8080;
  }
  location /mysql {
    proxy_pass http://127.0.0.1:3306;
  }
}
这样一来，
用户访问 http://a-bluecake.cc:80/mysql 或 http://a-bluecake.cc:80/users 时，将转发给不同的服务
由此可见，location负责匹配用户的访问路径，进行不同的操作

（注：浏览器默认http使用80端口，常写法是http://a-bluecake.cc/mysql ，不显示写上 :80 ，结果基本一致）

## nginx静态资源模式
关键字：alias root  index
alias 的作用是注明访问文件的路径
root 的作用是标注位置作为访问文件的根路径
index 用于匹配访问文件的名称
以如下配置举例：
server {
  listen 80;
  server_name a-bluecake.cc; "#这里写请求传入的域名"
  location /index {
    root /opt/common;
    index index.html error.html;
  }
  location /a {
    alias /opt/;
    index index.html error.html;
  }  
}
当用户访问 http://a-bluecake.cc/index , 服务器将在/opt/commom/index目录下寻找文件，找到index.html文件则返回 ，
找不到时其次寻找error.html文件并返回，如果都找不到则报错
当用户访问 http://a-bluecake.cc/a , 服务器将在/opt目录下寻找文件，找到index.html文件则返回 ，
找不到时其次寻找error.html文件并返回，如果都找不到则报错

# nginx进阶语法
## nginx 设置代理请求头
在一开始的简单配置中，存在一个坑：
server {
  listen 80;
  server_name a-bluecake.cc; "#这里写请求传入的域名"
  location / {
    proxy_pass http://127.0.0.1:8080;
  }
}
后端真实服务收到的请求信息由nginx发送, 后端无法获得真实请求里的一部分参数，
这些参数，nginx需要手动设置传给后端，如：
协议类型(http/https):  $scheme;
访问者IP：$remote_addr 或 $proxy_add_x_forwarded_for
访问者访问的域名： $host
访问者访问的端口号：$server_port









