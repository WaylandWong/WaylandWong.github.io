---
title: ASP.NET MVC 5 (七-2) Ninject进阶
date: 2018-10-16 15:31:57
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (七-2) Ninject进阶
---
上一篇说了Ninject的初步使用，这里在说一下稍微复杂的使用。

@[toc]
# 创建依赖项链
依赖项是什么呢？简单来说，上一篇当中Home控制器的构造器（构造函数）有IValueCalculator类型参数，那么IValueCalculator就是HomeConreoller的依赖项了。
当我们的依赖项又依赖于其他类型时，即依赖项又有依赖项时，这时Ninject又会继续解析并创建所需要的所有类的实例，一个又一个的依赖项的依赖性链接成一条依赖项链。比如，IValueCalculator的实现类LinqValueCalculator如果又一个依赖项IDiscountHelper，而IDiscountHelper的实现类是DefaultDiscountHelper类，这时就构成了一个依赖项。
为了演示这一过程，我们在Models文件夹内添加一个Discount.cs文件，修改其内容如下：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace EssentialTools.Models
	{
	    public interface IDiscountHelper
	    {
	        decimal ApplyDiscount(decimal totalParam);
	    }
	    public class DefaultDiscountHelper : IDiscountHelper
	    {
	        public decimal ApplyDiscount(decimal totalParam)
	        {
	            return (totalParam - (discountSize / 100m * totalParam));
	        }
	    }
	}
Discount文件内定义了IDiscountHelper接口，它的ApplyDiscount方法将一个十进制接口运用于一个十进制的数。DefaultDiscountHelper 类实现了该接口，并使用固定的10%的折扣。
然后我们为LinqValueCalculator类添加一个IDiscountHelper依赖项：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace EssensialTools.Models
	{
	    public class LinqValueCalculator:IValueCalculator
	    {
	        private IDiscountHelper discount;
	        public LinqValueCalculator(IDiscountHelper discountParam)
	        {
	            discount = discountParam;
	        }
	        public decimal ValueProducts(IEnumerable<Product> products)
	        {
	            return discount.ApplyDiscount(products.Sum(p=>p.Price));
	        }
	    }
	}
此时向之前做的那样，将DefaultDiscountHelper 和IDiscountHelper的绑定关系放到之前的依赖项解析器的AddBindings方法当中就大功告成了。

	 private void AddBindings()
	        {
	            kernel.Bind<IValueCalculator>().To<LinqValueCalculator>();
	            kernel.Bind<IDiscountHelper>().To<DefaultDiscountHelper>;
	        }
这时我们将这个折扣便应用到了计算总价当中了，这构成了一个依赖项链。
# 指定属性和构造器参数
在将接口和实现进行绑定时，可以为实现类的属性提供一些值。比如，在Discount.cs文件对DefaultDiscountHelper 类添加一个属性：

	public class DefaultDiscountHelper : IDiscountHelper
	    {
	        public decimal DiscountSize { get; set; }
	        public decimal ApplyDiscount(decimal totalParam)
	        {
	            return (totalParam - (DiscountSize / 100m * totalParam));
	        }
	    }
discountSize属性提供不同的折扣值，在对DefaultDiscountHelper和IDiscountHelper接口进行绑定的时候可以给这个属性赋值，修改AddBindings方法如下：

	private void AddBindings()
	{
		kernel.Bind<IValueCalculator>().To<LinqValueCalculator>();
		kernel.Bind<IDiscountHelper>().To<DefaultDiscountHelper>().WithPropertyValue("DiscountSize",50M);
	}
这样就可以在计算总价时使用50%的折扣了。
当有多个属性时可以链接带哦用WithConstructorArgument方法涵盖所有属性。
当然也可以使用构造器参数来实现参数传递，对DefaultDiscountHelper 类做如下修改：
	
	 public class DefaultDiscountHelper : IDiscountHelper
	    {
	        public decimal discountSize;
	        public decimal ApplyDiscount(decimal totalParam)
	        {
		        return (totalParam - (discountSize / 100m * totalParam));
	        }
	        public DefaultDiscountHelper(decimal discountParma)
	        {
	            discountSize = discountParma;
	        }
	    }
这样可以在依赖项解析器当中使用WithConstructorArgument方法为构造器参数赋值

	 private void AddBindings()
	 {
	    kernel.Bind<IValueCalculator>().To<LinqValueCalculator>();
		kernel.Bind<IDiscountHelper>().To<DefaultDiscountHelper>().WithConstructorArgument("discountParam", 50M);
	  }
# 使用条件绑定
在Models文件夹下添加一个FiexibleDiscountHelper类，它实现IDiscount接口，并按照总价给出不同的折扣值。内容如下:

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace EssensialTools.Models
	{
	    public class FlexibleDiscountHelper : IDiscountHelper
	    {
	        public decimal ApplyDiscount(decimal totalParam)
	        {
	            decimal discount = totalParam > 100 ? 70 : 50;
	            return (totalParam - (discount / 100m * totalParam));
	        }
	    }
	}
此时IDiscount接口有两个实现方法，在依赖项解析器中我们可以按照不同条件绑定不同的实现类，在AddBindings方法中作出如下修改：

	 private void AddBindings()
	 {
	   kernel.Bind<IValueCalculator>().To<LinqValueCalculator>();
	   kernel.Bind<IDiscountHelper>().To<DefaultDiscountHelper>().WithConstructorArgument("discountParam", 50M);
	   kernel.Bind<IDiscountHelper>().To<FlexibleDiscountHelper>().WhenInjectedInto<LinqValueCalculator>();
	 }
这样可使当要创建一个LinqValueCalculator对象时使用FlexibleDiscountHelper类作为IDiscountHelper接口的实现类，而不是默认的DefaultDiscountHelper实现类。
Ninject有一些绑定条件的方法：

方法|效果
-|-
When(谓词)|当‘谓词’（一个lambda表达式）的结果为true时，实施绑定
WhenClassHas<T>()|当被注入的类以T类型注解属性进行注释时，实施绑定
WhenInjectInto<T>()|当要被注入的类时烈性T时，实施绑定

# 设置对象作用域
首先在NinjectDependencyResolver类中引入`using Ninject.Web.Common;`命名空间。
此时对AddBindings方法做如下修改：

	private void AddBindings()
	{
	    kernel.Bind<IValueCalculator>().To<LinqValueCalculator>().InRequestScope();
	    ...//其他方法未给出
	 }

绑定的最后使用`InRequestScope`方法指定LinqValueCalculator对象的作用域为一个请求，即一个http请求只会创建一个对象而不管有几个依赖项对象。
比如修改Home控制器的初始化器：

	public HomeController(IValueCalculator calcParam, IValueCalculator calc2)
	{
	            calc = calcParam;
	}
它虽然请求两个IValueCalculator 对象，但MVC只会创建**一个**IValueCalculator 的实现类LinqValueCalculator的对象。

以下是指定其他作用域的方法：
-摘自《精通ASP.NET MVC5》(Adam FreeMan)
名称|效果
-|-
InTransientScope()|与未指定作用域相同，为每一个被解析的依赖项创建一个新的对象（每依赖项一实例）
InSingeltonScope()<br/>ToConstant(object)|创建一个单一实例，使其应用于整个应用程序（每应用一实例）
InThreadScope()|创建一个单一实例，将其用于解析一个线程中各个对象的实例（每线程一实例）
InRequestScope()|创建一个单一实例，用于解析一个HTTP请求的各个对象的依赖项（每请求以实例）
