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