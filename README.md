# Will's Blog

基于 [Hugo](https://gohugo.io/) 和 [Blowfish](https://blowfish.page/) 主题的个人博客，托管于 Azure Static Web Apps，通过 GitHub Actions 自动部署。

---

<details>
<summary><b>目录</b>（点击展开）</summary>

- [1. 功能特性](#1-功能特性)
- [2. 环境要求](#2-环境要求)
- [3. 本地配置](#3-本地配置)
- [4. 基本操作](#4-基本操作)
- [5. TRAE 集成技能](#5-trae-集成技能)
- [6. 部署](#6-部署)
- [7. 自定义配置](#7-自定义配置)
- [8. 常见问题](#8-常见问题)
- [9. 相关资源](#9-相关资源)
- [10. 许可](#10-许可)

</details>

---

## 1. 功能特性

- 使用 Blowfish 主题，支持深色/浅色模式切换
- 中英文双语支持
- 全文搜索
- 标签、分类、作者等分类体系
- 响应式设计，适配移动端
- 自动部署到 Azure Static Web Apps（国内访问更稳定）

## 2. 环境要求

- [Git](https://git-scm.com/downloads) — 版本管理
- [Hugo (Extended)](https://gohugo.io/installation/) — 静态站点生成器，需要 **Extended** 版本以支持 Sass/SCSS
- 代码编辑器（推荐 [VS Code](https://code.visualstudio.com/) 或 [TRAE](https://www.trae.ai/)）

## 3. 本地配置

克隆仓库并拉取子模块：

```bash
git clone --recurse-submodules https://github.com/will-last/will-hugo.github.io.git
cd will-hugo.github.io
```

> 如果已克隆但忘记加 `--recurse-submodules`，执行 `git submodule update --init --recursive`。

启动 Hugo 开发服务器实时预览修改：

```bash
hugo server --buildDrafts
```

启动后访问 http://localhost:1313 ，修改内容后浏览器会自动刷新。

## 4. 基本操作

**创建新文章：**

```bash
hugo new content posts/文章标题/index.md
```

生成的文件头包含元数据，将 `draft` 改为 `false` 即可发布：

```yaml
---
title: "文章标题"
date: 2026-01-01
draft: true
description: "文章描述"
tags: ["标签1", "标签2"]
categories: ["分类名"]
---
```

**文章编写：** 使用 Markdown 语法，图片放在 `assets/img/` 目录下，引用方式 `![描述](/img/图片名.png)`。Blowfish 支持丰富的短代码，详见 [Blowfish 文档](https://blowfish.page/docs/shortcodes/)。

**本地构建验证：**

```bash
hugo --gc --minify
```

构建产物在 `public/` 目录。

**目录结构：**

```
├── .trae/skills/              # TRAE 集成技能
├── config/_default/           # 站点配置
│   ├── hugo.toml              # 主配置
│   ├── params.toml            # 主题参数
│   ├── languages.*.toml       # 语言配置（中英文）
│   ├── markup.toml            # Markdown 渲染
│   └── menus.*.toml           # 菜单配置
├── content/                   # 文章内容
├── themes/blowfish/           # Blowfish 主题（子模块）
├── static/                    # 静态文件
├── assets/                    # 资源文件
└── .github/workflows/         # GitHub Actions 部署配置
```

## 5. TRAE 集成技能

本仓库内置了 **TRAE 集成技能**（位于 `.trae/skills/hugo-blowfish-blog/`），记录了从零搭建 Hugo + Blowfish 博客并部署的全流程。在 [TRAE IDE](https://www.trae.ai/) 中打开此项目后，可直接调用该技能自动完成搭建。

## 6. 部署

### 6.1 Azure Static Web Apps（当前方案）

博客托管在 Azure Static Web Apps（免费层），国内访问更稳定。推送到 `main` 分支后自动触发构建部署：

```bash
git add .
git commit -m "更新内容"
git push origin main
```

部署完成后访问：https://yellow-water-0e536b200.7.azurestaticapps.net

### 6.2 GitHub Pages（旧方案）

原 GitHub Pages 部署已在切换 Azure 后移除。如需重新启用，在仓库 Settings → Pages 中选择 GitHub Actions 作为部署源，并恢复 `.github/workflows/deploy.yml` 工作流。

### 6.3 部署流程

1. 推送到 `main` 分支
2. GitHub Actions 自动触发构建
3. Azure Static Web Apps 自动部署上线
4. 可在仓库 Actions 标签页查看构建状态

## 7. 自定义配置

**修改站点信息：** 编辑 `config/_default/hugo.toml`，修改 `baseURL` 和 `defaultContentLanguage`。

**修改个人资料：** 编辑 `config/_default/languages.zh-cn.toml` 中的 `[params.author]` 部分，包括名称、头像、简介和社交链接。

**修改主题配色：** 编辑 `config/_default/params.toml` 中的 `colorScheme` 字段，可选值包括 `blowfish`、`ocean`、`forest`、`fire`、`slate`、`noir` 等，完整列表见 `themes/blowfish/assets/css/schemes/`。

## 8. 常见问题

**本地预览看不到最新主题效果？** 确保主题子模块已正确拉取：`git submodule update --init --recursive`。

**构建报错？** 确认使用的是 Hugo **Extended** 版本（`hugo version`），并检查工作流中的 `HUGO_VERSION` 与本地版本一致。

**如何切换主题？** 删除 `themes/blowfish` 子模块，添加新的主题子模块，修改 `hugo.toml` 中的 `theme` 字段，根据新主题文档调整配置。

## 9. 相关资源

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [Blowfish 主题文档](https://blowfish.page/docs/)
- [Azure Static Web Apps 文档](https://learn.microsoft.com/zh-cn/azure/static-web-apps/)

## 10. 许可

本项目内容采用 MIT 许可协议。