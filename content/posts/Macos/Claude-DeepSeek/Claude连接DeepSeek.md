+++
date = '2026-05-09T00:00:00+08:00'
draft = false
title = 'Claude Code 连接 DeepSeek 完整教程'
description = '在 Mac 上使用 LiteLLM 把 Claude Code 请求转发到 DeepSeek API 的完整配置教程'
author = 'SmileSion'
tags = ["MacOs","Claude","DeepSeek","LiteLLM","教程"]
+++

# Claude Code 连接 DeepSeek 完整教程

Claude Code 默认走 Anthropic 的接口，如果想让它使用 DeepSeek 作为后端模型，可以在本机起一个 LiteLLM 代理：

```text
Claude Code  ->  LiteLLM  ->  DeepSeek API
```

简单理解：

- `Claude Code`：我们平时在终端里用的 `claude` 命令。
- `LiteLLM`：本地代理，把 Claude Code 发来的 Anthropic 格式请求转换成 DeepSeek 能理解的请求。
- `DeepSeek API`：真正负责推理和生成结果的模型服务。

本文以 macOS 为例，Windows / Linux 的思路也一样，只是安装命令略有区别。

> 说明：DeepSeek 官方现在已经提供 Anthropic 兼容接口，简单场景可以直接连 `https://api.deepseek.com/anthropic`。本文仍然使用 LiteLLM，是因为 LiteLLM 更方便做模型别名、统一密钥、日志、后续切换其他模型服务。

## 1. 准备工作

你需要准备：

- 一台能正常访问 DeepSeek API 的电脑。
- 一个 DeepSeek 平台账号。
- Python 3.10+，用于安装 LiteLLM。
- Node.js 18+，用于安装 Claude Code。
- 一个终端工具，macOS 自带 Terminal 或 iTerm2 都可以。

先检查本机环境：

```shell
python3 --version
node --version
npm --version
```

如果没有 Node.js，推荐用 Homebrew 安装：

```shell
brew install node
```

如果没有 Homebrew，可以先安装 Homebrew：

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 2. 申请 DeepSeek API Key

1. 打开 DeepSeek 开放平台：<https://platform.deepseek.com/>
2. 登录或注册账号。
3. 进入左侧的 `API Keys` 页面。
4. 点击 `Create API Key` 创建密钥。
5. 复制生成的 Key，格式一般类似：

```text
sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

注意事项：

- API Key 只会完整显示一次，复制后建议保存到密码管理器。
- 不要把 Key 写进博客、GitHub、截图或者公开配置文件。
- 使用 API 前需要确保账号里有可用额度。

DeepSeek 当前推荐的新模型名是：

| 用途 | 模型名 |
| --- | --- |
| 快速、便宜、日常代码问答 | `deepseek-v4-flash` |
| 更强推理、更适合复杂任务 | `deepseek-v4-pro` |

官方文档中也提到，旧模型名 `deepseek-chat` 和 `deepseek-reasoner` 会在 `2026-07-24` 废弃，所以新配置建议直接使用 `deepseek-v4-flash` 和 `deepseek-v4-pro`。

## 3. 配置 DeepSeek 环境变量

把 DeepSeek API Key 放到环境变量里，避免直接写死在配置文件中。

如果你用的是 macOS 默认的 zsh：

```shell
vim ~/.zshrc
```

在文件末尾添加：

```shell
export DEEPSEEK_API_KEY="sk-替换成你的DeepSeekKey"
```

让配置立即生效：

```shell
source ~/.zshrc
```

检查是否配置成功：

```shell
echo $DEEPSEEK_API_KEY
```

能看到 `sk-...` 开头的内容就说明环境变量已经生效。

## 4. 安装 Claude Code

Claude Code 官方要求 Node.js 18+。推荐使用 npm 安装：

```shell
npm install -g @anthropic-ai/claude-code
```

不要使用 `sudo npm install -g`，否则后面可能遇到权限问题。

安装完成后检查版本：

```shell
claude --version
```

正常情况下会输出 Claude Code 的版本号。

## 5. 安装 LiteLLM

推荐用 `uv` 安装 LiteLLM 的代理版本，隔离性更好：

```shell
brew install uv
uv tool install 'litellm[proxy]'
```

安装后检查：

```shell
litellm --version
```

如果你不想装 `uv`，也可以用 pip：

```shell
python3 -m pip install 'litellm[proxy]'
```

如果安装后提示找不到 `litellm` 命令，通常是 Python 的 bin 目录没有加入 `PATH`。可以先用下面的方式启动：

```shell
python3 -m litellm --help
```

## 6. 编写 LiteLLM 配置文件

创建配置目录：

```shell
mkdir -p ~/.litellm
vim ~/.litellm/config.yaml
```

写入下面的配置：

```yaml
model_list:
  - model_name: claude-fast
    litellm_params:
      model: deepseek/deepseek-v4-flash
      api_key: os.environ/DEEPSEEK_API_KEY
      api_base: https://api.deepseek.com

  - model_name: claude-pro
    litellm_params:
      model: deepseek/deepseek-v4-pro
      api_key: os.environ/DEEPSEEK_API_KEY
      api_base: https://api.deepseek.com

general_settings:
  master_key: sk-litellm-local

litellm_settings:
  drop_params: true
  set_verbose: true
```

这里有几个关键点：

- `model_name` 是暴露给 Claude Code 使用的名字，可以自己取。
- `model: deepseek/deepseek-v4-flash` 表示让 LiteLLM 调用 DeepSeek 的 `deepseek-v4-flash`。
- `api_key: os.environ/DEEPSEEK_API_KEY` 表示从环境变量读取 Key。
- `master_key` 是 Claude Code 连接 LiteLLM 时使用的本地代理密钥。
- `drop_params: true` 可以丢弃 DeepSeek 不支持的参数，减少兼容性报错。

本地自用时 `sk-litellm-local` 可以不改。如果你的代理会暴露到局域网或公网，请换成更长的随机字符串。

## 7. 启动 LiteLLM 代理

执行：

```shell
litellm --config ~/.litellm/config.yaml --port 4000
```

看到类似下面的输出就说明启动成功：

```text
Proxy running on http://0.0.0.0:4000
```

这个终端窗口先不要关闭。LiteLLM 需要一直运行，Claude Code 才能通过它转发请求。

## 8. 测试 LiteLLM 是否能调用 DeepSeek

另开一个终端，执行：

```shell
curl http://localhost:4000/v1/messages \
  -H "Authorization: Bearer sk-litellm-local" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-fast",
    "max_tokens": 300,
    "messages": [
      {
        "role": "user",
        "content": "用一句话介绍一下你自己"
      }
    ]
  }'
```

如果返回 JSON，并且里面有模型回复，说明 LiteLLM 到 DeepSeek 这一段已经通了。

如果报错，优先检查：

- `DEEPSEEK_API_KEY` 是否正确。
- DeepSeek 账号是否有余额。
- LiteLLM 终端里是否有具体错误日志。
- `model` 是否写成了配置里的 `claude-fast` 或 `claude-pro`。

## 9. 让 Claude Code 连接 LiteLLM

Claude Code 支持通过环境变量切换 API 地址。继续在新的终端里执行：

```shell
export ANTHROPIC_BASE_URL=http://localhost:4000
export ANTHROPIC_AUTH_TOKEN=sk-litellm-local
export ANTHROPIC_MODEL=claude-fast
```

然后启动 Claude Code：

```shell
claude
```

进入 Claude Code 后，可以问一句：

```text
你现在使用的后端模型是什么？
```

如果 LiteLLM 终端里出现请求日志，说明 Claude Code 的请求已经走到了本地代理。

如果你想切换到更强的模型：

```shell
export ANTHROPIC_MODEL=claude-pro
claude
```

这里的 `claude-fast` 和 `claude-pro` 并不是真正的 Claude 模型，而是我们在 LiteLLM 里给 DeepSeek 模型起的别名。

## 10. 写成一键启动脚本

每次手动 export 比较麻烦，可以写一个启动脚本。

创建文件：

```shell
vim ~/start-claude-deepseek.sh
```

写入：

```shell
#!/bin/zsh

export DEEPSEEK_API_KEY="sk-替换成你的DeepSeekKey"
export ANTHROPIC_BASE_URL="http://localhost:4000"
export ANTHROPIC_AUTH_TOKEN="sk-litellm-local"
export ANTHROPIC_MODEL="claude-fast"

claude
```

赋予执行权限：

```shell
chmod +x ~/start-claude-deepseek.sh
```

之后使用时：

1. 先启动 LiteLLM：

```shell
litellm --config ~/.litellm/config.yaml --port 4000
```

2. 再启动 Claude Code：

```shell
~/start-claude-deepseek.sh
```

更安全的写法是不要把 `DEEPSEEK_API_KEY` 写进脚本，而是保留在 `~/.zshrc` 里。这样脚本可以改成：

```shell
#!/bin/zsh

export ANTHROPIC_BASE_URL="http://localhost:4000"
export ANTHROPIC_AUTH_TOKEN="sk-litellm-local"
export ANTHROPIC_MODEL="claude-fast"

claude
```

## 11. 让 LiteLLM 在后台运行

如果不想一直开着一个终端窗口，可以用 `nohup`：

```shell
nohup litellm --config ~/.litellm/config.yaml --port 4000 > ~/.litellm/litellm.log 2>&1 &
```

查看日志：

```shell
tail -f ~/.litellm/litellm.log
```

查看进程：

```shell
ps aux | grep litellm
```

停止 LiteLLM：

```shell
pkill -f "litellm --config"
```

如果你经常使用，后续也可以用 `launchd`、`brew services` 或 `tmux` 来托管 LiteLLM。

## 12. 常见问题

### Claude Code 还是走官方 Claude

检查启动 `claude` 前是否设置了环境变量：

```shell
echo $ANTHROPIC_BASE_URL
echo $ANTHROPIC_AUTH_TOKEN
echo $ANTHROPIC_MODEL
```

必须在同一个终端会话里先 export，再运行 `claude`。

### 返回 401 Unauthorized

一般是 LiteLLM 的 `master_key` 和 `ANTHROPIC_AUTH_TOKEN` 对不上。

配置里是：

```yaml
general_settings:
  master_key: sk-litellm-local
```

那么终端里就必须是：

```shell
export ANTHROPIC_AUTH_TOKEN=sk-litellm-local
```

### 返回模型不存在

检查 Claude Code 使用的模型名是否是 LiteLLM 配置里的 `model_name`：

```shell
export ANTHROPIC_MODEL=claude-fast
```

而不是直接写：

```shell
export ANTHROPIC_MODEL=deepseek-v4-flash
```

除非你在 LiteLLM 的 `model_name` 里也这么配置。

### LiteLLM 报 DeepSeek 参数不支持

保留配置里的：

```yaml
litellm_settings:
  drop_params: true
```

Claude Code 可能会发送一些 DeepSeek 不支持的 Anthropic 参数，`drop_params` 可以让 LiteLLM 尽量丢弃这些额外参数。

### 请求很慢或中断

可以先换成 `claude-fast`，也就是 `deepseek-v4-flash`。如果任务复杂，再切换到 `claude-pro`。

```shell
export ANTHROPIC_MODEL=claude-fast
```

如果网络不稳定，也可以增加 Claude Code 的请求超时时间：

```shell
export API_TIMEOUT_MS=600000
```

## 13. 直接连接 DeepSeek 的简化方案

如果你不需要 LiteLLM 的模型别名和日志，也可以尝试直接连接 DeepSeek 的 Anthropic 兼容接口：

```shell
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=$DEEPSEEK_API_KEY
export ANTHROPIC_MODEL=deepseek-v4-flash
claude
```

但是我更推荐使用 LiteLLM：

- 后续切模型只需要改 `config.yaml`。
- 可以把不同模型映射成 `claude-fast`、`claude-pro` 这种固定名字。
- 可以继续扩展到 OpenAI、Gemini、OpenRouter、本地 Ollama 等服务。
- 排查问题时 LiteLLM 日志更直观。

## 14. 完整配置汇总

`~/.litellm/config.yaml`：

```yaml
model_list:
  - model_name: claude-fast
    litellm_params:
      model: deepseek/deepseek-v4-flash
      api_key: os.environ/DEEPSEEK_API_KEY
      api_base: https://api.deepseek.com

  - model_name: claude-pro
    litellm_params:
      model: deepseek/deepseek-v4-pro
      api_key: os.environ/DEEPSEEK_API_KEY
      api_base: https://api.deepseek.com

general_settings:
  master_key: sk-litellm-local

litellm_settings:
  drop_params: true
  set_verbose: true
```

启动 LiteLLM：

```shell
litellm --config ~/.litellm/config.yaml --port 4000
```

启动 Claude Code：

```shell
export ANTHROPIC_BASE_URL=http://localhost:4000
export ANTHROPIC_AUTH_TOKEN=sk-litellm-local
export ANTHROPIC_MODEL=claude-fast
claude
```

如果要使用更强模型：

```shell
export ANTHROPIC_MODEL=claude-pro
claude
```

## 参考链接

- DeepSeek API 文档：<https://api-docs.deepseek.com/>
- DeepSeek API Key 平台：<https://platform.deepseek.com/>
- Claude Code 安装文档：<https://docs.anthropic.com/en/docs/claude-code/getting-started>
- Claude Code 环境变量：<https://code.claude.com/docs/en/env-vars>
- LiteLLM 文档：<https://docs.litellm.ai/>
