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
```
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
```
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

关键字：
proxy_set_header 设置请求头参数

server {
  listen 80;
  server_name a-bluecake.cc; "#这里写请求传入的域名"
  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Photo $scheme;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
后端收到的请求里则带有这些写好的请求体参数

设置nginx与后端服务的连接等待时间（写在location的操作块内，不写则默认为60秒）：
 proxy_connect_timeout 60s  tcp连接需要在60秒内完成
 proxy_read_timeout 60s 后端60秒未响应则断开
 proxy_send_timeout 60s 前端发送数据卡住60秒则断开

## nginx 配置https
网站可能需要使用https安全访问，则nginx需要配置监听443端口，
原80端口的服务交付443端口处理，此时原80端口用于配置续签证书和转发请求到443端口

### 签发证书
https访问的前提是配置好证书：
ACME协议（自动化证书管理环境）：客户端与CA的交流协议
CA：证书颁发机构
CA有多个，这里介绍Let's Encrypt ，它是一个免费的CA
Let's Encrypt 有着封装好申请证书过程命令的客户端工具 ：certbot / acme.sh

过程详解：
1、本地生成密钥对
本地使用密钥生成工具生成一对密钥，作为账户密钥
2、注册账号
向Let's Encrypt 指定的url地址发送post请求，携带符合要求的payload，该请求用私钥加密，同时公钥在请求中传给Let's Encrypt，即完成注册
Let's Encrypt 会返回一个url地址，之后请求会在头部带上这个地址，Let's Encrypt 验证存在账号后，允许使用各种服务
3、验证域名属于本人
向Let's Encrypt 发送请求，签一张证书，请求体内带上自己的域名
Let's Encrypt进行验证，最常见的验证方式是：
在 http://域名/.well-known/acme-challenge/<token> 放指定内容：
    1、Let's Encrypt生成一个随机token，传给客户端
    2、客户端创建一个文本文件，文件名为<token>，位于/.well-known/acme-challenge/目录下，文件内容包含token 和 账户公钥
    3、完成后，向Let's Encrypt 发送请求，通知其开始验证
    4、验证到存在该文件并符合要求，Let's Encrypt将该账户此域名的签发证书状态改为同意
4、申请签发证书
本地再次使用密钥生成工具生成一对密钥，作为证书密钥
向Let's Encrypt发送CSR请求，CSR中包含了申请域名与证书公钥，用证书私钥签名
Let's Encrypt检查域名，检查签名与公钥是否匹配，确定后生成证书，证书保存了服务器公钥并用CA私钥签名，有效期为90天
服务器向Let's Encrypt申请下载证书，之后向客户端发送请求时附带证书，一般需要设置有效期过自动续签

对于申请证书，只需了解如果使用申请证书工具即可

### nginx配置申请证书

nginx需要在80端口留下一个CA验证的路径，用于申请证书与续签
关键路径：/.well-known/acme-challenge/
最简单的写法：
server {
    listen 80;
    server_name a-bluecake.cc;
    location /.well-known/acme-challenge/ {
        root /ca;
    }
}
CA访问时，url是一个文件的地址，文件存在自动返回，使用root控制文件存放的根地址即可
(这里注意虽然也是静态资源模式，但是不适用index匹配文件名)
但是如果该路径不存在验证文件，该请求不会终止，建议加上终止
关键字：try_files 匹配文件，区别于index，当访问路径是文件时也能生效
参数: $uri  url的路径部分 , =404 返回404
常见写法：
server {
    listen 80;
    server_name a-bluecake.cc;
    location /.well-known/acme-challenge/ {
        root /ca;
        try_files $uri =404;
    }
}
寻找文件$uri ，找不到再返回 404，nginx在返回404时，就会终止匹配请求

### nginx 80端写法
80端只需要完成两个服务，提供证书验证路径，将转发进的请求重定向到443端口

关键字 return 重定向：
return 状态码 重定向地址
通常301表示永久重定向，302表示临时重定向
当执行return时，nginx会向浏览器返回状态码，同时返回重定向地址，浏览器用重定向地址再访问
完整写法：
```
server {
    listen 80;
    server_name a-bluecake.cc;
    location /.well-known/acme-challenge/ {
        root /ca;
        try_files $uri =404;
    }
    location / {
        return 301 https://$host$request_uri;
    }
}
```
### nginx 443端写法
443端需要另写一个server处理
https使用ssl（tls）握手，需要配置ssl处理
ssl on; 开启ssl，现在通常使用 listen ssl; 的简化写法
ssl_certificate   签发证书的地址
ssl_certificate_key  服务器私钥的地址
```
server {
    listen 443 ssl;
    server_name a-bluecake.cc;

    ssl_certificate      /xxx/fullchain.pem; 
    ssl_certificate_key  /xxx/privkey.pem;  

    location /api {
        proxy_pass 127.0.0.1:8080;
    }
}
```
### 完整申请证书与nginx配置示范

使用acme.sh申请证书：

下载安装acme.sh脚本
curl -L https://get.acme.sh -o get-acme.sh

执行完该命令后，工作目录下获得一个文件get-acme.sh
执行该文件
sh get-acme.sh

脚本会自动安装好acme.sh客户端
安装完成后，刷新环境变量
source ~/.bashrc

验证是否成功安装，如果输出版本号则安装成功
acme.sh --version

切换CA服务商为Let's Encrypt
acme.sh --set-default-ca --server letsencrypt

申请证书：
nginx需提取配置好CA访问路径，例：
server {
    listen 80;
    server_name a-bluecake.cc;
    location /.well-known/acme-challenge/ {
        root /ca;
        try_files $uri =404;
    }
}
申请证书命令
`acme.sh --issue -d a-bluecake.cc --webroot /ca `
--issue表示申请证书
-d 后参数表示申请域名，可以为多个域名申请同一个证书
--webroot 后参数表示文件根路径，与nginx里面的root的路径需要一致
acme.sh会在目的地址临时生成文件，通过验证后自动删除
acme.sh通常会自动在临期一个月时自动续签证书

导出证书与服务器私钥
建议在/ca目录下新建一个ssl文件夹，用于存放CA证书与服务器私钥
acme.sh --install-cert -d a-bluecake.cc --key-file /ca/ssl/privkey.pem --fullchain-file /ca/ssl/fullchain.pem
--install-cert 表示导出一份复制件，用于交付代理服务器
-d 后参数表示目标域名
--key-file 后参数表示私钥地址
--fullchain-file 后参数表示证书地址

将https加上后
nginx完整配置
```
http {
  server {
    listen 80;
    server_name a-bluecake.cc;
    location /.well-known/acme-challenge/ {
      root /ca;
      try_files $uri =404;
    } 
    location / {
      return 301 https://$host$request_uri;
    }
  }
  server{
    listen 443 ssl;
    server_name a-bluecake.cc;
    ssl_certificate /ca/ssl/fullchain.pem;
    ssl_certificate_key /ca/ssl/privkey.pem;
    location / {
      proxy_pass 127.0.0.1:8080;
      proxy_set_header Host $host;
    }
  }
}
```










