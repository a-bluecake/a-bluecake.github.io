## headscale与tailscale
headscale用于自建私有控制服务器，其兼容 Tailscale 协议，用于替代 Tailscale 官方的控制服务器
你需要在一个拥有公网ip的服务器部署headscale

tailscale 负责主机互相连接，搭成局域网

**注：
使用客户端接入服务器时，只能通过https服务器接入，
为此，在服务器部署headscale服务时，需要为这个服务提供一个域名，
用于签订证书，获得https地址**

### 请使用大模型辅助安装
_全流程较复杂，建议使用agent助手进行辅助安装，
将下方流程发给你的agent作为参考即可_
手动安装请查询大模型各种配置文件写法

## 服务器部署headscale
**部署过程需要使用ngnix和docker，确保你已经安装好了这两个应用**

### 1、使用docker部署headscale

**目录结构准备：**
选择一个合适的工作目录，新建文件docker-compose.yml
在同目录下创建lib文件夹和config文件夹，config文件夹下新建文件config.yml
lib文件夹用于存放headscale 的**数据持久化目录**
config文件夹用于存放headscale的配置

**编写配置文件：**
编写当前目录下docker-compose.yml
在config目录下编写config.yml

**使用docker部署：**
切换到docker-compose.yml所在目录，
docker compose up -d  命令进行安装
docker compose ps  查看状态

通过IP加端口号（或域名）可以访问到你的服务即成功

### 2、使用ngnix进行反向代理并配置证书
**ngnix进行反向代理**
编写ngnix的配置文件即可

**配置证书**
推荐下载 acme.sh 脚本，签发证书后，在ngnix配置中添加证书位置
详细过程请查询相关命令

**验证**
浏览器访问你的域名，查看是否成功代理到你的headscale服务

### 3、使用headscale
docker exec headscale的意思是执行docker里面headsacle程序中的命令
**创建admin用户**
`docker exec headscale headscale users create admin`

**在 headscale 容器内创建预授权密钥**
user 1：归属用户 ID 为 1 的用户
reusable：密钥可重复使用（多台设备都能用这个 key 接入）
expiration 720h：有效期 720 小时（30 天）
`docker exec headscale headscale preauthkeys create --user 1 --reusable --expiration 720h`

**在 headscale 容器内，列出所有预授权密钥**
`docker exec headscale headscale preauthkeys list`

**在 headscale 容器内，列出全部用户**
`docker exec headscale headscale users list`

*保存好拿到用户admin的密钥*

## 主机安装tailscale
官网地址：`https://tailscale.com/download`
前往官网下载对应的安装包，下载完成后进行安装即可

**Windows系统**
安装完成后，打开cmd命令行
输入命令：`tailscale --version`
返回tailscale的版本号即成功
tailscale安装成功后不会在桌面显示快捷图标，使用搜索功能搜索tailscale找到该应用，手动创建快捷图标即可
tailscale会默认添加到环境变量中

**使用tailscale**
_注意不要点击启动tailscale，否则默认连接官方服务器_
用管理员模式打开命令行，输入命令：
将你在服务器上保存好的密钥替换到命令中后执行：
`tailscale up --login-server=https://hs.a-bluecake.cc --authkey=密钥 --accept-dns=false`

**查看连接状态**
tailscale status
通过会显示局域网内： ip 主机名 用户 操作系统

**断开连接**
tailscale down

**下次连接**
tailscale up



