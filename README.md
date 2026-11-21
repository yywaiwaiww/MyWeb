# 半页代码

个人博客站点，基于 **Hugo + PaperMod**，部署在 **GitHub Pages**。

- 构建：GitHub Actions（`.github/workflows/hugo.yml`）
- 内容：全部是 `content/` 下的 Markdown
- 无需服务器 / 数据库

---

## 一、本地预览

Hugo 已放在项目里（`.tools/hugo/hugo.exe`，未提交到仓库，避免 22MB 二进制入库）。

```powershell
cd D:\CODE\MyWeb
.\.tools\hugo\hugo.exe server -D
```

打开 <http://localhost:1313/>。`-D` 表示连草稿一起渲染，改文件会自动刷新。

只做一次完整构建（产物在 `public/`）：

```powershell
.\.tools\hugo\hugo.exe --gc --minify
```

`hugo.toml` 里 `baseURL = "/"` 是故意的：本地预览时链接走根相对路径，直接可用；部署时工作流用 `--baseURL` 注入真实地址，所以**不需要填自己的域名**。

> 换电脑或想全局安装：装 [Hugo extended](https://github.com/gohugoio/hugo/releases)（0.170.1 及以上），然后直接用 `hugo server -D`。必须是 **extended** 版，普通版缺 SCSS 支持。

---

## 二、写一篇新文章

```powershell
.\.tools\hugo\hugo.exe new content posts/文章名.md
```

生成的文件默认 `draft: true`，写完把这一行改成 `draft: false` 才会正式发布。

文章开头的 front matter：

```yaml
---
title: "标题"
date: 2026-11-20T10:30:00+08:00
draft: false
tags: ["标签A", "标签B"]
categories: ["技术"]
summary: "列表页显示的一句话摘要"
ShowToc: true          # 是否显示目录
---
```

文件名决定 URL：`content/posts/hugo-deploy.md` → `/posts/hugo-deploy/`。
想固定 URL，在 front matter 里加 `url: "/whatever/"`。

---

## 三、部署到 GitHub Pages

### 1. 建仓库并推送

需要一个 GitHub 账号。本机目前**没有安装 git**，先装一个（任选其一）：

```powershell
winget install --id Git.Git -e
```

或者装图形化的 [GitHub Desktop](https://desktop.github.com/)。

然后在 GitHub 上新建仓库（名字随意，比如 `MyWeb`），推上去：

```powershell
cd D:\CODE\MyWeb
git init
git add .
git commit -m "init blog"
git branch -M main
git remote add origin https://github.com/<你的用户名>/MyWeb.git
git push -u origin main
```

### 2. 打开 Pages

仓库 **Settings → Pages → Source**，把 **Deploy from a branch** 改成 **GitHub Actions**。

这一步不做，工作流会跑成功但页面永远 404 —— 是最常见的失败原因。

### 3. 等一分钟

Actions 里会看到 `Deploy Hugo site to Pages` 跑两次（build + deploy），完成后访问：

| 仓库类型 | 访问地址 |
| --- | --- |
| `MyWeb`（项目站） | `https://<用户名>.github.io/MyWeb/` |
| `<用户名>.github.io`（用户站） | `https://<用户名>.github.io/` |

站点的 `baseURL` **不用手动改**，工作流里的 `actions/configure-pages@v5` 会在构建时自动注入正确的地址。

---

## 四、上线前要改的占位内容

| 文件 | 改什么 |
| --- | --- |
| `hugo.toml` | `title`、`description`、`author`、`[params.homeInfoParams]` 里的自我介绍、`socialIcons` 里的 GitHub 链接 |
| `content/about.md` | 把中间那段「关于我」换成你自己的，补上联系方式 |
| `content/posts/*.md` | 三篇示例文章，可以删掉换成自己的 |

主题外观在 `hugo.toml` 的 `[params]` 里调，常用几个：

```toml
defaultTheme = "auto"        # auto / light / dark
ShowToc = true               # 文章目录
ShowReadingTime = true       # 阅读时长
ShowCodeCopyButtons = true   # 代码块复制按钮
hideSummary = false          # 列表是否显示摘要
```

---

## 五、目录说明

```text
MyWeb/
├── hugo.toml                 全站配置
├── content/
│   ├── about.md              关于页
│   ├── search.md             搜索页（layout: search）
│   └── posts/                文章
├── themes/PaperMod/          主题（改主题样式建议用 assets/css/extended/ 覆盖，别直接改这里）
├── assets/                   自定义 CSS 放这里（可选）
├── archetypes/default.md     新文章的默认模板
├── .github/workflows/hugo.yml  自动构建部署
├── .tools/hugo/hugo.exe      本地 Hugo（不提交）
└── public/                   构建产物（不提交）
```

想加自定义样式，在 `assets/css/extended/` 下建 `.css` 文件即可，主题会自动合并，升级主题时不会丢。

---

## 六、常见问题

**页面能开但样式全丢** —— `baseURL` 结尾少了 `/`，或 CSS 文件路径带了错误的子目录。

**文章列表是空的** —— `hugo.toml` 里 `mainSections` 必须和实际目录名一致（这里是 `posts`）。

**搜索框是灰的** —— 首页 `outputs` 少了 `JSON`，或 `content/search.md` 的 `layout` 不是 `search`。

**推上去没反应** —— 确认分支名是 `main`（工作流里写的是 `main`），或去 Actions 页面手动 `Run workflow`。