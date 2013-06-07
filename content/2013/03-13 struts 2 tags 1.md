# 复习笔记——Struts 2 标签（一）

- date: 213-03-13
- category: Work
- tags: struts2, tag

----

在讲解Struts2标签库之前，本节将首先关注数据通过Struts2标签API离开框架的环境下的OGNL表达式语言。我们会浏览OGNL表达式的语法，并研究它可以取出数据的位置，尤其要深入研究ValueStack和ActionContext。

### 1.入门

当一个请求到达框架时，Struts2首先要做的事情是创建存储请求的所有重要数据的对象。应用程序的特定领域数据(使用标签最常访问的数据)会存储在ValueStack中，而另一些更基础的数据也必须存储起来，所有的这些数据连通ValueStack都存储在一个叫做ActionContext的对象中。

#### 1.1 ActionContext和OGNL

实际上，OGNL表达式可以根据任何一系列对象求值，而ValueStack只是这些对象中的一个，默认的那个。OGNL可用来求值的更宽泛的一系列对象叫做ActionContext。ActionContex包含了框架的请求处理过程可以访问的所有数据，包含内容从应用程序数据到会话作用域或者应用程序作用域的映射。默认情况下OGNL解析会选择ValueStack，但你可以指定其他的对象的名字，例如会话作用域。

下图展示了ActionContext和它包含的对象，可以把OGNL解析指向其中的任何对象：

![ActionContex包含的对象.png][1]

*   parameters:当前请求中请求参数的映射
*   request:请求作用域的属性的映射
*   session:会话作用域的属性的映射
*   application:应用程序作用域的属性的映射
*   attr:按照page、request、session、application作用域的顺序，返回第一个出现的属性
*   ValueStack:包含当前请求的应用程序特定领域的所有数据

每一个OGNL表达式的解析都需要一个根对象，有了它才能开始引用的解析。现在让我们看看如何选择ActionContext中的对象让OGNL基于此对象解析。

**为OGNL选择根对象**

OGNL表达式可以以一个特殊的语法开始，从上下中命名一个解析表达式所根据的对象：

    #session['user']
    

OGNL表达式通过表达式语言的#操作符主动命名ActionContext中的会话映射。#操作符告诉OGNL表达式使用位于上下文中的命名对象作为初始对象解析表达式的剩余部分。

#### 1.2 虚拟对象ValueStack

当Struts2接收到一个请求时，它立即创佳一个ActionContext、一个ValueStack和一个动作对象。作为应用程序数据的承载者，动作被马上放在ValueStack中，以便框架可以通过OGNL访问它的属性。

ValueStack只有一个微妙的地方，当OGNL表达式根据ValueStack解析时，它装作一个对象。这个虚拟对象包含放在上面的所有对象的所有属性。如果相同的属性出现多次，栈下面对象的属性会被栈上层的同名属性覆盖。

![ValueStack是OGNL表达式默认解析对象.png][2]

如上图所示，一个给定属性的引用的解析指向栈中最高位置出现的属性。以name为例，ValueStack总会返回对象栈中出现在最顶层的name属性。在上述的情况下，动作对象有一个name属性，但由于上层模型对象也有一个name属性，所以动作对象中的name属性永远不能被访问到。

### 2. Struts2标签概要

Struts2标签API通过使用条件呈现以继承ValueStack上应用程序的域模型数据，提供了动态创建健壮的网页功能。这些标签可分为4类：数据标签、流程控制标签、UI标签和其他标签。现在我们先不看UI标签，先研究另外3种标签。

#### 2.1 Struts 2标签API语法

Struts2标签定义在比任何具体的视图层技术更高的层。使用这些标签与调用API一样简单，标签API指定了标签公开的属性和参数。标签API的接口已经全部在(JSP、Velocity和FreeMarker)中实现。

**1.JSP语法**

Struts2标签的JSP版本与其他JSP标签一样。异侠是property标签的简单语法：

    <s:property value="name"/>
    

其他唯一需要说明的是，在使用Struts 2标签之前，在页面的开始部分必须声明包含property标签库的声明：

    <%@ page contentType="text/html; charset=UTF-8" %>
    <%@ taglib prefix="s" uri="/struts-tags" %>
    

#### 2.2 使用OGNL设置标签属性

向标签的属性设置值时，这个属性期望一个字符串字面值，还是一个指向ValueStack上有具体类型的值的OGNL表达式呢？下面我们就讲解这个问题。

在JSP页面中，所有的内容都是字符串，我们应该很熟悉使用某种转义字符序列来强制这些属性作为表达式解析而不是字符串字面值解析，例如EL表达式的写法${expression}的写法。但是这样会导致标签的边界难于阅读，导出都是EL表达式。为了让OGNL在标签中使用更加直观、可读性更强，Struts2会假定每一个属性都期望得到什么类型的数据，尤其在处理标签的属性时，会区分谁的类型是String，谁的类型不是String。

**1.字符串和字符串类型的属性**

如果一个属性的类型是String，那么在真实的JSP页面中，写入属性的值会被作为字符串字面值解析。如果一个属性不是String类型，那么属性中写入的值会被当作OGNL表达式解析。

例如下面两个标签，value的值会默认作为OGNL表达式解析，而default的值则会作为字符串解析。

    nonExistingProperty on the ValueStack = <s:property value="nonExistingProperty" />
    nonExistingProperty on the ValueStack = <s:property value="nonExistingProperty" default="doesNotExist" />
    

**2.强制使用OGNL解析**

但是，假如前面的例子，我们想使用一个来源于ValueStack的String属性作为标签的default属性值。这时候就需要强制使用OGNL解析的方式了，写法如下，有些像EL表达式，但不同的是使用%:

    nonExistingProperty on the ValueStack =<s:property value="nonExistingProperty" default="%{myDefaultString}" />

 [1]: /media/2013/03/3539674110.png "ActionContex包含的对象.png"
 [2]: /media/2013/03/1296015878.png "ValueStack是OGNL表达式默认解析对象.png"