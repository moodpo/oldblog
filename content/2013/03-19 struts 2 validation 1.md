# 复习笔记——Struts 2 的验证框架(一)

- date: 2013-03-18
- category: Work
- tags: struts2, note

----

之前我们已经学习了如何通过Validateable接口的validate()方法实现动作本地的验证方式。虽然这种方式工作得很好，但是它的某些限制最终会变得难以负担。因此，在本节中我们将引入Struts 2框架的另一个高级机制——验证框架。数据验证框架提供了一个比Validateable接口更通用、更可维护的验证解决方案。验证框架中更强大的一点是Validator（验证器），它是一种可重用的组件，其中实现了特定类型的验证逻辑。

### 1.熟悉数据验证框架

数据验证框架早已成为了Web应用程序框架的一部分，但是Struts 2将它在优化、模块化、整洁继承方面带到了一个权限的水平。

#### 1.1 验证框架的架构

下图展示了验证框架的主要组件：

![Struts 2 验证框架.png][1]

如上图所示，在验证框架中有3个主要的组件：域数据、验证元数据和验证器。在验证工作中每一个组件都扮演了至关重要的角色，我们将一一解释这些内容。

**1.域数据**

首先，必须有一些验证数据，这些数据以属性的方式驻留在Struts 2的动作中，数据来自对象或者模型驱动对象。

**2.验证元数据**

验证器和数据属性之间有一个中间组件，这个中间组件就是元数据，这些数据将每一个数据属性与属性运行时数据的合法性验证关联起来。一个属性可以关联多个验证器，也可以不关联任何验证器。元数据层可以使用XML文件或者Java注解将数据属性映射到验证器。

**3.验证器**

所有这些数据的验证实际上都有验证器完成。验证器是一个包含了执行某种细粒度的验证行动的逻辑的可重用组件。

#### 1.2 Struts 2 工作流中的验证框架

现在看看所有的这些验证工作是如何完成的。验证框架实际上与基本数据验证共享了大部分功能，它使用ValidationAware接口存储错误信息。如果需要，workflow拦截器会将用户带回到input页面。实际上，唯一改变的是验证本身，但是这是一个重大的变化。

不管基本的验证示例还是验证框架都在Struts 2自带的defaultStack环境下工作。下面代码片段是跟当前话题相关的defaultStack部分，来自struts-default.xml文件：

    <interceptor-ref name="params"/>
    <interceptor-ref name="conversionError"/>
    <interceptor-ref name="validation"/>
    <interceptor-ref name="workflow"/>
    

我们需要注意validation拦截器，workflow拦截器调用validate()方法来实施基本验证还没有参与。当validation拦截器触发时，它会通过前一节提到的元数据实施所有已经定义的验证。一下是验证框架的工作流程，数据验证框架在数据转移、类型转换之后，workflow拦截器之前运行：

![验证框架的工作流程.png][2]

从图中可以看出整个流程有两个数据验证机制，一种是validation拦截器即数据验证框架的验证，一种是workflow调用validate()方法的验证。我们将会有两种选择，如果能够预测到一个验证逻辑将来还会被重用，那么使用一个自定义验证器来实现会更有意义。然而，如果验证逻辑确实是一个生僻的需求，并且很可能只适用于一次的情况，那么把它放在validate()方法中会更有意义。

### 2.将动作关联到验证框架

#### 2.1 使用ActionClass-validations.xml声明验证元数据

现在，我们呢使用XML文件声明需要验证的元数据，文件名以：动作类名+-+validations.xml为规范，以下是Register-validation.xml:

    <!DOCTYPE validators PUBLIC "-//OpenSymphony Group//XWork Validator 1.0.2//
        EN" "http://www.opensymphony.com/xwork/xwork-validator-1.0.2.dtd">
    <validators>
        <field name="password">
            <field-validator type="requiredstring">
            <message>You must enter a value for password.</message>
            </field-validator>
        </field>
        <field name="username">
            <field-validator type="stringlength">
            <param name="maxLength">8</param>
            <param name="minLength">5</param>
            <message>While ${username} is a nice name, a valid username must
                    be between ${minLength} and ${maxLength} characters long.
            </message>
            </field-validator>
        </field>
        <field name="portfolioName">
            <field-validator type="requiredstring">
            <message key="portfolioName.required"/>
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
    

**1.字段验证器**

字段验证器是用来操作一个独立字段的验证器。这里的字段(field)与数据属性意思相同，字段验证器中的“字段”的含义是它们来源于请求的HTML表单的字段。

例如第一个字段password字段，一旦为数据声明了一个field元素，我们只需在field元素内部放入field-validator元素来声明哪些验证器用来验证此数据。对于password，我们只声明了一个requiredstring验证器。message元素包含验证失败时显示的消息的文本。一个field元素可以声明任意多个验证器。

**2.非字段验证器**

你也可以声明验证逻辑不是适用在某个特定字段的验证器，这些验证器适用于整个动作，通常包含对多个字段值的检查。例如：expression验证器就是如此。这个验证器使用OGNL比较其他两个字段是否相等，如果不相等表达式返回true，验证通过，否则，验证失败最终用户会返回输入页面并显示message元素中的内容。

**3.消息元素的选择**

message元素用来指定验证错误的情况下用户应该看到的消息，我们可以使用OGNL动态的组织消息。例如username字段的声明那样。XML中OGNL使用$作为转义字符，而不是OGNL通常使用的%符号。

message元素的另一个选择是将这些消息抽出到外部的资源文件中以实现验证错误的提示信息的国际化，portfolioName字段的验证方式是一个很好的示例，message 的 key 属性指向的正式国际化文件中的引用。如下：

    user.exists=This user ${username} already exists.
    portfolioName.required=You must enter a name for your initial portfolio.
    email.invalid=Your email address was not a valid email address.
    

#### 2.2 内建的验证器

框架自带了一系列功能强大的验证器以满足常见的验证需求，以下列出了全部的内建验证器：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>
      数据验证器
    </td>
    
    <td>
      参数
    </td>
    
    <td>
      功能
    </td>
    
    <td>
      类型
    </td>
  </tr>
  
  <tr>
    <td>
      required
    </td>
    
    <td>
      没有
    </td>
    
    <td>
      检验值非空
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      requiredstring
    </td>
    
    <td>
      trim(默认为true)
    </td>
    
    <td>
      验证值非空，并且不是空字符串
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      stringlength
    </td>
    
    <td>
      trim(默认值为true)、minLenth、maxLength
    </td>
    
    <td>
      验证字符床的长度在参数指定的范围内。不指定的参数不做检查。
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      int
    </td>
    
    <td>
      min、max
    </td>
    
    <td>
      验证这个证书值在参数指定的最小值和最大值之间
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      double
    </td>
    
    <td>
      minInclusive、maxInclusive、minExclusive、maxExclusive
    </td>
    
    <td>
      验证浮点值在参数指定的范围内
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      date
    </td>
    
    <td>
      min、max
    </td>
    
    <td>
      验证日期值在指定的最小值和最大值之间。日期格式MM/DD/YYYY
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      email
    </td>
    
    <td>
      没有
    </td>
    
    <td>
      验证电子邮件地址格式
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      url
    </td>
    
    <td>
      没有
    </td>
    
    <td>
      验证URL格式
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      fieldexpression
    </td>
    
    <td>
      expression(必须)
    </td>
    
    <td>
      根据当前ValueStack解析OGNL表达式。表达式必须返回true或者false以决定验证是否成功
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      expression
    </td>
    
    <td>
      expression(必须)
    </td>
    
    <td>
      与fieldexpression相同，但用在动作级别
    </td>
    
    <td>
      动作
    </td>
  </tr>
  
  <tr>
    <td>
      visitor
    </td>
    
    <td>
      context、appendPrefix
    </td>
    
    <td>
      将域对象属性的验证转交给域对象本地的验证声明
    </td>
    
    <td>
      字段
    </td>
  </tr>
  
  <tr>
    <td>
      regx
    </td>
    
    <td>
      expression(必须)、caseSensitive、trim
    </td>
    
    <td>
      验证String遵循给定的正则表达式
    </td>
    
    <td>
      字段
    </td>
  </tr>
</table>

这些数据验证器中的大部分功能都很简单，唯一需要深入讨论的是visitor验证器，这个验证器允许你为每一个域模型(ModelDriven)类定义验证元素据。

### 3.编写自定义验证器

编写自定义验证器与编写任何其他的Struts 2组件都不同，下面我们将通过编写一个检查密码完整性的自定义验证器。

#### 3.1 检查密码强度的自定义验证器

所有验证器必须实现Validator接口或者FieldValidator接口，前者将实现非字段验证器，后者将实现字段验证器。通常情况下我们会扩展对应的ValidatorSupport或者FieldValidatorSupport这两个类。

我们设计的密码验证器将做一下3项检查：

*   密码必须包含一个大写字母或者小写字母；
*   密码必须包含0～9的一个数字；
*   密码必须至少包含一套特殊字符中的一个字符。

特殊字符有一个默认值，但可以通过一个参数配置，这与之前使用的stringlength参数很类似。以下是自定义验证器类代码：

    public class PasswordIntegrityValidator extends FieldValidatorSupport {
        static Pattern digitPattern = Pattern.compile( "[0-9]");
        static Pattern letterPattern = Pattern.compile( "[a-zA-Z]");
        static Pattern specialCharsDefaultPattern = Pattern.compile( "!@#$");
    
        public void validate(Object object) throws ValidationException {
            String fieldName = getFieldName();
            String fieldValue = (String) getFieldValue(fieldName, object );
            fieldValue = fieldValue.trim();
            Matcher digitMatcher = digitPattern.matcher(fieldValue);
            Matcher letterMatcher = letterPattern.matcher(fieldValue);
            Matcher specialCharacterMatcher;
            if ( getSpecialCharacters() != null ){
                Pattern specialPattern = Pattern.compile("[" + getSpecialCharacters() + "]" );
                specialCharacterMatcher = specialPattern.matcher( fieldValue );
            } else{
                specialCharacterMatcher = specialCharsDefaultPattern.matcher( fieldValue );
            }
            if ( !digitMatcher.find() ) {
                addFieldError( fieldName, object );
            }else if ( !letterMatcher.find() ) {
                addFieldError( fieldName, object );
            }else if ( !specialCharacterMatcher.find() ) {
                addFieldError( fieldName, object );
            }
        }
        private String specialCharacters;
        //省略get和set方法
    }
    

作为一个开发人员，只需要关注验证逻辑的细节，这个逻辑放在validate()方法中，这个方法是Validator接口定义的入口方法并且在扩展类的抽象支持类中没有实现。此外，还需要创建JavaBean属性，这些属性应该与所有公开给用户的参数匹配。以下代码片段展示了一个参数如何从XML文件传递到这个属性：

    <field-validator type="passwordintegrity">
        <param name="specialCharacters">$!@#?</param>
        <message>Your password must contain one letter, one number, and one
            of the following "${specialCharacters}".
        </message>
    </field-validator>
    

我们通过两个辅助方法 getFieldName()和getFieldValue()获取字段的值，这里注意，validate()方法接收了需要被验证的对象，由于我们在动作级别通过Register-validation.xml文件定义了验证，所以传入validate()方法的对象是动作本身。获取字段值后，我们进行了各种检查，最终将错误信息添加到存储的错误消息集中，当然还是使用从支持类继承的辅助方法。这就是全部内容了。

#### 3.2 使用自定义数据验证器

我们在应用程序的类路径下的validators.xml文件中声明自定义的验证器，也就是src文件夹下，一下是这个文件的所有代码：

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE validators PUBLIC
            "-//OpenSymphony Group//XWork Validator Config 1.0//EN"
            "http://www.opensymphony.com/xwork/xwork-validator-config-1.0.dtd">
    
    <validators>
       <validator name="passwordintegrity" class="manning.utils.PasswordIntegrityValidator"/>
    </validators>
    

然后在Register-validation.xml文件中把这个验证器添加到password字段对应的验证器中，以下是代码片段：

    <field name="password">
      <field-validator type="requiredstring">
         <message >Password is required.</message>
      </field-validator>
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

 [1]: /media/2013/03/1475721437.png
 [2]: /media/2013/03/1923682797.png