**`just-the-docs`** 是 GitHub 上极其火爆的一个 **Jekyll 主题**（拥有 10k+ Stars）。

它的核心定位不是那种花哨的“日记型”个人博客，而是一个**极其现代、清爽、专注于文档和知识库管理（Documentation Theme）**的专业主题。如果你想用 GitHub 做一个**个人的技术笔记库、项目文档、或者结构化的维基百科（Wiki）**，选它堪称完美。

因为它基于 Jekyll，所以**它享有 GitHub Pages 的原生支持**。这意味着你甚至不需要在本地配置任何 Node.js 或 Go 环境，直接在 GitHub 网页端就能完成搭建和发布。

以下是让你用最快速度把 `just-the-docs` 跑起来的两种方案：

---

### 方案一：零代码基础，直接通过 GitHub 网页模板秒级开通（最快 🚀）

官方非常贴心地提供了一个专门用来快速克隆的模板仓库。

#### 步骤 1：使用官方模板创建你的仓库

1. 打开 `just-the-docs` 的官方模板仓库：[just-the-docs-template](https://github.com/just-the-docs/just-the-docs-template)
2. 点击右上角绿色的 **"Use this template"** -> 选择 **"Create a new repository"**。
3. **关键命名：** * 如果你想让博客的首页网址是 `https://你的用户名.github.io/`，请将 Repository name 严格命名为：`你的用户名.github.io`。
* 如果你起成别的名字（比如 `my-notes`），那最终的网址就会是 `https://你的用户名.github.io/my-notes/`。


4. 勾选 **Public**（公开），点击 **Create repository**。

#### 步骤 2：激活 GitHub Pages 自动化部署

1. 进入你新创建的仓库，点击顶部菜单的 **Settings**。
2. 在左侧边栏找到 **Pages** 选项。
3. 在 **Build and deployment** 下方的 **Source** 下拉菜单中，保持默认的 **Deploy from a branch**。
4. 在 **Branch** 处，选择 **`main`** 分支，目录选择 **`/ (root)`**，然后点击 **Save**。

等待大约 1-2 分钟，刷新该页面，顶部就会出现一行绿色的：*“Your site is live at ...”*。点击链接，你的专业文档博客就已经上线了！

---

### 方案二：开发者推荐，本地运行与预览（适合高频写作）

如果你想在本地用 VS Code 写 Markdown 并实时预览效果，可以使用这个方法：

#### 步骤 1：克隆到本地

将你刚刚创建的仓库克隆下来：

```bash
git clone https://github.com/你的用户名/你的仓库名.git
cd 你的仓库名

```

#### 步骤 2：安装 Ruby 和 Bundler

Jekyll 依赖 Ruby 环境（Mac 通常自带，Windows 用户需要去 RubyInstaller 官网下载）。
在终端运行以下命令安装依赖：

```bash
gem install bundler
bundle install

```

#### 步骤 3：本地启动预览

```bash
bundle exec jekyll serve

```

启动后，控制台会输出一个地址（通常是 `http://localhost:4000`）。在浏览器打开它，你在本地修改任何 Markdown 文件，页面都会**实时刷新（Live Reload）**。

---

### 💡 `just-the-docs` 的核心用法与结构化配置

这个主题之所以强大，是因为它独特的 **目录树导航（Multi-level Navigation）** 管理。你只需要在 Markdown 文件的顶部（Front Matter）加几行参数，它就会自动帮你生成精美的左侧导航栏。

#### 1. 基础全局配置 (`_config.yml`)

打开根目录下的 `_config.yml`，修改基本信息：

```yaml
title: 我的技术知识库
description: 专注 AI 开发与基础面分析的个人笔记。

# 如果你的仓库名不是 "你的用户名.github.io"，这里必须填仓库名
baseurl: "" 

# 使用远端主题加载，确保 GitHub Pages 能直接编译
remote_theme: just-the-docs/just-the-docs

```

#### 2. 创建一个“顶级目录”页面 (例如：Python 笔记)

在根目录下新建一个 `python-notes.md`，内容如下：

```markdown
---
layout: default
title: Python 核心笔记       # 左侧导航栏显示的名称
nav_order: 1                # 排序，数字越小越靠前
has_children: true          # 声明：这个页面下面还包含子页面！
---

# Python 核心知识库
这里是我的 Python 学习主页...

```

#### 3. 创建一个“子页面/子文章” (例如：装饰器详解)

新建一个 `python-decorators.md`：

```markdown
---
layout: default
title: 深入理解装饰器
parent: Python 核心笔记      # 极其关键！名字必须和上面的顶级目录 title 完全一致
nav_order: 1                # 在子目录里的排序
---

# 深入理解装饰器
这里开始写你的正文内容，支持标准的 Markdown 语法、代码高亮...

```

只要你指定了 `parent`，`just-the-docs` 就会在左侧导航栏自动生成一个折叠菜单，把“深入理解装饰器”收纳到“Python 核心笔记”下方，瞬间拥有面包屑导航和专业 API 文档的质感！

接下来，你就可以在本地或者 GitHub 网页上愉快地通过添加 `.md` 文件来构建你自己的无限制、免费知识库了。
