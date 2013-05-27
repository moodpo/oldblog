# 复习笔记——Struts 2 的结果组件

- date: 2013-03-17
- category: Work
- tags: struts2, note

----

本节我们将构建一个自定义的结果组件，展示在Struts 2平台上开发Ajax应用程序的技术，通过这个示例你将了解Struts 2的结果组件，以及它们如何工作。

### 1. 动作之后

动作做的最后一件事是返回一个控制字符串以便告诉框架哪个可用的结果应该呈现视图。准确的说结果是什么呢？我们把结果描述为框架MVC架构中视图关注点的封装。在经典的Web应用程序中，这些视图关注点通常等同于创建返回客户的HTML页面，默认情况下，框架使用与JSP协作的结果类型来呈现这些响应页面。但是如果你想重定向到另一个URL而不是客户呈现一个页面，或者使用非标转的使用模式，例如非典型的Ajax应用程序模式，又要如何做呢？这些正是本节的内容。

#### 1.1 如何使用自定义的结果组件构建Struts 2 Ajax应用程序

传统的Web应用程序向客户返回一个完整的HTML页面响应，动作会默认使用转发结果。然而，Ajax应用程序与此完全不同，Ajax程序不请求完整的HTML页面，它们只需要数据。这些数据来源于多种方式，有些Ajax应用程序需要HTML片段作为响应，而另一些期望XML或者JSON响应。Strtus 2可以使用结果组件的灵活性非常容易的处理这种情况。

#### 1.2 实现JSON结果类型

在本节中，我们准备开发一个可以为Ajax客户返回JSON响应的结果类型，JSON提供了一种简洁的、基于文本的数据对象序列化的方式。JSON是Web应用程序服务器和Ajax客户之间数据通信的一种非常简洁和轻量级的方式。

**JSONResult编码**

如果我们计划使用Ajax客户端请求JSON响应，需要构造一个能够这样做的结果。这将需要自定义JSONResult。从某些方面来说，构建一个自定义的结果像编写一个动作一样简单。一个结果的实现唯一的需求是它必须实现接口com.opensymphony.xwork2.Result。这个接口如以下代码所示：

    public interface Result extends Serializable {
        public void execute(ActionInvocation invocation) throws Exception;
    }
    

因此我们只需要提供一个execute()方法作为入口点，这个方法接收一个ActionInvocation参数，结果通过这个参数能够访问所有与当前请求相关的信息，这就是结果组件访问数据的原理。

下面是JSONResult这个自定义结果组件的全部代码：

    public class JSONResult implements Result{
        private static final long serialVersionUID = 1L;
    
        // 这里定义了result组件的默认参数
        public static final String DEFAULT_PARAM = "classAlias";
    
        private String classAlias;
    
        public String getClassAlias() {
            return classAlias;
        }
    
        public void setClassAlias(String classAlias) {
            this.classAlias = classAlias;
        }
    
        public void execute(ActionInvocation invocation) throws Exception {
            // 设置内容类型
            ServletActionContext.getResponse().setContentType("text/plain");
            PrintWriter responseStream = ServletActionContext.getResponse()
                    .getWriter();
            ValueStack valueStack = invocation.getStack();
            // 从ValueStack上取得要序列化的对象
            // 动作会把向客户端发送的需要序列化的对象放在jsonModel属性上
            Object jsonModel = valueStack.findValue("jsonModel");
            XStream xstream = new XStream(new JettisonMappedXmlDriver());
            if (classAlias == null) {
                classAlias = "object";
            }
            xstream.alias(classAlias, jsonModel.getClass());
            responseStream.println(xstream.toXML(jsonModel));
        }
    }
    

一直与JSP一起使用的RequestDispatcher结果类型把JSP页面的位置作为默认参数，这与JSONResult定义classAlias为默认参数的方式相同。定义默认参数的好处是，向结果传递参数时不需要指定参数的名字。在以下的结果定义中，你可以看到我们只是把JSP的位置放在了result元素内部，而没有通过一个param标签来提供它：

    <result>/chapterEight/VisitorHomePage.jsp</result>
    

这个路径值被自动设置到RequestDispatcher结果对象的“位置”属性上。我们也可以显式的使用param标签想如下这样指定位置属性，但这不是必须的：

    <result type="dispatcher">
        <param name="location">/chapterEight/VisitorHomePage.jsp</param>
    </result>
    

经过序列化对象，最终JSON响应的内容会是这样：

    {"artist":{"username":"Mary","password":"max","portfolioName":"Mary's
    Portfolio","firstName":"Mary","lastName":"Greene",
    "receiveJunkMail":"false"}}
    

上述示例中，JSON定义了一个叫做artist的对象。这个artist对象是一个关联数组，包含了User对象的属性相同的名称对。JavaScript客户端代码能够从这些JSON格式的数据快速初始化一个对象。

**客户端的Ajax代码**

在客户端，我们使用JavaScript Ajax技术向Struts 2应用程序提交异步请求。下面是相关的Ajax JavaScript函数fetchUser()，它被页面产生的事件调用，用来提交请求。

    function fetchUser(){
        console=document.getElementById('console');
        var selectBox = document.getElementById('AjaxRetrieveUser_username');
        var selectedIndex = selectBox.selectedIndex;
        var selectedValue = selectBox.options[selectedIndex].value
        // 提交请求
        sendRequest("AjaxRetrieveUser.action","username=" + selectedValue , "POST");
    }
    

下面看一下Ajax XMLHttpRequest对象的回调函数如何处理JSON响应的：

    function onReadyState() {
        var ready=req.readyState;
        var jsonObject=null;
        if ( ready == READY_STATE_COMPLETE ){
            jsonObject=eval( "("+ req.responseText +")" );
            // 更新页面
            toFinalConsole ( jsonObject );
        }
    }
    

**动作**

编写一个Ajax动作与一个普通的动作没有什么不同。动作不需要知道结果中发生的任何事情，动作继续以原来的模式运行，从请求参数中取得数据，执行某些业务逻辑，准备最终的数据对象，之后将一切内容转交给结果。

    public class RetrieveUser extends ActionSupport {
        public String execute(){
            User user = getPortfolioService().getUser( getUsername() );
            setJsonModel(user);
            return SUCCESS;
        }
        private String username;
        private Object jsonModel;
        . . .
        JavaBeans Property Implementations Omitted for Brevity
        . . .
    }
    

这些代码看起来很像我们曾经使用的任何其他动作，它不会在意请求来源于JavaScript的XMLHttpRequest对象，或者响应会以JSON序列化的方式发送，传入和传出数据仍然由JavaBean属性来承载。最重要的是这个动作可以被重用，指向一个普通的完整页面的JSP结果而无需修改一行代码。

**声明和使用JSONResult结果类型**

之前，我们只是用过那些作为struts-default包中的一部分预先定义的结果。很明显，JSONResult没有在默认包中声明。因此，我们需要在自己的包中将JSONResult声明为可用的结果类型。为此可以在struts.xml中做如下声明：

    <result-types>
        <result-type name="customJSON" class="manning.chapterEight.JSONResult" />
    </result-types>
    

有了这个声明，就可以在这个包以及任何继承了这个包的任何地方使用这个结果类型。以下是使用的例子：

    <action name="AjaxRetrieveUser" class="manning.chapterEight.RetrieveUser">
        <result type="customJSON">artist</result>
    </action>
    

我们把artist作为默认的参数传入，这个参数会设置到结果的classAlias属性上，这个属性将作为对象的JSON序列化的名字。

### 2. 常用的结果类型

Struts2框架自带了几个内建的结果类型，下表中的结果覆盖了一个传统的Web应用程序的最常用的用例：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      结果类型
    </td>
    
    <td>
      用例
    </td>
    
    <td>
      参数
    </td>
  </tr>
  
  <tr>
    <td>
      dispatcher(转发)
    </td>
    
    <td>
      JSP，其他的Web应用程序资源，例如Servlet
    </td>
    
    <td>
      location(默认)、parse
    </td>
  </tr>
  
  <tr>
    <td>
      redirect(重定向)
    </td>
    
    <td>
      告诉浏览器重定向到另一个URL
    </td>
    
    <td>
      location(默认)、parse
    </td>
  </tr>
  
  <tr>
    <td>
      redirectAction
    </td>
    
    <td>
      告诉浏览器重定向到另一个Struts动作
    </td>
    
    <td>
      actionName(默认)、namespace，额外的参数会变成查询字符串参数
    </td>
  </tr>
</table>

所有的这些内建的结果类型都定义在struts-default.xml文件中，因此只有扩展struts-default包时它们才是内建的。result-type声明将逻辑名映射到类实现，以下代码片段展示了struts-default.xml文件中FreemarkerResult的声明：

    <result-type name="freemarker"
        class="org.apache.struts2.views.freemarker.FreemarkerResult"/>
    

#### 2.1 RequestDispatcher，也叫dispatcher

当你想呈现一个JSP页面作为动作的结果时使用这个结果类型。DispatcherRequest是默认的结果类型，因此不需要显示的声明。以下是来源与struts-defalut.xml指定dispatcher为默认类型的片段：

    <result-type name="dispatcher" 
        class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>  
    

下面是显式的声明结果类型为dispatcher的例子：

    <action name="SelectPortfolio" class="manning.chapterSeven.SelectPortfolio">
        <result type="dispatcher" >/chapterSeven/SelectPortfolio.jsp</result>
    </action>
    

更详细的声明方式：

    <action name="SelectPortfolio" class="manning.chapterSeven.SelectPortfolio">
        <result type="dispatcher" >
            <param name="location">/chapterSeven/SelectPortfolio.jsp</param>
            <param name="parse">true</param>
        </result>
    </action>
    

其中第一个参数是location，这个参数指定了请求转发到的Servlet资源的位置。第二个参数是parse，这个参数决定location指定的参数是否被当作OGNL表达式解析。这两个参数默认的设定就是如此，因此通常情况下可省略它们。

#### 2.2 ServletRedirectResult，也叫redirect

RequestDispatcher的特点是转发完全发生在服务器端，所有的处理都发生在Servlet API内部，并且在同一个线程内部。这意味着第一个资源的数据通过Servlet API和Struts 2的ActionContext都可以被第二个资源访问。而重定向认为当前的请求已经完成，发出一个HTTP重定向响应以便告诉浏览器指向一个新位置，这个行为正是关闭了当前请求，不管从Struts 2 ActionContext的角度看还是从Servlet请求的角度看都是如此。

使用重定向最常见的原因是需要改变浏览器中显示的URL。当重定向发送时，浏览器处理这个响应，之后向重定向响应中的新URL发送一个请求。重定向发出的URL会覆盖浏览器中原来的URL。

**声明重定向的结果类型**

以下是struts-default.xml中对于redirect结果类型的声明定义：

    <result-type name="redirect"
        class="org.apache.struts2.dispatcher.ServletRedirectResult"/>
    

以下代码片段展示了如何配置一个重定向结果以告诉浏览器指向其他URL：

    <action name="SendUserToSearchEngineAction" class="myActionClass">
        <result type='redirect'>http://www.google.com</result>
    </action>       
    

重定向结果支持两个参数location和parse。这两个参数与RequestDispatcher支持的两个参数完全相同。

**嵌入OGNL表达式创建动态的位置**

我们可以在location参数值中嵌入OGNL表达式。以下示例展示了在运行时如何从ValueStack中取值，并且将取得的值作为查询字符串的参数传递到与之前示例中使用的相同的URL中：

    <action name="SendUserToSearchEngineAction" class="myActionClass">
        <result type='redirect' >
            http://www.google.com/?myParam=${defaultUsername}
        </result>
    </action>
    

首先需要注意使用了美元符号这点。在页面中我们一直使用%作为OGNL表达式的转义序列，但是在struts.xml以及其他声明性框架的配置文件中，必须使用美元符号代替。除此之外，OGNL访问仍旧按照相同的方式工作。

#### 2.3 ServletActionRedirectResult，也叫redirectAction

redirectAction结果与简单的重定向结果做相同的事情，只有一点重要的不同，这个版本的重定向能够理解声明性架构中定义的Struts 2动作的逻辑名，即可以从动作和包的声明中提供redirectAction的名字和命名空间。以下展示了它的使用方法：

    <action name="Login" class="manning.chapterEight.Login">
        <result type="redirectAction">
            <param name="actionName">AdminPortfolio</param>
            <param name="namespace">/chapterEight/secure</param>
        </result>
        <result name="input">/chapterEight/Login.jsp</result>
    </action>
    

同样我们也可以容易的配置传递到目标动作的请求参数和，下面是用例：

    <action name="Login" class="manning.chapterEight.Login">
        <result type="redirectAction">
            <param name="actionName">AdminPortfolio</param>
            <param name="namespace">/chapterEight/secure</param>
            <param name="param1">hardCodedValue</param>
            <param name="param2">${testProperty}</param>
        </result>
        <result name="input">/chapterEight/Login.jsp</result>
    </action>       
    

除了添加的两个请求参数外，这个结果的配置与之前的示例没有什么不同。首先我们为param1参数硬编码了一个值。之后，使用前面学到的OGNL嵌入技巧动态传入了第二个参数，以下是结果会用来重定向浏览器的新URL：

    http://localhost:8080/manningSampleApp/chapterEight/
    secure/AdminPortfolio.action?param1=hardCodedValue&param2=777   
    

以上就是对最常用的结果类型的讲解。

### 3. 全局结果

有时候我们需要配置全局结果，这意味着这个结果可以被整个包内的所有动作使用，例如最常见的例子错误页面。以下是全局结果的声明方式，在包的global-results元素内声明：

    <global-results>
        <result name="error">/chapterFour/Error.jsp</result>
    </global-results>
