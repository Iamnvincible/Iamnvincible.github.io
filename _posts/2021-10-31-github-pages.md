<!--
 * @Author: Jie Lin
 * @Date: 2021-10-31 22:38:26
 * @LastEditTime: 2021-11-06 21:13:41
 * @LastEditors: Jie Lin
 * @Description: Github pages 创建步骤
 * @FilePath: /1st_step/github_pages.md
 * 
-->
# GitHub Pages 托管静态博客
在没有独立服务器的情况下，使用 GitHub Pages 托管一个静态网站作为博客是一种不错的选择。不过静态网站只能展示页面，没有数据库，需要一点额外操作才能启用评论等独立服务器用户拥有的高级功能。

首先需要在 GitHub 上创建一个仓库，网站展示的页面来源于仓库中特定目录下的 Markdown 文件，只需要在仓库中上传 Markdown 格式的文章，GitHub Pages 就能将文章转换成对应的网页。一篇文章生成后，目录、首页也会同时更新。这样，不需要复杂的建站步骤，每个 GitHub 用户都能拥有个人博客了！

可以从 [快速入门](https://docs.github.com/en/pages/quickstart) 或 [GitHub Pages 创建指南](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) 开始探索。需要注意的是，仓库名最好是你的 GitHub 用户名，这样静态网站的网址就会是 `username.github.io`，否则，这个网址会包含整个仓库名称，很长，不容易记忆。快速入门会让你手动添加文件到仓库中，然后就能通过特定链接访问了，但这样不能预览网站结果，没有目录，自定义操作需要修改 `_config.yml` 文件实现。为了能在发布网站前预览网站效果，我们需要使用 Jekyll。Jekyll 是 ruby 语言编写的静态网站生成器， 受到 GitHub Pages 内建支持。

为了满足更多静态网站的自定义需求，需要在本地搭建 Jekyll 环境，对网站样式满意之后将所有 Jekyll 配置上传到仓库，GitHub Pages 会根据配置生成和本地同样的网站样式。



## 使用 Jekyll 创建的步骤

### 准备 Jekyll 环境

Jekyll 本是一个通用的静态网站生成工具，但为匹配 GitHub Pages 的要求，需要使用 github-pages 包来创建 GitH站点。
由于依赖关系较为复杂，使用 GitHub Pages 提供的 Docker 镜像 [pages-gem](https://github.com/github/pages-gem) 来避免环境依赖问题。

### 克隆仓库

将仓库克隆到本地，使用下面的命令创建一个空白的分支，用于存放网站内容。
```bash
git checkout --orphan gh-pages
```

### 使用 Jekyll 创建站点

空分支创建完成后，使用 Jekyll 新建站点。

```bash
jekyll new --skip-bundle .
```
修改 Gemfile，向其中添加 github-pages 包信息，让 Jekyll 处理包依赖关系。
```bash
bundle install
```

### 运行站点
```bash
bundle exec jekyll serve -H 0.0.0.0
```



未完待续。
