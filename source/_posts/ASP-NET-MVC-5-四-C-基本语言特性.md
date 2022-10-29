---
title: 'ASP.NET MVC 5 (四) C# 基本语言特性'
date: 2018-10-16 15:30:42
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (四) C# 基本语言特性
---
@[toc]
介绍C# 语言的基本特性。
# 创建示例代码
使用ASP,.NET MVC Web 应用程序，如第一章说的，创建一个空的程序，命名为LanguageFeatures，创建完成后在控制器文件夹Controllers中添加一个控制器HomeController.cs，打开编辑修改为如下内容：
	
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.Mvc;
	using LanguageFeatures.Models; //此时该命名空间不会被解析，只要在Models文件夹中添加一个类文件之后就会解析了
	namespace LanguageFeatures.Controllers
	{
	    public class HomeController : Controller
	    {
	        // GET: Home
	        public string Index()
	        {
	            return "Navigate to a URL to show an example";
	        }
	    }
	}
	
在Index方法中右键单击添加视图，添加一个命名为Result.cshtml的视图文件，修改为一下内容：

	@model string
	@{
	    Layout = null;
	}
	<!DOCTYPE html>
	<html>
	<head>
	    <meta name="viewport" content="width=device-width" />
	    <title>Result</title>
	</head>
	<body>
	    <div> 
	        @Model
	    </div>
	</body>
	</html>
这是一个强类型视图，模型类型为String。

# 添加System.Net.Http程序集
后面的一些内容依赖于该程序集，所以需要添加。添加方法是在解决方案资源管理器中右键单击References选择Add References添加程序集，找到System.Net.Http程序集点击确定即可。

# 添加模型类Product.cs
在Models文件及右键单击Add->class，添加名称为Product.cs的模型类，打开并且修改为如下内容：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	namespace LanguageFeatures.Models
	{
	    public class Product
	    {
	        public string Name{get;set;}       
	    }
	}
	
在HomeController中添加对Product的使用：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using System.Web.Mvc;
	using LanguageFeatures.Models; /*此时该命名空间不会被解析，只要在Models文件夹中添加一个类文件之后就会解析了*/
	namespace LanguageFeatures.Controllers
	{
	    public class HomeController : Controller
	    {
	        // GET: Home
	        public string Index()
	        {
	            return "Navigate to a URL to show an example";
	        }
	
	   public ViewResult AutoProperty()
	        {
		//创建对象
	            Product myProduct = new Product();
		//设置属性值
	            myProduct.Name = "kayak";
		//读取属性值
	            string productName = myProduct.Name;
		//生成视图，使用View（stirng，object）方法的重载
	            return View("Result",(object)String.Format("Product name:{0}",productName));
	        }
	
	    }
	}
运行结果：运行程序后访问/Home/AutoProperty 会得到结果：
Product name ：kayak

# 使用扩展方法
扩展方法是指对那些不是我拥有的、不能直接修改的类添加方法的一种方式，如在Models文件夹中添加一个ShoppingCart.cs模型类，并修改为如下内容，它表示一个Products对象集合：

	using System;
	using System.Collections;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	namespace LanguageFeatures.Models
	{
	    public class ShoppingCart:IEnumerable<Product>
	    {
	        public List<Product> Products { get; set; }
	    }
	}
在Models文件夹中添加一个MyExtentionMethods.cs模型类文件并修改为如下内容：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace LanguageFeatures.Models
	{
	    public static class MyExtentionMethods
	    {
	        //对类ShoppingCart添加拓展方法
	       public static decimal TotalPrices(this ShoppingCart cardParam)
	        {
		 decimal total = 0;
		  foreach(Product prod in cardParam.Products)
		  {
		          total += prod.Price;
		   }
		    return total;
		}
	   }
	}
第一个this关键字把TotalPrices方法标记为ShoppingCart类的一个扩展方法，只要这个类在你的命名空间或using命名空间中即可。可以通过参数cardParam来引用ShoppingCart类的实例。这个扩展方法实现了把ShoppingCart类中Product集合的所有元素价格累计算出并返回。这样对类ShoppingCart就可以像引用它本身定义的方法一样来使用TotalPrices方法，比如在HomeController中添加如下方法：

	 public ViewResult UseExtension()
        {
            ShoppingCart cart = new ShoppingCart
            {
                Products = new List<Product>{
                    new Product{Name="kayak",Price=275M},
                    new Product{Name="LifeJack",Price=48.95M},
                    new Product{Name="Soccer",Price=19.58M},
                    new Product{Name="Corner",Price=34.95M}
                }
            };
	//如这行代码，TotalPrices方法像ShoppingCart类中定义的方法一样被调用
            decimal cartTotal = cart.TotalPrices();
            
            return View("Result", (object)String.Format("Total:{0:c}", cartTotal));
        }

运行程序并导航到/Home/UseExtension 会得到如下结果：
Total：￥378.48  或者Total：$378.48

自定义类或系统预定义的类都可以定义扩展方法，对接口同样可以运用扩展方法，即对实现的接口类像普通的类一样添加扩展方法即可。
创建过滤扩展方法。
对对象集合进行过滤的扩展方法，即定义一个扩展方法，在方法体中按照给定的参数和规则进行筛选，使用yield关键字返回符合筛选条件的对象集合，不符合条件的被丢弃。这是对IEnumerable<T>进行操作并返回IEnumerable<T>子集的扩展方法，添加之后任何实现该IEnumerable<T>接口的类都可以调用该扩展方法，如Array数组等。
如下，在MyExtentionMethods类中添加扩展方法：

	  public static IEnumerable<Product> FileterByCategory(this IEnumerable<Product> productEnum,string categoryParam)
	        {
	            foreach (Product prod in productEnum)
	                if (prod.Category == categoryParam)
	                {
	                    yield return prod;
	                }
	        }
	
在Home控制器中添加动作方法：

	  public ViewResult UseFilterExtensionMethod()
	        {
	            IEnumerable<Product> products = new ShoppingCart
	            {
	                Products = new List<Product>{
	                    new Product{Name="kayak",Price=275M},
	                    new Product{Name="LifeJack",Price=48.95M},
	                    new Product{Name="Soccer",Category="Soccer",Price=19.58M},
	                    new Product{Name="Corner",Category="Soccer",Price=34.95M}
	                }
	            };
	            Product[] productsArray ={
	                    new Product{Name="kayak",Price=275M},
	                    new Product{Name="LifeJack",Price=48.95M},
	                    new Product{Name="Soccer",Category="Soccer",Price=19.58M},
	                    new Product{Name="Corner",Category="Soccer",Price=34.95M}
	            };
	            decimal total = 0, total1 = 0;
		// IEnumerable<Product>对象调用IEnumerable<Product>接口的筛选扩展方法
	            foreach (Product prod in products.FileterByCategory("Soccer"))
	            {
	                total += prod.Price;
	            }
		//Array对象调用IEnumerable<Product>接口的筛选扩展方法
	            foreach (Product prod in productsArray.FileterByCategory("Soccer"))
	                total1 += prod.Price;
	            return View("Result", (object)String.Format("Total:{0},Array Total :{1}", total, total1));
	        }
	
运行程序并导航到/Home/UseFilterExtensionMethod 会得到结果：
Total:54.53,Array Total :54.53 
这说明对  IEnumerable<Product>或Array数组都可以调用筛选扩展方法FileterByCategory。

# C-Sharp的字段与属性和对象与集合的初始化器
在C# 的一个类中：

	public class Text
	{
		private string name；
		public string Name{
		                get{return name;}
		                set{name=value;}
		            }
	}
其get代码块（称为“getter（读取块）”）在读取该属性值时执行，set代码块（称为”setter（设置块）“）在对该属性值赋值时执行，特殊变量value表示要赋的值，属性是由其他类来使用的，比如向这个类传递一些值，就好像他是一个公有字段一样。但字段一般是私有的不可以被直接访问的。
属性名一般大写字母开头，而参数名和字段名消息字母开头。
属性是带有geeter块和setter块的成员。属性值的读取和设置就像对一个常规字段进行设置一样，使用属性要比字段更好，因为可以修改get和set语句而不需要修改依赖这个属性的类。
但当一个类有多个属性时，所有属性都要对字段协调访问，导致类文件冗长，此时可以使用自动实现的属性，可以采用字段支持模式属性而不需要定义这个字段或在getter和setter中指定代码，如下：

	public class Product
	    {
	        public string Name{get;set;}
	        public int ProductID{get;set;}
	        public string Description { get; set; }
	        public decimal Price { get; set; }
	        public string Category { get; set; }
	    }
当如果需要改变一个属性的实现方式，还可以返回属性规则的格式，但要保证同时实现getter和setter。
当需要使用一个类的实例时，除了要定义时更烦人的是需要多这个实例对象的各个属性进行赋值，如：

	Product myProduct = new Product();
	            myProduct.Name = "kayak";
	            myProduct.Category = "Soccer";
	            myProduct.Description = "description";
		…
幸运的是可以使用对象初始化器特性，在一步中创建并填写Product实例：

	Product myProduct = new Product{
	            myProduct.Name = "kayak",
	            myProduct.Category = "Soccer",
	            myProduct.Description = "description"};
调用的“{}”形成了初始化器，目的是为参数提供值，以此作为该对象的构造过程，同样可以对集合和数组采用初始化器进行内容的初始化：

	string [] arrays={"apple","orrange","plum"};
	List<int> intlist =new List<int>{10,20,30,40};
	Dictionary<string,int> myDict=new Dictionary<string,int>{
		{"apple",10},{"orange",20},{"plum",30}
	}

