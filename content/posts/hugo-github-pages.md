---
title: "用 Hugo + PaperMod 把博客挂到 GitHub Pages"
date: 2026-11-20T10:30:00+08:00
draft: false
tags: ["Hugo", "GitHub Pages", "建站"]
categories: ["技术"]
summary: "从零到上线：目录结构、主题配置、Actions 工作流，以及几个容易踩的坑。"
ShowToc: true
---

这个站就是按这套流程搭的：**Hugo 生成静态页 → PaperMod 提供主题 → GitHub Actions 构建 → GitHub Pages 托管**。整套东西没有服务器、没有数据库，一年维护成本接近于零。

## 一、目录结构

```text
MyWeb/
├── hugo.toml              # 全站配置
├── content/               # 所有 Markdown 都放这里
│   ├── _index.md          # 首页
│   ├── about.md           # 关于页
│   ├── search.md          # 搜索页
│   └── posts/             # 文章，每篇一个文件
│       └── hello-world.md
├── themes/
│   └── PaperMod/          # 主题（也可以改成 git submodule）
├── .github/workflows/
│   └── hugo.yml           # 自动构建 + 发布
└── public/                # 构建产物（不进仓库）
```

## 二、写一篇新文章

文件名就是 URL 的一部分。`content/posts/hugo-deploy.md` 会发布到 `/posts/hugo-deploy/`。

```bash
hugo new content posts/我的新文章.md
```

生成的文件默认 `draft: true`，写完了改成 `false` 才会进构建。本地预览时用 `-D` 也能看到草稿：

```bash
hugo server -D
```

## 三、配置文件里几个关键项

```toml
theme = "PaperMod"
baseURL = "https://example.github.io/"   # 部署时会被 Actions 覆盖

[params]
mainSections = ["posts"]     # 首页只列 posts 目录
ShowToc = true               # 自动生成目录
defaultTheme = "auto"        # 跟随系统深浅色

[outputs]
home = ["HTML", "RSS", "JSON"]   # 搜索功能依赖这里的 JSON
```

最后那条 `outputs` 特别容易漏。漏了不会报错，只是搜索框永远转圈。

## 四、GitHub Actions 工作流

`.github/workflows/hugo.yml` 做两件事：在 Ubuntu runner 上装 Hugo、构建，然后把 `public/` 作为 Pages 产物上传。

几个细节值得注意：

- `actions/configure-pages@v5` 会输出 `base_url`，构建时用 `--baseURL "${{ steps.pages.outputs.base_url }}/"` 传进去。**这样无论仓库叫 `xxx.github.io` 还是项目站 `xxx.github.io/blog`，都不用改配置。**
- 权限必须显式声明 `pages: write` 和 `id-token: write`，否则部署阶段会 401。
- `fetch-depth: 0` 是为了拿到完整提交历史，用于 `enableGitInfo` 之类的功能。

## 五、仓库设置（这一步最容易忘）

进仓库 **Settings → Pages**，把 **Source** 改成 **GitHub Actions**，不要选 "Deploy from a branch"。选错了工作流会跑成功但页面永远是 404。

然后推一次代码：

```bash
git add .
git commit -m "init blog"
git push origin main
```

第一次跑完，页面通常在 `https://<用户名>.github.io/<仓库名>/`。

## 六、踩过的坑

| 现象 | 原因 |
| --- | --- |
| 页面能开，但 CSS 全丢 | `baseURL` 少了结尾的 `/` |
| 文章列表空的 | `mainSections` 和实际目录名对不上 |
| 搜索框一直禁用 | 首页 `outputs` 没加 `JSON` |
| Actions 构建成功但 404 | Pages 的 Source 还是 "Deploy from a branch" |
| 中文标点被吞 | `hasCJKLanguage = true` 没开 |

## 七、以后要改的地方

现在这站还是"能用"的水平。后面打算补的：

- 文章封面图（PaperMod 支持 `cover.image`）
- 评论系统（utterances 基于 GitHub Issues，不需要额外服务）
- 归档页（主题自带 `layouts/archives.html`，加个菜单项就出来了）

先上线，再慢慢改。这大概是这类项目唯一正确的顺序。
