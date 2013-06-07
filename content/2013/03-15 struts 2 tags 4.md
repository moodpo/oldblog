# 复习笔记——Struts 2 标签（四）

- date: 2013-03-14
- category: Work
- tags: struts2, tag, J2EE

----

我们已经从宏观上介绍了UI组件架构，现在开始学习使用UI组件标签。首先从总结所有UI组件都使用的属性和方法开始。

### 1.通用属性

所有Struts 2 UI组件标签通用的属性有很多，大部分是底层HTML元素公开的很多属性，这里我们重点关注Struts 2标签最核心的使用。通常情况下，你可以假定Struts 2标签支持底层HTML元素的所有属性。

下面是这些常用通用属性的一个列表，如果属性的数据类型是String，那么属性值会被当作字符串字面值解析，除非使用%{expression}标记；所有非String类型都会被作为OGNL表达式自动求值。

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
    </td>
    
    <td>
      主题
    </td>
    
    <td>
      数据类型
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
      simple
    </td>
    
    <td>
      String
    </td>
    
    <td>
      设置表单输入元素的name属性。如果没有手工设置value属性，那么也会被传播到组件的value属性。组件自己使用name属性指向ValueStack上的属性作为提交的请求参数值的目标
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      simple
    </td>
    
    <td>
      Object
    </td>
    
    <td>
      指向ValueStack上的属性的OGNL表达式，用来为预填充设置表单输入元素的值。默认为name属性设置的值
    </td>
  </tr>
  
  <tr>
    <td>
      key
    </td>
    
    <td>
      simple
    </td>
    
    <td>
      String
    </td>
    
    <td>
      从ResourceBundle取得本地化的标签(label)，可以传播到name属性，也可以传播到value属性
    </td>
  </tr>
  
  <tr>
    <td>
      label
    </td>
    
    <td>
      XHTML
    </td>
    
    <td>
      String
    </td>
    
    <td>
      为组件创建一个HTML标签(label)，如果设定使用key属性和本地化的文本就不需要指定这个属性
    </td>
  </tr>
  
  <tr>
    <td>
      labelPosition
    </td>
    
    <td>
      XHTML
    </td>
    
    <td>
      String
    </td>
    
    <td>
      元素标签(label)的位置，可选的值是left或者top
    </td>
  </tr>
  
  <tr>
    <td>
      required
    </td>
    
    <td>
      XHTML
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      如果设定为true，那么标签(label)旁边会出现一个星号来暗示这是一个必须的字段。默认情况下，如果一个字段级别的数据验证器通过name属性与这个输入字段关联时，这个值为true
    </td>
  </tr>
  
  <tr>
    <td>
      id
    </td>
    
    <td>
      simple
    </td>
    
    <td>
      String
    </td>
    
    <td>
      HTML的id属性。如果id属性没有指定，组件会创建一个唯一标识。
    </td>
  </tr>
  
  <tr>
    <td>
      cssClass
    </td>
    
    <td>
      simple
    </td>
    
    <td>
      String
    </td>
    
    <td>
      HTML的class属性
    </td>
  </tr>
  
  <tr>
    <td>
      cssStyle
    </td>
    
    <td>
      simple
    </td>
    
    <td>
      String
    </td>
    
    <td>
      HTML的style属性
    </td>
  </tr>
  
  <tr>
    <td>
      disabled
    </td>
    
    <td>
      simple
    </td>
    
    <td>
      String
    </td>
    
    <td>
      HTML的disablled属性
    </td>
  </tr>
  
  <tr>
    <td>
      tabindex
    </td>
    
    <td>
      simple
    </td>
    
    <td>
      String
    </td>
    
    <td>
      HTML的tabindex属性
    </td>
  </tr>
  
  <tr>
    <td>
      theme
    </td>
    
    <td>
      N/A
    </td>
    
    <td>
      String
    </td>
    
    <td>
      在哪个主题下呈现这个组件，默认是xhtml
    </td>
  </tr>
  
  <tr>
    <td>
      templateDir
    </td>
    
    <td>
      N/A
    </td>
    
    <td>
      String
    </td>
    
    <td>
      用来覆盖从中取出模板的默认目录名
    </td>
  </tr>
  
  <tr>
    <td>
      template
    </td>
    
    <td>
      N/A
    </td>
    
    <td>
      String
    </td>
    
    <td>
      用来呈现UI标签的模板。
    </td>
  </tr>
</table>

除了上述属性，组件也支持常用的JavaScript事件处理属性，例如onclick和onchange。

### 2.简单组件

**1.head组件**

这个标签什么事也做不了，但是在支持其他标签方面它起了重要作用。如果某个标签依赖head标签引入的资源，那么省去head标签，这个标签就不能正常工作。下面是使用方法：

    <head>
        <title>Portfolio Registration</title>
        <s:head/>
    </head>
    

下面是它生成的内容：

    <link rel="stylesheet" href=" . . . styles.css" type="text/css"/>
    <script language="JavaScript" type="text/javascript" src=". . .dojo.js"/>
    <script language="JavaScript" type="text/javascript" src="dojoRequire.js"/>
    

**2.form组件**

form组件也许是这些组件中最重要的组件，它提供了重要的切入点，form组件是指向Struts 2 动作的表单。除了常用属性外，form组件还使用下面的一些属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      action
    </td>
    
    <td>
      String
    </td>
    
    <td>
      表单提交的目标，可以是Struts2动作的名字或者URL
    </td>
  </tr>
  
  <tr>
    <td>
      namespace
    </td>
    
    <td>
      String
    </td>
    
    <td>
      Struts2命名空间，在这个命名空间中查找命名动作(action属性)，或者从这里开始构建URL。默认为当前命名空间
    </td>
  </tr>
  
  <tr>
    <td>
      method
    </td>
    
    <td>
      String
    </td>
    
    <td>
      与HTML form属性相同，默认为POST
    </td>
  </tr>
  
  <tr>
    <td>
      target
    </td>
    
    <td>
      String
    </td>
    
    <td>
      与HTML form属性相同
    </td>
  </tr>
  
  <tr>
    <td>
      enctype
    </td>
    
    <td>
      String
    </td>
    
    <td>
      上传文件时设置为multipart/form-data
    </td>
  </tr>
  
  <tr>
    <td>
      validate
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      与验证框架一起使用，打开客户端JavaScript验证
    </td>
  </tr>
</table>

下面是使用的示例：

    <s:form action="Login">
        ......
    </s:form> 
    

会生成下面的代码：

    <form id="Login" name="Login" onsubmit="return true;"
        action="/manningSampleApp/chapterSeven/Login.action" method="POST">
    ......
    </form>
    

决定如何创建URL时需要遵循的步骤如下所述

- 如果没有指定action属性，则会重新指向生成当前页面的动作
- 如果指定了action属性，它首先被当作Struts 2动作解析。如果没有指定namespace属性，那么会在当前请求的命名空间中查找这个动作。如果指定了namespace属性，那么会在指定的命名空间中查找这个动作。注意，指定的动作没有.action的扩展名。
- 如果action属性中指定的值不是声明性架构中的Struts 2动作，那么这个值会被用来直接构建URL。如果指定的字符串以一个斜线(/)开头，那么它被假定为ServletContext相关，框架会在它之前加上ServletContext的庐江构建一个URL。如果指定的字符串没有已斜线开头，那么这个值会被直接当作URL。注意这种情况下，即使指定了namespace属性也不会使用。
    

下面是使用namespace的示例：

    <s:form action="Login" namespace="/chapterFour">
    

会生成下面的代码：

    <form id="Login" name="Login" onsubmit="return true;"
    action="/manningSampleApp/chapterFour/Login.action" >
    

下面是一个使用不映射动作的例子：

    <s:form action="/chapterSeven/PortfolioHomePage.jsp">
    

生成代码如下：

    <form id="PortfolioHomePage" onsubmit="return true;"
    action="/manningSampleApp/chapterSeven/PortfolioHomePage.jsp">
    

最后，如果你指定一个不是以斜线开始的值，并且这个值也不是一个动作，也不是一个完整的URL，那么这个值会被原封不动的输出到action属性中。下面是示例：

    <s:form action="MyResource">    
    

这是输出：

    <form id="MyResource" onsubmit="return true;" action="MyResource">
    

**3.textfield组件**

这个组件生成了无处不在的文本输入字段。除了通用属性外，textfield还经常使用一些自己的属性，如下：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      maxlength
    </td>
    
    <td>
      String
    </td>
    
    <td>
      字段数据的最大长度
    </td>
  </tr>
  
  <tr>
    <td>
      readonly
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      如果为true,字段不可编辑
    </td>
  </tr>
  
  <tr>
    <td>
      size
    </td>
    
    <td>
      String
    </td>
    
    <td>
      可视长度
    </td>
  </tr>
</table>

**4.password组件**

password标签实质上很像textfield标签，除通用属性外，它还经常使用如下属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      maxlength
    </td>
    
    <td>
      String
    </td>
    
    <td>
      字段数据的最大长度
    </td>
  </tr>
  
  <tr>
    <td>
      readonly
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      如果为true字段不可编辑
    </td>
  </tr>
  
  <tr>
    <td>
      size
    </td>
    
    <td>
      String
    </td>
    
    <td>
      文本字段的可视长度
    </td>
  </tr>
  
  <tr>
    <td>
      showPassword
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      如果设置为true，并且ValueStack中对应的属性有值，那么密码会预填充。默认为false。被填充的值会被伪装。
    </td>
  </tr>
</table>

**5.textarea组件**

textarea标签生成一个基于HTML的textarea元素的组件。除了通用属性外，它还有一些其他属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      cols
    </td>
    
    <td>
      Integer
    </td>
    
    <td>
      列数
    </td>
  </tr>
  
  <tr>
    <td>
      rows
    </td>
    
    <td>
      Integer
    </td>
    
    <td>
      行数
    </td>
  </tr>
  
  <tr>
    <td>
      readonly
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      如果为true字段不可编辑
    </td>
  </tr>
  
  <tr>
    <td>
      wrap
    </td>
    
    <td>
      String
    </td>
    
    <td>
      指定textarea中的内容是否应该被包装起来
    </td>
  </tr>
</table>

**6.checkbox组件**

checkbox组件使用单个HTML的checkbox元素创建一个Boolean型的组件。注意，这个组件与HTML的checkbox组件不是完全相同，它只能指定Boolean类型的值，与这个标签绑定的Java端的属性必须是Boolean类型的。除了之前的通用属性，checkbox组件也使用下列属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      fieldValue
    </td>
    
    <td>
      String
    </td>
    
    <td>
      被checkbox提交的哦真实值，可能是false或者true，默认是true
    </td>
  </tr>
  
  <tr>
    <td>
      value
    </td>
    
    <td>
      String
    </td>
    
    <td>
      与fieldValue一起使用，决定checkbox是否被选中。如果fieldValue=true，并且value=true，那么组件被选中
    </td>
  </tr>
</table>

预填充通常使用UI组件的value属性完成，对于checkbox组件来说要复杂一些。我们必须指定字段要提交的值以及Java端属性的当前值。在checkbox组件的上下文中他们是不同的值，而在textfield标签中是相同的。

### 3.基于集合的组件

Struts 2 标签支持数组、Map或者List集合类型。集合支持的组件的金币嗯逻辑是Java端的数据以一系列选项的方式呈现给用户，之后用户选择一个选项，这个值随着请求被提交。

**1.select组件**

select组件可能是最常见的基于几何的UI组件，在Java Web应用程序中，通常利用Collection、Map或者数组中的数据构建这一系列选项。

以下代码展示了select UI组件最简单的用例：

    <s:select name="user.name" list="{'Mike','Payal','Silas'}" />
    

select组件的list属性指向了为这个组件提供数据的数据集。通常情况下会使用一个OGNL表达式指向ValueStack上的一系列的数据，而不是用字面值生成的列表。但这里在介绍这个标签时我们尽量简化它的使用。以下代码是上述标签呈现的HTML标记：

    <select name="user.name" id="ViewPortfolio_user_name">
        <option value="Mike">Mike</option>
        <option value="Payal">Payal</option>
        <option value="Silas">Silas</option>
    </select>
    

列表中的每一个值都用来创建一个对应的option元素。但是如何预填充呢？由于没有指定value属性，所以会从name属性推断出value属性。如果标签呈现的过程中ValueStack的user.name属性包含了一个值，那么这个值会与所有option元素中的value属性逐一匹配，知道找到相同的值，并将其预选中。上面的情况没有一个选项被预选中，因此user.name属性是没有包含值的。

以下是select UI组件特有的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      lisst
    </td>
    
    <td>
      Collection\ Map\Array\Iterrator
    </td>
    
    <td>
      用来生成下拉列表选项的数据集合
    </td>
  </tr>
  
  <tr>
    <td>
      listKey
    </td>
    
    <td>
      String
    </td>
    
    <td>
      当list中的元素是复杂类型时，用来指定用作被提交的值的list元素的属性，默认是key
    </td>
  </tr>
  
  <tr>
    <td>
      listValue
    </td>
    
    <td>
      String
    </td>
    
    <td>
      当列表中的元素是复杂类型时，用来指定用作选项内容的list元素的属性。换句话说就是用户看到的字符串。默认是value
    </td>
  </tr>
  
  <tr>
    <td>
      headerKey
    </td>
    
    <td>
      String
    </td>
    
    <td>
      与标题一起使用，如果用户选择了标题，那么指定提交的值
    </td>
  </tr>
  
  <tr>
    <td>
      headerValue
    </td>
    
    <td>
      String
    </td>
    
    <td>
      作为list的标题呈现给用户，例如“States”、“Countries”
    </td>
  </tr>
  
  <tr>
    <td>
      emptyOption
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      与标题一起使用，在标题和真实选项之间放置一个空的间隔选项
    </td>
  </tr>
  
  <tr>
    <td>
      multiple
    </td>
    
    <td>
      Boolean
    </td>
    
    <td>
      用户可以选择多个值
    </td>
  </tr>
  
  <tr>
    <td>
      size
    </td>
    
    <td>
      String
    </td>
    
    <td>
      一次显示的选项个数
    </td>
  </tr>
</table>

下面是select标签的一个比较复杂的用例：

    <h5>Browse an Artist's Portfolios ( Demo of select component. )</h5>
    <s:form action="SelectPortfolio" >
        <s:select name="username" list='users' listKey="username"
            listValue="username" label="Select an artist" />
        <s:submit value="Browse"/>
    </s:form>
    

两个新的属性listKey和listValue是必须的，因为与我们展示的第一个例子不同，支持这个select组件的集合中的数据是复杂的User类型而不是简单的String。listKey属性将对象的一个属性与生成的option元素的value属性绑定，这个属性决定了选项被选中时提交的请求参数值。这里只需一个唯一的标识符即可，显然使用username属性非常适合(已假定username是唯一的)。listValue属性决定了UI下拉列表中用户看到的内容。它相当于生成的option元素包含的字符串。

下面是上述标签生成的HTML代码：

    <form id="SelectPortfolio" name="SelectPortfolio"
            action="/manningSampleApp/chapterSeven/SelectPortfolio.action" >
        <select name="username" id="SelectPortfolio_username">
            <option value="Jimmy">Jimmy</option>
            <option value="Chad">Chad</option>
            <option value="Mary">Mary</option>
        </select>
        <input type="submit" id="SelectPortfolio_0" value="Browse"/>
    </form>
    

下面展示了用Map类型的数据实现同样功能的代码：

    <h5>Browse an Artist's Portfolios ( Demo of select component. )</h5>
    <s:form action="SelectPortfolio" >
        <s:select name="username" list='users' listValue="value.username"
            label="Select an artist" />
        <s:submit value="Browse"/>
    </s:form>
    

需要注意的是遍历Map数据时，遍历的是类型为Entry的对象，而Entry有两个属性key和value，这正好是listKey和listValue的默认值，所以在这里无需指定listKey属性的值，而Entry的value属性对应的是User对象，因此listValue使用value.username的方式从中取的username属性的值。

**2.radio组件**

radio组件提供了很多与select组件相似的功能，但呈现方式不同。下面是除通用属性外，这个组件的一些常用属性列表：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      list
    </td>
    
    <td>
      Collection\ Map\Array\Iterrator
    </td>
    
    <td>
      用来生成单选项的数据集合
    </td>
  </tr>
  
  <tr>
    <td>
      listKey
    </td>
    
    <td>
      String
    </td>
    
    <td>
      集合的元素的属性，用来作为提交的值，默认是key
    </td>
  </tr>
  
  <tr>
    <td>
      listValue
    </td>
    
    <td>
      String
    </td>
    
    <td>
      集合的元素的属性，用来作为选项的内容，换句话说是用户看到的字符串。默认是value
    </td>
  </tr>
</table>

下面是这个组件在页面上的显示：

![radio tag.png][1]

这个标签的使用几乎与select组件完全相同，因此我们不会讲解这个组件的详细内容。

**3.checkboxlist组件**

checkboxlist组件与select组件也很相似。如下图所示，它呈现了相同的选项，但使用了多选框，因此它允许选择多个选项。

![checkboxlist tag.png][2]

此外，checkboxlist组件还经常使用下面的属性：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      属性
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
      list
    </td>
    
    <td>
      Collection\ Map\Array\Iterrator
    </td>
    
    <td>
      用来生成复选项的数据集合
    </td>
  </tr>
  
  <tr>
    <td>
      listKey
    </td>
    
    <td>
      String
    </td>
    
    <td>
      集合的元素的属性，用来作为提交的值，默认是key
    </td>
  </tr>
  
  <tr>
    <td>
      listValue
    </td>
    
    <td>
      String
    </td>
    
    <td>
      集合的元素的属性，用来作为选项的内容，换句话说是用户看到的字符串。默认是value
    </td>
  </tr>
</table>

**4.预选中基于集合的组件**

value属性指向了ValueStack上的一个属性，这个属性会用作组件的当前值预先选择一个选项。通常不设置value属性，而是让框架根据name属性推断出value属性。每一个HTML的选项都有一个value属性，这个属性表示这个选项被选中时整个组件的值。巧妙的地方是框架使用Struts2标签的value属性的值来匹配某一个选项的值，在匹配时，这个选项被选中。

例如下面的代码，其中defaultUsername中包含了后台传入的默认的用户名chad：

    <s:form action="SelectPortfolio" >
        <s:radio name="username" list='users' value="defaultUsername"
                listKey="username" listValue="username" label="Select an artist" />
        <s:submit value="Browse"/>
    </s:form>
    

以上代码最终会输出下面的HTML代码：

    <input type="radio" name="username" value="Jimmy"/>
    <input type="radio" name="username" value="Charlie Joe"/>
    <input type="radio" name="username" checked="checked" value="Chad"/>
    <input type="radio" name="username" value="Mary"/>
    

相同的预选中的过程同样适用于所有基于集合的组件，如果是多重选中的情况，value属性可以指向一个包含多个选择的属性，例如一个包含用户名的数组，那么这个数组会选中这些值中的每一个。

### 4.额外的组件

**1.label组件**

label(标签)组件不应该与前面讲解的UI组件生成的标签(label)相混淆。label组件在HTML只输出只读文本，很像一个只读的textfield组件。下面是一个用例：

    <s:label name="username" label="Username" />
    

**2.hidden组件**

hidden组件用来是实现隐藏域，以下是使用的标记及输出的HTML：

    <s:hidden name="username" />
    
    <input type="hidden" name="username" value="Chad"
              id="UpdateAccount_username"/>
    

**3.doubleselect组件**

doubleselect(关联下拉列表)组件可实现下拉列表的联动效果，下面是常见的使用方法，其中protfolios即使User对象的属性，而它本神又是一个集合；

    <h4>Select a portfolio to view.</h4>
    <s:form action="ViewPortfolio">
        <s:doubleselect name="username" list='users' listKey="username"
                listValue="username" doubleName="portfolioName"
                doubleList="portfolios" doubleListValue="name" />
        <s:submit value="View"/>
    </s:form>

 [1]: /media/2013/03/2146681631.png "radio tag.png"
 [2]: /media/2013/03/319284732.png "checkboxlist tag.png"