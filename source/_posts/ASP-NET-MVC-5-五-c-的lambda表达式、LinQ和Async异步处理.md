---
title: 'ASP.NET MVC 5 (五)c# 的lambda表达式、LinQ和Async异步处理'
date: 2018-10-16 15:31:14
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (五)c# 的lambda表达式、LinQ和Async异步处理
---
简单介绍Lambda表达式和LINQ表达式，详细的C# 语言语法查阅一下两个链接，详细实验和解释另外再开一个语法系列。
Lambda 表达式（C#  编程指南）
来自 <https://msdn.microsoft.com/zh-cn/library/bb397687.aspx> 

LINQ 查询表达式（C#  编程指南）
来自 <https://msdn.microsoft.com/zh-cn/library/bb397676.aspx>
 
@[toc]
# Lambda表达式基本语法样式
	参数列表=>表达式或语句块
符号"=>"读作“goes to”，它和赋值运算“=”有相同优先级,符号"=>"右侧的lambda表达式成为“表达式lambda”。
他有一下几种形式：

		• (int x,int y)=>x==y    //括号中多个参数之间用逗号隔开
		• (x,y)=>x==y              //可以省略参数类型
		• (int x)=>x>12           //一个参数可以省略括号
		• x=>{string n=x+" "+"World";Console.WriteLine(n);}//语句块
		• ()=>SomeMethod()     //无参表达式
表达式 Lambda 的主体可以包含一个方法调用。

当自己想调用自己的lambda表达式时可以使用委托调用，或者使用Func<>。
	
	//定义委托
	 delegate int mydelegate(int x, int y);
	public static void Main(string []args)
	{
		 mydelegate del = (x, y) =>  x * y ;
		int total=del(12, 12);     //total==12*12==144
	}
	//Func<>参数
	  public static IEnumerable<Product> Filter(
	            this IEnumerable<Product> productEnum, Func<Product, bool> selectorParam)
	        {
	            foreach (Product prod in productEnum)
	            {
	                if(selectorParam (prod))
	                    yield return prod;
	            }
	        }
	 public void UseFilterExtensionMethod()
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
	            Func<Product, bool> categoryFilter = prod => prod.Category == "Soccer"|| prod.Price>20;
		//检索满足条件：Category == "Soccer|| prod.Price>20“的元素组成集合
	            decimal total = 0, total1 = 0;
	            foreach (Product prod in products.Filter(categoryFilter))
	            {
	                total += prod.Price;
	            }
		//求满足条件的单价之和
	        }

# 自动接口类型
使用var关键字定义局部变量而不必指定该变量类型，称为 类型推断 或 隐式类型。

	var myVariable = new Product { Name="Kayak",Price=50};//合法
	int count = myVariable.Count;    //不合法，Product类没有该Count属性

# 匿名类型
可以不指定类型或结构，直接定义：

	var myAnonType=new{Name="MVC",Category="Pattern"};
myAnonType是一个匿名类型对象，但它不是动态的，依旧是强类型，即只可以对初始化器"{}"中已经定义的属性进行赋值或取值，而不允许修改其定义。编译器进行编译时会对这种匿名类型生成一个类，具有相同属性名的两个匿名类型对象会被分配给同样的类。

# 执行语言集成查询（LINQ）
LINQ类似于SQL语句，用于在对象集合上进行查询数据。比如：
在上一章的LanguageFeatures项目的HomeController中添加如下代码：

	 public ViewResult FindeProducts()
	        {
	            Product[] products = {
	                    new Product{Name="kayak",Category="Watesports",Price=275M},
	                    new Product{Name="LifeJack",Category="Watesports",Price=48.95M},
	                    new Product{Name="Soccer",Category="Soccer",Price=19.58M},
	                    new Product{Name="Corner",Category="Soccer",Price=34.95M}
	                };
            //LINQ语句，从products查询对象并按Prcie降序排列并只获取前三个，查询结果对象的Name属性和Price属性重新组成一个新的类型对象
            var foundProducts = from match in products
                                orderby match.Price descending
                                select new { match.Name, match.Price };
                    string result = "";
            int i = 0;
           foreach (var prod in foundProducts)
           {
               i++;
               if (i >3)
                   break;
               result += prod.Name + ":" + prod.Price + " ";
           }
            return View("Result",(object)result);
        }

运行并导航到/home/FindeProducts会得到结果：
kayak:275 LifeJack:48.95 Corner:34.95 


上述代码类似于SQL语句，这种风格称为 查询语法，它用起来舒服简单，但它为每一个源数据组中的每个Product返回一个匿名对象，需要另外进行处理。
LINQ还有另外的一种风格，叫做 点符号语法，它是基于扩展方法的。比如将上述代码替换为如下代码可以实现同样的代码：

	 public ViewResult FindeProducts()
	        {
	            Product[] products = {
	                    new Product{Name="kayak",Category="Watesports",Price=275M},
	                    new Product{Name="LifeJack",Category="Watesports",Price=48.95M},
	                    new Product{Name="Soccer",Category="Soccer",Price=19.58M},
	                    new Product{Name="Corner",Category="Soccer",Price=34.95M}
	                };
	            /*LINQ语句，从products查询对象并按Prcie降序排列，查询结果对象的Name属性和Price属性重新组成一个新的类型对象*/
	            var foundProducts = products.OrderByDescending(e => e.Price)
	                .Take(3).Select(e => new { e.Name, e.Price });
	            string result = "";
	           foreach (var prod in foundProducts)
	           {
	               result += prod.Name + ":" + prod.Price + " ";
	           }
	            return View("Result",(object)result);
	        }

# 一些LINQ扩展方法列表
摘自《精通ASP.NE MVC5》（[美]Adam Freeman 著，张成彬等译）

扩展方法|描述|延迟
-|-|-
All | 如果源数据中所有的条目都与谓词匹配，则返回true | 否
Any | 	如果源数据中至少有一个条目与谓词匹配，则返回true	 | 否
Contains	 | 如果数据源含有指定的条目或值，则返回true	 | 否
Count | 返回数据源的条目数	 | 否
First	 | 返回数据源的第一个条目 | 	否
FirstOrDefault	 | 返回数据源的第一个条目，或无条目时返回默认值| 否
Last	 | 返回数据源的最后一个条目 | 	否
LastOrDefault | 返回数据源的最后条目，或无条目时返回默认值 | 否
Max	<br>Min  | 返回lambda表达式表达的最大值或最小值 | 否
OrderBy<br>OrderByDescending  | 	基于lambda表达式返回的值对源数据进行排序 |是 
Reverse | 	反转数据源中的数据项的顺序 | 	是
Select	 | 设计一个查询结果	 | 是
SelectMany | 	把每一个数据项投射到一个条目序列之中，然后把所有这些结果序列连接成一个序列 | 	是
Single	|返回数据源的第一个条目，或者有多个匹配时抛出异常 | 	否
SingleOrDefault | 	返回数据源的第一个条目，或者无条目时，返回默认值；有多个匹配条目时，抛出一个异常	 | 否
Skip<br>SkipWhile	 | 跳过指定书目的元素，或者当谓词匹配时跳过	 | 是
Sum	 | 对谓词选定的值求和 | 	否
Take<br>TakeWhile	 | 从数据源的开始出选择指定条目的元素，或当谓词匹配是选择条目	 | 是 
ToArray<br>ToDictionary<br>ToList  | 	把数据源转换成数组或其他集合类型 | 	否
Where	 | 过滤掉数据源数据中与谓词不匹配的条目	 | 是

# 延迟的LINQ查询
延迟的意思是，直到对结果进行枚举后，才会执行该查询，每次对记过进行枚举时都会从头求取查询结果。如：

	Product[] products = {
			                    new Product{Name="kayak",Category="Watesports",Price=275M},
			                    new Product{Name="LifeJack",Category="Watesports",Price=48.95M},
			                    new Product{Name="Soccer",Category="Soccer",Price=19.58M},
			                    new Product{Name="Corner",Category="Soccer",Price=34.95M}
			                };
			 var foundProducts = products.OrderByDescending(e => e.Price)
			                .Take(3).Select(e => new { e.Name, e.Price });
			            string result = "";//延迟的LINQ查询定义的位置
			
	products[2]=new Product{Name="kayak1",Category="Watesports",Price=276M};
					//修改数据源
	           foreach (var prod in foundProducts)
	           {
	               result += prod.Name + ":" + prod.Price + " ";
	           }//枚举查询结果，它会显示修改数据源之后的结果

在定义LINQ查询得位置和修改数据源的位置分别插入断点，两个断点出查看foundProducts会发现发生了改变，尽管初始化foundProducts之后并没有对foundProducts进行更改，只是更改了源数据而已，此时延迟的LINQ这一特性便展现出来了。

使用async和await关键字是一种简单的异步处理方法。
如：

	using System;
	using System.Net.Http;
	using System.Threading.Tasks;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace LanguageFeatures.Models
	{
	    public class MyAsyncMethods
	    {
	       public async static Task<long?> GetPageLength()
	        {
	            HttpClient client = new HttpClient();
	            var httpMessage = await client.GetAsync("http://apress.com");
	
	            return httpMessage.Content.Headers.ContentLength;
	        }
	    }
	}

代码中使用了await关键字，它告诉编译器希望程序等待GetAsync方法返回的Task结果并继续执行同一方法中的其他语句。当使用await关键字时必须对该方法的的签名添加async关键字。但若想使用MVC控制器使用异步方法需要一项特殊技术，因此这里没有给出。这里只是想简单介绍一下async和await关键字的作用