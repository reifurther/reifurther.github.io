---
layout: post
title: "更新托管在github上的个人blog"
categories: [blog, github]
tags: [blog, github]
description: 快速更新个人blog.
---



### 新建md文件

首先需要取下个人blog的repository.
{% highlight ruby %}
   $git clone https://github.com/leipingr/leipingr.github.io.git
{% endhighlight %}
在 \_posts目录复制一个文件，修改描述信息，编辑完内容，保存。

(个人喜欢用Ulysses.)

### 本地调试
{% highlight ruby %}
   $jekyll s
{% endhighlight %}

查看效果。


> **Tips:**
有可能本地调试比较慢，是因为访问了google-analytics.com，使用了插件。需要更新下本机hosts文件，访问google较快即可。

### 提交到github

{% highlight ruby %}
$git add .
$git commit -m  "some description."
$git push origin master
{% endhighlight %}

结束。