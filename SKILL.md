---
name: wechat-to-md
description: Convert authorized WeChat Official Account articles into local Markdown files with images and metadata.
---

# WeChat Article to Markdown

## What this tool does

This tool converts a WeChat Official Account article into a local Markdown folder:

- Markdown body with YAML frontmatter
- Locally saved images
- Preserved code blocks
- Audio/video references when available
- Source URL, title, author, and publish time

Use it for content you are allowed to save or process, such as your own articles, authorized materials, or team-approved public references.

## Requirements

- Python 3.10+
- Dependencies installed with `pip install -r requirements.txt`
- Network access and a browser runtime for rendering the article page

## CLI

Single article:

```bash
python main.py "https://mp.weixin.qq.com/s/ARTICLE_ID"
```

Batch from file:

```bash
python main.py -f urls.txt -o ./output -v
```

Useful options:

| Flag | Description |
| --- | --- |
| `-f FILE` | Text file with one URL per line |
| `-o DIR` | Output directory, default `./output` |
| `-c N` | Image download concurrency, default `5` |
| `--no-images` | Keep remote image URLs |
| `--no-headless` | Show browser window for manual verification |
| `--force` | Overwrite existing output |
| `--no-frontmatter` | Use blockquote metadata instead of YAML |
| `-v` | Verbose logging |

## MCP Server

Run:

```bash
python mcp_server.py
```

Tools:

- `convert_article(url, output_dir, download_images, concurrency, use_frontmatter)`
- `batch_convert(urls, output_dir, download_images, concurrency)`

Client configuration example:

```json
{
  "mcpServers": {
    "wechat-to-md": {
      "command": "python",
      "args": ["mcp_server.py"],
      "cwd": "<path-to-this-project>"
    }
  }
}
```

## Notes

- Only `https://mp.weixin.qq.com/` article URLs are accepted.
- If a verification page appears, use `--no-headless` for manual confirmation.
- The tool should not be used for bulk collection, access-restricted content processing, or redistribution of unauthorized content.
