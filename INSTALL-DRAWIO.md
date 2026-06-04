# Draw.io MCP & Skill 安装教程

> **致 Agent 的说明**：这份教程是为你编写的。请按顺序阅读，判断用户的环境，然后**直接执行**对应的安装步骤。无需询问用户每一步——除非遇到岔路口，请自主完成安装。

---

## 第一步：判断用户使用哪种 Claude 客户端

在开始安装之前，先确认用户的环境：

| 用户使用的工具 | 推荐方案 |
|---|---|
| Claude Code（命令行 / IDE 插件） | **方案 C：Skill 安装**（最轻量，无需 MCP） |
| Claude Desktop | **方案 B：MCP Tool Server** |
| Claude.ai 网页版 | **方案 A：MCP App Server（托管版）** 或 **方案 D：Project Instructions** |
| VS Code + MCP 支持 | **方案 A：MCP App Server（托管版）** |

> 如果不确定，优先选择 **方案 C（Skill）**，它对 Claude Code 用户零依赖、最易安装。

---

## 方案 A：MCP App Server（托管版，无需安装）

**适用于**：Claude.ai、VS Code 等支持 MCP Apps 协议的客户端。

**原理**：图表直接在聊天界面内嵌显示，不会打开新标签页。

### 安装步骤

在 Claude.ai 或兼容客户端中，添加以下远程 MCP 服务器地址：

```
https://mcp.draw.io/mcp
```

**Claude.ai 操作路径**：设置 → Integrations → Add MCP Server → 粘贴上方 URL。

安装后，你拥有两个工具：
- `create_diagram`：将 draw.io XML 渲染为对话内嵌的交互式图表
- `search_shapes`：搜索 10,000+ 个图形（AWS、Azure、GCP、Kubernetes、UML 等）

**验证**：让用户说"帮我画一个简单的流程图"，检查图表是否嵌入显示。

---

## 方案 B：MCP Tool Server（本地 npm 包）

**适用于**：Claude Desktop、Claude Code 及其他 MCP 客户端。

**原理**：Agent 生成图表后，自动在浏览器中打开 draw.io 编辑器。支持 XML、CSV、Mermaid 三种格式。

### 前置要求

确认 Node.js 已安装：

```bash
node --version   # 需要 v16+
npm --version
```

### Claude Desktop 安装

**第一步**：找到配置文件位置：
- macOS：`~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows：`%APPDATA%\Claude\claude_desktop_config.json`

**第二步**：在配置文件中添加以下内容（如文件不存在则新建）：

```json
{
  "mcpServers": {
    "drawio": {
      "command": "npx",
      "args": ["-y", "@drawio/mcp"]
    }
  }
}
```

**第三步**：重启 Claude Desktop。

### Claude Code 安装

执行以下命令：

```bash
claude mcp add drawio -- npx -y @drawio/mcp
```

或者手动编辑项目的 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "drawio": {
      "command": "npx",
      "args": ["-y", "@drawio/mcp"]
    }
  }
}
```

### 安装后可用的工具

| 工具 | 功能 | 推荐度 |
|---|---|---|
| `open_drawio_mermaid` | 用 Mermaid 语法生成图表 | ★★★ 首选 |
| `open_drawio_xml` | 用 mxGraphModel XML 生成图表 | ★★★ 精确布局 |
| `open_drawio_csv` | 用 CSV 数据生成图表（组织架构等） | ★★ 偶尔失败 |

**验证**：让用户说"用 draw.io 画一个登录流程图"，检查浏览器是否自动打开 draw.io 编辑器。

---

## 方案 C：Skill + CLI（推荐，Claude Code 专用）

**适用于**：Claude Code（命令行工具 / IDE 插件）。

**原理**：只需复制一个文件，无需配置 MCP Server。Agent 直接生成 `.drawio` 文件并在本地打开，或导出为 PNG/SVG/PDF，或生成浏览器 URL。

**优势**：
- 零服务器依赖
- 图表数据完全留在本地
- 支持导出为图片格式（需 draw.io Desktop）
- 无需 Node.js 额外安装（Claude Code 自带）

### 前置要求

- Claude Code 已安装（必须）
- draw.io Desktop（可选，仅 PNG/SVG/PDF 导出时需要）

安装 draw.io Desktop（可选）：
- macOS / Windows / Linux：https://github.com/jgraph/drawio-desktop/releases

### 安装 Skill 文件

**方法一：从本仓库克隆的目录安装（推荐）**

如果你已经克隆了 drawio-mcp 仓库（执行以下命令）：

```bash
git clone https://github.com/jgraph/drawio-mcp.git /tmp/drawio-mcp
```

然后复制 Skill 文件：

**全局安装**（对所有项目生效）：

```bash
mkdir -p ~/.claude/skills/drawio
cp /tmp/drawio-mcp/skill-cli/drawio/SKILL.md ~/.claude/skills/drawio/SKILL.md
```

**项目级安装**（仅对当前项目生效）：

```bash
mkdir -p .claude/skills/drawio
cp /tmp/drawio-mcp/skill-cli/drawio/SKILL.md .claude/skills/drawio/SKILL.md
```

**方法二：直接下载（无需克隆整个仓库）**

```bash
# 全局安装
mkdir -p ~/.claude/skills/drawio
curl -o ~/.claude/skills/drawio/SKILL.md \
  https://raw.githubusercontent.com/jgraph/drawio-mcp/main/skill-cli/drawio/SKILL.md

# 或项目级安装
mkdir -p .claude/skills/drawio
curl -o .claude/skills/drawio/SKILL.md \
  https://raw.githubusercontent.com/jgraph/drawio-mcp/main/skill-cli/drawio/SKILL.md
```

### 使用方式

安装完成后，在 Claude Code 中使用 `/drawio` 命令：

```
/drawio 画一个用户登录流程图
/drawio png 系统架构图           → 生成 architecture.drawio.png
/drawio svg 数据库 ER 图         → 生成 er-diagram.drawio.svg
/drawio pdf 项目流程总览          → 生成 overview.drawio.pdf
/drawio url 微服务架构图          → 在浏览器中打开，同时保留本地 .drawio 文件
```

### 输出格式说明

| 命令格式 | 输出 | 需要 draw.io Desktop |
|---|---|---|
| `/drawio 描述` | `.drawio` 文件，在 draw.io 中打开 | 否 |
| `/drawio png 描述` | `.drawio.png`（内嵌 XML，可再次编辑） | 是 |
| `/drawio svg 描述` | `.drawio.svg`（内嵌 XML，可再次编辑） | 是 |
| `/drawio pdf 描述` | `.drawio.pdf`（内嵌 XML，可再次编辑） | 是 |
| `/drawio url 描述` | 浏览器打开 app.diagrams.net + 本地 `.drawio` | 否 |

**验证**：在 Claude Code 中运行 `/drawio 画一个简单的流程图`，检查当前目录是否生成了 `.drawio` 文件并自动打开。

---

## 方案 D：Project Instructions（无需安装）

**适用于**：Claude.ai Projects（网页版），完全不想安装任何东西的用户。

**原理**：将 draw.io 使用说明粘贴到 Claude Project 的系统提示中，Claude 通过生成 URL 来创建图表。

### 安装步骤

1. 在 Claude.ai 中创建或打开一个 Project
2. 进入 Project 设置 → Instructions
3. 将以下内容粘贴进去：

```
When asked to create a diagram, use draw.io URL format. 
Generate the diagram as draw.io XML (mxGraphModel format), 
compress it with Python's zlib, base64-encode it, and 
produce a clickable https://app.diagrams.net/#create=... link.
```

完整的 Project Instructions 内容参见：https://github.com/jgraph/drawio-mcp/tree/main/project-instructions

**验证**：在该 Project 中说"帮我画一个流程图"，检查 Claude 是否返回了可点击的 draw.io 链接。

---

## 常见问题排查

### Skill 安装后 `/drawio` 命令无响应

检查文件是否在正确位置：

```bash
ls ~/.claude/skills/drawio/SKILL.md    # 全局
ls .claude/skills/drawio/SKILL.md     # 项目级
```

确认 Claude Code 版本支持 Skills（需要较新版本）。

### MCP Tool Server 安装后图表不打开

确认 Claude Desktop 已完全重启（不是刷新）。检查配置文件格式是否正确（JSON 不能有语法错误）。

运行以下命令测试 MCP 是否正常启动：

```bash
npx @drawio/mcp
```

正常情况下会输出 MCP 服务器已启动的日志，无报错即可。

### PNG/SVG/PDF 导出失败（Skill 方案）

draw.io Desktop CLI 未找到。确认安装路径：

| 系统 | 路径 |
|---|---|
| macOS | `/Applications/draw.io.app/Contents/MacOS/draw.io` |
| Linux | `which drawio` |
| Windows | `C:\Program Files\draw.io\draw.io.exe` |

如果未安装 draw.io Desktop，改用 `/drawio url 描述` 或 `/drawio 描述`（不需要 Desktop）。

### draw.io URL 在 Windows/WSL2 上打开失败

Windows 的 `cmd.exe` 会截断 URL 中 `#` 后面的内容。Skill 已内置处理方案（写入临时 `.url` 文件），正常应自动解决。若仍有问题，手动复制输出的 URL 在浏览器中打开。

---

## 数据隐私说明

| 方案 | 图表数据是否离开本机 |
|---|---|
| MCP App Server（托管版） | 是，发送至 draw.io 服务器 |
| MCP App Server（自托管） | 否 |
| MCP Tool Server | 否（数据在 URL fragment 中，浏览器不发送给服务器） |
| Skill + CLI | 否（完全本地处理） |

---

## 安装完成后

安装完成后，Agent 可以直接响应用户的自然语言图表需求，例如：

- "画一个微服务架构图"
- "帮我做一个用户注册的流程图"
- "生成一个数据库 ER 图"
- "把这个系统设计画成时序图"

无需用户手动写 XML 或学习 draw.io 语法。

---

*参考来源：[jgraph/drawio-mcp](https://github.com/jgraph/drawio-mcp)*
