---
layout: post
title: "ubuntu12.04下使用eclipse的一些技巧"
categories: [ubuntu, eclipse]
tags: [ubuntu, eclipse]
description: 对于ubuntu12.04下使用eclipse的显示优化.
---

对于ubuntu12.04下使用eclipse的显示优化
本机环境：ubuntu12.04(64bit) + Eclipse juno4.2R 

### 隐藏quick access

如果不太使用quick access, 也觉得很占显示地方，可以隐藏掉它．
编辑 /opt/eclipse/plugins/org.eclipse.platform\_4.2.2.v201302041200/css,
\`\`\`vim
$vi e4\_classic\_winxp.css 
\`\`\`

> **note:**
Here is a quick hack which doesn't require any plugin installation, instead you just need to add a few lines to your current layout's CSS file. Works perfectly for me in v4.2.2 
Navigate to \<ECLIPSE\_HOME\>/plugins/org.eclipse.platform\_<VERSION>/css then open up the CSS file of whichever layout you are using, e.g. mine was e4\_default.css. Now append the following snippet to the file:
\`\`\`css
 SearchField {
 visibility:hidden;
 } 
\`\`\`
Now just restart Eclipse and the box is gone.

\`Attn: 我使用的是juno (ubuntu 12.04) ，编辑e4\_classic\_winxp.css (很奇怪)
\`

### 显示代码很好的字体

**YaHei Consolas Hybrid** *or* **Courier New**  *or* **Consolas** *or* **Courier 10 pitch**  
(个人感觉前三者差不多，最后一个显示较好)

YaHei字体获取： [http://dl.dbank.com/c01bo3a1eo\#][1]

### cvs插件迁出中文乱码问题解决

进入CVS Repository Exploring 视图，右键选择你的cvs repository , 选择properties, 
然后在对话框中选择Server Encoding, 更改text file encoding选项即可。

### 更改workspace目录

以下３种方法均可：

 1. 进入 Window \> Preferences \> General \> Startup and Shutdown 选中 Prompt
	for workspace on startup。
 2. 进入Eclipse的安装目录，找到configuration 目录下的 .settings 文件夹，里面有一个
	org.eclipse.ui.ide.prefs， 用UltraEdit等打开，也可以用写字板打开，找到RECENT\_WORKSPACES，按照它的格式修改一下。
 3. 先打开Eclipse，进入之后，再去打开一次，会提示 Workspace in use or cannot be created,
	choose a different one 这时候就会提示你更改workspace的目录了。

[1]:	http://dl.dbank.com/c01bo3a1eo#