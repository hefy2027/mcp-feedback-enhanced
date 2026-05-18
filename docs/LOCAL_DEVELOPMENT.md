# 本地开发与 Cursor 集成指南

本文档说明如何从源码运行 mcp-feedback-enhanced 并在 Cursor 中使用。

## 前提条件

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) 包管理器（`pip install uv`）
- Cursor IDE

## 1. 克隆项目并安装依赖

```bash
git clone https://github.com/hefy2027/mcp-feedback-enhanced.git
cd mcp-feedback-enhanced

# 安装所有依赖（含开发依赖）
uv sync --dev
```

## 2. 验证安装

```bash
# 检查版本
uv run mcp-feedback-enhanced version

# 测试 Web UI 是否正常启动
uv run mcp-feedback-enhanced test --web
# 浏览器会自动打开，按 Ctrl+C 停止
```

## 3. 在 Cursor 中配置 MCP

在 Cursor 中打开 MCP 配置文件（Settings → MCP → 添加服务器），或直接编辑 `.cursor/mcp.json`。

### 基础模式（base64 图片，默认行为）

```json
{
  "mcpServers": {
    "mcp-feedback-enhanced": {
      "command": "uv",
      "args": [
        "run",
        "--directory", "/你的路径/mcp-feedback-enhanced",
        "mcp-feedback-enhanced"
      ],
      "timeout": 600,
      "autoApprove": ["interactive_feedback"]
    }
  }
}
```

### 文件模式（图片存磁盘，AI 通过文件路径读取）

```json
{
  "mcpServers": {
    "mcp-feedback-enhanced": {
      "command": "uv",
      "args": [
        "run",
        "--directory", "/你的路径/mcp-feedback-enhanced",
        "mcp-feedback-enhanced"
      ],
      "timeout": 600,
      "env": {
        "MCP_IMAGE_DIR": "/tmp/mcp-images",
        "MCP_IMAGE_MODE": "filepath"
      },
      "autoApprove": ["interactive_feedback"]
    }
  }
}
```

### 文件模式（图片存磁盘，AI 通过 URL 获取）

```json
{
  "mcpServers": {
    "mcp-feedback-enhanced": {
      "command": "uv",
      "args": [
        "run",
        "--directory", "/你的路径/mcp-feedback-enhanced",
        "mcp-feedback-enhanced"
      ],
      "timeout": 600,
      "env": {
        "MCP_IMAGE_DIR": "/tmp/mcp-images",
        "MCP_IMAGE_MODE": "url"
      },
      "autoApprove": ["interactive_feedback"]
    }
  }
}
```

> **Windows 用户提示**：路径使用正斜杠或双反斜杠，如 `"D:/tmp/mcp-images"` 或 `"D:\\tmp\\mcp-images"`。

## 4. 关键参数说明

### `--directory` 参数

使用 `uv run --directory` 指向源码目录，这样 `uv` 会自动使用该目录的 `pyproject.toml` 和虚拟环境运行项目。

### 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `MCP_DEBUG` | 调试模式，输出详细日志 | `false` |
| `MCP_WEB_HOST` | Web UI 绑定地址 | `127.0.0.1` |
| `MCP_WEB_PORT` | Web UI 端口 | `8765` |
| `MCP_IMAGE_DIR` | 图片存储目录（设置后启用文件模式） | 未设置（base64 模式） |
| `MCP_IMAGE_MODE` | AI 图片引用方式（文件模式下） | `filepath` |

### MCP_IMAGE_MODE 说明

- **`filepath`**（默认）：AI 收到本地文件路径，如 `请查看图片: /tmp/mcp-images/session-xxx/image1.png`。AI 用 Read 工具直接读取文件。
- **`url`**：AI 收到 HTTP URL，如 `请查看图片: http://127.0.0.1:8765/api/images/session-xxx/image1.png`。AI 用 WebFetch 工具获取。

## 5. 开发调试

```bash
# 启动调试模式
MCP_DEBUG=true uv run mcp-feedback-enhanced test --web

# 运行代码质量检查
uv run ruff check .
uv run ruff format .
uv run mypy

# 运行测试
uv run pytest
```

## 6. 项目结构概览

```
src/mcp_feedback_enhanced/
├── __init__.py          # 包入口
├── __main__.py          # CLI 入口
├── server.py            # MCP 服务器核心
├── utils/
│   ├── image_storage.py # 图片存储管理器（base64/文件双模式）
│   ├── error_handler.py # 错误处理
│   └── resource_manager.py
└── web/
    ├── main.py          # WebUIManager
    ├── routes/          # FastAPI 路由
    ├── models/          # 数据模型
    ├── static/          # 前端资源
    └── templates/       # Jinja2 模板
```
