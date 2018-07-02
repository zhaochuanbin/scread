---
layout : post
title : "eclipse问题"
category : eclipse
tagline: ""
date : 2017-03-11
tags : [eclipse]
---

### reload maven project
An internal error occurred during: "reload maven project".
java.lang.NullPointerException
><workspace>\.metadata\.plugins\org.eclipse.e4.workbench目录
删除里面的workbench.xmi文件

### 启动卡死
eclipse上一次没有正确关闭，导致启动的时候卡死错误解决方法
><workspace>\.metadata\.plugins\org.eclipse.core.resources目录
删除文件 .snap

