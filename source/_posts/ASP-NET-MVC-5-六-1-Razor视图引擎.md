---
title: ASP.NET MVC 5 (六-1) Razor视图引擎
date: 2018-10-16 15:31:36
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (六-1) Razor视图引擎
---
**视图引擎**负责处理ASP.NET内容，并查找相关指令，将动态的内容插入到发送给浏览器的输出，而Razor是MVC框架视图引擎的名称。

@[toc]
# 准备示例项目
&nbsp;&nbsp;&nbsp;&nbsp; 使用Visual Studio 2013建立ASP.NET Web应用程序并命名为“Razor“，选择Empty（空）选项并勾选MVC复选框。
## 定义模型
&nbsp;&nbsp;&nbsp;&nbsp;右键Models文件夹选择添加Class，命名为Product并打开编辑为如下内容：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	namespace Razor.Models
	{
	    public class Product
	    {
	        public string Name { get; set; }
	        public int IntProductID { get; set; }
	        public string Description { get; set; }
	        public string Category { get; set; }
	        public decimal Price { get; set; }
	    }
	}
这和之前的文章中建立的Product类是一样的。
## 定义控制器
&nbsp;&nbsp;&nbsp;&nbsp;在Controllers文件夹上右键选择添加空的控制器（MVC 5 Controller-Empty），命名为HomeController，打开并修改为如下内容：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.Mvc;
	using Razor.Models;
	
	namespace Razor.Controllers
	{
	    public class HomeController : Controller
	    {
	        Product myproduct = new Product
	        {
	            Name = "Kayak",
	            Price = 275M,
	            IntProductID = 1,
	            Category = "Watersports",
	            Description = "A boat for one person"
	        };
	        // GET: Home
	        public ActionResult Index()
	        {
	            return View(myproduct);
	        }
	    }
	}
在控制器中创建了一个Product对象并对其属性进行填充同时还定义了一个Index动作方法，将Product对象传递给View。
## 创建视图
&nbsp;&nbsp;&nbsp;&nbsp;在Home控制器的Index方法右键选择添加视图，视图名称为Index，模板为空，模型类型为Product。
其内容如下：

	@model Razor.Models.Product
	@{
		Layout=null;
	    ViewBag.Title = "Index";
	}
	<html>
	<head>
	    <meta name="viewport" content="width=device-width" />
	    <title>@ViewBag.Title</title>
	</head>
	<body>
		<h2>Index</h2>
		@Model.Name
	</body>
	</html>

	
# 使用模型对象
&nbsp;&nbsp;&nbsp;&nbsp;从视图文件第一行开始
	
	@model Razor.Models.Product
Razor语句以**@**字符开头。此例中  `@model` 语句声明该强类型视图模型的对象模型，这样就可以通过`@Model`来引用视图模型对象的方法。字段和属性。如`@Model.Name` 便可以引用Product对象的Name属性。
有意思的是声明模型要使用`@model` 小写字母开头，而使用模型对象却要使用`@Model` 大写字母开头。
# 使用布局
&nbsp;&nbsp;&nbsp;&nbsp;Index视图另一个表达式是

	@{
			Layout=null;
		    ViewBag.Title = "Index";
		}
&nbsp;&nbsp;&nbsp;&nbsp;这是一个**Razor代码块** 的例子，这种代码块允许在视图中包含C# 代码，它以`@{`开始，以`}`结束。上述代码将Layout设置为null，它指明这个视图是自包含的、不采用布局模板，但当一个项目包含多个视图时，可能就需要一个布局模板来统一界面样式了。因此一下介绍如何建立并引用一个布局模板。
##  创建布局
&nbsp;&nbsp;&nbsp;&nbsp;在解决方案资源管理器中右键Views文件夹选择添加新建项，打开对话框中选择 MVC 5 Layout Page（Razor）（MVC5布局页（Razor））模板，将文件命名为_BasicLayout.cshtml（第一个字符为下划线）。需要说明的Views文件夹中是下划线开头的文件为视图支持文件，不会返回给客户，而其他的视图文件可以返回给客户。_BasicLayout.cshtml打开并修改为如下内容：
	
	<!DOCTYPE html>
	<html>
	<head>
	    <meta name="viewport" content="width=device-width" />
	    <title>@ViewBag.Title</title>
	</head>
	<body>
	    <h1>Product Name</h1>
	    <div>
	        @RenderBody()
	    </div>
	    <h2>DoongBo@outlook.com</h2>
	</body>
	</html>
&nbsp;&nbsp;&nbsp;&nbsp;布局是特殊形式的视图，`@RenderBody()` 方法调用会将动作方法指定的视图内容插入到布局标记里。另一个Razor表达式`@ViewBag.Title` 会在ViewBag中查找一个Title属性。
## 运用布局
&nbsp;&nbsp;&nbsp;&nbsp;运用布局于视图，只要在视图文件中设置Layout属性的值即可。布局中包含了html的结构，所以引用布局后视图文件就可以删除这些内容，因此Index引用_BasciLayout.cshtml布局后内容改为如下：

	@model Razor.Models.Product
	@{
	    ViewBag.Title = "Index";
	    Layout = "~/Views/_BasciLayout.cshtml";
	}
	<h2>Index</h2>
	@Model.Name
运行结果如下：

![使用Layout属性指定视图的布局](/uploads/6-1.jpeg)
# 使用共享布局
&nbsp;&nbsp;&nbsp;&nbsp;如果想对每一个视图运用布局，则没有必要对每一个视图文件制定Layout属性，只要使用起始布局文件即可。
在渲染一个视图时MVC会寻找一个_ViewStart.cshtml的文件并把它作为视图文件的一部分。因此在Views文件夹下添加一个_ViewStart.cshtml文件并打开编辑为如下内容：

	@{
	    Layout = "~/Views/_BasciLayout.cshtml";
	}
然后对需要采用_ViewStart.cshtml（第一个字符是下划线）指定的布局时，在视图文件中把Layout属性删掉即可。或者在创建视图时，选定Use a Layout page但不在下面的文本框中输入值，这样创建的视图就不会带有Layout属性而采用起始布局文件的指定内容。需要说明的是设置`Layout=null` 是指明本视图不采用布局，而不设置`Layout` 属性是指采用默认布局，两者是不同的。
&nbsp;&nbsp;&nbsp;&nbsp;添加_ViewStart.cshtml之后对Index视图修改，删除其Layout属性后运行结果未变。修改后的Index.cshtml：

	@model Razor.Models.Product
	@{
	    ViewBag.Title = "Index";
	}
	<h2>Index</h2>
	@Model.Name
运行结果
![视图文件使用起始布局文件，删除Layout属性运行结果](/uploads/6-2.jpeg)