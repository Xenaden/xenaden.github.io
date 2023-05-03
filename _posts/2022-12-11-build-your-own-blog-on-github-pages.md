---
layout: post
title: 在GitHub Pages 上搭建你的个人博客
date: 2022-12-11 10:36
modified_date:
categories: some-practice
tag: [github-pages, jekyll, git]
---
本篇大概简述一下使用[GitHub Pages](https://pages.github.com/) 搭建静态的个人博客，并在Ubuntu 中同步、修改GitHub Pages 相关连的仓库，还有关于[Jekyll](https://jekyllrb.com/) 主题的部分自定义。

### ***Git？ GitHub？ GitHub Pages？***
首先说一点，这些全都是可以供个人免费使用的。

**[Git](https://git-scm.com/)** 是一款开源分布式版本控制软件，是Linus Torvalds 最初为了更好地管理Linux 核心版本而开发出来的软件。作个不怎么准确的概述，就是Git 可以把你的文件在不同时间的状态保存下来（怎么有点像虚拟机快照？），保存到本地仓库进行管理。
  
而**GitHub** 就是将这个仓库搬到公共服务器上，这样世界各地的人们都可以协同合作修改文件了。  
**GitHub Pages** 就是一个静态网站，从GitHub 上的仓库创建，仓库里的文件就是网页的文件。  
而GitHub Pages 使用的是**Jekyll** 作为网页渲染器，其可以使用网页模板，也就是[主题](#liquid-sass)。

### ***本地搭建与远端同步***
为什么要搭建本地环境？因为对GitHub 仓库的修改，GitHub Pages 不会时实更新，在本地也可以更方便操作。  
至于Git 和Jekyll 的下载安装就不再赘述。

#### 创建本地Git 仓库和搭建本地GitHub Pages 环境
首先[初始化本地Git](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll#creating-your-site) ，这GitHub 的官方教程目的主要是为代码仓库创建网页，介绍当前仓库的代码项目，而我的目的就是搭博客，就只要GitHub 仓库的网页功能。  
```
~$ mkdir mysite		#以创建文件夹 mysite 为例
~$ cd mysite
~/mysite$ git init	#以当前文件夹为仓库，初始化仓库。此时没有指定仓库名，所以没有仓库名也不会再在当前文件夹创建一个文件夹作为仓库
```
接着教程第七步继续。
#### 与GitHub 的远程仓库同步

Jekyll 默认的主题是[Minima](https://jekyll.github.io/minima/)

### ***Liquid？ SASS？***
Jekyll 使用[Liquid](https://shopify.github.io/liquid/) 这个模板语言编写网页模板，HTML ，CSS 和[SASS](https://sass-lang.com/) 制作网页内容。  
要自定义主题，首先要搞清楚[Jekyll 的目录规范](https://jekyllrb.com/docs/structure/)，文件的作用和内容。以下继续以Minima 主题为例。  
```
_config.yml				#全局配置文件
_includes				#包含的模块布局
├── custom-head.html			#custom-* ，代表覆盖和原文件有相同的内容
├── disqus_comments.html
├── footer.html				#页脚
├── google-analytics.html
├── header.html				#页首
├── head.html				#头信息
├── social.html
├── social-icons
└── social-item.html
_layouts				#网页布局，在撰写文章头部信息指定
├── default.html
├── home.html
├── page.html
└── post.html
_posts
├── 2016-05-19-super-short-article.md
├── 2016-05-20-my-example-post.md
├── 2016-05-20-super-long-article.md
├── 2016-05-20-this-post-demonstrates-post-content-styles.md
└── 2016-05-20-welcome-to-jekyll.md
_sass
└── minima				#CSS样式
    ├── _base.scss			#设置基本样式
    ├── custom-styles.scss
    ├── custom-variables.scss
    ├── initialize.scss			#赋值样式变量
    ├── _layout.scss			#定义_layouts文件夹里所有布局的样式
    └── skins				#颜色变量赋值
```
<br>
现在进行布局调整，把页面右上角的About 块移动到左下角页脚  
![](/assets/images/about-block-on-right-header.png)

在网页，按 `F12` 调出开发者模式，点击元素抓取工具。可以看到，该元素是属于页首的布局。  
先在 `_config.yml`添加
```
footer_pages:		#定义该全局变量
 - about.md
```
在 `includes/footer.html` 添加
```
{%- raw -%}
 <li class="about">
   {%- for path in site.footer_pages -%}
   {%- assign my_page = site.pages | where: "path", path | last -%}
   {%- if my_page.title -%}
   <a class="page-link" href="{{ my_page.url | relative_url }}">{{ my_page.title | escape }}</a>
   {%- endif -%}
   {%- endfor -%}
</li>
{% endraw %}
```











