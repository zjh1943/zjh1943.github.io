Layout:slide
tags: 创业,演讲
description: 一个PPT，用来阐述创业精神
date: 2016-1-19 14:20
Status: hide

# 这一页用来干啥

## 子页面

### 三级页面

只能怪文

```jade:n
head
        +mobile_meta()
        meta(name="keywords", content=site.configs.keywords.escaped)
        meta(name="description", content=site.raw_content.escaped)
        block title
            title= request.args.s or post.title or tags.join('+') or category.title or site.title
        +get_resource("blog_basic.css")
        +load('pure#0.5.0')
        +load('/service/static_3rd/staticfile/ajax/libs/pure/0.5.0/grids-responsive-min.css')
        +load('/template/style.scss')
```

### 三级页面


# 一级页面