---
title: "通过hugo创建博客网站"
date: 2020-05-06T10:21:43+08:00
draft: false
toc: false
images:
tags: 
  - 个人站
---

## 环境

ubuntu18.04

## 下载hugo

推荐通过hugo的github去下载对应的release`https://github.com/gohugoio/hugo/releases`

执行`hugo version`显示版本号，无报错即是安装成功

## 生成博客目录

例如在/app目录下

执行`hugo new site xblog`,xblog指代博客名称

`cd xblog`

hugo会在此目录下生成以下的文件夹和config.toml文件：

> archetypes：设定自定义的头文件
> content：存放网站内容页，也就是博客文章，.md格式
> data：存放网站所要用到的一些配置、数据文件、自定义模板，toml/yaml/json格式
> layouts：存放的是网站的模板文件，渲染content目录下的内容
> static：存放js/css/img等静态资源
> themes：存放主题相关文件

切换到themes目录下，执行`git clone https://github.com/Track3/hermit.git`可以下载该主题到themes目录
hermit是其中的一款主题，需要选择更多的主题可以到<https://themes.gohugo.io/>进行选择

使用此主题需要将config.toml配置文件替换为主题包内的config.toml文件(具体的各个配置项和用法可以去<https://github.com/Track3/hermit>和<https://gohugo.io/documentation/>进行查看)

创建posts目录并生成第一篇文章,执行`hugo new posts/first_post.md`,会在content目录下生成posts文件夹和内部的md文件

执行`hugo server -D`(-D代表会显示草稿, md文件中头部配置中的draft: true代表草稿模式,完成文章后可改为false), 打开`http://localhost:1313/`查看站点预览效果

预览完后觉得效果可以,就可以生成站点的静态目录

在/app/xblog下执行hugo命令,会生成public目录,其中就包括整个静态站点的所有文件

## 上传github生成站点

在github上新建一个名叫`xxx.github.io`的仓库(必须以github.io结尾,xxx自定义)

修改config.toml文件中的baseURL为此仓库地址`https://xxx.github.io`

进入到刚才生成的public目录下执行

```
git init
git add .
git commit -m "xxx"
git remote add origin https://github.com/xxx/xxx.github.io.git
git push -u origin master
```

此时访问`https://xxx.github.io/`就可以看到和你预览效果同样的静态博客站点了

## 注意事项
hermit默认带的评论系统是Disqus,但是由于某些原因,Disqus在国内是无法访问的,所以要对评论模块进行替换

### utterances

这个是基于github issue的一个评论工具,轻量简单且好看

要安装utterances首先需要去自己的github app进行注册,进入<https://utteranc.es/>按照文档进行配置

选择对应的仓库,可以选择全部的仓库,但是建议只选择一个

![选择相应的仓库](/utterances.png)

配置好utterances之后,utterances官方文档上给出了一段script标签代码

```js
<script src="https://utteranc.es/client.js"
        repo="[ENTER REPO HERE]"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

将这一段代码替换原本的评论部分的代码,以hermit为例

找到hermit文件夹下的/hermit/layouts/posts/single.html打开会发现里面有这样的一段html

```html
<div id="comments" class="thin">
	{{- partial "comments.html" . -}}
</div>
```

这个就是文章的评论部分对应的代码了,但是中间还是引用的其他文件,那按照`{{- partial "comments.html" . -}}`的提示接着找,发现了`/hermit/layouts/partials/comments.html`文件,其中的内容是这样的

```
{{- if .Site.DisqusShortname }}
{{ template "_internal/disqus.html" . }}
{{- end }}
```

意思是只有在config.toml配置文件中添加了DisqusShortname选项,那么才会开启评论系统,可以看到默认的就是Disqus,但是我们不用管他写了啥,只要把其中的内容替换为上面的script标签代码即可,替换完成后在进行hugo命令生成新的public,在上传到github部署后,你的xxx.github.io的网站就可以看到新的评论系统了

好了,个人静态博客的搭建暂时完成,如果想接着对网站进行其他的配置,可能需要对hugo的官方文档进行深入的了解

#### peace

