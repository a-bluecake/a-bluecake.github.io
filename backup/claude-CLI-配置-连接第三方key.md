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

# 第一次启动claude 卡住
在国内环境，第一次使用启动claude时，可能不读取settings.json的内容，选择自动连接官网，
因为连接不上官网而卡住，此时可以手动设置环境变量（任意值或者空值）：
set ANTHROPIC_BASE_URL=
set ANTHROPIC_AUTH_TOKEN=
再启动claude，开始初始化
这次初始化就不会自动连接官网，完成初始化后会在配置目录下自动生成 .claude.json 文件，下次启动跳过初始化

也可以在配置目录下手动新建 .claude.json 文件，添加
  {"hasCompletedOnboarding": true}
再启动claude，进入第一次初始化环节，完成初始化后会在配置目录下自动补全 .claude.json 文件

## settings.json更多字段配置
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "密钥",
    "ANTHROPIC_BASE_URL": "站点url",
    "ANTHROPIC_MODEL": "默认模型",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "opus模型",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "sonnet模型"
  },
    "model": "进入claude后切换到该模型",
  "availableModels": [   <注释：切换模型时的可选项>
    "opus",
    "sonnet"
  ],
}
