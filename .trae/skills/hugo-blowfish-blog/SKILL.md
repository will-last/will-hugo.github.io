---
name: "hugo-blowfish-blog"
description: "在 Windows 上使用 Hugo + Blowfish 主题搭建博客，并部署到 GitHub Pages 的完整工作流。包括环境检测、站点初始化、主题安装、配置、Git 推送、Actions 部署和 README 生成。当用户要求搭建 Hugo 博客、使用 Blowfish 主题、或配置 GitHub Pages 部署时调用。"
---

# Hugo + Blowfish 博客搭建工作流

在 Windows 环境下，使用 Hugo 静态站点生成器和 Blowfish 主题，从零搭建个人博客并部署到 GitHub Pages 的完整流程。

## 关键注意事项

### Windows 终端 PATH 刷新

在 Windows 上，TRAE 的终端会话（PowerShell 5）**不会自动继承系统环境变量**。每次使用 `git`、`hugo` 等命令前，必须先手动刷新 PATH：

```powershell
$env:Path = [Environment]::GetEnvironmentVariable('Path', 'Machine') + ';' + [Environment]::GetEnvironmentVariable('Path', 'User')
```

> 此步骤必须在每次调用 RunCommand 时执行，否则会报 `CommandNotFoundException`。

### 适用于 GitHub Pages 项目站点

本工作流创建的仓库是 **项目站点**（`username.github.io/repo-name`），访问地址为 `https://username.github.io/repo-name/`。如需**用户/组织站点**（`username.github.io`），仓库名必须为 `username.github.io` 且 `baseURL` 设为 `https://username.github.io/`。

## 工作流步骤

### 1. 环境检测

```powershell
# 刷新 PATH
$env:Path = [Environment]::GetEnvironmentVariable('Path', 'Machine') + ';' + [Environment]::GetEnvironmentVariable('Path', 'User')

# 检查 Git
git --version
# 预期输出: git version 2.xx.x.windows.x

# 检查 Hugo
hugo version
# 预期输出: hugo v0.xxx.x+extended windows/amd64
# 必须为 Extended 版本以支持 Sass/SCSS

# 检查 GitHub CLI（可选，用于创建仓库）
gh --version
```

### 2. 初始化 Hugo 站点

```powershell
# 在目标目录下初始化 Hugo 站点
hugo new site . --force
```

### 3. 初始化 Git 仓库

```powershell
git init
# 将默认分支名改为 main
git branch -m master main
```

### 4. 安装 Blowfish 主题（作为 Git 子模块）

```powershell
git submodule add https://github.com/nunocoracao/blowfish.git themes/blowfish
```

### 5. 创建配置目录结构

```powershell
New-Item -ItemType Directory -Force -Path 'config\_default'
New-Item -ItemType Directory -Force -Path '.github\workflows'
New-Item -ItemType Directory -Force -Path 'content\posts'
New-Item -ItemType Directory -Force -Path 'static'
New-Item -ItemType Directory -Force -Path 'assets\img'
```

### 6. 创建配置文件

创建以下配置文件，参考 Blowfish 主题的 `themes/blowfish/exampleSite/config/_default/` 中的示例：

| 文件 | 用途 | 关键配置项 |
|------|------|-----------|
| `config/_default/hugo.toml` | 站点主配置 | `theme = "blowfish"`, `baseURL`, `defaultContentLanguage`, `hasCJKLanguage`, `[outputs]`, `[taxonomies]` |
| `config/_default/params.toml` | 主题参数 | `colorScheme`, `defaultAppearance`, `homepage.layout`, `mainSections`, `[article]`, `[list]`, `[header]`, `[footer]` |
| `config/_default/languages.zh-cn.toml` | 中文语言配置 | `locale = "zh-cn"`, `[params.author]` |
| `config/_default/languages.en.toml` | 英文语言配置 | `locale = "en"`, `[params.author]` |
| `config/_default/markup.toml` | Markdown 渲染 | `[goldmark.renderer] unsafe = true`, `[tableOfContents]` |
| `config/_default/menus.zh-cn.toml` | 中文菜单 | `[[main]]`, `[[footer]]` |
| `config/_default/menus.en.toml` | 英文菜单 | `[[main]]`, `[[footer]]` |

**关键配置示例** (`hugo.toml`)：

```toml
theme = "blowfish"
baseURL = "https://will-last.github.io/will-hugo.github.io"
defaultContentLanguage = "zh-cn"
hasCJKLanguage = true

[outputs]
  home = ["HTML", "RSS", "JSON"]

[taxonomies]
  tag = "tags"
  category = "categories"
  author = "authors"
  series = "series"
```

**主题参数示例** (`params.toml`)：

```toml
colorScheme = "blowfish"
defaultAppearance = "dark"
autoSwitchAppearance = true
enableSearch = true
mainSections = ["posts"]

[homepage]
  layout = "profile"
  showRecent = true
  showRecentItems = 6
```

### 7. 删除默认的根 hugo.toml

```powershell
Remove-Item hugo.toml
```

### 8. 创建 .gitignore

```gitignore
public/
.hugo_build.lock
resources/_gen/
.idea/
.vscode/
*.swp
*.swo
.DS_Store
Thumbs.db
*.log
```

### 9. 创建 GitHub Actions 部署工作流

创建 `.github/workflows/deploy.yml`，使用 GitHub Actions 构建并部署到 GitHub Pages：

```yaml
name: Deploy Hugo Site to GitHub Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.162.0    # 匹配本地 Hugo 版本
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 10. 创建示例内容

**首页** (`content/_index.md`):

```markdown
---
title: "欢迎来到我的博客"
description: "基于 Hugo 和 Blowfish 主题的个人博客"
---

欢迎来到我的博客！
```

**示例文章** (`content/posts/hello-world.md`):

```markdown
---
title: "Hello World"
date: 2026-08-02
draft: false
description: "第一篇文章"
tags: ["hello", "blog"]
categories: ["生活"]
---

## 你好，世界！

这是我的第一篇博客文章。
```

### 11. 本地构建验证

```powershell
hugo --gc --minify
```

检查输出是否包含 Pages 统计信息（中英文页面数量），确认无错误。构建产物在 `public/` 目录。

### 12. Git 提交与推送

```powershell
git add -A
git commit -m "Initial commit: Hugo blog with Blowfish theme"
git remote add origin https://github.com/{username}/{repo}.git
git push -u origin main
```

### 13. 生成 README 文档

生成包含以下内容的 `README.md`：

- 项目简介
- 环境要求（Git、Hugo Extended）
- 本地配置步骤（克隆仓库、拉取子模块、启动预览）
- 基本操作（创建文章、编写内容、本地构建、目录结构）
- TRAE 集成技能说明（介绍 `.trae/skills/` 目录的用途）
- 部署指南（自动部署流程）
- 自定义配置（站点信息、个人资料、主题配色）
- 常见问题

**README 排版要求：**
- 如果内容较长（超过 5 个一级章节），必须添加**目录（Table of Contents）**，位于标题下方、正文之前
- 所有章节标题必须使用**数字编号**（如 `## 1. 功能特性`、`### 3.1 克隆仓库`），方便读者定位
- 目录中的链接必须使用 Markdown 锚点格式，与标题编号对应（如 `#1-功能特性`、`#31-克隆仓库`）
- 目录必须使用 `<details>` 和 `<summary>` 标签包裹，实现**可折叠**效果，默认收起状态。示例：
  ```html
  <details>
  <summary><b>目录</b>（点击展开）</summary>

  - [1. 章节一](#1-章节一)
  - [2. 章节二](#2-章节二)

  </details>
  ```
- 目录与正文之间用 `---` 分隔线隔开

### 14. 提交并推送 README 与 TRAE 技能

```powershell
git add README.md .trae/
git commit -m "Add README documentation and TRAE integration skill"
git push
```

> TRAE 技能文件（`.trae/skills/`）应与项目代码一起推送至 GitHub，方便其他开发者或后续使用 TRAE IDE 时直接调用。

## 用户确认步骤

执行过程中需要用户确认：

1. **GitHub 仓库名称** — 询问用户仓库名（如 `username/repo-name`）
2. **GitHub 仓库创建** — 告知用户在 GitHub 上创建空仓库（不初始化任何文件）
3. **GitHub Pages 设置** — 告知用户在 Settings → Pages 中选择 **GitHub Actions** 作为部署源

## 故障排查

### 终端找不到命令

```powershell
# 每次新建终端会话时，先执行：
$env:Path = [Environment]::GetEnvironmentVariable('Path', 'Machine') + ';' + [Environment]::GetEnvironmentVariable('Path', 'User')
```

### 主题子模块未正确拉取

```bash
git submodule update --init --recursive
```

### Hugo 构建报错

- 确认使用 Extended 版本：`hugo version`
- 确认工作流中的 `HUGO_VERSION` 与本地版本一致
- 确认 `config/_default/hugo.toml` 中 `theme = "blowfish"` 已设置
- 确认 `markup.toml` 中 `[goldmark.renderer] unsafe = true`