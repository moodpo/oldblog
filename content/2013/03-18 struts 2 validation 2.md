# 复习笔记——Struts 2 的验证框架(二)

- date: 2013-03-18
- category: Work
- tags: struts2, note

----

前一节中我们介绍了Struts2的验证框架及怎样自定义验证器，本节将介绍一些使用验证框架的高级主题。一些高级主题只研究了验证机制的细微差别，而另一些高级主题则展示了如何将验证框架应用到一些更特殊的Struts 2开发模式。这些内容将涵盖验证器的继承，将验证映射到域模型对象而不是动作，验证器失败时短路跳出验证等。

### 1. 在域对象级别验证

在域对象级别是指动作把整个User对象作为JavaBean属性公开出来。在域对象级别验证，首先需要做的是定义元数据，以下是User-validation.xml文件的全部内容：

    <!DOCTYPE validators PUBLIC "-//OpenSymphony Group//XWork Validator 1.0.2//EN"
           "http://www.opensymphony.com/xwork/xwork-validator-1.0.2.dtd">
    <validators>
      <field name="password">
          <field-validator type="stringlength">
             <param name="maxLength">10</param>
             <param name="minLength">6</param>
             <message>Your password should be 6-10 characters.</message>
          </field-validator>
          <field-validator type="passwordintegrity">
              <param name="specialCharacters">$!@#?</param>
              <message>Your password must contain one letter, one number, and one of the following "${specialCharacters}".</message>
          </field-validator>
      </field>
      <field name="username">
          <field-validator type="stringlength">
             <param name="maxLength">8</param>
             <param name="minLength">5</param>
             <message>While ${username} is a nice name, a valid username must be between ${minLength} and ${maxLength} characters long. </message>
         </field-validator>
      </field>
      <field name="email">
          <field-validator type="requiredstring">
              <message>You must enter a value for email.</message>
          </field-validator>
           <field-validator type="email">
             <message key="email.invalid"/>
          </field-validator>
       </field>
      <validator type="expression">
          <param name="expression">username != password</param>
          <message>Username and password can't be the same.</message>
      </validator>
    </validators>
    

这个文件与之前的Register-validation.xml文件完全相同，但是我们需要把它放在User类同级目录下。User验证数据就绪后，我们需要把使用User的动作和这些验证元数据连接起来。这由visitor验证器完成。以下代码片段展示了UpdateAccount-validation.xml文件的简要内容，此文件将放在与UpdateAccount动作相同目录下：

    <!DOCTYPE validators PUBLIC "-//OpenSymphony Group//XWork Validator 1.0.2//EN"
           "http://www.opensymphony.com/xwork/xwork-validator-1.0.2.dtd">
    <validators>
       <field name="user">
          <field-validator type="visitor">
             <message>User:  </message>     
          </field-validator>    
      </field>
    </validators>
    

这个文件仅仅使用visitor验证器将验证细节全部转交给在这个指定的字段名的类上定义的验证元数据，即指定了user字段。visitor验证器使用这个信息找到User-validation.xml并使用这个文件描述的验证逻辑验证user属性上所有的数据。而在message元素上我们只定义了验证产生的错误信息的前缀。

**ModelDriven动作中使用验证框架**

当动作实现ModelDriven接口时，也可以使用上述技术，对于UpdateAccount-validation.xml文件我们只需要做一下轻微的改变：

    <validators>
       <field name="model">
          <field-validator type="visitor">
            <param name="appendPrefix">false</param>
            <message>User:  </message>      
          </field-validator>    
      </field>
    </validators>
    

做了两处变更，首先，由于ModelDriven动作的域对象通过getModel()获取方法公开出来，所以现在我们需要改变指向model的字段名。其次，我们需要添加appendPrefix参数告诉visitor验证器不需要在字段前添加user前缀来获取字段值。

### 2. 使用验证上下文优化验证

有时候可能需要一种更加细粒度的对什么时候运行哪些验证的控制级别，为了控制这些内容，验证框架引入了上下文的概念。数据验证上下文提供了一种简单的方法来识别我们想验证的数据，在使用这些数据的应用程序中的具体位置。

如果一个动作类中包含了多个方法作为动作的路口点时，我们就需要使用验证上下文。具体的做法是为这几个方法分别建立遵循：类名+-+方法名+-+validation.xml这个规则的文件，并在文件中定义验证逻辑。如果你仍然定义了一个类名+-+validation.xml的文件，那么这个文件中定义的验证也会被加载。

与visitor验证器和域对象一起使用验证上下文时，方法与此相同，比如只需在User类所在的目录下定义类名+-+方法名+-+validation.xml的文件分别添加验证规则即可。

显然这种方式很可能会发生冲突而不适合，幸运的是，我们可以在visitor验证器中使用context属性定义一个上下文名称来替换之前文件名中的方法名。使用如下：

    <field name="user">
        <field-validator type="visitor">
            <param name="context">admin</param>
            <message>User: </message>
        </field-validator>
    </field>
    

于是我们就可以使用User-admin-validation.xml的名称命名文件而避免冲突。

### 3. 验证继承

我们已经知道验证可以被声明在不同级别的不同上下文中，现在我们简要的描述一下验证声明的继承。下面展示了当框架开始处理时收集验证文件位置的顺序：

*   SuperClass-validation.xml
*   SuperClass-aliasName-validation.xml
*   Interface-validation.xml
*   Interface-aliasName-validation.xml
*   ActionClass-validation.xml
*   ActionClass-aliasName-validation.xml

在定义验证时，应该基于这个结构在这个搜索列表的更高层定义通用的验证，这样允许你重用这些定义。

### 4. 验证短路效应

验证框架的一个有用特性是当一个给定的验证失败时，它能够像短路一样停止后续验证。下面是一个用例：

    <field name="password">
        <field-validator type="stringlength" short-circuit="true">
            <param name="maxLength">10</param>
            <param name="minLength">6</param>
            <message>Your password should be 6-10 characters.</message>
        </field-validator>
        <field-validator type="passwordintegrity">
            <param name="specialCharacters">$!@#?</param>
            <message>Your password must contain one letter, one number, and one of the following "${specialCharacters}".
            </message>
        </field-validator>
    </field> 
    

这里我们追加的唯一内容是short-circuit属性，把它设置为true。这样做的目的是想在stringlength检查失败的情况下不让passwordintegrity检查运行。注意，虽然这个short-circuit定义在一个字段验证器上，但是这个字段剩余的验证都会成为短路的。如果在动作级别定义短路，那么所有的验证都将成为短路的。

### 5. 使用注解声明验证

使用注解声明验证和使用XML没有什么不同，下面是一个完整示例：

    @Validation
    
    public class RegisterValidationAnnotated extends ActionSupport implements SessionAware {
    
        @ExpressionValidator(expression = "username != password", message = "Username and password can't be the same.")
        public String execute(){
            /*
             * Create and move the data onto our application domain object, user.
             */
            User user = new User();
            user.setPassword( getPassword() );
            Portfolio newPort = new Portfolio();
            newPort.setName( getPortfolioName() );
            user.getPortfolios().add( newPort );
            user.setUsername( getUsername() );
    
            /* Login the newly created user */
            getPortfolioService().persistUser( user );
            session.put( Struts2PortfolioConstants.USER, user );
    
            return SUCCESS;
        }
    
        /* JavaBeans Properties to Receive Request Parameters */
        private String username;
        private String password;
        private String portfolioName;
        private boolean receiveJunkMail;
        private String email;
    
        @RequiredStringValidator(type = ValidatorType.FIELD, message="Email is required.")
        @EmailValidator(type = ValidatorType.FIELD, key="email.invalid", message="Email no good.")
        public void setEmail(String email) {
            this.email = email;
        }
        public String getEmail() {
            return email;
        }
    
        @RequiredStringValidator(type = ValidatorType.FIELD, message = "Portfolio name is required.")
        public String getPortfolioName() {
            return portfolioName;
        }
        public void setPortfolioName(String portfolioName) {
            this.portfolioName = portfolioName;
        }
    
        @StringLengthFieldValidator(type = ValidatorType.FIELD, minLength="5" , maxLength = "8",  message = "Password must be between ${minLength} and ${maxLength} characters.")
        @RequiredStringValidator(type = ValidatorType.FIELD, message = "Password is required.")
        public String getPassword() {
            return password;
        }
        public void setPassword(String password) {
            this.password = password;
        }
    
        @RequiredStringValidator(type = ValidatorType.FIELD, message = "Username is required.")
        @StringLengthFieldValidator(type = ValidatorType.FIELD, minLength="5" , maxLength = "8",  message = "Username must be between ${minLength} and ${maxLength} characters.")
        public String getUsername() {
            return username;
        }
        public void setUsername(String username) {
            this.username = username;
        }
    
        public boolean isReceiveJunkMail() {
            return receiveJunkMail;
        }
        public void setReceiveJunkMail(boolean receiveJunkMail) {
            this.receiveJunkMail = receiveJunkMail;
        }
    
        private PortfolioServiceInterface portfolioService;
    
        public PortfolioServiceInterface getPortfolioService( )     {
    
            return portfolioService;
    
        }
    
        public void setPortfolioService( PortfolioServiceInterface portService){
            portfolioService = portService;
        }
    
        private Map session;
    
        public void setSession(Map session) {
    
            this.session = session;
        }
    
    }
    

一个细微的不同是消息处理方式。message属性是这些注解必须的属性。如果想使用来自于属性文件资源包的消息，你不需要把关键字作为message属性的值。相反，向注解添加key属性即可，但你仍然需要在message中指定消息的值，当key查找失败时，将使用它作为默认消息。