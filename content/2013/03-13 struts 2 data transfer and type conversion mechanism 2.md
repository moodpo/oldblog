# 复习笔记——Struts 2 的数据转移和类型转换机制(二)

- date: 2013-03-13
- category: Work
- tags: struts2,J2EE

----

本节将讲述Struts2内建的类型转换器的具体细节。通过配置，Struts2框架能够处理几乎所有你可能需要的类型转换。

### 1.内建的类型转换器

#### 1.1 立即可用的类型转换器

Struts2框架自带了对HTTP本地字符串和以下列出的Java类型之间转换的内建支持。

    ■ String—有时候字符串就是字符串。
    ■ boolean/Boolean—true和false字符串可以被转换为Boolean的原始类型和对象类型。
    ■ char/Character—原始类型或者对象类型。
    ■ int/Integer, float/Float, long/Long, double/Double—原始类型或者对象类型
    ■ Date—当前Locale的SHORT格式的字符串版本 (例如，12/10/97)。
    ■ array—每一个字符串元素必须能够转换为数组的类型。
    ■ List—默认情况下使用String填充。
    ■ Map—默认情况下使用String填充。
    

当框架定位到一个给定的OGNL表达式指向的Java属性时，它会查找这个类型的转换器。如果这个类型在前面的列表中，你不需要任何事情，等着接收数据即可。

#### 1.2 使用OGNL表达式从表单字段名映射到属性

下面将按照上述类型转换器列表讲述每一个内建的类型转换器如何在两端建立对等的内容。

**1.原始类型和包装类**

指向ValueStack上特定属性的OGNL表达式例子：

    <h4>Complete and submit the form to create your own portfolio.</h4>
    <s:form action="Register">
        <s:textfield name="user.username" label="Username"/>
        <s:password name="user.password" label="Password"/>
        <s:textfield name="user.portfolioName" label="Enter a name "/>
        <s:textfield name="user.age" label="Enter your age as a double "/>
        <s:textfield name="user.birthday" label="Enter birthday. (mm/dd/yy)"/>
        <s:submit/>
    </s:form>
    

以下是公开的User对象的JavaBean属性：

    private User user;
    public User getUser() {
        return user;
    }
    public void setUser(User user) {
        this.user = user;
    }
    

而User对象也公开了相应的属性：

    public class User {
        private String username;
        private String password;
        private String portfolioName;
        private Double age;
        private Date birthday;
        public String getPassword() {
            return password;
        }
        public void setPassword(String password) {
            this.password = password;
        }
        ......
    }
    

注意类型转换和验证之间的区别，以及类型转换错误和验证错误之间的区别。验证代码用来从动作的业务逻辑的视角验证数据是否是数据类型的一个合法实例，它通过validation拦截器或者workflow拦截器对validate()方法的调用而触发。类型转换在将HTTP字符串值绑定到Java类型时发生，如在param拦截器转移请求数据时发生。

类型转换错误导致用户会被返回到输入页面，并展示一个默认的错误信息告知用户，你可以自定义类型转换问题的错误报告，将在以后章节学习。

**2.处理多值请求参数**

多个参数值可以被映射到传入请求中的一个参数名，一个表单有多种方式在一个参数名下提交多个值，也有多种方法将它们映射到Java端的类型，这意味着使用OGNL有很多种方式。Struts2提供了丰富的支持以便将多值的请求参数转移到Java端各种面向集合的数据类型，从数组到真正的Collection。下面将通过展示两端的示例说明怎样使用它。

**3.数组**

为数据转移指向数组属性(使用索引和不使用索引)：

    <s:form action="ArraysDataTransferTest">
        <s:textfield name="ages" label="Ages"/>
        <s:textfield name="ages" label="Ages"/>
        <s:textfield name="ages" label="Ages"/>
    
        <s:textfield name="names[0]" label="names"/>
        <s:textfield name="names[1]" label="names"/>
        <s:textfield name="names[2]" label="names"/>
        <s:submit/>
    </s:form>
    

OGNL输入字段名指向的数组属性：

    private Double[] ages ;
    public Double[] getAges() {
        return ages;
    }
    public void setAges(Double[] ages) {
        this.ages = ages;
    }
    private String[] names = new String[10];
    public String[] getNames() {
        return names;
    }
    public void setNames(String[] names) {
        this.names = names;
    }
    

注意这些属性不需要带索引参数的获取方法和设置方法，OGNL处理所有与索引相关的细节，我们只需通过一对获取方法和设置方法公开数组。

请求转发的结果页面代码：

    <h5>Congratulations! You have transferred and converted data to and from Arrays.</h5>
    <h3>Age number 3 = <s:property value="ages[2]" /> </h3>
    <h3>Name number 3 = <s:property value="names[2]" /> </h3>
    

**3.List**

使用List与使用数组几乎完全相同，仅有的不同是在Java 5之前List不支持类型指定，List没有类型的特性对Struts 2的类型转换机制有着重要的影响。对List来说，没有方法能够自动发现内部元素的类型。使用List时用两种选择，为元素指定类型或者接受默认类型(String)。接受默认行为时页面代码和使用数组相同，不同的是在Java端，没有类型说明，List中元素都会是String类型。

有些时候，除了使用String，你会想为List中的元素指定类型。这种情况下，我们只需告知OGNL某个给定属性我们希望的元素类型，这使用一个简单的属性文件实现。

为了给动作对象上的List属性指定元素类型，我们根据下图中的命名约定创建一个文件。

![List指定类型命名约定.png][1]

文件名约定是：动作名+-+conversion+属性文件扩展名，其中conversion是类型转换用的文件名后缀。之后把这个文件放在Java包中这个类的旁边。文件的内容约定如下图：

![List指定类型内容约定.png][2]

文件内容约定是：Element+_+List类型属性的名字=元素类型(如：Element-weights=java.lang.Double)，这样List属性会像数组属性一样工作，每一个独立元素都会被转换为指定的元素类型。例如页面可以这样写：

    <s:textfield name="weights[0]" label="weights"/>
    <s:textfield name="weights[1]" label="weights"/>
    <s:textfield name="weights[2]" label="weights"/>
    

当然也可以不使用索引，这取决于你的偏好，在Java端，动作对象上的属性没有变化。需要记住的一点就是在给List或者其他Collection指定类型时注意不要预先初始化List。

    private List weights;
    public List getWeights() {
        return weights;
    }
    public void setWeights(List weight) {
        this.weights = weight;
    }
    

下面是一个更全的功能的示例

    页面代码：
    <s:textfield name="users[0].username" label="Usernames"/>
    <s:textfield name="users[1].username" label="Usernames"/>
    <s:textfield name="users[2].username" label="Usernames"/>
    
    动作属性代码：
    private List users ;
    public List getUsers(){
        return users;
    }
    public void setUsers ( List users ) {
        this.users=users;
    }
    
    指定类型的属性文件内容：
    Element_users=manning.utils.User
    

**5.Map**

Map使用关键字而非索引来关联到它的值，使用关键字的特性对Struts2类型转换过程有两点含义。一是引用它们的OGNL表达式语法与List不同；而是与为Map属性指定类型相关。对Map来说，可以为关键字对象指定类型，如果不指定默认是String。

首先看一下页面代码，Java端代码和List的相同：

    <s:textfield name="maidenNames.mary" label="Maiden Name"/>
    <s:textfield name="maidenNames.jane" label="Maiden Name"/>
    <s:textfield name="maidenNames.hellen" label="Maiden Name"/>
    <s:textfield name="maidenNames['beth']" label="Maiden Name"/>
    <s:textfield name="maidenNames['sharon']" label="Maiden Name"/>
    <s:textfield name="maidenNames['martha']" label="Maiden Name"/>
    

现在在看一下指定类型的示例：

    指定类型属性文件内容
    Element_myUsers=manning.utils.User
    
    页面代码：
    <s:textfield name="myUsers['chad'].username" label="Usernames"/>
    <s:textfield name="myUsers['jimmy'].username" label="Usernames"/>
    <s:textfield name="myUsers['elephant'].username" label="Usernames"/>
    <s:textfield name="myUsers.chad.birthday" label="birthday"/>
    <s:textfield name="myUsers.jimmy.birthday" label="birthday"/>
    <s:textfield name="myUsers.elephant.birthday" label="birthday"/>
    
    Java端代码：
    private Map myUsers ;
    public Map getMyUsers(){
        return myUsers;
    }
    public void setMyUsers ( Map myUsers ) {
        this.myUsers=myUsers;
    }
    

前面已经说了，除了为元素指定类型，使用Map属性时也可以为关键字对象指定类型。与值一样，OGNL也会把参数的名字当作应该尝试转换为指定类型的字符串。假如我们想使用Integer作为关键字的myUsers版本。应该在属性文件内这样写：

    Key_myOrderedUsers=java.lang.Integer
    Element_myOrderedUsers=manning.utils.User
    

以下是提交表单页面：

    <s:textfield name="myOrderedUsers['1'].birthday" label="birthday"/>
    <s:textfield name="myOrderedUsers['2'].birthday" label="birthday"/>
    <s:textfield name="myOrderedUsers['3'].birthday" label="birthday"/>
    

最后，如果使用Java5或更高版本，可以使用泛型来类型化Collection和Map，在类型转换时Struts2可以使用这些信息，使得属性文件配置不再必要。

### 2.自定义类型转换

虽然内建的类型转换器功能强大且涵盖面广，但有时我们还是需要构建自定义的类型转化器。你可以指定一个转换逻辑将任何字符串翻译为任何Java类型。唯一需要做的就是创建字符串语法和对应的Java类，之后是使用一个类型转换器把他们连接起来。

#### 2.1 实现类型转换器

类型转换器是OGNL的一部分，因此类型转换器必须实现ognl.TypeConverter接口。一般情况下，OGNL的类型转换器能实现任何两种数据类型之间的转换。在Web应用程序领域，我们只需实现所有Java类型和HTTP字符串之间的转换即可。

Struts2提供了一个方便的基类：org.apache.struts2.util.StrutsTypeConverter作为自定义类型转换器的扩展点。下面代码是这个类所定义的抽象方法：

    public abstract Object convertFromString(Map context, String[] values,Class toClass);
    public abstract String convertToString(Map context, Object o);
    

第一个方法经请求转换为Java对象，第二个方法将Java对象转换为请求字符串。当你编写一个自定义的转换器时，你只需继承这个类并将自己的逻辑填充到这两个方法中即可。

#### 2.2 在字符串和Java对象间转换

![自定义类型转换逻辑.png][3]

以下代码实现了上图所示的转换规则：

    public class CircleTypeConverter extends StrutsTypeConverter {
        public Object convertFromString(Map context, String[] values,Class toClass) {
            String userString = values[0];
            Circle newCircle = parseCircle ( userString );
            return newCircle;
        }
        public String convertToString(Map context, Object o) {
            Circle circle = (Circle) o;
            String userString = "C:r" + circle.getRadius();
            return userString;
        }
        private Circle parseCircle( String userString ) throws TypeConversionException {
            Circle circle = null;
            int radiusIndex = userString.indexOf('r') + 1;
            if (!userString.startsWith( "C:r") )
                throw new TypeConversionException ( "Invalid Syntax");
            int radius;
            try {
                radius = Integer.parseInt( userString.substring( radiusIndex ) );
            } catch ( NumberFormatException e ) {
                throw new TypeConversionException ( "Invalid Value for Radius"); 
            }
            circle = new Circle();
            circle.setRadius( radius );
            return circle;
        }
    }
    

#### 2.3 配置框架使用自定义转换器

构造了自定义转换器之后，必须让框架知道什么时候、在哪里使用它。这里有两种选择，可以配置转换器用在一个给定动作的局部或者全局。前者将只告诉框架在处理特定动作的特定属性时才使用转换器，而后者在应用程序的任何地方，每次通过OGNL设置或者取得一个特定属性时都会使用它。

**1.属性专用**

与前面Map和List指定元素类型相同，在同样的文件中再添加以下内容：

    circle=manning.utils.CircleTypeConverter
    

这一行简单的内容将属性名circle和类型转换器关联起来。现在当OGNL想给Circle属性设置值时，它会自动使用我们定义的类型转换器。

**2.全局类型转化**

我们可以指定所有的Circle类型的属性都使用我们的转换器，这个过程和之前局部指定仅有一点不同：在定义全局类型转换器时，不使用动作名+-+conversion+属性文件扩展名的约定文件，而是使用xwork-conversion.properties，并且文件内容修改为如下：

    manning.utils.Circle=manning.utils.CircleTypeConverter
    

此外，还需将xwork-conversion.properties文件放在类路径下，例如在WEB-INF/classess/中。


 [1]: /media/2013/03/344633058.png "List指定类型命名约定.png"
 [2]: /media/2013/03/879049580.png "List指定类型内容约定.png"
 [3]: /media/2013/03/2746878052.png "自定义类型转换逻辑.png"