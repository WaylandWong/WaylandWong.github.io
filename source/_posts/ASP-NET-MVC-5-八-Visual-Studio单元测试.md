---
title: ASP.NET MVC 5 (八) Visual Studio单元测试
date: 2018-10-16 15:32:05
categories: ASP.NET
tags:
- ASP
- .NET
- MVC
description:
- ASP.NET MVC 5 (八) Visual Studio单元测试
---
这里记录如何使用Visual Studio的内置单元测试

@[toc]
# 准备示例
这里继续使用上一篇使用的EssensialTools项目做演示。不过需要对IDiscountHelper接口添加一个新的实现，在Models文件夹内添加一个MinimumDiscountHelper的类文件，修改其内容如下：

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	
	namespace EssensialTools.Models
	{
	    public class MinimumDiscountHelper : IDiscountHelper
	    {
	        public decimal ApplyDiscount(decimal totalParam)
	        {
		       throw new ArgumentOutOfRangeException();
	          
	        }
	    }
	}
它对不同价格总额采取不同的折扣：

 - 总额大于￥100的，折扣为10%
 - 总额介于￥100到￥10（包括100和10）的，折扣为￥5
 - 总额小于￥10的，无折扣
 - 总额为负值时，抛出一个ArgumentOutOfRangeException异常
 但暂时没有实现任何行为。Adam Freeman遵循测试驱动开发（TDD）方法编写单元测试并随之编写实现代码。
## 创建单元测试项目
为EssensialTools项目添加单元测试项目的步骤是，在解决方案资源管理器的顶级条目（"EssensialTools"的解决方案）右键选择添加新项目（New Project），也可以在创建MVC项目的同时添加单元测试----在MVC项目选择的对话框中，有一个“Add Unit Tests”(添加单元测试)选项。
在打开的对话框，在左侧面板的“Visual C# ”中选择“Test”，并在中间面板中选择“Unit Test Project(单元测试项目)”，将项目名称设置为EssensialTools.Tests然后点击确定。
## 添加项目引用
添加之后会在解决方案资源管理器中看到添加的单元测试，但此时还需要添加对一个应用程序项目的引用，使单元测试能够调用这个项目中的类进行测试。方法是，在EssensialTools.Tests项目的References上右键，选择添加引用（Add references），在打开的面板左侧选择Solution，选中右侧的EssensilTools项目，然后点击确定即可。
# 添加单元测试
## 第一个单元测试
然后我们可以在EssensialTools.Tests的UnitTest.cs中添加单元测试了，将其内容修改为如下内容：

	using System;
	using Microsoft.VisualStudio.TestTools.UnitTesting;
	using EssensialTools.Models;
	
	namespace EssensialTools.Tests
	{
	    [TestClass]
	    public class UnitTest1
	    {
	
	        private IDiscountHelper getTestObject()
	        {
	            return new MinimumDiscountHelper();
	        }
	
	        [TestMethod]
	        public void Discount_Above_100()
	        {
	            //准备 arrange
	            IDiscountHelper target = getTestObject();
	            decimal total = 200;
	
	            //动作 act
	            var discountedTotal = target.ApplyDiscount(total);
	
	            //断言
	            Assert.AreEqual(total * 0.9M, discountedTotal);
	        }
	    }
	}

我们只添加了一个单元测试，含有测试类的类使用`TestClass`注解属性进行注释的，其中的各个测试都是用`TestMethod`注解属性进行注释的方法。当然并不是所有的方法都是单元测试，比如`getTestObject`方法。要注意的时要添加`using EssensialTools.Models;`代码引入命名空间。
考察上面的单元测试方法，它是采用“准备(Arrange)”-“动作(Act)”-“断言(Assert)”三部分组成。

 - 准备，调用`getTestObject`方法返回一个`IDiscountHelper` 接口的实现类`MinimumDiscountHelper`的实例，定义表示总价的变量`total`并给定初值200。
 - 动作，使用接口（或实现类）的ApplyDiscount方法计算折扣后的总价并赋值给`discountedTotal`
 - 断言，通过 `Assert.AreEqual`方法判断通过接口实现的方法计算之后的值和它应当得出的值是否相等，如果不相等则抛出异常。

`Assert`类还有诸多在单元测试中使用的静态方法，从MSDN中https://msdn.microsoft.com/zh-cn/library/microsoft.visualstudio.testtools.unittesting.assert.aspx 可以查看。

## 添加其他单元测试
上述内容中给出测试超过￥100的折扣测试方法，现在在UnitTest1类中添加其他条件下的测试方法：
	
	using System;
	using Microsoft.VisualStudio.TestTools.UnitTesting;
	using EssentialTools.Models;
	
	namespace EssiontialTools
	{
	    [TestClass]
	    public class UnitTest1
	    {
	        
	        private IDiscountHelper getTestObject()
	        {
	            return new MinimumDiscountHelper();
	        }
	
	        [TestMethod]
	        public void Discount_Above_100()
	        {
	            //准备 arrange
	            IDiscountHelper target = getTestObject();
	            decimal total = 200;
	
	            //动作 act
	            var discountedTotal = target.ApplyDiscount(total);
	
	            //断言
	            Assert.AreEqual(total * 0.9M, discountedTotal);
	        }
	
	        [TestMethod]
	        public void Discount_Between_10_And_100()
	        {
	            //准备 arrange
	            IDiscountHelper target = getTestObject();
	
	            //动作 act
	            decimal TenDollarDiscount = target.ApplyDiscount(10);
	            decimal HundredDollarDiscount = target.ApplyDiscount(100);
	            decimal FiftyDollarDiscount = target.ApplyDiscount(50);
	
	            //断言
	            Assert.AreEqual(5,TenDollarDiscount,"$10 discount is wrong");
	            Assert.AreEqual(95, HundredDollarDiscount, "$100 discount is wrong");
	            Assert.AreEqual(45, FiftyDollarDiscount, "$50 discount is wrong");
	
	        }
	
	        [TestMethod]
	        public void Discount_Less_Than_10()
	        {
	            //准备 arrange
	            IDiscountHelper target = getTestObject();
	
	            //动作 act
	            decimal discount5 = target.ApplyDiscount(5);
	            decimal discount0 = target.ApplyDiscount(0);
	
	            //断言
	            Assert.AreEqual(5, discount5);
	            Assert.AreEqual(0, discount0);
	        }
	
	        [TestMethod]
	        [ExpectedException(typeof(ArgumentOutOfRangeException))]
	        public void Discount_Negative_Total()
	        {
	            //准备 arrange
	            IDiscountHelper target = getTestObject();
	
	            //动作 act
	            target.ApplyDiscount(-1);
	          
	
	        }
	    }
	}
# 运行单元测试并发现错误
Visual Studio提供了"Test Explorer(测试资源管理器)"，用于管理和运行单元测试，从“Test（测试）”菜单中选择“Windows（窗口）”->“Test Explorer(测试资源管理器)”便可以显示出来。然后点击该窗口左上角的"Run All（全部运行）"可以看到测试结果如图：
![第一此运行单元测试并发现错误](/uploads/8-1.jpeg)
# 实现特性
可以看到结果显示，有三个测试结果出现错误，超过100、100~10之间和小于10的折扣出现错误，这是因为我们的实现类MinimumDiscountHelper并没有实现当初预定的算法特性。
然后我们根据要求修改MinimumDiscountHelper的ApplyDiscount方法如下：

	 public decimal ApplyDiscount(decimal totalParam)
	        {
	            if (totalParam < 0)
	            {
	                throw new ArgumentOutOfRangeException();
	            }
	            else if (totalParam > 100)
	            {
	                return totalParam * 0.9M;
	            }
	            else if (totalParam > 10 && totalParam <= 100)
	            {
	                return totalParam - 5;
	            }
	            else
	            {
	                return totalParam;
	            }
	        }
# 测试并修正代码
重新运行全部测试，这时发现有一个测试并没有通过，可以看到错误提示为 ：

	Result Message:	Assert.AreEqual failed. Expected:<5>. Actual:<10>. $10 discount is wrong
说明希望得到的测试值是5，但实际得到的值是10。此时则需要重审代码，会发现ApplyDiscount设置的10~100之间的算法并没有包含10这一数值。修正原有代码` else if (totalParam > 10 && totalParam <= 100)`为`else if (totalParam >= 10 && totalParam <= 100)`，然后重新运行测试会发现测试成功。
在MSDN上https://msdn.microsoft.com/zh-cn/library/dd264975.aspx 有VS的单元测试的详细介绍。