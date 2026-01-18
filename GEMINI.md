# JachinShen 的个人博客

本项目是一个基于 Jekyll 构建并托管在 GitHub Pages 上的个人博客。主要包含技术文章、学习笔记和个人思考。

## 项目概览

- **静态网站生成器:** [Jekyll](https://jekyllrb.com/)
- **主题:** [so-simple-theme](https://mmistakes.github.io/so-simple-theme/) (远程主题)
- **托管平台:** GitHub Pages
- **核心技术:**
    - **Markdown 解析:** kramdown
    - **数学公式支持:** MathJax
    - **访问统计:** Google Analytics
    - **评论系统:** Disqus
    - **插件:** `jekyll-seo-tag`, `jekyll-sitemap`, `jekyll-feed`, `jekyll-paginate`

## 目录结构

- `_posts/`: 存放 Markdown 格式的博客文章。文件命名遵循 `YYYY-MM-DD-title.md` 格式。
- `_data/`:
    - `navigation.yml`: 定义站点的导航菜单。
    - `authors.yml`: 包含作者信息 (Jachin Shen)。
- `_includes/`: 自定义 HTML 代码片段 (例如 `scripts.html`)。
- `assets/`: 文章相关的资源文件，如图片、PDF 和代码片段，通常按日期或项目分类。
- `images/`: 站点通用图片，包括 Logo 和作者头像。
- `_config.yml`: Jekyll 的主配置文件，用于设置站点、主题和插件。
- `Gemfile`: Ruby 环境的依赖管理文件。

## 构建与运行

在本地运行博客进行开发：

1.  **安装依赖:**
    ```bash
    bundle install
    ```

2.  **启动 Jekyll 服务器:**
    ```bash
    bundle exec jekyll serve
    ```
    站点将在 `http://localhost:4000` 运行。

## 开发规范

- **创建新文章:**
    - 在 `_posts/` 目录下创建新文件，命名为 `YYYY-MM-DD-your-title.md`。
    - 确保 Front Matter 包含 `layout: post`, `title`, `date` 以及相关的 `categories` 和 `tags`。
- **资源管理:**
    - 将特定文章相关的图片或其他文件存储在 `assets/` 或 `images/` 下对应的子目录中。
- **导航管理:**
    - 修改 `_data/navigation.yml` 来添加或删除页眉/页脚中的链接。
- **Python 类型注解:** (来自 GEMINI.md 上下文) 如果项目中添加了 Python 脚本，优先使用现代 Python 类型注解，如 `list[str], dict[str]`。