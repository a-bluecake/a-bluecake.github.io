### frp（fast reverse proxy），中文叫快速反向代理，是一款开源、免费的内网穿透工具

## frp核心两个程序：frps（server 服务端）、frpc（client 客户端）

通常我们使用一个具有公网ip的主机作为服务器，部署frps
本地主机部署frpc，用于连接frps服务器，伪装成服务器上的端口，服务器将该端口收到的包发送给本地主机
即通过反向代理，让**外网可以访问没有公网 IP 的本地内网主机上的服务**

## frp的安装
frp_0.38.0_github版本
参考安装包链接：https://pan.baidu.com/s/1xfM1LJPO0gOHlisJluPnrQ?pwd=ya65
来自B站视频 BV13L411w7XU

**主要关注以下文件**
Linux版本  frps二进制运行文件与ini配置文件
<img width="507" height="42" alt="Image" src="https://github.com/user-attachments/assets/584dcc9a-3c70-46a4-807c-d657ad08abad" />

Windows版本   frpc  exe运行文件与ini配置文件
<img width="507" height="43" alt="Image" src="https://github.com/user-attachments/assets/ce12e437-129d-4d83-8789-71914901a836" />

## frp实现ssh远程连接本地主机

**用VScode打开Linux版本的frps.ini配置文件**
[common]
bind_port = 端口号1
auth_token = 随机字符串
**该配置的作用是创建服务端frps监听tcp端口**
后续客户端将创建隧道，通过该端口服务器进行反向代理

**用VScode打开Windows版本的frpc.ini配置文件**
[common]
server_addr = 服务器公网id
server_port = 服务器frps的监听tcp端口号
auth_token = 服务器的随机字符串
**该配置的作用是连接服务器frps**

**在frpc.ini配置文件中添加隧道**
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 本机ssh端口号
remote_port = 服务器frps反向代理端口号

** 客户端向服务器注册，服务器端frps将创建tcp连接监听remote_port端口 **
** 该端口的包将转发至local_ip:local_port **
** 隧道名取名ssh （任取名）**

** 连接命令：ssh 本地主机用户名@服务器公网ip -p remote_port **

完整流程 ：其它主机向 服务器的remote_port端口发送包时，被frps截获发给注册的frpc ，frpc将包发给local_ip:local_port 

包最后一步是本地frpc转发给本地ssh监听端口，所以local_ip填写127.0.0.1或本机实际ip地址

## frp创建内网实现游戏联机

**用VScode打开Linux版本的frps.ini配置文件**
[common]
bind_port = 端口号1
auth_token = 随机字符串
bind_udp_port =  端口号2

**用VScode打开Windows版本的frpc.ini配置文件**
[common]
server_addr = 服务器公网id
server_port = 服务器frps的监听tcp端口号
auth_token = 服务器的随机字符串

服务器使用端口号1与客户端进行连接，使用端口号2通过udp协议帮助客户端之间进行连接

对于房主
**在frpc.ini配置文件中添加隧道**
[game-host]
type = sudp
role = server
sk = 房间密码
local_ip = 0.0.0.0
local_port = 联机游戏端口号

对于访客
**在frpc.ini配置文件中添加隧道**
[visit-game]
type = sudp
role = visitor
server_name = game-host
sk = 房间密码
bind_addr = 127.0.0.1
bind_port = 联机游戏端口号

## 运行frp
对于服务器，首先创建文件夹，内部包含frps和frps.ini
在文件夹目录下，命令行输入 .\frps -c frps.ini

对于客户端，首先创建文件夹，内部包含frpc和frpc.ini
在文件夹目录下，命令行输入 .\frpc -c frpc.ini

ssh远程连接：
服务器开启frps后，客户端运行frpc，提示success，说明连接成功

** 之后该连接命令：ssh 本地主机用户名@服务器公网ip -p remote_port 可以成功使用**

创建内网实现游戏联机：
服务器开启frps后，
房主运行frpc，提示success
访客运行frpc，提示success，查看房主对应cmd出现提示 incoming a new work connection for sudp proxy
说明连接成功


建议调试完成后写start.bat启动脚本，方便运行
<img width="570" height="124" alt="Image" src="https://github.com/user-attachments/assets/8f37c237-3286-4ec3-bf2e-783c4bc88774" />

