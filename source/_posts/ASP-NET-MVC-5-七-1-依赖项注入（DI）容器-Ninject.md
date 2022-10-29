---
title: ASP.NET MVC 5 (七-1)依赖项注入（DI）容器-Ninject
date: 2018-10-16 15:31:46
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (七-1)依赖项注入（DI）容器-Ninject
---
MVC模式最重要的特性之一是它支持关注分离，应用程序中的组件尽可能独立，而只有很少的几个可控依赖项，在理想的情况下，每个组件都不了解其他组件，而只是通过抽象接口来处理应用程序的其他区域 ，这称为“松耦合”，它使得应用程序更易测试和修改。
**依赖项注入**可以实现获取实现某接口的对象而不必直接创建该对象，这也称为**控制反转**。简单理解就是说，为了使得在一个类中不必使用`new`关键字创建一个对象而可以直接实现一个接口对象，添加一个依赖项注入容器管理应用程序中所有的依赖项。这里采用的方式为**Ninject**DI容器 （Dependncy Injection Container，依赖项注入容器）。也有其他的类似DI工具，比如微软的Unity。

@[toc]
# 准备示例项目
创建ASP.NET MVC WEB 应用程序， 解决方案命名为EssentialTools，像以前一样，选择Empty(空)模板并选中MVC复选框。然后向解决方案中添加模型、控制器和视图。
## 添加模型类
 - 在Models文件夹内添加C# 类Product，这是之前见过很多次的一个模型类了，它是产品类。编辑为以下内容：

		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Web;
		
		namespace EssensialTools.Models
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

 - 如果想有对一组产品进行一些逻辑上的计算处理的方法，比如求总价、求平均价格等等，我们可以专门建立一个类来处理这些问题。因此又在Models文件夹内添加一个C# 类LinqValueCalculator，暂时只让它可以计算一组产品的总价，则修改其内容为：

		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Web;
		
		namespace EssentialTools.Models
		{
		    public class LinqValueCalculator
		    {
		        public decimal ValueProducts(IEnumerable<Product> products)
		        {
		           return products.Sum(p => p.Price);
		        }
		    }
		}
这个类中的ValueProducts方法使用LINQ的Sum方法将传入的一组Product对象的Price属性值相加并返回该值。

 - 最后想建立一个简单的购物车类ShoppingCart，它存储一组产品信息，并可以使用上述ValueProducts计算组的总价。
编辑其内容为：
	
		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Web;
		
		namespace EssentialTools.Models
		{
		    public class ShoppingCart
		    {
			    //定义LinqValueCalculator对象
		        private LinqValueCalculator calc;
		        
		        //重构构造函数
		        public ShoppingCart(LinqValueCalculator calcParam)
		        {
		            calc = calcParam;
		        }
				//产品组
		        public IEnumerable<Product> Products { get; set; }
		        //计算总价
		        public decimal CalculateProductTotal()
		        {
		            return calc.ValueProducts(Products);
		        }
		    }
		}

## 添加一个控制器
在Controllers文件夹内添加第一个控制器HomeController，编辑为以下内容：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.Mvc;
	using EssensialTools.Models;
	
	namespace EssensialTools.Controllers
	{
	    public class HomeController : Controller
	    {
	    //定义一组产品的信息
	        private Product[] products ={
	                                       new Product{Name="Kayak",Price=275M,Category="Watersports"},
	                                       new Product{Name="Lifejacket",Price=48.95M,Category="Watersports"},
	                                       new Product{Name="Soccer",Price=19.50M,Category="Soccer ball"},
	                                       new Product{Name="Kayak",Price=34.95M,Category="Corner flag"}
	                                   };
	        //默认方法            
	        public ActionResult Index()
	        {
	            LinqValueCalculator calc=new LinqValueCalculator();
	            //创建购物车对象并给属性Products赋值
	            ShoppingCart cart = new ShoppingCart(calc) { Products = products };
	            //使用购物车对象调用计算总价的方法
	            decimal totalValue = cart.CalculateProductTotal();
	            //返回视图，视图的数据模型为decimal
	            return View(totalValue);
	        }
	    }
	}
注意增加引用EssensialTools.Models命名空间。
# 添加视图
在Home控制器的Index方法内右键选择添加视图，并确定视图名称为Index，其他的选项是否选中以不再重要，因为当你把视图文件代码修改为以下内容时，你的选择会随之发生改变。Index.cshtml修改内容为如下：

	@model decimal
	@{
	    Layout = null;
	    ViewBag.Title = "Index";
	}
	<!DOCTYPE html>
	<html>
	<head>
	    <meta name="viewport" content="width=device-width" />
	    <title>Value</title>
	</head>
	<body>
	    <div>
	        Total value is $@Model
	    </div>
	</body>
	</html>
该视图的是decimal的强类型视图，Razor表达式`@Model`这时会代表一个decimal数。
此时就可以运行项目，会得到界面内容如下：
![初始界面](/uploads/7-1.jpeg)

# 使用Ninject
Ninject是依赖项注册容器，那我们为什么需要依赖项注入呢，什么是依赖项呢？依赖项要解决的基本问题是紧耦合类。
简单的来说，上面的几个C# 类中，ShoppingCart类中定义了一个LinqValueCalculator类的对象，因此两者之间是紧耦合的，HomeController类中使用`new` 关键字定义了一个ShoppingCart类的对象和LinqValueCalculator类对象，因此三者是紧耦合的。这意味着如果想替换LinqValueCalculator类，就必须修改与它有紧耦合关系的所有类中找出对它的引用并替换掉，这在大型项目中是非常讨厌的。
## 使用接口
通过使用C# 接口可以解决部分问题。
在Models文件夹内添加一个接口类，添加方式是，右键Models文件夹->添加->新项，选择Interface并命名为IValueCalculator，编辑其内容为如下：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace EssentialTools.Models
	{
	        public interface IValueCalculator
	        {
	            decimal ValueProducts(IEnumerable<Product> products);
	        }
	}
注意类定义为`public`型，该接口抽象出计算总价的方法。然后可以在LinqValueCalculator类中实现这一接口，如下：


		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Web;
		
		namespace EssentialTools.Models
		{
		    public class LinqValueCalculator:IValueCalculator
		    {
		        public decimal ValueProducts(IEnumerable<Product> products)
		        {
		           return products.Sum(p => p.Price);
		        }
		    }
		}
这个接口可以~~让~~解除ShoppingCart与LinqValueCalculator类之间的紧耦合关系，修改ShoppingCart类内容如下：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace EssentialTools.Models
	{
	    public class ShoppingCart
	    {
	        private IValueCalculator calc;
	        public ShoppingCart(IValueCalculator calcParam)
	        {
	            calc = calcParam;
	        }
	        public IEnumerable<Product> Products { get; set; }
	        public decimal CalculateProductTotal()
	        {
	            return calc.ValueProducts(Products);
	        }
	    }
	}
此时ShoppingCart与LinqValueCalculator之间的紧耦合关系已经解除，因为在使用ShoppingCart类时只需要为其构造器（构造函数）传递一个IValueCalculator 接口对象就行了。
但C# 要求在接口实例化时要指定其实现类，因为要知道想用哪一个实现类（如果有多个类实现了该接口，则在实例化接口时就要指明用的哪一个类实现的），这意味着当在Home控制器中实例化IValueCalculator 对象时需要指明其实现类LinqValueCalculator，此时HomeController与LinqValueCalculator依旧是紧耦合的关系。如HomeController使用IValueCalculator 对象的代码如下：

	...
	public ActionResult Index()
	        {
	            IValueCalculator calc = new LinqValueCalculator();
	            ShoppingCart cart = new ShoppingCart(calc) { Products = products };
	            decimal totalValue = cart.CalculateProductTotal();
	            return View(totalValue);
	        }
	...
要处理这种问题就需要采用Ninject了，来实现对IValueCalculator 接口的实现进行实例化，但它不是在HomeController代码中实现的，否则就没有实际意义了。即HomeController中` IValueCalculator calc = new LinqValueCalculator();`代码不需要出现在控制其中，就可以知道是使用哪一个实现类实现的本接口。
## 将Ninject引入Visual Studio项目
使用VS的NuGet集成可以快速简单的实现该目的。打开VS菜单栏的”Tools（工具）“->”Libarary Package Manager“->"Package Manager Console"打开NuGet命令行，或者”Tools（工具）“->”Libarary Package Manager“->"NuGet Manager Console"。
输入如下命令：
	
	Install-Package Ninject -version 3.0.1.10
	
	Install-Package Ninject.Web.Common -version 3.0.0.7

	Install-Package Ninject.MVC3 -version 3.0.0.6
等待安装Ninject完成后就可以使用了。
## Ninject初步
要得到Ninject的基本功能，要做得工作分为3个阶段，如HomeController的Index方法已修改如下（当然需要添加命名空间` using Ninject;`）：

	 public ActionResult Index()
	        {
	            IKernel ninjectKernel = new StandardKernel();
	            ninjectKernel.Bind<IValueCalculator>().To<LinqValueCalculator>();
	            IValueCalculator calc = ninjectKernel.Get<IValueCalculator>();
	            ShoppingCart cart = new ShoppingCart(calc) { Products = products };
	            decimal totalValue = cart.CalculateProductTotal();
	            return View(totalValue);
	        }
第一个阶段是准备使用Ninject，创建一个Ninject的Kernel（内核）实例，它是一个内核对象，负责解析依赖项并创建新的对象（比如用内核创建一个IValueCalculator 对象，而不再使用`new`关键字）。

 - 第一阶段的代码为：

	IKernel ninjectKernel = new StandardKernel();
StandardKernel类是Ninject.IKernel 接口的标准实现类（标准内核），它足以完成我们想要的任何功能要求。
 - 第二阶段则是配置内核，即指定某一个接口是由那个类实现的，代码如下：

	 ninjectKernel.Bind<IValueCalculator>().To<LinqValueCalculator>();
它说明IValueCalculator接口的依赖项解析应该通过创建LinqValueCalculator类的实例来进行。
 - 第三阶段就是使用Ninject（或者说使用内核）创建（IValueCalculator）对象了。其代码如下：

	 IValueCalculator calc = ninjectKernel.Get<IValueCalculator>();

这里已经取得可一些进展，但它依旧没有解决Home控制器和LinqValueCalculator的紧耦合问题，那需要做的就是下一步，**依赖项注入**了。
## 建立MVC的依赖项注入
依赖项注入实现的功能就是，将上问Home控制器中的 ***内核创建、配置和创建对象***拿出来放到一个专门的类中，并保证在应用程序运行时依赖项得到解析。
## # 创建依赖项解析器
创建一个单独的类做依赖项的解析器，因此创建一个新的文件夹Infrastructure来存放不适合放在其他文件夹内的类，并创建一个NinjectDependencyResolver类做依赖项解析器。这个类修改为一下内容：
	
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web.Mvc;
	using EssensialTools.Models;
	using Ninject.Web.Common;
	using Ninject;
	
	namespace EssensialTools.Infrastructure
	{
	    public class NinjectDependencyResolver : IDependencyResolver
	    {
	        private IKernel kernel;
	        public NinjectDependencyResolver(IKernel kenelParam)
	        {
	            kernel = kenelParam;
	            AddBindings();
	        }
	        public object GetService(Type serviceType)
	        {
	            return kernel.TryGet(serviceType);
	        }
	        public IEnumerable<object> GetServices(Type serviceType)
	        {
	            return kernel.GetAll(serviceType);
	        }
	        private void AddBindings()
	        {
	             kernel.Bind<IValueCalculator>().To<LinqValueCalculator>();
	        }
	    }
	}
NinjectDependencyResolver 类实现了System.Web.Mvc命名空间中的IDependencyResolver接口，MVC框架用它来获取所需的对象。
MVC框架在需要类实例以便对一个传入请求进行服务时，会调用`GetService`和`GetServices`方法。依赖性解析器器要做的工作就是创建这一实例。
上述依赖性解析器也是建立Ninject绑定（自定义接口和实现类的绑定）的地方，在`AddBindings`方法中便实现了IValueCalculator接口和LinqValueCalculator实现类之间关系的配置。
## # 注册依赖项解析器
当然创建完依赖项解析器后还是无法实现功能，因为我们还没有告诉MVC去使用它，那此时该做的事情就是注册该解析器让MVC去使用它。
当我们引入Ninject时它会在App_Start文件夹中创建一个NinjectWebCommon.cs文件，它定义了应用程序启动时会自动调用的一些方法。在NinjectWebCommon类中的RegisterServices方法中我们填写一条语句来创建一个依赖项解析器NinjectDependencyResolver 的实例，并用 System.Web.Mvc.DependencyResolver类定义的SetResolver静态方法将其注册为MVC框架的解析器，如下：

	  private static void RegisterServices(IKernel kernel)
	        {
	            System.Web.Mvc.DependencyResolver.SetResolver(
	                new EssentialTools.Infrastructure.NinjectDependencyResolver(kernel));
	        }   
直到此时我们已经做得工作有：创建依赖项解析器，并在其中将LinqValueCalculator绑定为IValueCalculator接口的实现类，并且告诉MVC使用我们的依赖项解析器。准备工作已经准备就绪了，看看我们怎样使用费了这么大劲终于完成的依赖项注入吧。
## # 重构Home控制器

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.Mvc;
	using EssentialTools.Models;
	using Ninject;
	
	namespace EssentialTools.Controllers
	{
	    public class HomeController : Controller
	    {
	     
	        private Product[] products ={
	                                       new Product{Name="Kayak",Price=275M,Category="Watersports"},
	                                       new Product{Name="Lifejacket",Price=48.95M,Category="Watersports"},
	                                       new Product{Name="Soccer",Price=19.50M,Category="Soccer ball"},
	                                       new Product{Name="Kayak",Price=34.95M,Category="Corner flag"}
	                                   };
	        private IValueCalculator calc;                          
	        public HomeController(IValueCalculator calcParam,IValueCalculator calc2)
	        {
	            calc = calcParam;
	        }
	    
	        public ActionResult Index()
	        {
	          ShoppingCart cart = new ShoppingCart(calc) { Products = products };
	            decimal totalValue = cart.CalculateProductTotal();
	            return View(totalValue);
	        }
	    }
	}
所做的更改主要有

 1. 创建HomeController类的带参构造函数，参数类型为接口IValueCalculator ，并在类中创建一个该接口的实例来存储构造函数带来的参数。当MVC创建HomeController实例时发现它带有IValueCalculator 参数便要求解析器解析它，此时我们自定义并注册的解析器就会解析到IValueCalculator 与LinqValueCalculator之间的依赖项关系，此时Ninject就会创建一个LinqValueCalculator对象返回给MVC框架，MVC利用该返回实例创建控制器实例。
 2. 删除Index方法中任何关于Ninject或LinqValueCalculator有关的代码，因为此时已经不需要它们了。
现在再看Home控制器，会发现它与LinqValueCalculator之间的紧耦合关系已经被打破了。
这样当需要修改IValueCalculator 的实现类时只需要修改依赖性解析器就可以了。

运行结果向未曾改变一样。
![依赖项注入之后的运行结果](/uploads/7-1.jpeg)