# 复习笔记——Struts 2 标签（二）

- date: 2013-03-14
- category: Work
- tags: struts2, tag, J2EE

----

本节将学习Struts2标签库的详细内容，包括数据标签，流程控制标签和其他标签。

### 1.数据标签

数据标签能从ValueStack上取得数据，或者将变量、对象放在ValueStack上。本小节将讨论property、set、push、bean和action标签，并演示这些标签的常见用例。

#### 1.1 property标签

property标签提供了一种将属性写入呈现的HTML页面的快速、方便的方法。下面是这个标签的重要属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      默认值
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      否
    </td>
    
    <td>
      栈最顶端
    </td>
    
    <td>
      Object
    </td>
    
    <td>
      被显示的值
    </td>
  </tr>
  
  <tr>
    <td>
      default
    </td>
    
    <td>
      否
    </td>
    
    <td>
      空(null)
    </td>
    
    <td>
      String
    </td>
    
    <td>
      值为空时使用的默认值
    </td>
  </tr>
  
  <tr>
    <td>
      escape
    </td>
    
    <td>
      否
    </td>
    
    <td>
      true
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      是否转义HTML
    </td>
  </tr>
</table>

实际的使用很简单，代码如下：

    <h4>Property Tag</h4>
    The current user is <s:property value="user.username"/>.
    

#### 1.2 set标签

在标签的上下文中，set意味着给一个属性指定一个别名。有时候一个属性需要使用一个很长的、复杂的OGNL表达式才能引用到它，通过为这个属性重新指定一个顶层的名字可以更容易、更快捷的访问它。默认情况下，这个属性会变成ActionContext中的一个命名对象，与ValueStack、会话映射齐名。所以你可以使用如#myObjec这样的OGNL表达式把它作为一个顶层的命名对象引用。当然，你也可以指定这个新引用被放置在ActionContext中的某个作用域中。下面是set标签的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      name
    </td>
    
    <td>
      是
    </td>
    
    <td>
      String
    </td>
    
    <td>
      被设置在指定作用域内的变量的引用名
    </td>
  </tr>
  
  <tr>
    <td>
      scope
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      application、session、request、page或者action，默认是action
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      否
    </td>
    
    <td>
      Object
    </td>
    
    <td>
      设置值的表达式
    </td>
  </tr>
</table>

下面是实际的使用：

    <s:set name="username" value="user.username"/>
    Hello, <s:property value="#username"/>. How are you?    
    

以下代码经把新引用设置为application作用域映射内的一个实体：

    <s:set name="username" scope="application" value="user.username"/>
    Hello, <s:property value="#application['username']"/>. How are you?
    

#### 1.3 push标签

set标签允许你创建指向值的新引用，而push标签允许你把属性放到ValueStack上。当你需要对一个对象进行很多操作时这个标签会很有用。将对象放在ValueStack顶端之后，它的所有属性可以通过第一级别的OGNL表达式访问。

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      是
    </td>
    
    <td>
      Object
    </td>
    
    <td>
      放到ValueStack中的值
    </td>
  </tr>
</table>

以下是使用这个标签的示例：

    <s:push value="user">
        This is the "<s:property value="portfolioName"/>" portfolio,
        created by none other than <s:property value="username"/>
    </s:push>
    

push标签把user属性放到ValueStack的顶端，这是我们就可以把user的属性当作ValueStack这个虚对象上的顶级属性访问，这样OGNL表达式就变简单了。最后push的结束标签把user从ValueStack顶端删除。

#### 1.4 bean标签

bean标签像是set标签和push标签的混合标签。主要的不同是不需要使用一个已经存在的对象，bean标签会自己new这个对象，并把它放到ValueStack上或者设置为ActionContext的顶级引用。默认是将对象放在ValueStack上，并在标签存在期间一直保留在ValueStack上。以下是bean标签的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      name
    </td>
    
    <td>
      是
    </td>
    
    <td>
      String
    </td>
    
    <td>
      被传入bean的包名和类名
    </td>
  </tr>
  
  <tr>
    <td>
      var
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      在结束标签作用域外引用这个bean时使用的变量名
    </td>
  </tr>
</table>

下面示例展示了如何创建和存储一个bean作为ActionContext中的命名参数：

    <s:bean name="org.apache.struts2.util.Counter" var="counter">
        <s:param name="last" value="7"/>
    </s:bean>
    <s:iterator value="#counter">
        <li><s:property/></li>
    </s:iterator>
    

默认情况下创建的对象会存储在ValueStack中，使用var后会在ActionContext中存储这个对象的引用，正是因为如此，在iterator标签中，我们使用它的方式需要用#操作符，即OGNL表达式#counter。在许多Struts2标签中，都存在可选的var属性，它的作用与此类似。

bean标签是我们研究的第一个参数化标签。现在看看如何使用bean标签将新创建的bean放在ValueStack上而不是存储在ActionContext上。而且我们我们还是用param标签演示怎样向自定义对象提供参数。

    <s:bean name="manning.utils.JokeBean" >
        <s:param name="jokeType">knockknock</s:param>
        <s:property value="startAJoke()"/>
    </s:bean>
    

上面我们使用了OGNL方法调用语法starAJoke()，bean标签不需要完全遵守JavaBean标准，但是OGNL能够使用这个方法，我们将在OGNL详解中讲解它的机制。

最后，注意我们向JokenBean传递了一个参数，这个参数用来控制这个bean讲述的笑话的类型。只要这个对象实现了一个与这个参数同名的属性，它就能自动接收传入的参数。

#### 1.5 action标签

这个标签允许我们从当前的视图层调用其他的动作。action标签的实际使用很简单，只需指定其他需要调用的动作。

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      name
    </td>
    
    <td>
      是
    </td>
    
    <td>
      String
    </td>
    
    <td>
      动作名
    </td>
  </tr>
  
  <tr>
    <td>
      namespace
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      动作的命名空间，默认使用当前页面的命名空间
    </td>
  </tr>
  
  <tr>
    <td>
      var
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      在页面后续代码中使用的动作对象的引用名
    </td>
  </tr>
  
  <tr>
    <td>
      executeResult
    </td>
    
    <td>
      否
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      设置为true时排除动作的结果(默认值为false)
    </td>
  </tr>
  
  <tr>
    <td>
      flush
    </td>
    
    <td>
      否
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      设置为true时在action标签的结尾会刷新写出缓冲(默认为true)
    </td>
  </tr>
  
  <tr>
    <td>
      ignoreConte xtParams
    </td>
    
    <td>
      否
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      设置为true时动作被调用时不包含请求参数(默认值为false)
    </td>
  </tr>
</table>

以下是一个选择包含辅助动作结果的示例：

    <h3>Action Tag</h3>
    <h4>This line is from the ActionTag action's result.</h4>
    <s:action name="TargetAction" executeResult="true"/>
    

通常情况下，你希望辅助动作被触发，但不写结果，而是提供一个在ActionContext上某个地方存储域数据，在控制返回后，主动作可以访问这些数据，下面示例正是如此：

    <h4>This line is before the ActionTag invokes the secondary action.</h4>
    <s:action name="TargetAction"/>
    <h4>Secondary action has fired now.</h4>
    <h5>Request attribute set by secondary action = </h5>
    <pre> <s:property value="#request.dataFromSecondAction"/></pre>
    

### 2.控制标签

Struts2中有一系列的标签可以容易的控制页面执行的流程，包括iterator标签和if/else标签等。

#### 2.1 iterator标签

使用iterator表亲啊可以很容易的便利集合对象。下面是这个标签的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      是
    </td>
    
    <td>
      Object
    </td>
    
    <td>
      被遍历的对象
    </td>
  </tr>
  
  <tr>
    <td>
      status
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      如果被指定，IteratorStatus对象会使用这个属性指定的名字被放在ActionContext中
    </td>
  </tr>
</table>

下面是一个使用示例：

    <s:iterator value="users" status="itStatus">
        <li>
        <s:property value="#itStatus.count" />
        <s:property value="portfolioName"/>
        </li>
    </s:iterator>
    

上面的代码很直观，在标签内部，每一个对象会轮流被放在ValueStack顶端，我们可以很容易的访问用户属性。注意，iterator标签通过指定status属性，也声明了IteratorStatus对象。你为status属性指定的名字会用来作为关键字从ActionContext中取得遍历状态对象。

**使用IteratorStatus**

有时候我们非常想知道当前发生的循环的状态信息。如果定义了status属性，那么会在ActionContext中提供一个IteratorStatus对象，这个对象可以提供诸如集合大小、当前索引、当前对象的索引是奇数还是偶数的简单信息。下面是IteratorStatus的共有方法：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      方法名
    </td>
    
    <td>
      返回值
    </td>
  </tr>
  
  <tr>
    <td>
      getCount
    </td>
    
    <td>
      int
    </td>
  </tr>
  
  <tr>
    <td>
      getIndex
    </td>
    
    <td>
      int
    </td>
  </tr>
  
  <tr>
    <td>
      isEven
    </td>
    
    <td>
      boolean
    </td>
  </tr>
  
  <tr>
    <td>
      isFirst
    </td>
    
    <td>
      boolean
    </td>
  </tr>
  
  <tr>
    <td>
      isLast
    </td>
    
    <td>
      boolean
    </td>
  </tr>
  
  <tr>
    <td>
      isOdd
    </td>
    
    <td>
      boolean
    </td>
  </tr>
  
  <tr>
    <td>
      modulus(int operand)
    </td>
    
    <td>
      boolean
    </td>
  </tr>
</table>

#### 2.2 if和else标签

很多语言都提供了熟悉的if和else控制逻辑，Struts2标签亦是如此。使用这些标签很容易，它们都只有一个属性，即布尔型test:

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      test
    </td>
    
    <td>
      是
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      被求值和测试的布尔表达式
    </td>
  </tr>
</table>

以下是如何使用这个标签的示例：

    <s:if test="user.age > 35">This user is too old.</s:if>
    <s:elseif test="user.age < 35">This user is too young</s:elseif>
    <s:else>This user is just right</s:else>
    

### 3.其他标签

这一节将讨论Struts2的inclued标签、URL标签、i18n标签和text标签，最后了解下详细的param标签的使用。这些标签很有用，但是不太容易分类。

#### 3.1 inclued标签

虽然JSP有自己的包含标签<jsp:include>，但是Struts2提供了一个与Struts2集成的更好、有更多高级特性的include标签。简而言之，这个标签可以执行Servlet API样式的包含。下面是其唯一的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      是
    </td>
    
    <td>
      String
    </td>
    
    <td>
      页面、动作、Servlet以及其他可以被引用的URL的名字
    </td>
  </tr>
</table>

这个标签很简单，无须展示例子了吧。include标签的行为很想JSP的include标签，但是它与框架集成的更好并且提供了对ValueStack(使用%{...}从ValueStack取值)和可扩展参数模型的本地访问(使用<s:param>标签传递查询字符串参数)。

#### 3.2 URL标签

Struts2提供了URL标签来帮助管理URL，这个标签支持与URL相关的所有操作，从控制参数到Cookie缺失时自动持久化会话。下面是它的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      基础URL，默认为呈现当前页面的URL
    </td>
  </tr>
  
  <tr>
    <td>
      action
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      生成的URL指向的动作名，使用声明性架构中配置的动作名(不带.action扩展名)
    </td>
  </tr>
  
  <tr>
    <td>
      var
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      如果指定，这个URL不会被重写，而会存储在ActionContext留待后用
    </td>
  </tr>
  
  <tr>
    <td>
      includeParams
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      从all、get、none中选择参数，默认值为get
    </td>
  </tr>
  
  <tr>
    <td>
      includeContext
    </td>
    
    <td>
      否
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      如果为true，那么生成的URL会使用应用程序的Context作为前缀，默认值为true
    </td>
  </tr>
  
  <tr>
    <td>
      encode
    </td>
    
    <td>
      否
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      如果用户浏览器不支持Cookie，会将session ID追加到生成的URL中
    </td>
  </tr>
  
  <tr>
    <td>
      scheme
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      指定协议，默认使用当前协议(HTTP或HTTPS)
    </td>
  </tr>
</table>

下面是两个示例：

    URL = <s:url value="IteratorTag.action"/>
    <a href='<s:url value="IteratorTag.action" />'> Click Me </a>
    

URL标签只是把生成URL输出为字符串，下面是页面输出：

    URL = IteratorTag.action
    <a href='IteratorTag.action'> Click Me </a>
    

如果我们想指向一个动作，那么应该使用action属性：

    URL = <s:url action="IteratorTag" var="myUrl">
                    <s:param name="id" value="2"/>
                </s:url>
    <a href='<s:property value="#myUrl" />'> Click Me </a>
    

这样页面输出会是这样：

    URL =
    <a href='/manningHelloWorld/chapterSix/IteratorTag.action?id=2'>
        Click Me
    </a>
    

#### 3.3 i18n和text标签

这两个标签被用在国际化功能中，它们使用name属性指定关键字，通过这个关键字取得对应的文本消息。

下面是text标签支持的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      name
    </td>
    
    <td>
      是
    </td>
    
    <td>
      String
    </td>
    
    <td>
      在ResourceBundle中查找用的关键字
    </td>
  </tr>
  
  <tr>
    <td>
      var
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      如果找到，文本会使用这个名字保存在ActionContext中
    </td>
  </tr>
</table>

如果你想手动指定应该使用的ResourceBundle，可以使用i18n标签，下面是i18n的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      name
    </td>
    
    <td>
      是
    </td>
    
    <td>
      String
    </td>
    
    <td>
      ResourceBundle的名字
    </td>
  </tr>
</table>

下面是一个示例，它展示了如何使用i18n标签设置ResourceBundle，以及如何使用text标签从中取出文本消息：

    <s:i18n name="manning.chapterSix.myResourceBundle_tr">
        In <s:text name="language"/>,
        <s:text name="girl" var="foreignWord"/>
    </s:i18n>
    "<s:property value="#foreignWord"/>" means girl.
    

#### 3.4 param标签

param标签什么也不做，但它是最重要的标签之一。它不仅在上述标签中起重要作用，而且在许多UI组件标签中也起重要作用。下面是它的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      是否必须
    </td>
    
    <td>
      类型
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      name
    </td>
    
    <td>
      否
    </td>
    
    <td>
      String
    </td>
    
    <td>
      参数名
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      否
    </td>
    
    <td>
      Object
    </td>
    
    <td>
      参数值
    </td>
  </tr>
</table>

在之前的例子中已经展示了param标签的作用，包括作为一种自定义的工具对象传递参数的方法。只要有了这个整体思路，剩下的就是细读标签API看看哪个标签可以接受参数。
