---
title: 五天学会一种web开发框架
date: 2016-07-30 19:23:00
tags: 
- framework
- web
categories: phper
---
web framework层出不穷，特别是ruby/python,各有10+个,php/java也是一大堆 根据我自己的经验写了一个to do list,按照这个清单，一条一条的学习，事半功倍，很快就能掌握 一共25条，即便很磨蹭，2小时也能搞定一条，25*2=50。只需要50小时就能掌握任意一种web框架.  

建议把本文打印到一张A4纸，搞定一条打个勾。

#### web框架学习列表  ####
1. 如何定义 url route.
2. 如何组织 request handler 函数.
    - 写一个最简单的request handler 函数
    - 如何从get/post请求中取出参数
    - 如何定义全局url 拦截函数
    - 如何获取/修改/存储 cookie,session数据
    - 如何修改/输出 http header 数据  
3. 如何部部署app 程序.
    - 服务器部署可以参考读python web 程序的9种部署方式
    - 如何配置开发环境
    - 如何配置静态文件访问
4. 如何访问数据库.
    - 是否支持ORM
        - 如何维护表结构的变更
        - 如何定义/组织/初始化 数据表
        - 如何对接orm系统和现有的表结构
        - 掌握基本的add/delete/按字段查询/counts/slice/order by
        - 如何直接使用sql访问数据库
    - 不支持orm（这样的框架，不用也罢）
5. 如何使用模板系统
    - 如何组织/访问模板文件的目录结构
    - 如何在模板中嵌入代码
    - 模板是否支持继承结构
    - 模板之间如何include
    - 如何自定义模板函数
6. 如何通过http get/post 获取远程数据
7. 如何parse json
8. 如何parse xml
9. 如何输出为json
10. 如何处理状态码：404和50x
11. 如何处理文件上传
#### 可选学习项目 ####
- 发送email
- log
- 图片处理
#### 误区 ####
- 表单验证辅助函数，很多框架的表单验证部分实现的特别复杂，初学者完全不需要，手写代码处理就够用。
- ORM中的hasone,manytomany,onetomany关系，概念复杂，其实只是多写/少写一个查询字段的关系，学习成本太高，初学者完全不需要理会，直接跳过。
#### 现代web开发框架六大元素 ####
1. url route/map，表面看这个就是一组正则表达式，其实这是web开发框架的核心，它决定了代码的组织方式.
2. html 模版子系统.
3. orm，或者说数据库访问层.
4. form 前后端验证，表单验证是整个网站开发过程当中最复杂的一步，需要写手很多代码.
5. helper，包括：分页，cache，session,sitemap,rss,email等支持.
6. deployment 支持，简单说，这个框架起码要内置一个http server，更进一步要支持fastcgi，这样才能部署到生产环境.  

原文地址：[5天学会一种 web 开发框架](http://lutaf.com/148.htm)
