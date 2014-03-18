---
layout: post
title: python函数式编程之-装饰器(Decorators)
category: 技术
tags: Python
keywords: Python,函数式编程,装饰器，Decorators
description: 
---

# python函数式编程之-装饰器(Decorators)

----------

## 函数装饰器
装饰器为我们提供了一个增加已有函数或类的功能的有效方法。听起来是不是很像Java中的面向切面编程(Aspect-Oriented Programming)概念？两者都很简单，并且装饰器有着更为强大的功能。举个例子，假定你希望在一个函数的入口和退出点做一些特别的操作(比如一些安全、追踪以及锁定，打印日志，函数执行时间等操作)就可以使用装饰器。

装饰器是一个包装了另一个函数的特殊函数：主函数被调用，并且其返回值将会被传给装饰器，接下来装饰器将返回一个包装了主函数的替代函数，程序的其他部分看到的将是这个包装函数。
语法糖@标识了装饰器。 如下方式

```python
def decorator(func):
    '''
    Decorator that reports the execution time.
    '''
    pass
 
@decorator
def func_method(n):
    '''
    Decorator that reports the execution time.
    '''
    pass

```

下面我们将用装饰器做一些更典型的操作：在不改变函数内部逻辑计算函数执行时间,以及在函数执行前后打印log。
这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。
```python
import time
from functools import wraps
 
def timethis(func):
    '''
    Decorator that reports the execution time.
    '''
    @wraps(func)
    def wrapper(*args, **kwargs):
	print 'call %s():' % func.__name__
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end-start)
        return result
    return wrapper
 
@timethis
def countdown(n):
    while n > 0:
        n -= 1
 
countdown(100000)

# call countdown():
# ('countdown', 0.006999969482421875)
```

讲解：
> * 装饰器即如下描述
```python
@timethis
def countdown(n):
```

> * 上述语法糖即表示在编译器执行时分别执行如下步骤：
```python
def countdown(n):
    ...
countdown = timethis(countdown)
```

> * decorator接受一个函数作为参数，并返回一个函数。我们要借助Python的@语法，把decorator置于函数的定义处：
> * 当编译器查看以上代码时，countdown()函数将会被编译，并且函数返回对象将会被传给装饰器代码，装饰器将会在做完相关操作之后用一个新的函数对象代替原函数
> * 装饰器函数中的代码创建了一个新的函数(正如此例中的wrapper函数)，它用 *args 和 **kwargs 接收任意的输入参数，并且在此函数内调用原函数并且返回其结果。
你可以根据自己的需要放置任何额外的代码(例如本例中的计时操作)，新创建的包装函数将作为结果返回并取代原函数。



## 带参数的函数装饰器

如果decorator本身需要传入参数，那就需要编写一个返回decorator的高阶函数，写出来会更复杂。比如，要自定义log的文本：

```python
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print '%s %s():' % (text, func.__name__)
            return func(*args, **kw)
        return wrapper
    return decorator

# 这个3层嵌套的decorator用法如下：
@log('execute')
def countdown(n):
    while n > 0:
        n -= 1
    print 'end'
 
countdown(100000)
# execute countdown():
# end


countdown = log('execute')(countdown)
# execute countdown():
# end
# 我们来剖析上面的语句，首先执行log('execute')，返回的是decorator函数，再调用返回的函数，参数是countdown函数，返回值最终是wrapper函数。


print countdown.__name__
# wrapper

# 为何经过decorator装饰之后的函数，它们的__name__已经从原来的'now'变成了'wrapper'：
# 因为返回的那个wrapper()函数名字就是'wrapper'，因此就是wrapper

```

为避免上述问题需要把原始函数的__name__等属性复制到wrapper()函数中，否则，有些依赖函数签名的代码执行就会出错。

不需要编写wrapper.__name__ = func.__name__这样的代码，Python内置的functools.wraps就是干这个事的，所以，一个完整的decorator的写法如下：
```python
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print '%s %s():' % (text, func.__name__)
            return func(*args, **kw)
        return wrapper
    return decorator

```


一个更实际的例子：但是这种方式需要更改函数调用参数

```python
def decorator(func):
    def modify(*args, **kwargs):
        variable = kwargs.pop('variable', None)
        print variable
        x,y=func(*args, **kwargs)
        return x,y
    return modify
 
@decorator
def func(a,b):
    print a**2,b**2
    return a**2,b**2
 
func(a=4, b=5, variable="hi")
func(a=4, b=5)
 
# hi
# 16 25
# None
# 16 25

```

## 面向对象的decorator
在面向对象（OOP）的设计模式中，decorator被称为装饰模式。OOP的装饰模式需要通过继承和组合来实现。
Python除了能支持OOP的decorator外，直接从语法层次支持decorator。Python的decorator可以用函数实现，也可以用类实现。
装饰器定义成类更容易理解其功能，并且这样更能发挥装饰器机制的威力。
对装饰器的类实现唯一要求是它必须能如函数一般使用，也就是说它必须是可调用的。所以，如果想这么做这个类必须实现__call__方法。
这样的装饰器应该用来做些什么？它可以做任何事，但通常它用在当你想在一些特殊的地方使用原函数时，但这不是必须的，例如：

```python
class decorator(object):
 
    def __init__(self, f):
        print("inside decorator.__init__()")
        f() # Prove that function definition has completed
 
    def __call__(self):
        print("inside decorator.__call__()")
 
@decorator
def function():
    print("inside function()")
 
print("Finished decorating function()")
 
function()
 
# inside decorator.__init__()
# inside function()
# Finished decorating function()
# inside decorator.__call__()

```
讲解：
> * 语法糖@decorator相当于function=decorator(function)，在此调用decorator的__init__打印“inside decorator.__init__()”
> * 随后执行f()打印“inside function()”
> *  随后执行“print(“Finished decorating function()”)”
> * 最后在调用function函数时，由于使用装饰器包装，因此执行decorator的__call__打印 “inside decorator.__call__()”。



#     golang-python学习心得     
#微信ID：fuhao1121  
网址：http://fuhao715.github.io  
QQ：243312452   
编程学习心得轻松学编程   
回复:『 p 』查看python课程回复  
回复:『 g 』查看golang课程回复  
回复:『 t 』查看习题回复  
回复:『 w 』查看其他文章   
点击"阅读全文"进入http://fuhao715.github.io  