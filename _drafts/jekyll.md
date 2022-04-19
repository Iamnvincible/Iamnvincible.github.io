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