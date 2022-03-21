---
title: Hexo+GitHub博客管理命令汇总
date: 2018-03-06 20:13:39
tags:
	- 工具
categories: 技术
---
## [Hexo]部署博客及更新博文

### 新建博文

使用 Git Shell 进入本地 Blog 文件夹，输入以下命令：

		hexo n "文章题目" 

命令执行完后，就会发现在Blog/source/_posts目录中多了一个文件 博文名.md，这就是我们刚才新建的博文。
<!--more-->

### 新建页面
上面新建的博文是显示在单个文章界面，这里新建的页面是作为单个页面显示的，比如分类、标签、归档和关于我，你点击后都是显示为单个页面。输入以下命令：

		hexo n page "页面名称"

命令执行完后，就会发现在在Blog/source目录中多了一个文件夹，里面还有一个index.md，这就代表新建了一个页面。

### 写博文

用文本编辑器打开上面新建的博文，如下图所示：
新建的页面略有不同，没有tags和categories标签。
三个”-“后面就是博文的正文内容，接下来就是正儿八经地撰写博文了。

因为这里博文都是用Markdown语言写的，所以首先需要一个好用的Markdown编辑器。目前只用过MarkdownPad，其实好用的Markdown编辑器一大堆，这里推荐两个方便使用的：

* 本地编辑器：Haroopad，非常小众的一款Markdown编辑器，左边编辑右边实时预览效果，非常轻便；

* 在线编辑器：MaHua，也是比较小众的一款Markdown编辑器，但效果确实很棒

现在打开新建的博文，开始编写，具体参考这里的Markdown教程：[Markdown——入门指南](http://www.jianshu.com/p/1e402922ee32/)
### 发博文

依然在 Git Shell 中进入 Blog文件夹，执行下面几条命令，将博客部署到 GitHub 上：

		hexo clean
		hexo generate(若要本地预览就先执行 hexo server)
		hexo deploy
快捷命令：

		hexo g == hexo generate
		hexo d == hexo deploy
		hexo s == hexo server
		hexo n == hexo new
还能组合使用，如：

		hexo d -g
		hexo s --port=4100
刷新你的个人博客，就可以看到新鲜出炉的博文了，赶紧邀请小伙伴们来欣赏吧。
### 一个可能出现的错误
		spawn git ENOENT
解决方法在这里：[spawn git ENOENT解决方法](http://liangwenhao.cn/2016/08/24/article18/)
