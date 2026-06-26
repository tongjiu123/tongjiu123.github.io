# Blog of Yang

个人技术博客，基于 [Hexo](https://hexo.io/) + [NexT](https://theme-next.js.org/) 主题，部署在 GitHub Pages。

**在线地址**: [https://yangzongyou.com](https://yangzongyou.com)

---

## 技术栈

- **框架**: Hexo 7
- **主题**: NexT v8.27 (Gemini scheme)
- **部署**: GitHub Pages (`gh-pages` 分支)
- **域名**: yangzongyou.com
- **功能**: 本地搜索、阅读进度条、不蒜子访问统计、代码复制、书签

---

## 目录结构

```
blog/
├── _config.yml          # Hexo 主配置
├── _config.next.yml     # NexT 主题配置
├── source/
│   ├── _posts/          # 博客文章 (Markdown)
│   │   ├── BEVFusion.md
│   │   ├── CoBEVT.md
│   │   ├── VICOD.md
│   │   ├── deeptime.md
│   │   ├── lstfsurvey.md
│   │   ├── scaleOIJ.md
│   │   ├── mininet1.md
│   │   └── sftp-sql-1.md
│   ├── about/           # About 页面
│   ├── tags/            # 标签页
│   ├── categories/      # 分类页
│   ├── images/          # 全局图片资源
│   └── CNAME            # 自定义域名
├── scaffolds/           # 文章模板
├── package.json
└── .github/workflows/   # GitHub Actions 自动部署
```

---

## 常用命令

```bash
# 安装依赖（首次或换电脑时）
npm install

# 新建文章
npx hexo new "文章标题"

# 本地预览（http://localhost:4000）
npx hexo server

# 清理缓存 + 部署到 GitHub Pages
npx hexo clean && npx hexo deploy

# 仅生成静态文件（不部署）
npx hexo generate
```

---

## 写文章

### 1. 创建新文章

```bash
npx hexo new "My New Post"
```

会在 `source/_posts/` 下生成 `My-New-Post.md` 和同名资源文件夹。

### 2. 编辑文章

打开生成的 Markdown 文件，编辑 front-matter 和正文：

```markdown
---
title: My New Post
date: 2026-02-09
categories:
  - Paper
tags:
  - LSTF
  - Deep Learning
---

文章摘要，显示在首页。

<!-- more -->

正文内容...
```

- `<!-- more -->` 之前的内容会作为摘要显示在首页
- 图片放在同名文件夹中，用相对路径引用：`![描述](image.png)`

### 3. 分类和标签

- **categories**: 文章分类（如 Paper, problem）
- **tags**: 文章标签（如 LSTF, Auto_vehicle, Streaming）

### 4. 预览和部署

```bash
# 本地预览
npx hexo server

# 确认无误后部署
npx hexo clean && npx hexo deploy
```

---

## 配置修改

### 站点配置 (`_config.yml`)

- **title/subtitle/description**: 站点基本信息
- **url**: 站点 URL
- **deploy**: 部署目标仓库和分支

### 主题配置 (`_config.next.yml`)

- **scheme**: 主题方案（Muse/Mist/Pisces/Gemini）
- **menu**: 导航菜单
- **avatar**: 头像
- **social**: 社交链接
- **local_search**: 本地搜索开关

---

## 域名与 DNS

- **GitHub 仓库**: [tongjiu123/tongjiu123.github.io](https://github.com/tongjiu123/tongjiu123.github.io)
- **GitHub Pages 源**: `gh-pages` 分支
- **自定义域名**: yangzongyou.com

DNS 记录：

| 类型 | 主机记录 | 记录值 |
|------|---------|--------|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | tongjiu123.github.io |

---

## 备注

- 源码在 `main` 分支，静态文件通过 `hexo deploy` 推送到 `gh-pages` 分支
- `.github/workflows/deploy.yml` 也配置了 GitHub Actions 自动构建（push 到 main 时触发）
- 文章图片使用 post asset folder 模式，每篇文章的图片放在同名文件夹中
