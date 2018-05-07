# Golang 101系列基础篇——Struct和Interface

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1522650631/1.pic_enaskv.jpg)

这篇Blog里面我们来聊一聊Golang的结构体和接口(Interface)。前者跟C系的编程语言比较接近比较好理解，后者接口被称为Golang里面一个很精妙的设计，所以比较奇特。是的，就像Go创始人之一Rob Pike描述的朴素真理一样，空的接口啥都没有说。Hhhhhh.

## struct
结构体定义了成员，让各个成员组成一个有机的、可以更好被理解的数据对象结构。比如，一个立方体拥有长、宽、高三个变量，那么通过关键词type......struct 可以定义为如下：

``` go

type Cube struct{
	l float64
	w float64
	h float64
}

//or a collapse way:
type Cube struct{
	l, w, h float64
}
```

使用的时候也是挺简单：

``` go
	c1 := Cube{2, 3, 4}
	fmt.Println(c1.l, c1.w, c1.h) //2, 3, 4
	
	func GetVolume(c Cube) float64 {
		return c.l * c.w * c.h
	}
	
	fmt.Println(GetVolume(c1)) //24
```

在这个例子当中，我们把struct传入给GetVolume函数求体积。这里struct其实是被复制了一遍的，所以在可以的情况，我们可以使用别的办法避免复制，节约资源。第一个想法就是使用指针， 让函数接受指针，传参的时候使用地址，效果跟前一种方法是一样的：

``` go
	
	func GetVolume(c *Cube) float64 {
		return c.l * c.w * c.h
	}
	
	fmt.Println(GetVolume(&c1)) //24
```

当然，在类似于面向对象这样的思想，我们希望struct可以主动调用这个函数。于是，该函数可以一个指向Cube的指针，传入作为pointer receiver：

``` go
func (c *Cube) GetVolume() float64 {
	return c.l * c.w * c.h
}

fmt.Println(c1.GetVolume()) //24
```

下面再给出一个练习时的完整例子，这里contactInfo struct是套在person struct里面的，使用起来也蛮方便。

``` go
package main

import "fmt"

type contactInfo struct {
	email string
	zip   int
}

type person struct {
	firstName string
	lastName  string
	contact   contactInfo
}

func (p *person) updateName(firstName string) {
	(*p).firstName = firstName
}

func (p person) print() {
	fmt.Printf("%+v\n", p)
}

func main() {
	alex := person{firstName: "Alex", lastName: "Hammer", contact: contactInfo{email: "1@123.com", zip: 94568}}
	alex.print()

	alex.updateName("GG")
	alex.print()
}
```

这里值得注意的还是updateName使用了pointer ceceiver, *person。这里主要原因是一是避免复制，二是因为我们将改变原有对象（Person）的属性值，所以需要用指针，跟C++里面reference那一套道理很像。因此，将上述updateName下面这个函数是不会起作用的：

``` go
func (p person) updateName(firstName string) {
	p.firstName = firstName
}
```

## interface
接口定义了一系列的方法集合(collection of method signatures)，任何实现方法则被称为“实现了该接口“。这里的哲学被称为是Duck Type: “If it walks like duck, swims like a duck and quacks like a duck, it’s a duck”（如果一个东西走路像鸭子，游泳像鸭子，叫声像鸭子，那么它就是鸭子.）

在下面的例子中，我的两个struct各自有自己的GetSalary()和GetWorkContent()实现，做不同的事情。那么我则可以提供一个interface，interface内声明有GetSalary()和GetWorkContent()两个方法，那么其他函数中，我可以传该interface来代替任何一个struct，这样可以增加代码的复用，使代码更加简洁。

``` go
package main

import (
	"fmt"
)

type Employee interface {
	GetSalary() int
	GetWorkContent() string
}

type Engineer struct {
	base int
	tech string
}

type SalesPerson struct {
	base, bonus int
	activity    string
}

func (e Engineer) GetSalary() int {
	return e.base
}

func (s SalesPerson) GetSalary() int {
	return s.base + s.bonus
}

func (e Engineer) GetWorkContent() string {
	return "Coding with " + e.tech
}

func (s SalesPerson) GetWorkContent() string {
	return s.activity
}

func ShowJobDescription(em Employee) {
	fmt.Println(em)
	fmt.Println("Income :", em.GetSalary())
	fmt.Println(em.GetWorkContent())
}

func main() {
	Tony := Engineer{5000, "C++"}
	Sarah := SalesPerson{3000, 2000, "business"}

	ShowJobDescription(Tony)
	ShowJobDescription(Sarah)
}

```