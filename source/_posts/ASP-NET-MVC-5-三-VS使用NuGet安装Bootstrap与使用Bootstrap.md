---
title: ASP.NET MVC 5 (三) VS使用NuGet安装Bootstrap与使用Bootstrap
date: 2018-10-16 15:30:34
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (三) VS使用NuGet安装Bootstrap与使用Bootstrap
---

@[toc]
# 加载bootstrap
在Visual Studio中Tools（工具）菜单中找到NuGet程序包管理器中打开包管理器控制台，在其中输入以下命令并按回车键：

	PM> Install-Package -version 3.0.0 bootstrap
该命令会下载bootstrap中的jQuery、js和css文件，CSS文件会被存储在Content文件夹中，会自动新建Scripts文件夹并存储javascript文件，同时会建立一个fonts文件夹存储一些特定文件。
现在文件目录结构如下：
![这里写图片描述](/uploads/3-1.jpeg)
Content中有.min.前缀的配对文件，它是压缩即删除所有空格的javascript和css文件，
以减少浏览器传递内容所需的带宽量，并且vs自动管理。
现在需要将所需的.css文件引用到所需的视图中，方法与引用Style.css文件一致，然后可以对每一个元素进行class标签属性的赋值。
# 设置Index视图

Index.cshtml添加bootstrap
	
	@{
	    Layout = null;
	}
	
	<!DOCTYPE html>
	
	<html>
	<head>
	    <meta name="viewport" content="width=device-width" />
	    <link href="~/Content/bootstrap-theme.css" rel="stylesheet" />
	    <link href="~/Content/bootstrap.css" rel="stylesheet" />
	    <title>Index</title>
	    <style>
	        .btn a{color:white;text-decoration:none}
	        body{background-color:# f1f1f1;}
	    </style>
	</head>
	<body>
	    <div class="text-center">
	        <h2>
	            We are going to have an exciting party!
	        </h2> 
	        <h3>
	            And you are invited!
	        </h3>
	        <div class="btn btn-success">
	            @Html.ActionLink("RSVP Now", "RsvpForm")
	        </div>
	    </div>
	</body>
	</html>

![这里写图片描述](/uploads/3-2.jpeg)

其他视图修改方法类似，不过有一点需要指明，就是采用html辅助器生成的标记元素指定class的方法：
	
	@Html.TexBoxFor(x=>x.Name,new{@class="form-control"})
关于如何使用Razor的布局（Laout）和分布视图来减少代码和重复标记，将在以后介绍。
本章到此结束，第一个简单的MVC示例完成。