---
title: Markdown模板
date: 2017-12-22 12:48:48
categories: 杂记
tags: markdown模板
---

# Markdown 基本要素

这篇文件意在简要介绍 [GitHub Flavored Markdown 写作](https://guides.github.com/features/mastering-markdown/)。

## 语法说明

### 标题

```markdown
# 这是 <h1> 一级标题

## 这是 <h2> 二级标题

### 这是 <h3> 三级标题

#### 这是 <h4> 四级标题

##### 这是 <h5> 五级标题

###### 这是 <h6> 六级标题
```

### 强调

<!-- prettier-ignore -->
```markdown
*这会是 斜体 的文字*
_这会是 斜体 的文字_

**这会是 粗体 的文字**
__这会是 粗体 的文字__

_你也 **组合** 这些符号_

~~这个文字将会被横线删除~~
```

### 列表

#### 无序列表

```markdown
- Item 1
- Item 2
  - Item 2a
  - Item 2b
```

#### 有序列表

```markdown
1. Item 1
1. Item 2
1. Item 3
   1. Item 3a
   1. Item 3b
```

### 添加图片

```markdown
![GitHub Logo](/images/logo.png)
Format: ![Alt Text](url)
```

### 链接

```markdown
http://github.com - 自动生成！
[GitHub](http://github.com)
```

### 引用

```markdown
正如 Kanye West 所说：

> We're living the future so
> the present is our past.
```

### 分割线

```markdown
如下，三个或者更多的

---

连字符

---

星号

---

下划线
```

### 行内代码

```markdown
我觉得你应该在这里使用
`<addr>` 才对。
```

### 代码块

你可以在你的代码上面和下面添加 <code>\`\`\`</code> 来表示代码块。

#### 语法高亮

你可以给你的代码块添加任何一种语言的语法高亮

例如，给 ruby 代码添加语法高亮：

    ```ruby
    require 'redcarpet'
    markdown = Redcarpet.new("Hello World!")
    puts markdown.to_html
    ```

会得到下面的效果：

```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```

### 任务列表

```markdown
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item
```

### 表格

<!-- prettier-ignore -->
```markdown
First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column
```

## 扩展的语法

### 上标

```markdown
30^th^
```

### 下标

```markdown
H~2~O
```

### 脚注

```markdown
Content [^1]

[^1]: Hi! This is a footnote
```

### 缩略

```markdown
_[HTML]: Hyper Text Markup Language
_[W3C]: World Wide Web Consortium
The HTML specification
is maintained by the W3C.
```

### 标记

```markdown
==marked==
```

### CriticMarkup

CriticMarkup 缺省是禁用的，你可以通过插件设置来启动它。  
有关 CriticMarkup 的更多信息，请查看 [CriticMarkup 用户指南](http://criticmarkup.com/users-guide.php).

这里有 5 种基本语法：

- 添加 `{++ ++}`
- 删除 `{-- --}`
- 替换 `{~~ ~> ~~}`
- 注释 `{>> <<}`
- 高亮 `{== ==}{>> <<}`

> CriticMarkup 仅可用于 markdown-it parser，不与 pandoc parser 兼容。

## 参考

- [Mastering Markdown](https://guides.github.com/features/mastering-markdown/)
- [Daring Fireball: Markdown Basics](https://daringfireball.net/projects/markdown/basics)

```markdown
- [Mastering Markdown](https://guides.github.com/features/mastering-markdown/)
- [Daring Fireball: Markdown Basics](https://daringfireball.net/projects/markdown/basics)
```
> 参考文档


