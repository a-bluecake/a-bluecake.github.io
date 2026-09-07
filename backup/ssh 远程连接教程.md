# 使用OpenSSH完成ssh远程连接
OpenSSh是目前最主流、开源的远程连接工具，用于连接两台电脑
分为ssh-client客户端，ssh-server服务端，
客户端用于主动连接别的机器，服务端用于允许别的机器 SSH 连进本机

## 安装
windows11系统默认安装了ssh客户端，ssh服务端安装需要主动在 设置-添加可选功能 中选择安装
<img width="409" height="225" alt="Image" src="https://github.com/user-attachments/assets/7198e49f-0d4e-4b0a-b633-b9f05e8ae4ec" />

## 上述安装方法可能失败，可以前往下方链接下载离线安装压缩包，进行主动安装
https://github.com/PowerShell/Win32-OpenSSH/releases

 以OpenSSH-Win64.zip为例
下载完成后，解压路径到
C:\Program Files\OpenSSH-Win64
以管理员身份打开powershell
移动目录到OpenSSH-Win64根目录
执行ssh服务端安装脚本   .\install-sshd.ps1

提示成功后输入Get-Service sshd
出现如图提示说明成功
<img width="365" height="85" alt="Image" src="https://github.com/user-attachments/assets/90c221f0-5c4f-4034-af0e-eb92e206d6d2" />
打开服务，搜索OpenSSH SSH Server ，设置启动、开机自启
 OpenSSH Authentication Agent属于私钥管理工具，建议暂不使用
<img width="426" height="35" alt="Image" src="https://github.com/user-attachments/assets/2eaa2a9f-954c-4332-b59c-ce59fd5c47ad" />

手动下载安装调整环境变量
系统变量path中会新增一条 C:\Program Files\OpenSSH-Win64
而原电脑自带环境变量 C:\Windows\System32\OpenSSH\
调整C:\Program Files\OpenSSH-Win64 的上下位置至底端，电脑将优先使用原ssh.exe
建议不调整，统一使用自己下载的ssh版本

## ssh连接
命令行输入命令   ssh 用户名@主机地址
提示输入密码
成功后即连接成功

## 密钥免密连接
密钥生成示例 ssh-keygen -t ed25519 -f ~/.ssh/test_key
生成一对密钥
ssh-keygen 密钥生成工具
-t ed25519 使用ed25519算法加密
-f ~/.ssh/test_key 生成密钥路径
根据需求修改
生成密钥时弹出提示设置密钥密码，连续回车，不设置密码生成密钥
生成test_key.pub公钥与test_key私钥

<img width="521" height="49" alt="Image" src="https://github.com/user-attachments/assets/2aac66aa-55d1-4058-999d-c9ae6175e673" />

通常ssh服务端在本机家目录的.ssh文件夹里面存放一个authorized_keys文件
一行表示一个公钥
本机将生成的私钥保留，公钥保存到服务器authorized_keys文件中
将sshd_config 配置打开PubkeyAuthentication yes ，即可识别密钥登录

远程推送到服务器命令 ： ssh-copy-id -i 公钥.pub -p 端口号 用户名@公网IP
也可以手动复制黏贴到服务器的authorized_keys文件中

之后ssh连接使用命令
ssh -i 私钥路径 -p 端口号 用户名@公网IP
即可完成免密连接

## 自定义ssh服务端监听端口
对于windows
修改 C:\ProgramData\ssh 目录下sshd_config 配置，#Port22 删除注释符号#，数字改为想修改的端口号
如果提示不能保存，用管理员身份powershell打开当前目录配置文件
对于Linux系统
修改/etc/ssh/sshd_config 配置，#Port22 删除注释符号#，数字改为想修改的端口号

## 自定义ssh启动命令
修改 C:\Users\   “填写当前用户名”  \.ssh 下的config 配置
以一个ssh远程连接为例：
’
Host github.com             # ssh github.com 时自动使用下方配置
  HostName ssh.github.com          # 连接时ip地址或域名地址
  User git                                       # 连接时账号名
  Port 443                                      # 连接端口号
  IdentityFile ~/.ssh/git_github     # 私钥位置与私钥名
‘
当 ssh github.com 自动根据配置补全 ssh -i 私钥路径 -p 端口号 用户名@公网IP

###
(可以下载VSCode插件Remote - SSH ，通过vscode左侧远程资源管理器远程ssh连接，图形化查看远程主机文件)
