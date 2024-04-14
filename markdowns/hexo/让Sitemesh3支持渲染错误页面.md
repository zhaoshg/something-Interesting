---
title: 让Sitemesh3支持渲染错误页面
url: sitemesh_404
date: 2017-05-22 15:48:58
categories: jsp
tags: 
    - sitemesh
---

```xml
<filter-mapping>
    <filter-name>sitemesh</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>ERROR</dispatcher>
    <dispatcher>FORWARD</dispatcher>
</filter-mapping>

```