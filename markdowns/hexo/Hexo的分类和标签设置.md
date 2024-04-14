---
title: Hexo的分类和标签设置
date: 2017-04-19 16:29:59
url: hexo_tags_categories
categories: other
tags: 
	- hexo
	- 分类
	- categories
	- 标签
	- tags
---
# 设置分类列表

## 修改根目录下_config.yml
``` yml
# Directory
source_dir: source
public_dir: public
#这个是tags的目录
tag_dir: tags
archive_dir: archives
#这个是category的目录
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
```

<!--more-->

## 修改themes下_config.yml

```yml
menu:
  home: /
  #categories要打开
  categories: /categories
  #about: /about
  archives: /archives
  #tags要打开
  tags: /tags
  #sitemap: /sitemap.xml
  #commonweal: /404.html
```

## 生成tags和categories页面
在根目录下执行
```ps
$hexo n page "tags"
$hexo n page "categories"
```

然后去source目录下找到categories/index.md
修改：
```yml
---
title: categories
date: 2017-04-19 16:19:25
type: categories
comments: false
---
```

去source目录下找到tags/index.md
修改：
```yml
---
title: tags
date: 2017-04-19 16:19:25
type: tags
comments: false
---
```


这样就配置好了

# 在文章中添加tag、分类

## 模板中添加
编辑 /sacaffolds/post.md
```yml
---
title: {{ title }}
date: {{ date }}
categories: 
tags:
---
```
这样，new出来的文章默认就带上categories和tags了

## 多个tag的写法
```yml
tages: 
    - 标签1
    - 标签2
    ...
    - 标签n
```