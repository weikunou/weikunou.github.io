---
title: Hexo-Config
date: 2021-01-27 17:55:00
tags: Hexo
---

在根目录下，有个叫 _config.yml 的文件，用文本编辑器打开，可以看到很多配置字段。

<!--more-->

# Site

| 字段        | 描述                                           | 作用                     |
| ----------- | ---------------------------------------------- | ------------------------ |
| title       | 网站的标题                                     | 默认会显示在网站顶部     |
| subtitle    | 网站的副标题                                   | 默认会显示在标题下方     |
| description | 网站的描述                                     | 没有显示，跟搜索引擎有关 |
| keywords    | 网站的关键词                                   | 没有显示，跟搜索引擎有关 |
| author      | 作者                                           | 默认会显示在网站底部     |
| language    | 网站的语言，使用 2 个字符的代码描述，中文是 zh | 没有显示，跟搜索引擎有关 |
| timezone    | 网站的时区，Hexo会默认使用计算机上的设置       | 没有显示，跟搜索引擎有关 |

# URL

| 字段                       | 描述                  | 作用                                                         |
| -------------------------- | --------------------- | ------------------------------------------------------------ |
| url                        | 网站的 URL            | 如果网站在域名下的一个子目录里，则需要设置为 https://网址/子目录/ |
| root                       | 网站的根目录          | 如果网站在域名下的一个子目录里，则需要设置为 /子目录/        |
| permalink                  | 文章的永久链接格式    | public 文件夹和网址的层级格式                                |
| permalink_defaults         | 永久链接格式的默认值  | 为 permalink 设置默认值                                      |
| pretty_urls                | 美化永久链接          | 可以让 URL 更简洁                                            |
| pretty_urls.trailing_index | 链接尾部的 index.html | 设置为 false 可以让链接尾部的 index.html 不显示              |
| pretty_urls.trailing_html  | 链接尾部的 .html      | 设置为 false 可以让链接尾部的 .html 不显示                   |

# Directory

| 字段         | 描述                                               | 作用                                                         |
| ------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| source_dir   | 资源文件夹                                         | 存放文章、图片等资源                                         |
| public_dir   | Hexo 发布的文件夹                                  | 可以看到发布后的静态页面文件                                 |
| tag_dir      | 标签文件夹                                         | 给文章设置标签后，会多出一个标签文件夹，网站的侧边栏也会多出两栏标签导航栏 |
| archive_dir  | 档案文件夹                                         | 默认按照月份将文章归档                                       |
| category_dir | 分类文件夹                                         | 跟标签文件夹差不多，同样在文章的顶部进行设置                 |
| code_dir     | 代码文件夹                                         | 存放代码                                                     |
| i18n_dir     | 国际化文件夹                                       | 跟语言相关                                                   |
| skip_render  | 跳过渲染，将路径下的文件直接拷贝到 public 文件夹下 | 可以在发布时，将一些资源原封不动地拷贝到 public 文件夹，避免渲染后出现问题，比如 README.md 文件 |

# Writing

| 字段                  | 描述                                                | 作用                                                         |
| --------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| new_post_name         | 新建文章的名称格式，默认是 title.md                 | 新建文章后生成的文件名会是 标题.md 的格式                    |
| default_layout        | 新建文章的默认布局，默认是 post                     | 新建文章时使用的模板                                         |
| titlecase             | 将标题转换成首字母大写的标题                        |                                                              |
| external_link         | 外部链接                                            |                                                              |
| external_link.enable  | 是否在新标签页打开外部链接                          |                                                              |
| external_link.field   | 对整个网站还是当前文章生效                          |                                                              |
| external_link.exclude | 排除哪些域名                                        |                                                              |
| filename_case         | 把文章的名称改为小写（1）或大写（2），默认不改（0） |                                                              |
| render_drafts         | 是否渲染草稿                                        | 设置为 true 后，将会显示通过草稿模板创建的文章               |
| post_asset_folder     | 是否启用资源文件夹                                  | 设置为 true 后，新建文章的同时，会在文章同级目录下生成一个同名文件夹存放资源 |
| relative_link         | 是否把链接改为根目录的相对路径                      |                                                              |
| future                | 是否显示未来日期的文章                              | 设置为 false 后，将不会显示未来日期的文章                    |
| highlight             | 代码块设置                                          |                                                              |
| highlight.enable      |                                                     |                                                              |
| highlight.line_number |                                                     |                                                              |
| highlight.auto_detect |                                                     |                                                              |
| highlight.tab_replace |                                                     |                                                              |
| highlight.wrap        |                                                     |                                                              |
| highlight.hljs        |                                                     |                                                              |
| prismjs               | 代码块设置                                          |                                                              |
| prismjs.enable        |                                                     |                                                              |
| prismjs.preprocess    |                                                     |                                                              |
| prismjs.line_number   |                                                     |                                                              |
| prismjs.tab_replace   |                                                     |                                                              |

# Home page setting

| 字段                     | 描述               | 作用 |
| ------------------------ | ------------------ | ---- |
| index_generator          | 主页设置           |      |
| index_generator.path     | 主页的路径         |      |
| index_generator.per_page | 每页显示的文章数目 |      |
| index_generator.order_by | 文章排序           |      |

# Category & Tag

| 字段             | 描述     | 作用 |
| ---------------- | -------- | ---- |
| default_category | 默认分类 |      |
| category_map     | 分类别名 |      |
| tag_map          | 标签别名 |      |

# Metadata elements

| 字段           | 描述                         | 作用 |
| -------------- | ---------------------------- | ---- |
| meta_generator | 是否在页面头部插入 meta 标签 |      |

# Date / Time format

| 字段           | 描述                                         | 作用 |
| -------------- | -------------------------------------------- | ---- |
| date_format    | 日期格式                                     |      |
| time_format    | 时间格式                                     |      |
| updated_option | 如果文章中没有指定创建日期，就会使用更新日期 |      |

# Pagination

| 字段           | 描述                       | 作用 |
| -------------- | -------------------------- | ---- |
| per_page       | 文章分页时，每页的文章数目 |      |
| pagination_dir | 分页的目录                 |      |

# Include / Exclude file(s)

| 字段    | 描述         | 作用                                            |
| ------- | ------------ | ----------------------------------------------- |
| include | 包括的文件   | Hexo 会将这个路径下的文件拷贝到 source 文件夹下 |
| exclude | 不包括的文件 | Hexo 会将这个路径下的文件排除在外               |
| ignore  | 忽略的文件   | Hexo 不会对这些文件进行操作                     |

# Extensions

| 字段  | 描述 | 作用           |
| ----- | ---- | -------------- |
| theme | 主题 | 网站显示的样式 |



# Deployment

| 字段          | 描述                          | 作用                                               |
| ------------- | ----------------------------- | -------------------------------------------------- |
| deploy        | 部署设置                      |                                                    |
| deploy.type   | 部署类型，默认是 git          |                                                    |
| deploy.repo   | 部署地址，github 远程仓库地址 | 将发布后的文件部署到相应的 Github 地址             |
| deploy.branch | 部署分支                      | 将发布后的文件部署到相应的 Github 地址的一个分支下 |

