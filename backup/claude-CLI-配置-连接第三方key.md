## claude配置文件位置
**通过claude.exe程序运行claude**
_建议将claude.exe所在目录加入环境变量path，方便命令行启动claude_

`claude.exe` 默认读取 `%USERPROFILE%\.claude` 作为全局配置目录
win+r 打开 %USERPROFILE% ,查看`.claude`文件夹，其中包含了所有配置文件
如果不存在，`claude.exe`在启动时会自动创建 `.claude`文件夹

## 配置连接第三方key
在.claude文件夹下新建 settings.json , 内容包含
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "密钥",
    "ANTHROPIC_BASE_URL": "站点url"
  },
}
`claude.exe` 运行会自动读取ANTHROPIC_BASE_URL和ANTHROPIC_AUTH_TOKEN ，向指定站点发送请求与密钥，获得响应
（请求转发至 `ANTHROPIC_BASE_URL`，请求头携带 `ANTHROPIC_AUTH_TOKEN`）

## 自定义配置目录
配置环境变量 CLAUDE_CONFIG_DIR ，设置变量值，自定义更换配置目录位置

## 项目级配置目录
项目自定义配置，在项目目录下新建  .claude/settings.json , claude.exe会优先读取项目级配置