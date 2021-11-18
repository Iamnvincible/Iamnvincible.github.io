---
layout: post
title: "在 GitHub Pages 上搭建博客"
categories: GitHub-Pagges
tags: github pages
permalink: /1/
---

# GitHub Pages 托管静态博客

在没有独立服务器的情况下，使用 GitHub Pages 托管一个静态网站作为博客是一种不错的选择。不过静态网站只能展示页面，没有数据库，需要一点额外操作才能启用评论等独立服务器用户拥有的高级功能。

首先需要在 GitHub 上创建一个仓库，网站展示的页面来源于仓库中特定目录下的 Markdown 文件，只需要在仓库中上传 Markdown 格式的文章，GitHub Pages 就能将文章转换成对应的网页。一篇文章生成后，目录、首页也会同时更新。这样，不需要复杂的建站步骤，每个 GitHub 用户都能拥有个人博客了！

可以从 [快速入门](https://docs.github.com/en/pages/quickstart) 或 [GitHub Pages 创建指南](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) 开始探索。需要注意的是，仓库名最好是你的 GitHub 用户名，这样静态网站的网址就会是 `username.github.io`，否则，这个网址会包含整个仓库名称，很长，不容易记忆。快速入门会让你手动添加文件到仓库中，然后就能通过特定链接访问了，但这样不能预览网站结果，没有目录，自定义操作需要修改 `_config.yml` 文件实现。为了能在发布网站前预览网站效果，我们需要使用 Jekyll。Jekyll 是 ruby 语言编写的静态网站生成器， 受到 GitHub Pages 内建支持。

为了满足更多静态网站的自定义需求，需要在本地搭建 Jekyll 环境，对网站样式满意之后将所有 Jekyll 配置上传到仓库，GitHub Pages 会根据配置生成和本地同样的网站样式。



## 使用 Jekyll 设置 GitHub Pages 站点

### 准备 Jekyll 环境

Jekyll 是一个通用的静态网站生成工具，但 GitHub Pages 后端生成内容时使用了[特定版本](https://pages.github.com/versions/)，为了能让本地预览效果 GitHub Pages 呈现的效果一致，需要安装与 GitHub Pages 后端一致的版本。

由于依赖关系较为复杂，可以使用 GitHub Pages 提供的 Docker 镜像 [pages-gem](https://github.com/github/pages-gem) 来避免环境依赖问题。该 Docker 镜像构建后会得到 2 个镜像，gh-pages 和 ruby 2.7.3。其中 gh-pages 镜像可以在运行时指定一个 Jekyll 项目目录，容器会以该目录为源生成网站内容，并且运行容器时会自动端口映射，浏览器访问本地 4000 端口即可查看。ruby 2.7.3 镜像有匹配版本的 ruby 和 Jekyll 环境，gh-pages 镜像以此为基础增加了 gh-pages 这个 ruby 包，专门用于 GitHub Pages 的生成。

由于在使用 gh-pages 镜像时出了点问题，我的经验是先通过运行 ruby 2.7.3 容器，并将本地仓库目录挂载到容器中，使用里面的 Jekyll 命令创建站点。站点创建后，会生成 Jekyll 站点需要的文件和目录，稍加修改即可运行。



### 创建 Jekyll 站点

1. 克隆仓库。
   
    在合适的目录下将仓库克隆到本地后，使用下面的命令创建一个空白的分支，用于存放网站内容。

    ```bash
    git checkout --orphan gh-pages
    ```
2. 运行容器。
   
   运行容器时，将本地仓库目录挂载到容器中，同时指定端口映射和容器名。
   
   ```bash
   docker run -itd --name ghpages -p 4000:4000 -v {仓库绝对路径}:/home/gh ruby:2.7.3
   ```
3. 进入容器。
   
   ```bash
   # 进入交互式环境
   docker container exec -it ghpages /bin/bash
   ```
   其他可能用得到的容器命令。
   ```bash
   # 停止容器
   docker container stop {容器 ID}
   # 启动容器
   docker container start {容器 ID}
   ```

4. 使用 Jekyll 创建站点。

   进入 `/home/gh` 目录下，使用 Jekyll 新建站点。

    ```bash
    jekyll new --skip-bundle .
    ```
   命令完成后，会在当前目录下生成下列文件。

   ```bash
   404.html  Gemfile  _config.yml	_posts	about.md  index.md
   ```

5. 修改 Gem 包依赖。


   编辑 Gemfile 文件，添加 github-pages 包。

   ```bash
   # 注释下面这一行
   #gem "jekyll", "~> 3.9.1"
   # 新增 github-pages 行，其中 GITHUB-PAGES-VERSION 用实际版本号替换
   # 具体版本见 https://pages.github.com/versions/
   gem "github-pages", "~> GITHUB-PAGES-VERSION", group: :jekyll_plugins
   ```

6. 处理依赖。
   
   Gemfile 文件指定了项目需要的 ruby 包，bundle 根据文件信息下载需要的包并处理依赖。
    
    ```bash
    bundle install
    ```

7. 运行站点

    ```bash
    bundle exec jekyll serve -H 0.0.0.0
    ```
    浏览器访问本地 4000 端口即可看到站点内容。

## Ruby

![Ruby](/assets/ruby-logo.png)

如果你和我一样没用过 Ruby 语言，那么可以看看这篇 [Ruby 101](https://jekyllrb.com/docs/ruby-101/)。
### Gem

Gem 代表 Ruby 语言的代码库，提供特定的功能。Jekyll 是一个 Gem，Jekyll 的插件也是 Gem。

### Gemfile
Gemfile 文件列出了站点需要的 Gem，每个站点都会有一个。

### Bundler
Bundler 是用于安装 Gemfile 中需要的 Gem 的工具，同样，也是一个 Gem。在一个包含 Gemfile 文件的目录下，运行 `bundle install` 即可一次性安装 Gemfile 指定的 Gem。

未完待续。
