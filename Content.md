一、介绍
====
1.概述
----
JEP是一个用于解析和计算数学表达式的Java类库。通过使用这个包你可以把公式看作字符串并快速计算它们。其中内置了大量公共的数学函数和常量供用户使用。另外，你也可以通过自定义变量、常量、函数等方式扩展JEP。
###  1.1特性
- 占用空间小（281kb类库）
- 快速计算
- 支持任意精度的算术
- 支持字符串、向量和复数
- 包括常用数学函数和预定义常量
- 支持布尔表达式(!, &&, ||, <, >, !=, ==, >=, and <=)
- 支持隐式乘法（允许使用“3x”代替“3*x”这种表达式）
- 支持赋值表达式（例如x=5）
- 允许选择声明变量或未声明变量
- 可扩展的自定义函数
- 可定制的语法表达式
- 兼容J2SE 5
- 支持字符编码（包括希腊符号）
- 丰富的文档
- 包括JavaCC语法

2.解析和计算
----
使用JEP计算一个字符串表达式包括两个步骤，如下图所示。首先是解析表达式，从字符串结构解析为树形结构。表达式的树形结构表示允许接下来的简单、快速的表达式计算。接下来将更详细地讨论这两个步骤。  
![image](https://github.com/time-out01/JEP-DOC-CN/blob/master/image/img01.png)
###2.1解析
一般来说，可以描述解析作为一系列字符作为输入并产生一组包含字符集合信息的结构化表示。解析虽然用于其他领域例如自然语言解析等，但是这里的重点是数学表达式。<br>
数学表达式可以通过树形结构表示，其中数字、变量、操作符和函数全部通过节点表示。数字和字符称为“子节点”，操作符和函数称为“父节点”。父节点操作它的子节点。例如，表达式“1+2*3”可以表示为一棵包含五个节点的树，如上图所示。<br>
你也许会问为什么“乘”节点是“加”节点的一个子节点。这是运算优先级的原因。当解析表达式时，需要遵守优先级的顺序。乘法的优先级高于加法的优先级。所以，如上例所示，它需要在加1之前计算2乘以3。在树中，这是由乘法操作符的子节点数字2和3组成的。最后，乘法节点自己作为加法节点的一个子节点，计算乘法的结果用于计算加法操作。解析器按照优先级顺序负责确定语法。<br>
解析器通常使用解析器生成器生成，例如JavaCC、ANTLR、SableCC、Yacc或者Bison。在众多解析器生成器中，JEP使用JavaCC。使用JccParser中定义的语法。JavaCC创建一组类来执行解析过程的核心任务。一个可配置的Java解析器也包括其中。<br>
###2.2计算
计算把表达式从一个树形表示转变为一个表达式的值。从树通过操作符优先级建立，计算方法不需要知道操作符优先级。树可以使用一个简单的方法确定表达式的值来遍历。
一个简单的方法是从树的根节点（最顶层的节点）开始，运用递归方法。这个可以使用访问者模式来实现。不同的方法可以为每个节点类型创建。计算一个操作或者一个函数节点，首先子节点要被计算。然后，用子节点的值，操作符或者函数用于确定节点本身的值。常量（数字）或者变量节点，节点计算常量或者变量的值。使用这样的递归方法迅速地通过深度优先算法遍历树。<br>

二、基本用法
====
1.入门指南
----
在你的项目中使用JEP非常简单。下面的步骤将帮助你快速开始。<br>  

1.  下载JEP包 
2.  解压包
3.  移动jep-java-x.x.x.jar文件(位于主目录)到所选择的目录中
4.  重要：当编译程序时，确保Java编译器能够找到JEP 类，它需要知道它们的位置。你需要设置IDE中jar的位置来让编译器发现类库（如何去做请查看你的IDE帮助手册），如果你不是用IDE，你需要把jar加入到你的CLASSPATH环境变量中，例如 C:\java\packages\jep-java=x.x.x.jar，这取决于你把jar放到哪一路径。
5.  将以下代码加入到你的程序中。<br>
			
			import com.singularsys.jep.Jep;
			import com.singularsys.jep.JepException;
			public class Demo {
				public static void main(String[] args) {
					Jep jep = new Jep();
					try {
			jep.addVariable("x", 10);
			jep.parse("x+1");
			Object result = jep.evaluate();
			
			System.out.println("x + 1 = " + result);
		} catch (JepException e) {
			System.out.println("An error occurred: " + e.getMessage());
		}
				}
			    }
第五行创建了一个Jep解析器对象，第七行添加一个变量“x”，并为其赋初始值“10”，第八行和第九行，解析表达式“x+1”，并计算；计算结果保存在result中；最后，打印结果。
6.  你应该能够编译运行你的应用，并看到打印值。
接下来是什么？通过浏览文档明白Jep都能做什么。也可以看下简单的示例applets。<br>

2.错误处理
----
当错误发生，异常将被解析和计算方法抛出。解析方法抛出ParseException实例，计算方法抛出EvaluationException实例。两个异常类都继承自JepException类，所以如果你不需要区分解析错误和计算错误，你可以简单地抛出JepException实例。<br>
异常类通过getMessage()方法提供关于错误发生的信息。<br>

3.默认设置
----
 如果你通过使用默认设置创建一个Jep实例，你可以解析带有以下函数列表中的函数的所以功能，你也可以使用以下常量：
 <table>
   <tbody>
    <tr>
     <td>pi</td><td>3.1415…&nbsp;圆的周长比圆的直径（Math.PI）</td>
    </tr>
    <tr>
     <td>e</td><td>2.718…&nbsp;双精度欧拉常数（Math.E）</td>
    </tr>
    <tr>
     <td>true</td><td>布尔常量（Boolean.TRUE）</td>
    </tr>
    <tr>
     <td>false</td><td>布尔常量（Boolean.FALSE）</td>
    </tr>
    <tr>
     <td>i</td><td>虚数单位</td>
    </tr>
  </tbody>
 </table>
4.计算方法
----
三个用于计算表达式的方法：
- Object evaluate()：计算最后解析的表达式并作为对象返回结果。
- Object evaluate(Node root)：通过根节点和计算解析表达式并作为对象返回结果。
- double evaluateD()：计算最后解析的表达式并作为double类型返回结果，如果结果不能转换为double类型，将抛出EvaluationException异常。

你可能总是不知道表达式结果类型。例如，它依赖表达式解析可能是Double，Vector，Boolean或者String。你可以用instanceof操作符去确定结果类型，然后把结果转换为相应的类。

5.快速重复计算
----
当在没有重复解析表达式而变量的值改变的时候执行重复计算表达式是可能的。当然，这比重复解析每次需要的计算要快的多。<br>
下面的代码展示addVariable(String,Object)和evaluate()方法如何重复调用去改变变量的值以及重复计算表达式。<br>
		
		// add variable to allow parsing
		jep.addVariable("x", 0);
		jep.parse("2*x-3");
		// evaluate expression for x = 0 to x = 99
		for (int i = 0; i < 100; i++) {
			// update the value of x
			jep.addVariable("x", i);
			// print the result
			System.out.println("Value at x = " + i + ": " + jep.evaluate());
		}
		

 你可以在任何计算器包括RealEvaluate()中使用快速重复计算。<br>

6.Decimal运算
----
 在Jep 3中，使用Decimal运算计算表达式已经成为可能。这种数据类型在BigDecimal而不是double模式中表示数字。<br>
  简单地使用下面的代码构建一个新的Jep实例。<br>
		
		 	jep = new Jep(new BigDecComponents());
		 	

这种不同的好处通过例子展示。当执行两个double类型相乘的操作，10\*0.09计算结果为0.8999999999999999。但是当使用decimal运算（BigDecimal）执行同样的计算时，10\*0.09的结果为0.9。<br>
如果你对高精度算法感兴趣，参考文档的BigDecimal部分。<br>

7.隐式乘法
----
你可以设置setImplicMul(true)使用隐式乘法操作。默认设置为true（允许隐式乘法）。<br>
隐式乘法允许表达式“2 x”代替“2\*x”。注意，两个变量之间的空格被看作是乘。这个方法适用于一个变量后跟着一个数字。例如“y 3”解释为“y\*3”，但是“y3”解释为一个单独的变量“y3”。<br>
如果一个变量在数字之前，它们之间不需要空格为隐式乘法。<br>

8.处理多个表达式
----
Jep可以同时处理多个表达式。简单的方法是在多个字符串中多次调用Jep.parse()方法。方法返回一个Node类型的对象表示表达式树。这些节点可以被存储用来以后使用或者使用Jep.evaluate(Node n)方法计算。<br>
		
			Jep jep = new Jep();
		try {
			Node n1 = jep.parse("y=x^2");
			Node n2 = jep.parse("z=x+y");
			for (double x = 0.0; x <= 1.0; x += 0.1) {
				jep.addVariable("x", x);
				Object value1 = jep.evaluate(n1);
				Object value2 = jep.evaluate(n2);
			}
		} catch (JepException e) {
		}
		

另一种方法允许所有的表达式包含在一个单独的字符串中。一系列等式可以使用Jep.initMultiParse(String s)方法从一个单独的字符串或者Reader之中，Jep.initMultiParse(Reader r)和continueParsing()允许所有的表达式包含在一个字符串之中。等式用分号隔开，解析器通过initMultiParse初始化，表达式通过使用continueParsing()轮流读。例如：
		
		Jep jep = new Jep();
		String s = "x=1; y=2; x+y";
		jep.initMultiParse(s);
		try {
			Node n;
			while ((n = jep.continueParsing()) != null) {
				Object res = jep.evaluate(n);
			}
		} catch (JepException e) {
		}

9.使用RealEvaluator快速计算
----
从Jep 3.2到Jep 3.3，通过 FastEvaluator代替默认的StandardEvaluator组件使得计算速度有了重大提升。然而，如果你的表达式操作仅仅是单精度浮点数，你可以实现一个额外的RealEvaluator可以被加载通过Jep构造器Jep jep = new Jep(new RealEvaluator())；或者通过在构造器之后设置计算器jep.setComponent(new RealEvaluator())。<br>
从Jep 3.2到Jep 3.3，通过 FastEvaluator代替默认的StandardEvaluator组件使得计算速度有了重大提升。然而，如果你的表达式操作仅仅是单精度浮点数，你可以实现一个额外的RealEvaluator可以被加载通过Jep构造器Jep jep = new Jep(new RealEvaluator())；或者通过在构造器之后设置计算器jep.setComponent(new RealEvaluator())。<br>

三、变量
====
1.基础
----
变量被Variable类表示并储存在VariableTable中。变量的值通过Jep.addVariable(String name,Object val)方法设置，通过Object Jep.getVariableValue(String name)方法检索。addVariable()方法也可以被用于改变已经存在的变量的值。对于double值，用 Jep.addVariable(String, double, double)方法。进一步的方法，getVariable(String)，返回Variable对象。这个类允许变量被观察，并提供了一个略微快速的方法getting和setting变量值。

2.未声明和未定义的变量
----
在解析期间，解析器查找是否存在变量。Jep.setAllowUndeclared(boolean flag)为未声明变量设置行为：
 1. **允许未声明变量 (默认)：**当解析表达式时，未声明变量将被加入到Variable
 2. **不允许未声明变量：**这种情况下，必须在在解析之前使用addVariable()方法加入变量。如果遇到未声明变量，将抛出ParseException 异常。如果你希望限制一个变量出现在表达式中，这个选项是很有用的。
