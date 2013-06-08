# 复习笔记——Struts 2 拦截器(一)

- date: 2013-03-10
- category: Work
- tags: struts2,J2EE

----

从开发人员日常工作的角度来看，动作组件可能是框架的核心和灵魂，然而在后台工作的默默无闻的拦截器却可以说是真正的英雄——拦截器负责完成了框架的大部分处理工作。

### 1.拦截请求的意义

#### 1.1 清理MVC

拦截器消除了动作组件中的横切任务。日志记录功能是典型的横切任务，它不是某一个动作所特有的，而是横向关联所有动作。作为软件工程师，我们会把这个任务提到更高的层面，让它处在任何需要日志记录的请求上(或者说之前)，达到将MVC关注点分离的目的。

拦截器承担的另一些任务被称为预处理或者后加工。预处理任务的一个很好的示例就是常见的数据转移，它通过params拦截器实现。

在Struts2中没有一个动作被单独调用，动作调用是一个分层的过程，总是包含一系列的拦截器在动作执行之前或者之后执行。框架不直接调用动作的execute()方法，而是创建一个叫做ActionInvocation的对象，它封装了动作的一系列被配置的动作执行之前之后触发的拦截器。下图展示了ActionInvocation类封装的完整的动作执行过程。

![ActionInvocation 流程.png][1]

拦截器的强大功能之一是改变调用工作流。有时候某个拦截器会决定动作不应该执行，此时，拦截器可以自己返回一个控制字符串，从而终止工作流。以workflow拦截器为例，这个拦截器做两件事情。首先，如果动作实现了Validateable接口，那么调用动作的validate()方法。其次，检查动作上是否出现了错误信息。如果出现了错误信息，它返回控制字符串并终止后续执行。

#### 1.2 拦截器的好处

使用拦截器分层让软件更整洁，并增加了可读性和可测试性，也提高了灵活性。从这种灵活性中得到的两个主要益处是可重用和可配置。

每一个人都想重用软件，想达到可重用很简单，只需将想重用的逻辑隔离到清晰分割的单元。把这些逻辑隔离到拦截器之后，我们可以把它们放在任何地方，应用到所有动作类上。例如我们通常通过继承defaultstack获得的代码重用。

除代码重用之外，拦截器分层的能力还给我们带来了另外一些非常重要的好处。我们能够很容易的配置它们的顺序和数量。

### 2.拦截器的工作原理

#### 2.1 总指挥 ActionInvocation

ActionInvocation封装了与特定动作执行相关的所有处理细节，所以知道ActionInvocation做什么就等于知道Struts2如何处理请求。当框架收到一个请求时，它首先必须决定这个URL映射到哪个动作，这个动作的一个实例会被加入到一个新创建的ActionInvocation示例中。接着，框架咨询声明性架构(通过xml或者Java注解)，以发现哪些拦截器应该触发，指向这些拦截器的引用将被加入到ActionInvocation中。

#### 2.2 如何触发拦截器

框架创建了ActionInvocation，并填充了需要的所有对象和信息，通过调用ActionInvocation公开的invoke()方法开始动作的执行，此时拦截器栈中的第一个拦截器开始调用过程。ActionInvation负责跟踪执行过程达到的状态，并把控制交给栈中合适的拦截器。ActionInvation通过调用拦截器的intercept()方法将控制转交给拦截器。后续拦截器继续执行，最终执行动作。这些都是通过递归调用ActionInvocation的invoke()方法实现。

  1. 调用ActionInvocation对象的invoke()方法
  2. 开始调用第一个拦截器
  3. 调用拦截器的intercept()方法，并把ActionInvocation实例作为参数传递
  4. 拦截器调用传递的参数即ActionInvocation实例的invoke()方法
  5. 开始调用第二个拦截器
     ......
  N. 栈中没有拦截器开始触发动作
    

在此过程中，ActionInvocation在内部管理处理状态，它总是能知道现在处在栈的哪里。

拦截器有一个三阶段的、有条件的执行周期，如下所述：

  1. 做一些预处理
  2. 通过调用invoke()方法将控制转移给后续的拦截器，最后直到动作；或者通过返回一个控制字符串终端执行
  3. 做一些后加工
    

### 3.Struts2 内置的拦截器

Struts2框架自带了一系列功能强大的内置拦截器，它们提供了你想从Web框架得到的大部分功能。除了defaultStack这个拦截器栈，框架还自带了很多拦截器和已经预先配置好的拦截器栈。

#### 3.1工具拦截器

这些拦截器提供了辅助开发、调优及解决问题的简单工具。

##### 3.1.1 timer拦截器

这个简单的拦截器只记录执行花费的时间，在拦截器栈中的位置决定了它实际测量什么时间。

##### 3.1.2 logger拦截器

这个拦截器提供了一个简单的日志记录机制，记录了在预处理事的进入声明以及在后加工时的退出声明。

#### 3.2 数据转移拦截器

##### 3.2.1 params拦截器(defaultStack)

这个熟悉的拦截器将请求参数转移到通过ValueStack公开的属性(包括ModelDriven动作通过域模型对象提供的属性)上。params拦截器不知道这些数据最终会去哪里，它只是把数据转移到在ValueStack上发现的第一个匹配的属相上。

动作总是在请求处理周期开始时被放到ValueStack上。ModelDriven动作则被modelDriven拦截器放在ValueStack上。

##### 3.2.2 static-params拦截器(defaultStack)

这个拦截器也将参数转移到ValueStack公开的属性上，不同的是参数来源于声明性框架(XML)的动作元素中。

    <action name="Upload" class="com.moodpo.review.action.Upload">
        <!-- 上传路径 -->
        <param name="storePath">E:\temp\</param>
        <result>/Uploaded.jsp</result>
        <result name="input">/UploadForm.jsp</result>
    </action>
    

注意一点，defaultStack中static-params拦截器在params拦截器之前触发，这意味着请求参数会覆盖XML中param元素的值。

##### 3.2.3 autowiring拦截器

这个拦截器为使用Spring管理应用程序资源提供了一个集成点。复习Spring的时候再说吧。

##### 3.2.4 servlet-config拦截器(defaultStack)

这个拦截器将来源于Servlet API的各种对象注入到动作中，动作只需实现必要的接口即可。以下的接口可以用来取得与Servlet环境相关的不同对象。

    ServletContextAware设置ServletContext
    ServletRequestAware设置HttpServletRequest
    ServletResponseAware设置HttpServletResponse
    ParameterAware设置Map类型的请求参数
    RequestAware设置Map类型的请求属性
    SessionAware设置Map类型的会话属性
    ApplicationAware设置Map类型的应用程序领域属性
    PrincipalAware设置Principal对象(安全相关)
    

每一个接口包含一个方法——当前资源的设置方法。最佳实践建议避免使用Servlet API对象，因为他们会将动作代码绑定到Servlet API.

##### 3.2.5 fileUpload拦截器(defaultStack)

fileUpload拦截器将文件和元数据从多重请求转换为常规的请求参数，以便能够将它们像普通参数一样设置到动作上。

#### 3.3 工作流拦截器

工作流拦截器提供改变请求处理的工作流的机会，这里的工作流是指贯穿拦截器、动作、结果，最后又回到拦截器的处理路径。工作流拦截器检查处理的状态，有条件的干涉、改变正常路径。

##### 3.3.1 workflow拦截器(defaultStack)

确实有一个拦截器就叫workflow。它与动作协作，提供数据验证以及验证错误发生时改变后续工作流的功能。workflow拦截器还引入了另一个重要的拦截器概念——使用params调整拦截器的执行。workflow拦截器可以使用以下几个参数：

    alwaysInvokeValidate(true或者false,默认是true，意味着validate()方法会被调用)
    inputResultName(验证失败时选择的结果的名字，默认值是Action.INPUT)
    excludeMethods(workflow拦截器不应执行的方法名，这样可以略过一个特定入口方法的验证检查)
    

这些内容在defaultStack中进行了配置，如下来源于struts-default.xml的代码片段：

    </interceptor-stack>
        ......
        <interceptor-ref name="workflow">
            <param name="excludeMethods">input,back,cancel,browse</param>
        </interceptor-ref>
    </interceptor-stack>
    

这样配置将排除input,back,cancel,browse方法上的validate()验证。

##### 3.3.2 validation拦截器(defaultStack)

validation拦截器是Struts2验证框架的一部分，提供了声明性的方式验证你的数据。不用编写验证代码，验证框架让你使用XML或者Java注解描述数据的验证规则。

注意：validation拦截器在workflow拦截器之前触发。

##### 3.3.3 prepare拦截器(defaultStack)

prepare拦截器提供了一种向动作追加额外工作流处理的通用入口点。当prepare拦截器执行时，它在动作上查找prepare()方法(首先查找是否实现了Preparable接口)，它允许执行任何类型的预处理。

prepare拦截器也很灵活，例如它可以在一个动作上为不同的执行方法定义特别的预备方法。以下是预备方法的命名约定：

    动作方法名    预处理方法1                  预处理方法2 
    input()        prepareInput()          prepareDoInput() 
    update()     prepareUpdate()       prepareDoUpdate()
    

##### 3.3.4 modelDriven(defaultStack)

modelDriven拦截器通过调用getModel()方法改变执行的工作流，并将模型对象放在ValueStack上从请求接受参数。在不使用这个拦截器的情况下，参数会被params拦截器直接转移到动作对象上。

#### 3.4 其他拦截器

##### 3.4.1 exception拦截器(defaultStack)

exception拦截器在defaultStack中出现在第一位，也应该在任何你创建的自定义栈中出现在第一位。exception拦截器捕获异常，并根据类型将它们映射到用户自定义的错误页面。它的位置在栈的顶端，这样可以保证它能够捕获动作调用所有阶段可能生成的所有异常，而在后加工阶段又会最后一个触发，因此能够捕获所有的异常。当捕获异常时，exception拦截器会创建一个ExceptionHolder对象，并将其放在ValueStack的最顶端，ExceptionHolder是一个异常的包装器，它把跟踪栈和异常作为JavaBean属性公开出来，可以在错误页面中通过标签访问这些属性。

##### 3.4.2 token拦截器和token-session拦截器

token和token-session拦截器可以作为避免表单重复提交系统的一部分。token拦截器用来向被拦截器检查的请求传入一个令牌，如果唯一的令牌再次来到拦截器，那么这个请求被视为重复的。

##### 3.4.3 scoped-modelDriven拦截器(defaultStack)

这个拦截器为动作的模型对象提供跨请求的向导式的持久性，它增加了modelDriven拦截器的功能，允许你将模型对象存储到绘画作用域中。

##### 3.4.4 execAndWait拦截器

当一个请求需要执行很长时间时，最好能给用户一些反馈，execAndWait拦截器可帮助用户避免急躁。

#### 3.5 内置的拦截器栈

大部分可能会用到的其他内置拦截器栈都是defaultStack的简化版本，这些较小的栈是为了成为模块化的构建单元，用来构建更大的栈。

一般情况下推荐使用defaultStack，在某些情况下，多余的拦截器并不会那么影响性能，况且混乱的组织拦截器可能会迅速增加调试的复杂度，因此还是使用内置的最容易的途径——defaultStack吧。

> Struts2不仅非常灵活，而且它也代表着更多的开箱即用。

 [1]: /media/2013/03/3358556227.png "ActionInvocation 流程.png"