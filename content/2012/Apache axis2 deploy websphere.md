# Apache Axis2在WAS上的部署

- date: 2012-08-14
- category: Work
- tags: axis2, was

----

在WebSphere Application Server上部署Axis2需要修改几个配置，否则，Axis2服务可以访问，但大多数功能是无法使用的。

由于在WebSphere Application Server 7.0 中，加入了对JAX-WS的支持，它是基于一个Axis2的修改版本的，当WAS启动时，它的类加载器将会加载这些类。于是当真正部署Axis2 1.6时就会发生冲突，怎么解决这种冲突呢，很简单：

1. 在部署axis2.war之前，修改war包中的`axis2.xml`文件。找到`EnableChildFirstClassLoading`参数,将其修改为`true`(默认为`false`)。这个参数只支持axis2的1.5.5以上版本。

2. 在部署完axis2.war之后，修改应用模块的类加载方式为父类最后：在WebSphere的管理员控制台，找到企业级应用的配置页面，点击Manage Modules，定位到包含Axis2的模块，将`Class loader order`项修改为 `Classes loaded with local class loader first (parent last)`

经过以上配置之后，重启axis2应用即可。