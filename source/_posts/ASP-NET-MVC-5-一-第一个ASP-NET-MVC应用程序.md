---
title: ASP.NET MVC 5(一) 第一个ASP.NET MVC应用程序
date: 2018-10-16 15:30:00
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5(一) 第一个ASP.NET MVC应用程序
---

***征途才刚刚开始：***这一系列ASP.NET MVC笔记是我大三上学期寒假，借助《精通ASP.NET MVC5》(Adma Freeman著，张成斌译)这本书学习基于.NET 4.5框架开发MVC5模式的ASP.NET WEB应用程序时记录的。使用的编程工具是Visaul Studio 2013 Commiunity。两年多的经验告诉我，读书时要紧跟练习，并且如果不想出现学过却不记得的情况，就要自己坚持每天做一个详细笔记，成为一个优秀程序员的道路是漫长的，而我才刚刚开始。
    在学习网站搭建的时候一直听前辈们推崇着MVC框架的优秀，在接触了C# 语言并开发过WPF桌面、WEB窗体程序，也接触过PHP语言介于MVC框架的网站开发之后，我觉得有一定的基础来比较深入的学习ASP.NET MVC的原理和概念，而不是只把它当做一个黑匣子会用就好。 所以建立的第一个ASP.NET MVC程序选择的模板是Empty而不是微软提供的任何模板，这也是学习这本书时作者采用的方式。这本书的代码示例可以从http://www.apress.com/cn/中获取，我的示例代码也会存储在github上：https://github.com/DoongBo

# 开始创建

 打开VS2013，文件->新建项目，选择ASP.NET WEB Application模板，按照书中的案例起名为PartyInvities。
新建项目选择ASP.NET Web应用程序
![这里写图片描述](/uploads/1-1.jpeg)
点击确定后初始项目配置选择空模板：Empty，并且在下方Add folders and core referrences(添加文件夹和核心引用)部分选中MVC复选框，来创建一个空的ASP.NET MVC项目。
![这里写图片描述](/uploads/1-2.jpeg)
创建完成后会在解决方案 资源管理器中看到初始的文件目录，先大概介绍一下MVC的特点有助于理解项目的组成结构。

#MVC框架简介## M--MODEL:
模型，数据模型，用于数据类型设计，在ASP.NET中可以理解为一系列自定义的类，它是项目中存储数据的模版。比如表示书本信息的类：Book类等等。有面向对象程序设计基础比如学习过JAVA或者C# 后，可以比较好的理解。
## V--VIEW:
视图，经过解析后HTTP传递展现的样式，在ASP.NET中视图文件类型为cshtml，php中可以是.php文件，他们大部分样式或者内容由html语言编写，其他内容因为采用技术不同而有所差异。
          这里需要说明的是，WEB页面由服务器生成到浏览器展示需要经过三步：第一步，渲染：视图引擎对视图文件进行解释将代码转化为html 标记，第二步，传递：将html标记传递到客户端，第三步，呈现：浏览器将html处理为web页面显示给客户。
## C--Controller:
控制器，处理请求的工具，在ASP.NET中为一些C# 类，它包含了许多函数方法，
      在MVC中不再是一个url对应一个真实的物理文件，而是对应一个控制器中的动作方法，每一个动作方法的默认视图与它同名，并且约定默认控制器为Home，默认动作方法为Index。在MVC中我们通常认为，约定优于配置，即最好按照事先的默认约定处理问题而不是个性化的配置，这不仅使得项目代码易读并且会避免许多麻烦。
目录文件：
![这里写图片描述](/uploads/1-3.jpeg)
        -App_Start文件夹中的.cs文件用于设置初始约定；
        -Controllers文件夹存储了控制器，这些控制且以Controller为文件名后缀，比如Home控制器的名称约定为HomeContrller；
        -Views文件夹存储了需要的视图文件而且一般每一个控制器对应的视图会放到与控制器同名的子文件夹下，比如Home控制器对应的视图所在的位置为~/Home/，默认会有视图文件Index.cshtml对应HomeCtroller.cs控制器中的Index()动作方法（~/Home/Index.cshtml）；
          Models文件夹存储自定义数据类模板。当然不是必须按照这种结构但约定如此。
          当想要添加一个控制器时可以在Controller文件夹上右键添加选择控制器，选择MVC 5 控制器-空，添加视图时右键View文件夹添加视图，添加模板时右键Model添加类，并且在控制器中可以在某一个动作方法出选择右键添加视图生成该动作方法的视图。

# 添加第一个控制器
在Controllers文件夹上右键添加选择控制器，选择MVC 5 控制器-空，点击添加后重命名为HomeController，点击添加后会在Controllers文件夹下看到文件HomeController.cs。
![这里写图片描述](/uploads/1-4.jpeg)
![这里写图片描述](/uploads/1-5.jpeg)
打开HomeController.cs会看到C# 代码
	
	using PartyInvities.Models;
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.Mvc;
	
	namespace PartyInvities.Controllers
	{
	    public class HomeController : Controller
	    {
	        // GET: Home
	        public ViewResult Index()
	        {
	                     return View();
	        }
	        }
	    }
	}
	
修改HomeController.cs

	using PartyInvities.Models;
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.Mvc;
	namespace PartyInvities.Controllers
	{
	    public class HomeController : Controller
	    {
	        // GET: Home
	        public string Index()
	        {
	                     return “Hello Word！”;
	        }
	        }
	    }
	}
	
经过修改后点击Debug按钮（绿色三角按钮，可以选择进行调试的浏览器，运行时出现红色方块按钮为停止按钮，只有停止调试后才可以继续修改代码）可以得到一个web页面，只显示了一行文字“ Hello Word！”。而当不经过修改直接运行时会提示找不到页面，这是因为return View()时返回的页面为当前动作的默认视图~Views/Home/Index.cshtml而你的项目中并没有这一视图。所以当我们不修改HomeController.cs而想程序直接运行时需要在Views文件夹中添加视图Index.cshtml，具体添加方法如下。
	
# 添加第一个视图
	
## 方法一：手动添加
由于需要添加的视图对应的控制器为Home，所以需要在Views文件夹下新添文件夹Home，再Home文件夹右键，添加MVC 5 视图页并命名为Index.cshtml
	
## 方法二：快速添加
在HomeController.cs的Index方法内右键选择添加视图，命名为Index，选择模板为Empty (不具有模型)，并将下面的三个复选框取消选中，点击添加后vs会自动创建Home控制器对应的视图文件夹Home并在里面添加Index方法对应的视图Index.cshtml。

Index.cshtml代码如下，主题部分由html提供，其他部分之后再做详细介绍。这是一个空页，可以给他添加一行文字
	@{
	    Layout = null;
	}
	
	<!DOCTYPE html>
	
	<html>
	<head>
	    <meta name="viewport" content="width=device-width" />
	    <title>Index</title>
	</head>
	<body>
	    <div> 
	    </div>
	</body>
	</html>
	
更改后的视图文件Index.cshtml：

	@{
	    Layout = null;
	}
	
	<!DOCTYPE html>
	
	<html>
	<head>
	    <meta name="viewport" content="width=device-width" />
	    <title>Index</title>
	</head>
	<body>
	    <div> 
		Hello World！（from the view）
	    </div>
	</body>
	</html>	
点击运行效果如下：
![这里写图片描述](../uploads/1-6.jpg)
	访问地址是http://localhost:61675，即访问本地服务器端口61675上的默认页面（Home）默认方法（Index），此端口号是随机产生的。


# 添加第一个模型
设置情景：朋友决定举办一个晚会，要做一个web程序来实现一下功能：
	1. 一个显示晚会信息的首页
	2. 一个用来进行电子回复（RSVP）的表单
	3. 对表单验证，显示一个“感谢你”的页面
	4. 完成RSVP时，给晚会主人发送一份电子邮件
因此设计一个数据模型以存储回复RSVP表单的信息，创建过程如下：
右键Models文件夹->添加->类，重命名为GuestRespones.cs，并修改其内容如下：
	
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.ComponentModel.DataAnnotations;
	
	namespace PartyInvities.Models
	{
	    public class GuestRespones
	    {
	        public string Name { get; set; }//姓名
	        public string Email { get; set; }//邮箱地址
	        public string Phone { get; set; }//电话号码
	        public bool? WillAttend { get; set; }//是否会参加
	    }
	}
	
其中特殊的WillAttend数据类型为可控的bool类型，即它的值可以是true，false或者null。它的值需要在控制器中验证是否为空，所以需要有null值，而bool只能为true或flase，这可能会导致变量未赋值的错误，除非页面加载时给它赋初值。

本篇笔记就到这里，下一篇将继续介绍如何使用视图、控制器和模型完成此次情景所需功能。