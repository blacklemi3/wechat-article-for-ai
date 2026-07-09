# wechat-article-for-ai

把微信公众号文章整理成适合本地归档和 AI 阅读的 Markdown 文件。

这个工具会打开文章页面，读取正文、标题、作者、发布时间、图片和代码块，然后输出一个本地目录：

```text
output/
  文章标题/
    文章标题.md
    images/
      img_001.png
      img_002.jpg
```

生成的 Markdown 可以放进个人知识库、笔记系统、RAG 资料库，或者继续交给 AI 做摘要、检索和结构化整理。

GitHub：<https://github.com/blacklemi3/wechat-article-for-ai>

## 它做了什么

一篇微信文章在网页里通常不是一段干净文本，而是混着页面样式、懒加载图片、代码块、音视频占位、脚本和各种页面元素。这个工具把这些内容拆出来，再重新整理成本地文件。

流程可以理解成：

```text
微信公众号文章链接
        |
        v
浏览器加载页面
        |
        v
提取标题、作者、时间、正文、图片、代码块
        |
        v
下载图片并替换成本地路径
        |
        v
生成 Markdown + images 目录
```

## 主要功能

- 单篇或批量转换微信公众号文章。
- 生成带 YAML frontmatter 的 Markdown。
- 下载正文图片到本地 `images/` 目录。
- 保留代码块，并清理常见的行号和样式噪音。
- 提取音频、视频等媒体引用。
- 支持 CLI 使用，也可以作为 MCP server 接入本地 AI 工具。
- 遇到验证页或异常页面时返回明确提示，便于人工处理。

## 使用边界

请只处理你有权保存、归档或继续使用的文章内容，例如：

- 自己发布的文章。
- 已获得授权保存的文章。
- 团队内部允许归档的公开材料。

这个工具不适合用于大规模抓取、处理访问受限内容、复制未授权内容或重新分发他人文章。遇到平台验证、访问限制或内容不可用时，应停止或改为人工确认。

## 安装

需要 Python 3.10+。

```bash
git clone https://github.com/blacklemi3/wechat-article-for-ai.git
cd wechat-article-for-ai
pip install -r requirements.txt
```

首次运行时，浏览器运行环境可能会自动下载所需组件。

## CLI 使用

转换单篇文章：

```bash
python main.py "https://mp.weixin.qq.com/s/ARTICLE_ID"
```

从文件批量读取链接：

```bash
python main.py -f urls.txt -o ./output -v
```

`urls.txt` 示例：

```text
# one URL per line
https://mp.weixin.qq.com/s/ARTICLE_ID_1
https://mp.weixin.qq.com/s/ARTICLE_ID_2
```

常用参数：

| 参数 | 说明 |
| --- | --- |
| `urls` | 一个或多个微信公众号文章链接 |
| `-f, --file FILE` | 从文本文件读取链接 |
| `-o, --output DIR` | 输出目录，默认 `./output` |
| `-c, --concurrency N` | 图片下载并发数，默认 `5` |
| `--no-images` | 不下载图片，Markdown 中保留远程图片地址 |
| `--no-headless` | 显示浏览器窗口，便于人工处理验证页 |
| `--force` | 覆盖已有输出 |
| `--no-frontmatter` | 不使用 YAML frontmatter，改为普通引用块元数据 |
| `-v, --verbose` | 输出调试日志 |

## 输出示例

Markdown 开头会类似这样：

```markdown
---
title: 示例文章标题
author: 示例作者
date: 2026-07-08 10:00:00
source: https://mp.weixin.qq.com/s/ARTICLE_ID
---

# 示例文章标题

这里是转换后的正文。

![图片](images/img_001.png)
```

## MCP server

这个项目也可以作为 MCP server 运行，让本地 AI 工具调用转换功能。

```bash
python mcp_server.py
```

提供两个工具：

- `convert_article`：转换单篇文章。
- `batch_convert`：批量转换多篇文章。

MCP 客户端配置示例：

```json
{
  "mcpServers": {
    "wechat-to-md": {
      "command": "python",
      "args": ["mcp_server.py"],
      "cwd": "/path/to/wechat-article-for-ai"
    }
  }
}
```

## 项目结构

```text
wechat_to_md/
  cli.py          # 命令行入口和批量处理
  scraper.py      # 浏览器加载页面、重试、验证页识别
  parser.py       # 提取标题、作者、正文、图片、代码块、媒体引用
  converter.py    # HTML 转 Markdown、frontmatter、图片路径替换
  downloader.py   # 异步下载图片
  mcp_server.py   # MCP server 工具入口
main.py           # CLI 入口
mcp_server.py     # MCP server 入口
SKILL.md          # 本地 AI 工具的技能说明
```

## 当前验证

本地已验证：

```bash
python main.py --help
```

结果：命令行参数可以正常显示。

## 常见情况

| 情况 | 处理方式 |
| --- | --- |
| 出现验证页或环境异常 | 使用 `--no-headless` 打开浏览器窗口，人工确认后再继续 |
| 内容为空 | 可能是页面结构变化、文章不可访问或访问频率受限，稍后重试 |
| 图片下载失败 | Markdown 会保留远程图片链接，可以用 `--force` 重新运行 |
| 输出目录已存在 | 默认跳过，使用 `--force` 覆盖 |

## License

MIT. See [LICENSE](LICENSE).
