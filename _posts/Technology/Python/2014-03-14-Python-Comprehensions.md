---
layout: post
title: Python推导式演变(Comprehensions)
category: 技术
tags: Python
keywords: Python,List,推导表达式,Comprehensions
description: 
---

# Python推导式演变(Comprehensions)

----------

如果你已经使用了很长时间的Python，那么你至少应该听说过列表推导(list comprehensions)。这是一种将for循环、if表达式以及赋值语句放到单一语句中的一种方法。换句话说，你能够通过一个表达式对一个列表做映射或过滤操作。

一个列表推导式包含以下几个部分：
> * 一个输入序列
> * 一个表示输入序列成员的变量
> * 一个可选的断言表达式
> * 一个将输入序列中满足断言表达式的成员变换成输出列表成员的输出表达式

举个例子，我们需要从一个输入列表中将所有大于0的整数平方生成一个新的序列，你也许会这么写：

```python
num = [1, 4, -5, 10, -7, 2, 3, -1]
filtered_and_squared = []
for number in num:
    if number > 0:
        filtered_and_squared.append(number ** 2)
print filtered_and_squared
 
# [1, 16, 100, 4, 9]
```

很简单是吧？但是这就会有4行代码，两层嵌套外加一个完全不必要的append操作。首先我们想到的是使用filter、lambda和map函数，则能够将代码大大简化：

```python
num = [1, 4, -5, 10, -7, 2, 3, -1]
filtered_and_squared = map(lambda x: x ** 2, filter(lambda x: x > 0, num))
print filtered_and_squared
 
# [1, 16, 100, 4, 9]
```

如果我们使用列表推导表达式呢？将会更大化简化代码：

```python
num = [1, 4, -5, 10, -7, 2, 3, -1]
filtered_and_squared = [ x**2 for x in num if x > 0]
print filtered_and_squared
 
# [1, 16, 100, 4, 9]
```

![Comprehensions](/public/upload/Python_Comprehensions.jpg)
> * 迭代器(iterator)遍历输入序列num的每个成员x
> * 断言式判断每个成员是否大于零
> * 如果成员大于零，则被交给输出表达式，平方之后成为输出列表的成员。


下面我们总结一下列表推导表达式的过滤列表语法：
> [mapping-expression for element in source-list if filter-expression]

也可表达为：

> methodList = [method for method in dir(object) if callable(getattr(object, method))]

这行看上去挺复杂,确实也很复杂,但是基本结构都还是一样的。整个过滤表达式返回一个列表，并赋值给 methodList 变量。表达式的前半部分是列表映射部分。映射表达式是一个和遍历元素相同的表达式，因此它返回每个元素的值。dir(object) 返回 object 对象的属性和方法列表――你正在映射的列表。所以唯一新出现的部分就是在 if 后面的过滤表达式。

过滤表达式看上去很恐怖，其实不是。你已经知道了 callable、getattr 和 in。正如你在前面的部分中看到的，如果 object 是一个模块，并且 method 是上述模块中某个函数的名称，那么表达式 getattr(object, method) 将返回一个函数对象。

所以这个表达式接收一个名为 object 的对象，然后得到它的属性、方法、函数和其他成员的名称列表，接着过滤掉我们不关心的成员。执行过滤行为是通过对每个属性/方法/函数的名称调用 getattr 函数取得实际成员的引用，然后检查这些成员对象是否是可调用的，当然这些可调用的成员对象可能是方法或者函数，同时也可能是内置的 (比如列表的 pop 方法) 或者用户自定义的 (比如 odbchelper 模块的 buildConnectionString 函数)。这里你不用关心其它的属性，如内置在每一个模块中的 __name__ 属性。

列表推导式被封装在一个列表中，所以很明显它能够立即生成一个新列表。这里只有一个type函数调用而没有隐式调用lambda函数，列表推导式正是使用了一个常规的迭代器、一个表达式和一个if表达式来控制可选的参数。

另一方面，列表推导也可能会有一些负面效应，那就是整个列表必须一次性加载于内存之中，这对上面举的例子而言不是问题，甚至扩大若干倍之后也都不是问题。但是总会达到极限，内存总会被用完。

针对上面的问题，生成器(Generator)能够很好的解决。生成器表达式不会一次将整个列表加载到内存之中，而是生成一个生成器对象(Generator objector)，所以一次只加载一个列表元素。

生成器表达式同列表推导式有着几乎相同的语法结构，区别在于生成器表达式是被 ***圆括号包围，而不是方括号***：

```python
num = [1, 4, -5, 10, -7, 2, 3, -1]
filtered_and_squared = ( x**2 for x in num if x > 0 )
print filtered_and_squared
 
# <generator object <genexpr> at 0x00583E18>
 
for item in filtered_and_squared:
    print item
 
# 1, 16, 100 4,9
```

这比列表推导效率稍微提高一些，

```python
num = [1, 4, -5, 10, -7, 2, 3, -1]
 
def square_generator(optional_parameter):
    return (x ** 2 for x in num if x > optional_parameter)
 
print square_generator(0)
# <generator object <genexpr> at 0x004E6418>
 
# Option I
for k in square_generator(0):
    print k
# 1, 16, 100, 4, 9
 
# Option II
g = list(square_generator(0))
print g
# [1, 16, 100, 4, 9]
```

除非特殊的原因，应该经常在代码中使用生成器表达式。但除非是面对非常大的列表，否则是不会看出明显区别的。
