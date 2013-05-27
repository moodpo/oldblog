# 简明 Python 教程笔记(上)

- date: 2013-03-03
- category: Work
- tags: python, note

-----

利用周末时间终于看完了《简明Python教程》，在线地址在[这里][1]，说来真是惭愧，本来想过年放假时看完的，想不到竟然托到了现在。废话不多说，还是在这里记录下看书时做的笔记吧。

### 1. 控制流

在Python中有三种控制流语句——if、for和while。

#### 1.1 if语句

    number = 23
    guess = int(raw_input('Enter an integer : '))
    
    if guess == number:
        print 'Congratulations, you guessed it.' 
        print "(but you do not win any prizes!)" 
    elif guess < number:
        print 'No, it is a little higher than that' 
    else:
        print 'No, it is a little lower than that' 
    
    print 'Done'
    

#### 1.2 while语句

    number = 23
    running = True
    
    while running:
        guess = int(raw_input('Enter an integer : '))
    
        if guess == number:
            print 'Congratulations, you guessed it.' 
            running = False 
        elif guess < number:
            print 'No, it is a little higher than that' 
        else:
            print 'No, it is a little lower than that' 
    else:
        print 'The while loop is over.' 
    
    print 'Done'
    

#### 1.3 for语句

    for i in range(1, 5):
        print i
    else:
        print 'The for loop is over'
    

除此之外，还有`break`和`continue`语句，与java语言相同。

### 2. 函数

#### 2.1 函数定义

    def sayHello():
        print 'Hello World!' # 函数体
    
    sayHello() # 调用函数
    

#### 2.2 文档字符串

    def printMax(x, y):
        '''Prints the maximum of two numbers.
    
        The two values must be integers.'''
    
        ...... # 函数体
    

文档字符串的惯例是一个多行字符串，它的首行以大写字母开始，句号结尾。第二行是空行，从第三行开始是详细的描述。DocStrings也适用于模块和类。可使用**doc**（注意双下划线）调用。

    print printMax.__doc__
    

### 3. 模块

#### 3.1 模块的**name**

在程序本身被使用的时候运行主块，而在它被别的模块输入的时候不运行主块，以通过模块的**name**属性完成。

    if __name__ == '__main__':
        print 'This program is being run by itself'
    else:
        print 'I am being imported from another module'
    

测试结果：

    $ python using_name.py
    This program is being run by itself
    
    $ python
    >>> import using_name
    I am being imported from another module
    >>>
    

#### 3.2 dir()函数

dir函数用来列出模块定义的属性和方法。

    $ python
    >>> import sys
    >>> dir(sys) # get list of attributes for sys module
    

#### 3.3 其他内容

sys.argv变量是一个字符串的"列表"，它包含了 命令行参数 的列表，即使用命令行传递给你的程序的参数。

sys.path包含输入模块的目录名列表。

help()函数将打印 "文档字符串"内容。

raw_input('Enter an integer:') 用于接收用户的输入。

### 4. 数据结构--列表、元组和字典

#### 4.1 列表

    shoplist = ['apple', 'mango', 'carrot', 'banana'] # 定义
    len(shoplist)  # 内置的返回列表大小的函数
    
    for item in shoplist: # 循环打印列表元素内容
        print item
    
    shoplist.append('rice')
    
    shoplist.sort()
    
    olditem = shoplist[0]
    
    del shoplist[0] # 删除列表中的元素
    

#### 4.2 元组

元组同字符串一样，是不可变的，无法修改的。

    zoo = ('wolf', 'elephant', 'penguin') # 定义元组
    new_zoo = ('monkey', 'dolphin', zoo)
    new_zoo[2][2]
    

只有一个元素的元组这样写，必须加逗号

    singleton = (2,)
    

元组最通常的用法是用在打印语句中

    age = 22
    name = 'Swaroop'
    print '%s is %d years old' % (name, age)
    print 'Why is %s playing with that python?' % name
    

#### 4.3 字典

字典类似"键值对"。

    d = {key1 : value1, key2 : value2 } # 定义
    d[key1]
    del d[key1]
    
    for key,value in d:
        print '%s is $s'%(key,value)
    
    if key in d:
        ......

 [1]: http://woodpecker.org.cn/abyteofpython_cn/chinese/