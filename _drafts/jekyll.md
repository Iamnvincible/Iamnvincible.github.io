# Jekyll 从入门到入门

## 依赖

- Ruby 2.5.0+
- RubyGems
- GCC、Make

## 快速开始

### 安装 Jekyll

1. 安装 Jekyll。
   ```sh
   gem install jekyll bundle
   ```
2. 创建 `Gemfile` 文件，以列出项目所需依赖。
   ```sh
   bundle init
   ```
3. 编辑生成的 `Gemfile` 文件，添加 Jekyll 作为依赖。
   ```sh
   gem "jekyll"
   ```
4. 运行 `bundle` 安装依赖。以后所有使用 Jekyll 的命令都可以加上前缀 `bundle exec` 以保证使用的 Jekyll 是 `Gemfile` 中指定的版本。
   > 这一步会安装很多 Gem，他们都会安装在系统目录下 `/usr/local/bundle/gems`

### 创建站点

1. 创建站点。为站点创建一个目录，也可以使用一个 Git 仓库，以后将这个目录称为**根目录**。
2. 在根目录创建一个 `index.html`，内容如下。
   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <meta charset="utf-8" />
       <title>Home</title>
     </head>
     <body>
       <h1>Hello World!</h1>
     </body>
   </html>
   ```

### 构建

Jekyll 是静态网站生成器，所以在预览网站前需要构建站点。

- `jekyll build`，构建站点，并在 `_site` 目录下存放构建结果。
- `jekyll server`，构建站点，并在本地 4000 端口开放站点内容，可以通过浏览器访问 `http://localhost:4000`。在站点内容修改后需要重新构建。

## Liquid

Liquid 是一个模板语言，可以让 Jekyll 更个性化。Liquid 包括三个组件 `objects`、`tags`、`filters`。

### Objects

Object 用于在页面中定义变量，使用两对花括号表示：`{{ variable }}`。

例如，`{{ page.title }}` 将会在页面中显示 `page.title` 变量。

### Tags

Tag 用于在模板中定义逻辑和控制流，使用一对花括号和百分号表示：`{% ... %}`。

例如，下面的模板代码使用 `page.show_sidebar` 变量来控制是否显示边栏。

```liquid
{% if page.show_sidebar %}
  <div class="sidebar">
    sidebar content
  </div>
{% endif %}
```

[了解更多 Tag 的使用说明](https://jekyllrb.com/docs/liquid/tags/)。

### Filters

Filter 用于改变 Object 的输出样式，使用 `|` 表示。

例如，下面的代码可以将输出大写。

```liquid
{{ "hi" | capitalize }}
```

[了解更多 Filter 的使用说明](https://jekyllrb.com/docs/liquid/filters/)。

### 试用 Liquid

在修改上文的例子中的页面内容为小写。

```liquid
<h1>{{ "Hello World!" | downcase }}</h1>
```

为了让 Jekyll 能够处理，需要在页面顶部添加 front matter（下文将会提及）。

```liquid
---
# front matter tells Jekyll to process Liquid
---
```

修改后，`index.html` 文档内容如下。

```liquid
---
---

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Home</title>
  </head>
  <body>
    <h1>{{ "Hello World!" | downcase }}</h1>
  </body>
</html>
```

保存后，浏览器将会显示 `hello world!`。Jekyll 中可以将 Liquid 和其他功能结合使用。

## Front Matter

Front Matter 是写在文档顶部用一对三个短横线包围的 YAML 代码块。可以在 front matter 中设置当前页面的变量。

```liquid
---
my_number: 5
---
```

有了变量定义，就可以使用 Liquid 来使用。在页面中，定义的变量作为 `page` 变量的属性/字段。下面的例子用于显示上面定义的变量。

```liquid
{{ page.my_number }}
```

### 试用 front matter

将上面例子中，用变量显示页面标题。

```liquid
---
title: Home
---
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ page.title }}</title>
  </head>
  <body>
    <h1>{{ "Hello World!" | downcase }}</h1>
  </body>
</html>
```

如果要让 Jekyll 处理页面而不使用变量，在 front matter 中留空即可。

```liquid
---
---
```

## Layouts

Jekyll 在生成页面时，支持处理 Markdown 和 HTML。在组织简单文本时，使用 Markdown 比 HTML 简单的多。

在根目录中创建一个名为 `about.md` 的 Markdown 文件。可以从之前的 `Index` 文件中复制内容过来再将其修改为**关于**页面，不过这样就会产生重复的代码，每创建一个新页面就会重复一遍。例如，如果要在 `<head>` 标签中增加新的 CSS 内容，就需要为所有页面逐个添加，这会很浪费时间。

### 创建 layout

Layout 是页面样式的模板，可以在任意页面中使用，为一系列页面统一样式。Layout 模板内容保存于 `_layouts` 目录中。

在根目录中创建 `_layouts` 目录，并在其中新建 `default.html` 文件，内容如下。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>{{ page.title }}</title>
  </head>
  <body>
    {{ content }}
  </body>
</html>
```

文件内容和 `index.html` 大致相同，不过没有 front matter，并且页面内容用 `content` 变量替换了。
`content` 是一个特殊的变量，其内容为调用这个 layout 的页面渲染完毕后的内容。

### 调用 layouts

为了能让 `index.html` 页面用到新建的样式模板，在 `index.html` 的 front matter 中增加一个 `layout` 变量，修改后的 `index.html` 内容如下。

```liquid
---
layout: default
title: Home
---
<h1>{{ "Hello World!" | downcase }}</h1>
```

刷新页面后，页面内容将保持不变。这是因为 layout 包含了页面内容，可以在 layout 文件中调用 front matter 中创建的变量，例如 `page`，layout 将会使用调用页面的 front matter。

### 创建关于页面

将下面的内容添加到 `about.md` 文件中，能够使用上文创建的样式模板。

```md
---
layout: default
title: About
---

# About page

This page tells you a little bit about me.
```

在浏览器中访问 `http://localhost:4000/about.html` 可以看到创建的关于页面。

## Includes

为了将站点中的页面链接起来，需要用到导航。导航需要在每个页面使用，所以也要写在 layout 里，并在合适的地方调用。为介绍 includes ，这里不直接添加到 layout 里。

### Include 标签

`include` 标签可以用于包含存放于 `_includes` 目录中的另一个文件。Includes 在需要使用重复代码的时候非常有用。

用于导航的代码可能会有些复杂，所以有时候会使用 include。

### Include 用法

创建 `_includes/navigation.html` 文件，文件内容如下。

```html
<nav>
  <a href="/">Home</a>
  <a href="/about.html">About</a>
</nav>
```

随后，在 `_layouts/default.html` 文件中引入 include 标签 用于页面间导航。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>{{ page.title }}</title>
  </head>
  <body>
    {% include navigation.html %} {{ content }}
  </body>
</html>
```

用浏览器访问 `http://localhost:4000`，现在可以在页面顶部导航。

### 高亮当前页面

为了在导航中高亮当前所在的页面，需要让 `navigation.html` 知道当前页面的 URL，随后添加高亮样式。Jekyll 有一个变量 `page.url` 可以表示当前页面 URL。

使用 `page.url` 变量获得当前页面，并将当前页面的导航链接改为红色。

```html
<nav>
  <a href="/" {% if page.url == "/" %}style="color: red;"{% endif %}>
    Home
  </a>
  <a href="/about.html" {% if page.url == "/about.html" %}style="color: red;"{% endif %}>
    About
  </a>
</nav>
```

再次访问站点，将能看到当前所在页面的导航颜色有所改变。

## 数据文件

Jekyll 可以从位于 `_data` 目录的 YAML、JEON 和 CSV 文件中读取数据，数据文件可以将内容和源码分离，使站点更易于维护。

这里将导航内容存放在数据文件中，随后通过 include 引入。

### 数据文件用法

YAML 是一种在 Ruby 生态圈中普遍使用的文件格式，可以将导航项目存放于数组中，并且为每一项指定链接。

创建 `_data/navigation.yml` 文件，内容如下。

```yml
- name: Home
  link: /
- name: About
  link: /about.html
```

使用变量 `site.data.navigation` 可以访问到这个数据文件的内容。现在可以遍历数组获得导航项，而不是像上例那样将所有导航项写入 `_includes/navigation.html` 文件中。

```html
<nav>
  {% for item in site.data.navigation %}
    <a href="{{ item.link }}" {% if page.url == item.link %}style="color: red;"{% endif %}>
      {{ item.name }}
    </a>
  {% endfor %}
</nav>
```

修改后，页面现实内容与上例一致。使用数据文件后，可以在增加导航项时更易维护。

## Assets

Jekyll 站点中也可以直接使用 CSS、JS 和图像，将这些资源放在 `assets` 目录下即可。为了更好组织，创建子目录 `css`，`js`，`images`。另外，在根目录下创建 `_sass` 目录，稍后将会用到。

### Sass

在 `_includes/navigation.html` 文件中内联了样式，这并不是一个好的做法，更好的做法是为特定对象标记一个 CSS class。将其中的具体样式移除，并用一个 class 来取代，内容如下。

```html
<nav>
  {% for item in site.data.navigation %}
    <a href="{{ item.link }}" {% if page.url == item.link %}class="current"{% endif %}>{{ item.name }}</a>
  {% endfor %}
</nav>
```

可以使用标准的 CSS 文件来控制样式，不过这里使用 Sass，一门 CSS 的扩展语言。创建 `assets/css/styles.scss`，内容如下。

```scss
---
---

@import "main";
```

文件头部的空白 front matter 标志能让 Jekyll 处理此文件。`@import "main"` 表示 Sass 会调用 在 `_sass` 目录下 的 `main.scss` 文件。本例中只用到了一个样式文件，对于较大的项目，这将是组织 样式文件的好办法。

创建 `_sass/main.scss` 文件，内容如下，可将当前访问页面的导航项目设为绿色。

```scss
.current {
  color: green;
}
```

随后，在 `_layouts/default.html` 中引用样式表。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>{{ page.title }}</title>
    <link rel="stylesheet" href="/assets/css/styles.css" />
  </head>
  <body>
    {% include navigation.html %} {{ content }}
  </body>
</html>
```

其中引用的 `styles.css` 由 Jekyll 从 `styles.scss` 构建生成。

刷新页面后，当前页面链接导航项将变为绿色。

## 博客

博客可以没有数据库，Jekyll 只使用文本文件就可以组建一个博客。

### Posts

博客文章存放于 `_posts` 目录，Jekyll 对文章文件名有要求，也就是`年-月-日-标题.文件扩展名`。

创建一篇博客文章，文件名为 `_posts/2018-08-20-bananas.md`，内容如下。

```md
---
layout: post
author: jill
---

A banana is an edible fruit – botanically a berry – produced by several kinds
of large herbaceous flowering plants in the genus Musa.

In some countries, bananas used for cooking may be called "plantains",
distinguishing them from dessert bananas. The fruit is variable in size, color,
and firmness, but is usually elongated and curved, with soft flesh rich in
starch covered with a rind, which may be green, yellow, red, purple, or brown
when ripe.
```

和先前创建的 `about.md` 文件内容结构相似，只是多了 `author` 和 `layout` 两个变量。`author` 是一个自定义变量，可以没有这个变量，或者把变量名改为 `creator` 也无妨。

### Layout

这里用到的 `post` 样式现在还没有，需要创建在 `_layout/post.html` 文件中，内容如下。

```html
---
layout: default
---

<h1>{{ page.title }}</h1>
<p>{{ page.date | date_to_string }} - {{ page.author }}</p>

{{ content }}
```

这里用到了样式的继承，继承了 `default` 样式。`post` 样式输出页面标题、日期、作者和正文，其中正文由 `default` 样式包含。

注意，这里用到了 `date_to_string` filter，可以将日期转化为更友好的格式来显示。

### 文章列表

现在还没有办法导航到博客文章，通常博客都会有一个页面来展示博客列表。

Jekyll 将博客列表保存在 `site.posts` 变量中，可以通过遍历将博文列出。在根目录下创建 `blog.html`，内容如下。

```html
---
layout: default
title: Blog
---

<h1>Latest Posts</h1>

<ul>
  {% for post in site.posts %}
  <li>
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </li>
  {% endfor %}
</ul>
```

- `post.url` 是文章路径，由 Jekyll 设置。
- `post.title` 是文章文件名中指定的标题，如果文章内容的 front matter 也指定了 title 变量，那么 `post.title` 为 title 变量值。
- `post.excerpt` 默认是文章第一段的内容。

有了 `blog.html` 页面后，就需要有链接导航到这个页面，修改 `_data/navigation.yml` 文件，添加 blog 页面和链接。

```yml
- name: Home
  link: /
- name: About
  link: /about.html
- name: Blog
  link: /blog.html
```

### 再多几篇

一个博客只有一片文章显得不太像样，这里再增加两篇。

第二篇博客文章，`_posts/2018-08-21-apples.md`。

```md
---
layout: post
author: jill
---

An apple is a sweet, edible fruit produced by an apple tree.

Apple trees are cultivated worldwide, and are the most widely grown species in
the genus Malus. The tree originated in Central Asia, where its wild ancestor,
Malus sieversii, is still found today. Apples have been grown for thousands of
years in Asia and Europe, and were brought to North America by European
colonists.
```

第三篇，`_posts/2018-08-22-kiwifruit.md`。

```md
---
layout: post
author: ted
---

Kiwifruit (often abbreviated as kiwi), or Chinese gooseberry is the edible
berry of several species of woody vines in the genus Actinidia.

The most common cultivar group of kiwifruit is oval, about the size of a large
hen's egg (5–8 cm (2.0–3.1 in) in length and 4.5–5.5 cm (1.8–2.2 in) in
diameter). It has a fibrous, dull greenish-brown skin and bright green or
golden flesh with rows of tiny, black, edible seeds. The fruit has a soft
texture, with a sweet and unique flavor.
```

添加完成后，再次访问，看看新增的博文。

## Collections

为了将某一个作者的文章用单独的页面展示出来，需要用到 collections 功能。Collections 类似于 posts，不过文章不以日期分组。

### 配置

使用 collections 功能需要在 jekyll 配置文件中指定，默认的配置文件是 `_config.yml`。在根目录创建 `_config.yml` 文件，内容如下。

```yml
collections:
  authors:
```

需要重启 Jekyll 服务以重新加载配置文件。

### 添加 authors

Collections 的分类类别存放在根目录的 `_connection_name` 目录中，本例中以 `authors` 作为分类依据，所以类别项目存放于 `_authors` 目录中。为每个 author 创建一个文件，内容如下。

- `_authors/jill.md`

  ```md
  ---
  short_name: jill
  name: Jill Smith
  position: Chief Editor
  ---

  Jill is an avid fruit grower based in the south of France.
  ```

- `_authors/ted.md`

  ```md
  ---
  short_name: ted
  name: Ted Doe
  position: Writer
  ---

  Ted has been eating fruit since he was baby.
  ```

  ### Staff 页

  为了列出所有 authors，创建一个 `staff.html` 页。上一节所有 authors 的信息保存在 `site.authors` 变量中，可以在 `staff.html` 中遍历这个变量以列出所有 authors。

  ```html
  ---
  layout: default
  title: Staff
  ---

  <h1>Staff</h1>

  <ul>
    {% for author in site.authors %}
    <li>
      <h2>{{ author.name }}</h2>
      <h3>{{ author.position }}</h3>
      <p>{{ author.content | markdownify }}</p>
    </li>
    {% endfor %}
  </ul>
  ```

  由于 `author.content` 这个变量内容是 markdown，显示时需要用 `markdownify` filter。

  既然新增了页面，就要在导航上新增一项。修改 `_data/navitation.yml`，内容如下。

  ```yml
  - name: Home
    link: /
  - name: About
    link: /about.html
  - name: Blog
    link: /blog.html
  - name: Staff
    link: /staff.html
  ```

  ### 为 authors 输出单独页面

  默认情况下，collections 不会为每一个 author 输出一页，只需要修改 `_config.yml` 即可实现。

  ```yml
  collections:
    authors:
      output: true
  ```

  再次重启 Jekyll 服务以重新加载配置文件。

  再为每个 author 添加一个链接，以链接到单个 author 页面。修改 `staff.html` 内容如下。

  ```html
  ---
  layout: default
  title: Staff
  ---

  <h1>Staff</h1>

  <ul>
    {% for author in site.authors %}
    <li>
      <h2><a href="{{ author.url }}">{{ author.name }}</a></h2>
      <h3>{{ author.position }}</h3>
      <p>{{ author.content | markdownify }}</p>
    </li>
    {% endfor %}
  </ul>
  ```

  再为 authors 页面创建一个 layout。

  - `_layouts/author.html`

    ```html
    ---
    layout: default
    ---

    <h1>{{ page.name }}</h1>
    <h2>{{ page.position }}</h2>

    {{ content }}
    ```

### Front matter 默认值

现在可以为 `author` 页面指定 `author` 这个 layout 样式模板，可以像之前那样加在 front matter 里，这里介绍另一种方法。

可以为每个类型的博客文章设定一个默认的 layout，其他类型的博客文章使用 default layout。这些默认值在 `_config.xml` 中指定，内容如下。

```yml
collections:
  authors:
    output: true

defaults:
  - scope:
      path: ""
      type: "authors"
    values:
      layout: "author"
  - scope:
      path: ""
      type: "posts"
    values:
      layout: "post"
  - scope:
      path: ""
    values:
      layout: "default"
```
再次重启 Jekyll 服务以重新加载配置文件。

### 列出每个 author 的文章

现在可以为每个 author 创建单独页面了。首先需要匹配每篇文章中的 author 变量到 author 的 `short_name` 属性，使用这个变量来将博客文章按 author 分类。继续修改 `_layouts/author.html`。

```html
---
layout: default
---

<h1>{{ page.name }}</h1>
<h2>{{ page.position }}</h2>

{{ content }}

<h2>Posts</h2>
<ul>
  {% assign filtered_posts = site.posts | where: 'author', page.short_name %} {%
  for post in filtered_posts %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
```

### 链接到 author 页面

每篇文章都会显示 author，这样可以通过 author 链接到其署名的所有文章。可以在 `_layouts/post.html` 中修改以达到此效果。

```html
---
layout: default
---

<h1>{{ page.title }}</h1>

<p>
  {{ page.date | date_to_string }} {% assign author = site.authors | where:
  'short_name', page.author | first %} {% if author %} -
  <a href="{{ author.url }}">{{ author.name }}</a>
  {% endif %}
</p>

{{ content }}
```
完成后，重新打开站点，检查一下 staff 页和 author 链接是否正确。