---
title: ASP.NET MVC 5 (六-2) 使用Razor表达式
date: 2018-10-16 15:31:35
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (六-2) 使用Razor表达式
---
&nbsp;&nbsp;&nbsp;&nbsp;上一章介绍**Razor视图与布局**，现在可以在此基础上使用**Razor表达式**来创建视图内容。

@[toc]
# 开始使用Razor表达式
&nbsp;&nbsp;&nbsp;&nbsp;因为MVC是注重并强迫应用程序各部分之间的分离，因此虽然Razor表达式可以使用C# 语句，但最好不要使用Razor执行业务逻辑或对域模型进行操作。同样不应该对动作方法传递给视图的数据进行格式化而应该传递一个完整的模型对象。如：

	...
	 public ActionResult NameAndPrice()
	        {
	            return View(myproduct);
	        }
	...
这样在视图中使用Razor表达式可以得到要插入属性的值：
	
	...
	The product name is @Model.Name and it costs $@Model.Price 
	...

# 插入数据值
&nbsp;&nbsp;&nbsp;&nbsp;当希望控制器向视图插入数据时可以采用`@ModelRazor表达式`，也可以使用`@ViewBag`表达式以引用试图包特性动态定义的属性。以下的例子则采用这两种方式向视图传递数据。
在HomeConroller.cs文件添加一个DemoExpression动作方法：
	
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
	        public ActionResult NameAndPrice()
	        {
	            return View(myproduct);
	        }
	        public ActionResult DemoExpression()
	        {
	            ViewBag.ProductCount = 1;
	            /*使用ViewBag动态属性传递数据*/
	            ViewBag.ExpressShip = true;
	            ViewBag.ApplyDiscount = false;
	            ViewBag.Supplier = null;
	            /*向视图传递Product对象*/
	            return View(myproduct);
	        }
	    }
	}
&nbsp;&nbsp;&nbsp;&nbsp;在Views/Home文件夹中创建一个名称为DemoExpression.cshtml的强类型视图来演示这些基本表达式传递的数据。

	@model Razor.Models.Product
	@{
	    ViewBag.Title = "DemoExpression";
	} 
	<table>
	<thead>
        <tr>
            <th>Property</th><th>Value</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Name</td><td>@Model.Name</td>
        </tr>
        <tr>
            <td>Pric</td>
            <td>@Model.Price</td>
        </tr>
        <tr>
            <td>Stock Level</td>
            <td>
               <b>@ViewBag.ProductCount</b>
            </td>
        </tr>
    </tbody>
	</table>
&nbsp;&nbsp;&nbsp;&nbsp;这里创建了一个html表格，使用模型对象和视图包的一些属性填充了单元格的值。启动项目并导航到/Home/DemoExpression，可以看到结果如下：
![使用viewbag和模型对象传递数据](/uploads/6-1-1.jpeg)
# 设置标签属性值
&nbsp;&nbsp;&nbsp;&nbsp;Razor表达式不仅可以设置元素内容，还可以设置标签属性的值，在DemoExpression.cshtml最后添加一个div和一组checkbox，修改为如下内容：

	@model Razor.Models.Product
		@{
		    ViewBag.Title = "DemoExpression";
		} 
		<table>
		<thead>
	        <tr>
	            <th>Property</th><th>Value</th>
	        </tr>
	    </thead>
	    <tbody>
	        <tr>
	            <td>Name</td><td>@Model.Name</td>
	        </tr>
	        <tr>
	            <td>Pric</td>
	            <td>@Model.Price</td>
	        </tr>
	        <tr>
	            <td>Stock Level</td>
	            <td>
	               <b>@ViewBag.ProductCount</b>
	            </td>
	        </tr>
	    </tbody>
		</table>
	<div data-discount="@ViewBag.ApplyDiscount" dataexpress="@ViewBag.ExpressShip" data-supplier="@ViewBag.Supplier">
	    The containing element has data attributes
	</div>
	
	Discount:<input type="checkbox" checked="@ViewBag.ApplyDiscount" />
	Express:<input type="checkbox"checked="@ViewBag.ExpressShip" />
	Supplier:<input type="checkbox" checked="@ViewBag.Supplier" />
		
&nbsp;&nbsp;&nbsp;&nbsp;这里使用Razor表达式在div元素上设置了一些data标签属性的值。
&nbsp;&nbsp;&nbsp;&nbsp;Razor会将值为null的视图包属性或模型属性渲染成空字符串，并且Razor处理checked这种属性标签时，当值false或null时会完全删除该标签属性。因为当checked属性为false、null或空字符串时，浏览器会显示复选框已勾选，而Razor直接删除该属性则使得复选框处于未选中状态了。
&nbsp;&nbsp;&nbsp;&nbsp;运行结果如下：
![Razor设置标签属性](/uploads/6-2-1.jpeg)

# 使用条件语句
&nbsp;&nbsp;&nbsp;&nbsp;Razor能够处理条件语句，就可以根据视图数据的值，对视图输出进行剪辑而创造复杂而流畅的视图，并使其易于阅读和维护。
如在DemoExpression.cshtml视图中添加了一个条件语句;
	
	@model Razor.Models.Product
		@{
		    ViewBag.Title = "DemoExpression";
		} 
		<table>
		<thead>
	        <tr>
	            <th>Property</th><th>Value</th>
	        </tr>
	    </thead>
	    <tbody>
	        <tr>
	            <td>Name</td><td>@Model.Name</td>
	        </tr>
	        <tr>
	            <td>Pric</td>
	            <td>@Model.Price</td>
	        </tr>
	        <tr>
	            <td>Stock Level</td>
	            <td>
	                 @switch((int)ViewBag.ProductCount)
                {
                    case 0:
                        @:Out of stock
                        break;
                    case 1:
                        <b>Low Stock (@ViewBag.ProductCount)</b>
                        break;
                    default:
                        @ViewBag.ProductCount
                        break;   
                }
	            </td>
	        </tr>
	    </tbody>
		</table>
&nbsp;&nbsp;&nbsp;&nbsp;要开始一个条件语句，要在C# 条件关键字前面放一个`@`字符，以`{`结束。需要注意的是在switch中，Razor表达式不能求取动态属性的值，因此必须进行强制类型转换为int类型。
在Razor代码块内部，只要定义html以及Razor表达式就可以将html元素和数据插入到视图输出，如

	...
	 <b>Low Stock (@ViewBag.ProductCount)</b>
	...
或者	像这样：
			
	@ViewBag.ProductCount
不必使用其他任何特殊方式来表示html元素和Razor表达式。还有就是，可以使用如下一个Razor辅助工具直接显示文本文字：
	
	 @:Out of stock
`@：`字符阻止Razor将此行解释为一条C# 语句。
&nbsp;&nbsp;&nbsp;&nbsp;以下是Razor视图使用if语句：

	@model Razor.Models.Product
		@{
		    ViewBag.Title = "DemoExpression";
		} 
		<table>
		<thead>
	        <tr>
	            <th>Property</th><th>Value</th>
	        </tr>
	    </thead>
	    <tbody>
	        <tr>
	            <td>Name</td><td>@Model.Name</td>
	        </tr>
	        <tr>
	            <td>Pric</td>
	            <td>@Model.Price</td>
	        </tr>
	        <tr>
	            <td>Stock Level</td>
	            <td>
	@if (ViewBag.ProductCount == 0) { 
                    @:Out of Stock
                }
                else if (ViewBag.ProductCount == 1)
                {
                    <b>Low Stock (@ViewBag.ProductCount)</b>
                }
                else
                {
                 }
                </td>
	        </tr>
	    </tbody>
		</table>
# 枚举数组和集合
&nbsp;&nbsp;&nbsp;&nbsp;在Home控制器中定义`DemoArray`动作方法演示如何在MVC中传递和使用枚举和数组集合。

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
	        ...省略了其他动作方法
	        public ActionResult DemoArray()
	        {
	            Product[] arry ={
	                               new Product{Name="Kayak",Price=275M},
	                               new Product{Name="LifeTackt",Price=48.95M},
	                               new Product{Name="Soccer",Price=19.50M},
	                               new Product{Name="Corner flag",Price=34.95M}
	                           };
	            return View(arry);
	        }
	      }
	}
&nbsp;&nbsp;&nbsp;&nbsp;该动作方法创建了Product[]对象并传递给View方法，以使用默认视图来渲染这些数据。Visual Studio无法指定一个数组为模型类型，但可以在视图文件中手动修改强制模型视图的模型类型，以下是添加的DemoArray.cshtml视图文件代码：

	@using Razor.Models
	@model Razor.Models.Product[]
	@{
	    ViewBag.Title = "DemoArray";
	}
	
	@if (Model.Length > 0)
	{
	    <table>
	        <thead>
	            <tr>
	                <th>Product</th>
	                <th>Price</th>
	            </tr>
	        </thead>
	        <tbody>
	            @foreach (Product p in Model)
	            {
	                <tr>
	                    <td>@p.Name</td>
	                    <td>$@p.Price</td>
	                </tr>
	            }
	        </tbody>
	    </table>
	}
	else
	{
	    <h2>No product data</h2>
	}

&nbsp;&nbsp;&nbsp;&nbsp;`@model Razor.Models.Product[]`指定本视图的强制模型类型为Product[]，这里使用了一个`@if-else`语句，以便根据数据内容修改视图。另外，使用一个`@foreach`表达式来枚举数组的内容，并为每一条数据生成一个html表格行。可以看出来，在`foreach`循环中创建了一个`p`局部变量，然后永Razor表达式`@p.Name`和`@p.Prcie`引用了它的属性。
导航到/Home/DemoArray，运行结果如下：
![Razor使用条件语句](/uploads/6-3-1.jpeg)
# 处理命名空间
&nbsp;&nbsp;&nbsp;&nbsp;上例中

	@using Razor.Models
代码像.cs类中一样引用了一个命名空间，只不过需要在`using`前面加一个`@`符号而已。
# 番外篇
&nbsp;&nbsp;&nbsp;&nbsp;倒霉的是，这两天遇到了一个尴尬的问题：Visual Studio编译运行ASP.NET web应用程序时打开浏览器总是会显示“无法显示此页”，查找了各种处理方法都没法解决，之前重新启动Visual Studio或者重启电脑就可以解决了，但这次这种粗暴的方法好像被免疫了。我之前用的VS2013，就算我再安装了VS2015同样不好用。
&nbsp;&nbsp;&nbsp;&nbsp;有人说是浏览器的问题，但我电脑上的5个浏览器（Edge、IE、Google Chrome、360安全浏览器、世界之窗浏览器）都不行；有人说防火墙或者安全软件的问题，但我关闭了防火墙和安全软件（没有安装NOD32安全防护，使用的是腾讯电脑管家 ）也没有卵用；有人说网卡驱动的问题，我使用驱动精灵更新了所有驱动，可悲是的电脑直接崩溃了，祸不单行啊！
&nbsp;&nbsp;&nbsp;&nbsp;这下好了，电脑黑屏一下午就是无法开机，用win10自带的修复工具重装了系统才能重新开机，这下终于又能愉快的调试了！！
&nbsp;&nbsp;&nbsp;&nbsp;不过也因祸得福，电脑磁盘一下干净多了，而且其他盘里的资料都保留下来了，重新装VS和Office和LOL又费了我一整天时间，哈哈哈哈！！