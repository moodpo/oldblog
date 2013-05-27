# 简明 Python 教程笔记(下)

- date: 2013-03-04
- category: Work
- tags: python, note

-----

今天星期一，周末综合症发作，头痛不行，下班后休息了一会终于活了过来。继续上篇的笔记吧。

### 1. 类

#### 1.1 类的定义

    class Person:
        pass # 空表达式
    
    p = Person()
    print p
    

#### 1.2 类的方法

    class Person:
        def sayHi(self):
            print 'Hello, how are you?'
    
    p = Person()
    p.sayHi()
    

在类中定义方法时，类的第一个参数必须是`self`。

**init**方法在类的一个对象被建立时，马上运行。这个方法可以用来对你的对象做一些你希望的初始化 。类似与Java中类的构造方法。

    class Person:
        def __init__(self, name):
            self.name = name
        def sayHi(self):
            print 'Hello, my name is', self.name
    
    p = Person('Swaroop')
    p.sayHi()
    

Python 中所有的类成员(包括数据成员)都是 公共的 ,所有的方法都是有效的 。只有一个例外:如果你使用的数据成员名称以 双下划线前缀 比如__privatevar,Python 的名称管理体系会有效地把它作为私有变量。

当对象不再被使用时，**del**方法就会运行。

#### 1.3 继承

    class SchoolMember:
        ......
    class Teacher(SchoolMember):
        ......
    class Student(SchoolMember):
        ......
    

一个术语的注释——如果在继承元组中列了一个以上的类,那么它就被称作多重继承 。

### 2. 输入/输出操作

    f = file('poem.txt','w')  # r读模式 w写模式 a追加模式 
    f.write(pome)
    f.close()
    
    f = file('poem.txt')
    
    while True:
        line = f.readline()
        if len(line) == 0:
            break
        print line
    f.close()
    

### 3. 存储器pickle

Python提供一个标准的模块，称为pickle。使用它你可以在一个文件中储存任何Python对象，之后你又可以把它完整无缺地取出来。这被称为 持久地 储存对象。

    import cPickle as p
    
    shoplist = ['apple','mango','carrot']
    
    f = file('shoplist.data','w')
    p.dump(shoplist,f)
    f.close
    
    del shoplist
    
    f = file('shoplist.data')
    storedlist = p.load(f)
    print storedlist
    

### 4. 异常

#### 4.1 处理异常

    try:
        ......
    except EOFError:
        ......
    except:
        ......
    finally:
        ......
    

#### 4.2 引发异常

使用 raise 语句 引发 异常。你还得指明错误/异常的名称和伴随异常触发的 异常对象。

    if len(s) < 3:
        raise ShortInputException(len(s), 3)
    #参数：实际值/期望值
    

类似于java中的throw