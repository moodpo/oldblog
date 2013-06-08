# 复习笔记——Struts 2 Action

- date: 2013-03-09
- category: Work
- tags: struts2,J2EE

----

Struts2是一个面向动作的框架，Action是它的核心。

### 1.Struts2中Action的作用

封装业务单元 为数据转移提供场所 为结果选择路由

### 2.使用包(package)机制将Action分组

包是一种逻辑容器，想Java的包一样，它提供了一种基于功能或者领域的共性将action组件分组。包级别上定义的URL可用来映射到动作；此外它还具有继承的特性，你能够继承框架已经定义好的一些组件。 包声明只需设置它的四个属性：

    属性          描述
    name(必须)    包的名字
    namespace    包内所有action的命名空间
    extends      被继承的父包
    abstract     如果为true,这个包只能用来定义可继承的组件，不能定义动作
    
    <package name="chapterThreeSecure" namespace="/chapterThree/secure" extends="struts-default">
            <action name="AdminPortfolio">
                <result>/chapterThree/AdminPortfolio.jsp</result>
            </action>
            <action name="AddImage">
                <result>/chapterThree/ImageUploadForm.jsp</result>
            </action>
            <action name="RemoveImage">
                <result>/chapterThree/ImageRemoved.jsp</result>
            </action>
    </package>
    

上面的配置展示了包的用法。当一个请求(比如AddImage)到达时，框架会在/chapterThree/secure命名空间下查找AddImage的action方法，如果没有设置命名空间，请求动作就会进入默认命名空间查找。因为继承了struts-default包，而使用其中的默认命名空间。

**struts-default包中的组件**

只要继承struts-default包，就会自动使用它已经定义好的组件。大部分常用的拦截器在其中已经定义好了，这个包在struts2-core.jar的struts-default.xml中定义，它声明了大多数应用程序用到的拦截器栈：

      <package name="struts-default" abstract="true">
             ........
    
            <!-- A complete stack with all the common interceptors in place.
                 Generally, this stack should be the one you use, though it
                 may do more than you need. Also, the ordering can be
                 switched around (ex: if you wish to have your servlet-related
                 objects applied before prepare() is called, you'd need to move
                 servletConfig interceptor up.
    
                 This stack also excludes from the normal validation and workflow
                 the method names input, back, and cancel. These typically are
                 associated with requests that should not be validated.
                 -->
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
    
            ......
    
       </interceptors>
    
        <default-interceptor-ref name="defaultStack"/>
    
        <default-class-ref class="com.opensymphony.xwork2.ActionSupport" />
    </package>
    

注意那个叫做params的拦截器，从请求到动作的数据自动转移就是由它完成的。

### 3.实现动作(Action)

struts2中的action只需要实现一个返回控制字符串的execute()方法即可，不需要实现action接口，因此基本上任何一个类都可以成为一个动作(action).即使如此框架还是提供了一些接口和实现来让工作变得更简单，成本更小。

**Action接口**  
如果一个action实现了com.opensymphony.xwork2.Action接口，它就可以使用一下常量，而这些也正是框架中使用的默认值：

    public static final String ERROR="error";
    public static final String INPUT="input";
    public static final String LOGIN="login";
    public static final String NONE="none";
    public static final String SUCCESS="success";
    

**ActionSupport类**  
此类提供了数据验证、错误信息传输及错误信息国际化等功能。

**Ⅰ.基本验证**

ActionSupport类使用两个接口与默认拦截器栈的DefaultWorkflowInterceptor拦截器配合完成基本验证。

            <interceptor-ref name="workflow">
                <param name="excludeMethods">input,back,cancel,browse</param>
            </interceptor-ref>
    

当workflow拦截器被触发时，它会在action中查找并执行validate()方法，ActionSupport实现了该方法，不过在实际项目中我们一般要使用自己的业务验证逻辑重写此方法。

    @Override
    public void validate() {
        // TODO Auto-generated method stub
    
        //添加一个字段的数据错误
        addFieldError("fieldName", "errorMessage");
        //使用国际化
        addFieldError("name", getText("properties_name"));
        //添加一个action范围内的错误
        addActionError("anErrorMessage");
    
    }
    

当执行完validate()方法后，workflow拦截器会检查是否有错误信息。如果有错误信息，它将改变请求的工作流，它会立即停止请求的处理，将用户带回到表单页面并显示相应的错误提示。

**Ⅱ.使用资源包(国际化)**

ActionSupport实现了两个接口，它们协作完成了国际化文本信息的功能。首先是实现com.opensymphony.xwork2.TextProvider接口，可以从资源属性文件中通过关键字获取文本的本地化信息。下面是regist_en.properties文件的内容：

    user.exists=This user already exists.
    username.required=Username is required.
    password.required=Password is required.
    portfolioName.required=Portfolio Name is required.
    

属性文件创建之后，在action中使用getText("property_name")方法即可引用资源属性文件中的本地化信息。

其次，ActionSupport实现了com.opensymphony.xwork2.LocaleProvider接口，它只提供了一个方法getLocale(),可通过浏览器发送来的地域语言信息获取用户所在的地域，从而判断使用那一种资源属性文件来国际化。

### 4.action传递域对象数据

action把请求中的数据放在简单的JavaBean对象上，这种做法很好很强大，但是当JavaBean对象足够复杂之后(如具有很多属性)，这种做法就不够强大了。既然框架提供了数据转移的机制，我们何不直接使用域对象进行数据转移呢？是的，框架允许我们这么做。

有两种实现方式可以做的这一点：

使用JavaBean对象，让数据直接传输到这个对象上 使用模型驱动(ModelDriven)的action

**Ⅰ.使用JavaBean方式**

    public String execute(){
        getPortfolioService().createAccount( user ); 
        return SUCCESS;
    }
    
    private User user; 
    
    public User getUser() {
        return user;
    }
    
    public void setUser(User user) {
        this.user = user;
    }
    
    public void validate(){
    
        if ( getUser().getPassword().length() == 0 ){           
            addFieldError( "user.password", getText("password.required") );
        }
        if ( getUser().getUsername().length() == 0 ){           
            addFieldError( "user.username", getText("username.required") );
        }
                ....
    }
    

后台代码变的简单了，前台代码就变复杂点了，就是这样无奈。

    <s:textfield name="user.name" label="姓名"/>
    

**Ⅱ使用模型驱动(ModelDriven)**

使用ModelDriven与使用JavaBean方法不同，它使用getModel()方法将域对象公开。它引入了新的接口和拦截器，但拦截器已经包含在默认拦截器栈中了，我们只需要实现getModel()方法即可。

    public class ModelDrivenRegister extends ActionSupport implements ModelDriven {
    
        public String execute(){//注意在execute方法中只能使用user本来的引用，而不能做任何将其
                                        //改变引用的操作
            getPortfolioService().createAccount( user ); 
            return SUCCESS;
        }
    
        private User user = new User();//注意使用模型驱动需要初始化模型对象
    
        public Object getModel() {
            return user;
        }
    
        public void validate(){
            PortfolioService ps = getPortfolioService();
    
            if ( user.getPassword().length() == 0 ){            
                addFieldError( "password", getText("password.required") );
            }
            ......
        }
    
        public PortfolioService getPortfolioService( )  {
            return new PortfolioService();
        }
    }
    

值得庆幸的是使用模型驱动的方式传递数据在视图层不需要做任何改变，这也许就是为什么一般选择使用模型驱动而不选择JavaBean方式的原因了。

**Ⅲ.使用域对象传递数据的潜在危险**

像上面那样使用域对象传递数据的做法很好，但是这样做的前提是要将域对象公开出去，无论是使用JavaBean方式还是模型驱动方式都是如此。但是无论如何将域对象整个的暴露给用户总不是一个很好的做法，它还存在一些潜在的威胁。比如说域对象中有一些敏感的数据你不想公开给外界；一个恶意的用户在请求中添加一个适当命名精心设计的查询语句，这样这个参数就会被自动写入到域对象的属性上。当然你可以进行适当的过滤，但是这样一来使用域对象的目的就变的无效了。这个问题现在也没有很好的解决方法，看的人自己想想，在实际的开发中记住这一点就行。