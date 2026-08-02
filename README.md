# Will's Blog

基于 [Hugo](https://gohugo.io/) 和 [Blowfish](https://blowfish.page/) 主题的个人博客，托管于 GitHub Pages，通过 GitHub Actions 自动部署。

---

## 目录

- [1. 功能特性](#1-功能特性)
- [2. 环境要求](#2-环境要求)
- [3. 本地配置](#3-本地配置)
  - [3.1 克隆仓库](#31-克隆仓库)
  - [3.2 本地预览](#32-本地预览)
- [4. 基本操作](#4-基本操作)
  - [4.1 创建新文章](#41-创建新文章)
  - [4.2 文章编写](#42-文章编写)
  - [4.3 本地构建](#43-本地构建)
  - [4.4 目录结构](#44-目录结构)
- [5. TRAE 集成技能](#5-trae-集成技能)
- [6. 部署](#6-部署)
  - [6.1 自动部署（推荐）](#61-自动部署推荐)
  - [6.2 部署流程](#62-部署流程)
- [7. 自定义配置](#7-自定义配置)
  - [7.1 修改站点信息](#71-修改站点信息)
  - [7.2 修改个人资料](#72-修改个人资料)
  - [7.3 修改主题配色](#73-修改主题配色)
- [8. 常见问题](#8-常见问题)
  - [8.1 本地预览看不到最新主题效果？](#81-本地预览看不到最新主题效果)
  - [8.2 构建报错？](#82-构建报错)
  - [8.3 如何切换主题？](#83-如何切换主题)
- [9. 相关资源](#9-相关资源)
- [10. 许可](#10-许可)

---

## 1. 功能特性

- 使用 Blowfish 主题，支持深色/浅色模式切换
- 中英文双语支持
- 全文搜索
- 标签、分类、作者等分类体系
- 响应式设计，适配移动端
- 自动部署到 GitHub Pages

## 2. 环境要求

- [Git](https://git-scm.com/downloads) — 版本管理
- [Hugo (Extended)](https://gohugo.io/installation/) — 静态站点生成器，需要 **Extended** 版本以支持 Sass/SCSS
- 代码编辑器（推荐 [VS Code](https://code.visualstudio.com/) 或 [TRAE](https://www.trae.ai/)）

## 3. 本地配置

### 3.1 克隆仓库

```bash
git clone --recurse-submodules https://github.com/will-last/will-hugo.github.io.git
cd will-hugo.github.io
```

> `--recurse-submodules` 参数会同时拉取 Blowfish 主题子模块。如果已克隆但忘记加此参数，执行：
> ```bash
> git submodule update --init --recursive
> ```

### 3.2 本地预览

启动 Hugo 开发服务器，实时预览修改：

```bash
hugo server --buildDrafts
```

启动后访问 http://localhost:1313 即可看到博客。修改内容后浏览器会自动刷新。

## 4. 基本操作

### 4.1 创建新文章

```bash
hugo new content posts/文章标题/index.md
```

会在 `content/posts/文章标题/` 目录下生成一个 Markdown 文件，文件头包含以下元数据：

```yaml
---
title: "文章标题"
date: 2026-01-01
draft: true           # true 表示草稿，本地预览需加 --buildDrafts
description: "文章描述"
tags: ["标签1", "标签2"]
categories: ["分类名"]
---
```

将 `draft` 改为 `false` 即可发布。

### 4.2 文章编写

- 使用 Markdown 语法编写
- 文章图片放在 `assets/img/` 目录下，引用方式：`![图片描述](/img/图片名.png)`
- Blowfish 主题支持丰富的短代码（Shortcodes），如警告框、代码块、图表等，详见 [Blowfish 文档](https://blowfish.page/docs/shortcodes/)

### 4.3 本地构建

```bash
hugo --gc --minify
```

构建生成的静态文件在 `public/` 目录下，可用于本地验证最终效果。

### 4.4 目录结构

```
├── .trae/skills/              # TRAE 集成技能
│   └── hugo-blowfish-blog/    # 博客搭建工作流技能
├── config/_default/           # 站点配置
│   ├── hugo.toml              # 主配置
│   ├── params.toml            # 主题参数
│   ├── languages.en.toml      # 英文语言配置
│   ├── languages.zh-cn.toml   # 中文语言配置
│   ├── markup.toml            # Markdown 渲染配置
│   ├── menus.en.toml          # 英文菜单
│   └── menus.zh-cn.toml       # 中文菜单
├── content/                   # 文章内容
│   ├── _index.md              # 首页
│   └── posts/                 # 文章目录
├── themes/blowfish/           # Blowfish 主题（子模块）
├── static/                    # 静态文件
├── assets/                    # 资源文件（图片、CSS、JS）
└── .github/workflows/         # GitHub Actions 部署配置
```

## 5. TRAE 集成技能

本仓库内置了 **TRAE 集成技能**（位于 `.trae/skills/hugo-blowfish-blog/`），记录了从零搭建 Hugo + Blowfish 博客并部署到 GitHub Pages 的完整工作流。在 [TRAE IDE](https://www.trae.ai/) 中打开此项目后，可直接调用该技能自动完成博客搭建流程。

技能内容涵盖：

- Windows 环境检测与 PATH 配置
- Hugo 站点初始化与 Blowfish 主题安装
- 站点配置文件生成
- GitHub Actions 自动化部署配置
- README 文档生成规范
- 常见故障排查

## 6. 部署

### 6.1 自动部署（推荐）

代码推送到 `main` 分支后，GitHub Actions 会自动构建并部署到 GitHub Pages。

```bash
git add .
git commit -m "更新内容"
git push origin main
```

部署完成后访问：https://will-last.github.io/will-hugo.github.io/

### 6.2 部署流程

1. 推送到 `main` 分支
2. GitHub Actions 自动触发构建
3. 构建完成后部署到 GitHub Pages
4. 可在仓库 Actions 标签页查看构建状态

## 7. 自定义配置

### 7.1 修改站点信息

编辑 `config/_default/hugo.toml`：

```toml
baseURL = "https://will-last.github.io/will-hugo.github.io"
defaultContentLanguage = "zh-cn"
```

### 7.2 修改个人资料

编辑 `config/_default/languages.zh-cn.toml` 中的 `[params.author]` 部分：

```toml
[params.author]
  name = "Will"
  image = "img/blowfish_logo.png"
  headline = "个人博客"
  bio = "欢迎来到我的博客，记录技术与生活。"
  links = [
    { github = "https://github.com/will-last" },
  ]
```

### 7.3 修改主题配色

编辑 `config/_default/params.toml`，修改 `colorScheme` 字段：

```toml
colorScheme = "blowfish"  # 可选: blowfish, ocean, forest, fire, slate, noir 等
```

完整配色方案列表见 `themes/blowfish/assets/css/schemes/` 目录。

## 8. 常见问题

### 8.1 本地预览看不到最新主题效果？

确保主题子模块已正确拉取：

```bash
git submodule update --init --recursive
```

### 8.2 构建报错？

- 确认使用的是 Hugo **Extended** 版本：`hugo version`
- 确认 Hugo 版本与工作流中配置的一致（`HUGO_VERSION` in `.github/workflows/deploy.yml`）

### 8.3 如何切换主题？

Hugo 支持多种主题，更换主题的操作步骤：

1. 删除 `themes/blowfish` 子模块
2. 添加新的主题子模块
3. 修改 `config/_default/hugo.toml` 中的 `theme` 字段
4. 根据新主题的文档调整配置

## 9. 相关资源

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [Blowfish 主题文档](https://blowfish.page/docs/)
- [Blowfish GitHub 仓库](https://github.com/nunocoracao/blowfish)

## 10. 许可

本项目内容采用 MIT 许可协议。