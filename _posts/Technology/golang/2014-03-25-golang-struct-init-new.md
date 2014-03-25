---
layout: post
title: golang中结构体的初始化方法(new方法)
category: 技术
tags: golang
keywords: golang,结构体,初始化方法,struct
description: golang中结构体的初始化方法(new方法)
---



## 1、自定义一个结构体

```
type Vertex struct {
    X, Y float64
}
```

## 2、初始化方法-指针：

```
rect1 := new(Vertex )
rect2 := &Vertex {}
rect3 := &Vertex {1, 2}
rect4 := &Vertex {X:100, Y:200}
```

***注意***: 这几个变量全部为指向Rect结构的指针(指针变量)，因为使用了new()函数和&操作符．

## 3、初始化方法-类型实例

```
a := Rect{}
b := Rect{3, 4}
c := Rect{X=5, Y=6}
```

则表示这个是一个Rect{}类型．两者是不一样的．

## 4、区别 
下面这个例子能展现之间区别：

```
package main
import "fmt"

type Vertex struct {
        X, Y float64
} 
func main() {
    rect1 := new(Vertex)
	rect2 := &Vertex{1, 2}
	fmt.Printf("%v  %T  %v \n",  rect1,  rect1,  *rect1)
	fmt.Printf("%v  %T  %v \n",  rect2,  rect2,  *rect2)

	rect3 := Vertex{X: 5, Y: 6}
	fmt.Printf("%v  %T\n",  rect3,  rect3)
    
}
// 输出：
/*
&{0 0}  *main.Vertex  {0 0} 
&{1 2}  *main.Vertex  {1 2} 
{5 6}  main.Vertex
*/
```


从结果中可以清楚的看到两者的不同．
> * 用 new 分配内存 内建函数 new 本质上说跟其他语言中的同名函数功能一样：new(T) 分配了零值填充的 T 类型的内存空间，并且返回其地址，一个 *T 类型的值。用 Go 的术语说，它返回了一个指针，指向新分配的类型 T 的零值。记住这点非常重要。 这意味着使用者可以用 new 创建一个数据结构的实例并且可以直接工作。

> * 务必记得 make 仅适用于 map，slice 和 channel，并且返回的不是指针。应当用 new获得特定的指针。


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
