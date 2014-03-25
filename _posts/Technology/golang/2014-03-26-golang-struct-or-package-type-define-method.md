---
layout: post
title: golang结构体和包中的类型或基础类型定义方法
category: 技术
tags: golang
keywords: golang,结构体,初始化方法,struct,method,add
description: golang结构体和包中的类型或基础类型定义方法
---



## 1、对结构体定义方法
首先看下面这个例子：

```
package main

import (
    "fmt"
    "math"
)

type Vertex struct {
    X, Y float64
}

func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    v := &Vertex{3, 4}
    fmt.Println(v.Abs())
    fmt.Println(v,*v)
    
    x := Vertex{3, 4}
    fmt.Println(x.Abs())
    fmt.Println(x)   
}
```

Go 没有类。然而，仍然可以在结构体类型上定义方法。方法接收者 出现在 func 关键字和方法名之间的参数中---    (v *Vertex)

```
func (v *Vertex) FunName() float64 {
      // 
}
```

## 2、对包中的任意类型定义任意方法  
对包中的 任意 类型定义任意方法，而不仅仅是针对结构体。不能对来自其他包的类型或基础类型定义方法。-----  (f MyFloat)
如下示例：

```
package main

import (
    "fmt"
    "math"
)

type MyFloat float64

func (f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}

func main() {
    f := MyFloat(-math.Sqrt2)
    fmt.Println(f.Abs())
}

```

下面用如下这个例子来区二者区别以及用法：

```
package main

import (
    "fmt"
    "math"
)
// 普通类型
type MyFloat float64

func (f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}

func (f *MyFloat) P() float64{
    if *f < 0 {
        return float64(-*f)
    }
    return float64(*f)
}

// 结构体
type Vertex struct {
    X, Y float64
}

func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}


func main() {
    // 基础类型的初始化和调用方式,注意用()而不是{}
    f := MyFloat(-math.Sqrt2)
    fmt.Println(f.Abs(),f.P())
    
    /* 
	错误调用方式1:
    f1 := &MyFloat(-math.Sqrt2)
    fmt.Println(*f.Abs(),*f.P())

	错误调用方式2:
	f1 := &MyFloat{-math.Sqrt2}
    fmt.Println(*f.Abs(),*f.P())
	错误调用方式3:
	f1 := MyFloat{-math.Sqrt2}
    fmt.Println(f.Abs(),f.P())
    */
    v := &Vertex{3, 4}
    fmt.Println(v, *v,&v,v.Abs())
    v.Scale(5)
    fmt.Println(v, *v,&v,v.Abs())
    
    v1 := Vertex{6, 7}
    fmt.Println(v1,&v1, v1.Abs())
    v1.Scale(8)
    fmt.Println(v1, v1.Abs())
}
// 输出：
/*
1.4142135623730951 1.4142135623730951
&{3 4} {3 4} 0x10500178 5
&{3 4} {3 4} 0x10500178 5
{6 7} &{6 7} 9.219544457292887
{6 7} 9.219544457292887
*/
```

**注意区别：**

```
func (v *Vertex) Abs() float64 
func (v Vertex) Abs() float64 
```

## 3、注意两者区别：
> * func (v *Vertex) Abs() float64 
    调用方法时传递的是对象的实例的指针，即可改变对象的值；
> * func (v Vertex) Abs() float64 
调用方法时传递的是对象的实例的一个副本，即不可改变对象的值；

总结：对象的实例针对数据操作时必须定义为指针的类型，然后才能传递正确的地址，否则v参数只是对象的一个副本，

下面这个实例则可佐证此观点：

```
package main

import (
    "fmt"
    "math"
)
// 普通类型
type MyFloat float64

func (f MyFloat) Abs()  {
    if f < 0 {
        f = MyFloat(float64(-f))
    }
	 f = MyFloat(float64(f))
}

func (f *MyFloat) Abs_1() {
    if *f < 0 {
        *f = MyFloat(float64(-*f))
    }
	*f = MyFloat(float64(*f))
}

// 结构体
type Vertex struct {
    X, Y float64
}

func (v Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func (v *Vertex) Scale_1(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}


func main() {
    // 基础类型的初始化和调用方式,注意用()而不是{}
    f := MyFloat(-math.Sqrt2)
    fmt.Println(f, &f)
    f.Abs()
    fmt.Println(f, &f)
    f.Abs_1()
    fmt.Println(f, &f)
    
    
    v := &Vertex{3, 4}
    fmt.Println(v, *v,&v)
    v.Scale(5)
    fmt.Println(v, *v,&v)
    v.Scale_1(5)
    fmt.Println(v, *v,&v)
    
    v1 := Vertex{6, 7}
    fmt.Println(v1,&v1)
    v1.Scale(8)
    fmt.Println(v1, &v1)
    v1.Scale_1(8)
    fmt.Println(v1, &v1)
}
// 输出结果：
/*
-1.4142135623730951 0x10500168
-1.4142135623730951 0x10500168
1.4142135623730951 0x10500168
&{3 4} {3 4} 0x10500188
&{3 4} {3 4} 0x10500188
&{15 20} {15 20} 0x10500188
{6 7} &{6 7}
{6 7} &{6 7}
{48 56} &{48 56}
*/
```


总结：golang隐式传递指针，但是不隐式定义指针，此坑需同学们注意。


-----------------------------------------------
golang-python学习心得   
微信公众号：golang-python  
个人微信ID：fuhao1121    
网址：http://fuhao715.github.io  
QQ：243312452   
编程学习心得轻松学编程   
回复:『 p 』查看python课程回复  
回复:『 g 』查看golang课程回复  
回复:『 p 』查看项目管理  
回复:『 w 』查看其他文章   
点击"阅读全文"进入http://fuhao715.github.io  