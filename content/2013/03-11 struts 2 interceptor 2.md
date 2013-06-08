# 复习笔记——Struts 2 拦截器(二)

- date: 2013-03-11
- category: Work
- tags: struts2,J2EE

----

![拦截器.jpg][1]

### 1.声明拦截器

XML是声明拦截器的唯一选择，注解机制现在还不支持声明拦截器。

#### 1.1 声明独立的拦截器和拦截器栈

通常，拦截器声明包含声明可用的拦截器并把它们与应该触发的动作关联起来。像所有框架组件的声明一样，拦截器的声明必须在package元素内部。以下是struts-default.xml文件中struts-default包的各个拦截器的声明：

    <package name="struts-default" abstract="true">
        <interceptors>
             ......
            <interceptor name="execAndWait" class="org.apache.struts2.interceptor.ExecuteAndWaitInterceptor"/>
            <interceptor name="exception" class="com.opensymphony.xwork2.interceptor.ExceptionMappingInterceptor"/>
            <interceptor name="fileUpload" class="org.apache.struts2.interceptor.FileUploadInterceptor"/>
             ......
    
            <interceptor-stack name="defaultStack">
                    <interceptor-ref name="exception"/>
                    <interceptor-ref name="alias"/>
                    <interceptor-ref name="servletConfig"/>
                    <interceptor-ref name="i18n"/>
                    <interceptor-ref name="prepare"/>
                    <interceptor-ref name="chain"/>
                    <interceptor-ref name="debugging"/>
                    <interceptor-ref name="scopedModelDriven"/>
                    <interceptor-ref name="modelDriven"/>
                    <interceptor-ref name="fileUpload"/>
                    <interceptor-ref name="checkbox"/>
                    <interceptor-ref name="multiselect"/>
                    <interceptor-ref name="staticParams"/>
                    <interceptor-ref name="actionMappingParams"/>
                    <interceptor-ref name="params">
                        <param name="excludeParams">dojo\..*,^struts\..*</param>
                    </interceptor-ref>
                    <interceptor-ref name="conversionError"/>
                    <interceptor-ref name="validation">
                        <param name="excludeMethods">input,back,cancel,browse</param>
                    </interceptor-ref>
                    <interceptor-ref name="workflow">
                        <param name="excludeMethods">input,back,cancel,browse</param>
                    </interceptor-ref>
                </interceptor-stack>
    
                <default-interceptor-ref name="defaultStack"/>
        </interceptors>
    </package>
    

interceptors元素包含这个包内所有的interceptor和interceptor-stack的声明。每个interceptor元素声明了一个可以在包内使用的拦截器。这些声明实际上没有创建一个拦截器或者把这个拦截器与任何动作关联，它们只是把一个名字映射到一个类。struts-default包声明了几个栈，其中中最重要的是defaultStack。

interceptor-stack元素的内容是一系列interceptor-ref元素，这些引用必须都指向interceptor元素创建的某个逻辑名，interceptor-ref元素也可以传入参数来配置引用创建的拦截器实例。

最后，一个包可以定义一组默认的拦截器，default-interceptor-ref定义的拦截器组会与这个包内没有显示声明自己的拦截器的所有动作关联，它只指向一个逻辑名，这里是defaultStack。

**XML文档结构**

声明使用的XML文档必须遵守特定的顺序规则，这可以通过DTD查看：

    <!ELEMENT struts ((package|include|bean|constant)*, unknown-handler-stack?)>
    <!ELEMENT package (result-types?, interceptors?, default-interceptor-ref?, default-action-ref?, default-class-ref?, global-results?, global-exception-mappings?, action*)>
    

上述语句指出，struts-default.xml文件以struts元素开始，这个元素能够包含4种不同的元素类型的0个或者多个实例。与struts的元素不同，package元素的内容必须遵照特定的顺序。另外，所有包含在一个package元素中的元素，出action元素外，都只能出现一次。interceptors元素必须只出现一次或者一次也不出现，并且不需出现在result-types元素之后，并且在default-interceptor-ref元素之前。

#### 1.2 将拦截器映射到动作组件

可以使用interceptor-ref元素完成拦截器和动作的关联，以下代码片段显示了如何将一系列拦截器和一个特定的动作关联起来：

    <action name="MyAction" class="com.moodpo.review.MyAction">
        <interceptor-ref name="timer"></interceptor-ref>
        <interceptor-ref name="logger"></interceptor-ref>
        <!-- 需要再次声明使用defaultStack -->
        <interceptor-ref name="defaultStack"></interceptor-ref>
        <result>/success.jsp</result>
    </action>
    

首先，这个动作命名了拦截器，但没有提及在struts-default包中声明的defaultStack。正因如此，这个动作必须在一个扩展了struts-default的包中。其次，虽然没有定义任何interceptor-ref的动作会继承默认的拦截器，但是只要动作声明了自己的拦截器，它就失去了自己的默认值，并且为了使用defaultStack就必须显式指出。

#### 1.3 设置、覆盖拦截器参数

很多拦截器可以被参数化。如果一个拦截器接受参数，那么interceptor-ref元素是向它们传入参数的地方。比如之前所说的workflow拦截器被参数化以忽略对动作的某些方法名的访问：

    <interceptor-ref name="workflow">
        <param name="excludeMethods">input,back,cancel,browse</param>
    </interceptor-ref>
    

如果我们想重用包含了defaultStack的拦截器栈，但是我们想改变excludeMethods参数的值要怎么办呢？很简单，代码如下：

    <action name="MyAction" class="com.moodpo.review.MyAction">
        <interceptor-ref name="defaultStack">
            <param name="workflow.excludeMethods">dosomething</param>
        </interceptor-ref>
        <result>/success.jsp</result>
    </action>
    

### 2.构建自定义拦截器

除了安排栈的顺序和在调试中学习解决顺序问题需要多加留意外，编写拦截器其实很容易。

#### 2.1 实现Interceptor接口

在编写一个拦截器时需要实现com.opensymphony.xwork2.interceptor.Interceptor接口。

    public interface Interceptor extends Serializable {
        public void destroy();
        public void init();
        public String intercept(ActionInvocation actionInvocation) throws Exception;
    }
    

可以看到，这个简单的接口只定义了3个方法，前两个方法是典型的生命周期方法，真正的业务逻辑发生在intercept()方法中。

#### 2.2 声明拦截器并构建新的默认栈

在定义完拦截器类后，需要将这个拦截器应用在动作上，所以可以构建一个自定栈，它的大致写法如下：

    <package name="demo4Secure" namespace="/secure" extends="struts-default">
    
        <interceptors>
            <interceptor name="yourInterceptor" class="com.moodpo.review.utils.YourInterceptor"></interceptor>
    
            <interceptor-stack name="myStack">
                <interceptor-ref name="yourInterceptor"></interceptor-ref>
                <interceptor-ref name="defaultStack"></interceptor-ref>
            </interceptor-stack>
        </interceptors>
    
        <default-interceptor-ref name="myStack"></default-interceptor-ref>
    
        <action name="yourAction" class="com.moodpo.review.action.YourAction">
            <result>/success.jsp</result>
        </action>
    
    </package>
    

首先，我们必须使用一个interceptors元素来包含所有的interceptor和interceptor-stack的声明。我们必须使用一个interceptor元素将Java类映射到一个逻辑名(yourInterceptor)。其次，我们构建了一个新栈，这个栈使用defaultStack并且把新拦截器追加到它的上面(当然根据业务的不同，也可以放在下面)。最后，我们把myStack声明为这个包的默认拦截器栈。以上这样做的好处是拦截器与动作代码完全分离，并且实现了可重用。

 [1]: /media/2013/03/2759728019.jpg "拦截器.jpg"