# 复习笔记——Struts 2 的国际化

- date: 2013-03-19
- category: Work
- tags: struts2, 笔记

----

本节我们将快速浏览Struts 2的国际化机制，重点在于怎样使用国际化方法满足我们的各种需求，而不深究其内部详细的原理。

### 1. Struts 2 框架和Java i18n

Java平台早已内置了对i18n的支持，Struts 2提供了一个高层的、极其方便的本地Java对i18n支持的封装。首先我们先简要介绍下Java的基础概念。

#### 1.1 使用ResourceBundle和Locale取得本地化文本

**1. 在ResourceBundle中存储资源**

Java的ResourceBundle是一个抽象类，其子类的实现管理包中包含了资源。ResourceBundle的子类能按照它们喜欢的任意方式管理它们的资源。Java平台提供了两个方便的子类供你使用，其中最常用的是PropertyResourceBundle，这个类从普通文本属性文件中加载资源，这些资源包括了各种版本内容的文件。

**2. 使用本地Java ResourceBundle**

如果我们想去的一个本地化文本，必须先获取对包含这个文本的包的引用。以下代码展示了这个功能：

    Locale currentLocale = new Locale( "tr");
    ResourceBundle myMessages = ResourceBundle.getBundle("EmailClientMessages",currentLocale);
    String greetingLabel = myMessages.getString( "greeting");
    

getBundle()方法获取参数后会在其拥有的资源中进行查找，这个过程包含两个阶段。首先，它查找与接收到的包名参数匹配的属性文件，之后它会查找包的Java类实现。这些类实现由与属性文件命名相似的Java类来实现，例如EmailClientMessages_tr.java，最后她会发现这些属性文件，并使用这些属性文件创建一个ResourceBundle实例。

#### 1.2 Struts 2如何解决本地Java对i18n支持的问题

Struts 2框架为i18n付出了很多努力。框架仍然使用我们刚刚看过的Java类，但事情变得更容易了。首先，不需要实例化ResourceBundle，Struts 2会自动创建它，并处理确定需要哪个包的所有琐碎内容。另一方面，框架也处理确定正确地域的过程，它通过检查来源于浏览器的HTTP头信息自动确定当前的Locale。当然，你也可以覆盖这个行为，并使用其他方式确定地域信息。

Struts 2包创建的过程使用约定和配置混合的方法来定位属性文件，它是一个双向的通道。Struts 2告诉你它在哪里查找属性文件，同时你也可以告诉它在哪里查找属性文件。

#### 1.3 Struts 2的i18n如何工作

框架驱动国际化功能的机制由如下步骤完成：

1.  让动作扩展ActionSupport，以便它们继承默认的TextProvider实现， 
2.  将一些属性文件放在默认的TextProvider可以发现的位置， 
3.  使用Struts 2的text标签或者通过OGNL直接调用getText()方法将消息取到页面中。

#### 1.4 Struts 2 默认的TextProvider ResourceBundle搜索算法

TextProvider的默认实现会在几个常见的地方搜索开发人员创建的属性文件以创建ResourceBundle，这些位置中有很多都遵循与超类或实现的接口的命名相似的模式。以下展示了Struts 2尝试填充的ResourceBundle的名字和出处：

1.  ActionClass——是否有一个ResourceBundle与当前动作类的名字相同？换句话说就是是否有一系列的属性文件命名为ActionClass.properties、ActionClass*es.properties等？ 
2.  MyInterface——如果动作实现了任何接口，有没有与这些接口关联的ResourceBundle?也就是说，如果当前类实现了MyInterface接口，是否有一系列的属性文件命名为MyInterface.properties、MyInterface*es.properties等？并且每一个接口的超接口也会被查找，更具体的接口优先于更高层的超接口。 
3.  MySuperClass——如果动作扩展了一个父类，那么是否有一个ResourceBundle与这个超类关联？这和接口的查找类似。但在继承链上低级别类的ResourceBundle优先于高级别类的ResourceBundle。也就是说，如果Object.properties存在，那么它是最后一个。 
4.  如果动作实现了ModelDriven接口，那么会使用模型对象的类查找ResourceBundle。也就是说，如果模型对象是User类，那么User.properties文件会被加载。 
5.  package.properties——接着，搜索过程会尝试加载当前动作类所在包的ResourceBundle，以及这个链上的每一个父包。注意这些属性文件都叫做package.properties。 
6.  通过关键字引用的ValueStack上的域模型对象，这与第4步的ModelDriven很相似。对于一个给定的关键字，如user.username，如果ValueStack上公开了一个叫做user的属性，那么这个属性的类会用来加载ResourceBundle。
7.  默认的ResourceBundle——Struts 2允许指定全局包，这个包总是可以访问的。

包搜索顺序中的前6步列出了Struts 2搜索属性文件过程中基于的约定位置，如果基于约定的包都不存在，那么就会使用默认包，也就是第7步所说的全局包。我们可以在struts.properties文件中或者某一个XML配置文件(例如struts.xml或者它包含的一个文件)的constant元素中设置这个属性和所有Struts 2属性。以下是具体的使用方法；

    <constant name="struts.custom.i18n.resources" value="global-messages" />
    

以下是在struts.properties文件中配置的相同内容：

    struts.custom.i18n.resources=global-messages
    

可以指定使用逗号分隔的一系列包，这些包按照给定的顺序被搜索，也可以使用包空间指定包的位置，如下所示：

    struts.custom.i18n.resources=global-messages,manning.utils.otherBundle  
    

注意，如果同时使用这两种方式，struts.properties文件优于XML文件中的constant元素。

### 2. 从包中取得消息文本

在前面几节的讲解中我们已经捎带着说明了怎样从保重取得消息文本的方法，包括：使用UI组件标签的key属性、验证框架中使用、validate()方法中使用getText()方法、页面中使用text标签等，就剩下本地化类型转换错误消息的国际化了，下面我们就讲解它。

**本地化类型转换错误消息的国际化**

例如页面中有一个Age字段，当用户输入字母在后台会发生类型转换错误，为了给Age字段追加自定义类型转换错误消息，你只需要在某个可访问的保重追加一个属性，这个属性遵循的命名约定为invalid.fieldvalue.fieldname。以下属性为Age字段定义的一个错误消息：

    invalid.fieldvalue.age=Please enter a numerical value for your age.
    

只需把上述内容追加到Register动作可访问的包(例如Register.properties文件)就可以使用了。此外，你可能想改变本地化默认的类型转换消息。你可以在默认的global-message保重追加以下属性：

    xwork.default.invalid.fieldvalue=We can not convert that to a Java type.

