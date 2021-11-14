<!--
 * @Author: Jie Lin
 * @Date: 2021-10-31 22:38:26
 * @LastEditTime: 2021-11-06 21:13:41
 * @LastEditors: Jie Lin
 * @Description: Github pages 创建步骤
 * 
-->

# GitHub Pages 托管静态博客

在没有独立服务器的情况下，使用 GitHub Pages 托管一个静态网站作为博客是一种不错的选择。不过静态网站只能展示页面，没有数据库，需要一点额外操作才能启用评论等独立服务器用户拥有的高级功能。

首先需要在 GitHub 上创建一个仓库，网站展示的页面来源于仓库中特定目录下的 Markdown 文件，只需要在仓库中上传 Markdown 格式的文章，GitHub Pages 就能将文章转换成对应的网页。一篇文章生成后，目录、首页也会同时更新。这样，不需要复杂的建站步骤，每个 GitHub 用户都能拥有个人博客了！

可以从 [快速入门](https://docs.github.com/en/pages/quickstart) 或 [GitHub Pages 创建指南](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) 开始探索。需要注意的是，仓库名最好是你的 GitHub 用户名，这样静态网站的网址就会是 `username.github.io`，否则，这个网址会包含整个仓库名称，很长，不容易记忆。快速入门会让你手动添加文件到仓库中，然后就能通过特定链接访问了，但这样不能预览网站结果，没有目录，自定义操作需要修改 `_config.yml` 文件实现。为了能在发布网站前预览网站效果，我们需要使用 Jekyll。Jekyll 是 ruby 语言编写的静态网站生成器， 受到 GitHub Pages 内建支持。

为了满足更多静态网站的自定义需求，需要在本地搭建 Jekyll 环境，对网站样式满意之后将所有 Jekyll 配置上传到仓库，GitHub Pages 会根据配置生成和本地同样的网站样式。



## 使用 Jekyll 设置 GitHub Pages 站点

### 准备 Jekyll 环境

Jekyll 是一个通用的静态网站生成工具，但 GitHub Pages 后端生成内容时使用了[特定版本](https://pages.github.com/versions/)，为了能让本地预览效果 GitHub Pages 呈现的效果一致，需要安装与 GitHub Pages 后端一致的版本。

由于依赖关系较为复杂，可以使用 GitHub Pages 提供的 Docker 镜像 [pages-gem](https://github.com/github/pages-gem) 来避免环境依赖问题。该 Docker 镜像构建后会得到 2 个镜像，gh-pages 和 ruby 2.7.3。其中 gh-pages 镜像可以在运行时指定一个 Jekyll 项目目录，容器会以该目录为源生成网站内容，并且运行容器时会自动端口映射，浏览器访问本地 4000 端口即可查看。ruby 2.7.3 镜像有匹配版本的 ruby 和 Jekyll 环境，gh-pages 镜像以此为基础增加了 gh-pages 这个 ruby 包，专门用于 GitHub Pages 的生成。

由于没有 Jekyll 的知识储配，我的经验是先通过运行 ruby 2.7.3 容器，使用 Jekyll 命令创建站点后，会生成站点基本目录，目录中包含所有必须的配置文件，完成基本配置后，这个目录可以从容器中取出，用于 gh-pages 容器的运行。



### 创建 Jekyll 站点

1. 运行容器

   ```bash
   # 使用交互式环境启动容器，并映射端口
   docker run -itd -p 4000:4000 ruby:2.7.3
   # 如果从交互式环境中退出，再次进入需要先获得容器 ID
   docker container ls --all
   # 再次进入交互式环境
   docker container exec -it {容器 ID} /bin/bash
   # 停止容器
   docker container stop {容器 ID}
   # 启动容器
   docker container start {容器 ID}
   # 启动容器后可以使用进入交互式环境的命令进入容器
   ```

2. 进入容器

3. 克隆仓库

   在合适的目录下将仓库克隆到本地后，使用下面的命令创建一个空白的分支，用于存放网站内容。

    ```bash
    git checkout --orphan gh-pages
    ```

4. 使用 Jekyll 创建站点

   空分支创建完成后，使用 Jekyll 新建站点。

    ```bash
    jekyll new --skip-bundle .
    ```

5. 修改 Gemfile

   创建站点完成后，会在当前目录下生成若干文件。

   ```bash
   404.html  Gemfile  _config.yml	_posts	about.md  index.md
   ```

   编辑 Gemfile 文件，添加 github-pages 包。编辑完成后，使用 bundle 自动处理版本依赖。

   ```bash
   # 注释下面这一行
   #gem "jekyll", "~> 3.9.1"
   # 新增 github-pages 行，其中 GITHUB-PAGES-VERSION 用实际版本号替换
   # 具体版本见 https://pages.github.com/versions/
   gem "github-pages", "~> GITHUB-PAGES-VERSION", group: :jekyll_plugins
   ```

6. 处理依赖
   
    Gemfile 文件指定了项目需要的 ruby 包，bundle 根据文件信息下载需要的包并处理依赖。
    
    
    ```bash
    bundle install
    ```

7. 运行站点

    ```bash
    bundle exec jekyll serve -H 0.0.0.0
    ```
    
    浏览器访问本地 4000 端口即可看到站点内容。



未完待续。
