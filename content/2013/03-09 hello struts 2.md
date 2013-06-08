# 复习笔记——Hello Struts 2

- date: 2013-03-09
- category: Work
- tags: struts2,J2EE

----

本文主要复习Struts2的两种声明性架构：

*   基于xml的声明性架构； 
*   基于java注解的声明性架构.

需要的jar包：

*   commons-fileupload-1.2.1.jar
*   commons-io-1.3.2.jar
*   commons-logging-1.0.4.jar
*   commons-logging-api-1.1.jar
*   freemarker-2.3.16.jar 
*   javassist-3.7.ga.jar
*   ognl-3.0.jar
*   struts2-core-2.2.1.1.jar
*   xwork-core-2.2.1.1.jar

### 1.基于xml的声明性架构

web.xml配置：

    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    

struts.xml配置：

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
        "http://struts.apache.org/dtds/struts-2.0.dtd">
    <struts>
        <!-- 运行于开发者模式 -->
        <constant name="struts.devMode" value="true"></constant>
        <package name="struts2demo1" namespace="/demo1" extends="struts-default">
            <action name="Name">
                <result>/nameCollection.jsp</result>
            </action>
            <action name="Hello" class="xiaoxie.review.struts2demo.HelloworldAction">
                <result name="SUCCESS">/helloworld.jsp</result>
            </action>
        </package>
    </struts>
    

这个示例中包含了两个动作(action),其中一个几乎什么都没做。package元素是一个重要的容器元素，示例中它声明了一个当框架将url映射到动作时需要使用的命名空间：

    http://+localhost:8080/+struts2demo/+demo1/+helloworld.action
    协议+主机名：端口+servlet上下文+命名空间+动作
    

nameCollection.jsp

    <%@ page language="java" contentType="text/html; charset=UTF-8"
        pageEncoding="UTF-8"%>
    <%@ taglib prefix="s" uri="/struts-tags" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Name Collection</title>
    </head>
    <body>
        <center>
            <div>
                <h4>输入姓名</h4>
                <s:form action="Hello">
                    <s:textfield name="name" label="姓名"/>
                    <s:submit value="Submit"></s:submit>
                </s:form>
            </div>
        </center>
    </body>
    </html>
    

helloworld.jsp

    <%@ page language="java" contentType="text/html; charset=UTF-8"
        pageEncoding="UTF-8"%>
    <%@ taglib prefix="s" uri="/struts-tags" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>HelloWorld</title>
    </head>
    <body>
        <center>
            <h4>Hello</h4>
            <div><s:property value="coutomeName"/></div>
        </center>
    </body>
    </html>
    

HelloworldAction.java

    package xiaoxie.review.struts2demo;
    
    public class HelloworldAction {
        private String name;
        private String coutomeName;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    
        public String getCoutomeName() {
            return coutomeName;
        }
        public void setCoutomeName(String coutomeName) {
            this.coutomeName = coutomeName;
        }
        public String execute(){
            setCoutomeName("Hello "+getName());
            return "SUCCESS";
        }
    }
    

使用JavaBean的方式编写action使领域数据总是存储在action中,action的execute方法可以很方便的访问到数据；而为了在其他地方(比如jsp)也能访问到数据，框架底层也把数据放到了ValueStack中。ValueStack的机制是将action中的所有属性作为它的第一级属性公开出来，这样就可以使用ONGL来访问(<code><s:property value="coutomeName"/></code>).

### 2.基于java注解的声明性架构

在web.xml中添加以下代码

    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
        <init-param>
            <!-- 扫描xiaoxie包，查找注解 -->
            <param-name>actionPackages</param-name>
            <param-value>xiaoxie</param-value>
        </init-param>
    </filter>
    

标记哪个类是action类有两种方法：

*   让action类实现com.opensymphony.xwork2.Action接口
*   使用命名约定，类名以Action结尾 
    
	@Result(name="SUCCESS",location="/helloworld.jsp")  
	public class HelloworldAction{ ...... }