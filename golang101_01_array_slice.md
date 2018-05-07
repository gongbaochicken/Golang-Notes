# Golang 101系列基础篇 String, Array, Slice

## 前言
2017年年底，我才开始正式接触GO语言，这是一个偶尔机会的必然相遇。在最初认识Golang的时候，我开始犯我的老毛病，犹豫这是否是一个值得投入宝贵时间和精力的技术，犹豫学习Golang之后所带给我的好处。其实这种犹豫是大部分人都有的，毕竟现代人都需要认识到精力的有限，只不过每个人对这样的犹豫程度不同罢了。

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1521526970/images2_lcl5sf.png)

事实上，在最初的时候，我跟很多人想法不一样。很多人觉得Golang背后有Google，感觉靠山很强，很好棒棒哒，而我反而觉得当一门编程语言被标上Google的标签后，并不见得是一件好事。一旦因为公司政策和方向改变，说不定未来的发展会发生变数。况且以Google一贯的作风（Angular 2，3，4）很难说之后的维护会是怎么样的。在各种中外论坛上，也有很多人对Golang嗤之以鼻，其中比较有名的驳斥，当然是王垠称Golang在设计上很垃圾。

我在研究过一小段时间后，发现Golang实在亮点突出：简单。想起《神雕侠侣》中的玄铁重剑，`“重剑无锋，大巧不工”`。也许从大神们的角度和学术的角度来说，Golang有诸多缺点，但是从工程的角度上，它的设计是优秀的：简洁的语法，剔除古怪无用的概念，完整的工具链，强大的社区支持，高性能。

至少我从C++开始学起入门的经历，我觉得Golang的上手实在轻松，写着写着竟有写Python的感觉，想想学C++的苦，眼泪快掉下来。所以国内现在Golang一片欣欣向荣，老板招了人，即便从来没有写过Go，只要基础不错，稍微培训一下也能干活。国外也有很多公司慢慢开始转移技术栈，或者一些新产品直接基于Golang。我们公司SAP在Big Data Hub这个大部门下的新产品，很多都是基于Go和K8的。

我使用Go主要目的是方便研究分布式和存储，为以后的职场挑战做准备。我觉得需要开一个系列慢慢记录我的学习过程，既然是笔记性质的，所以可能有些地方不详细，望谅解。Golang系列的笔记，范围从Golang基础，web和Network的应用，著名项目源码和架构解读再到Toy Project的开发。积跬步，至千里。

## 环境
安装过程就略了。
配置环境，我就直接使用Visual Studio Code和Go的插件，可以自动完成排版，纠错等功能。

## 基础语法
### 类型
基本类型主要有`string`, `bool`, `int(8,16,32或者64位)`, `float`, `byte(uint8)`, `rune(32位)`,`uint(8,16,32或者64位)`，`complex32`，`complex64`。

复合类型，主要有`struct`, `slice`, `array`, `union`, `pointer`, `map`, `channel`, `interface`。至于每个复合类型具体是干什么的，我们来一一探究。

### string
Go中，字符串string相当于只读的byte slice，以下是一些字符串的常见用法。

``` go
package main

import "fmt"

func main() {
	//the way to declare a var, which is suggested in standard
	var s1 string
	s1 = "Apple"
	fmt.Println(s1) //"Apple"
	fmt.Println(s1[3], string(s1[3]), s1[1:3], s1[1:], s1[:2], len(s1))
	// 108 l pp pple Ap 5

	//another way to declare
	s2 := "Pear"

	//Concat
	fmt.Println(s1 + " " + s2) //print concat string "Apple Pear"
	fmt.Println(s1, s2)        //print with multiple items: "Apple Pear"
}
```

### array和slice
在Golang里面，array和slice感觉是很容易混淆的，不管是形式上，还是从我们的直觉认知上。我目前认为array和slice最大的区别是array作为参数传给函数的时候传的copy， 而slice传的是reference。显然，大多数情况slice更有效，因为slice本身就可以看成对应某个array的一部分（或者全部）reference。

``` go
//声明array的时候，需要制定size：
id := [3]int{503, 402, 222}
products := [100]string{}
names := [...]string{"Ben", "Terry"} //让编译器来数个数

//声明slice的时候不需要指定size:
id := []int{503, 402, 222}
products := make([]string, 100)

```

数值Array的未初始化的值是零值，字符串array的未初始化的值是空字符。

slice除了上述声明方法，还可以使用built-in函数make。
> func make([]T, len, cap) []T

比如：
``` go
slice1 := make([]byte, 10) //len = 10, cap = 10
```

传入想生成的类型，长度和capacity，可生成需要的零值slice。如果省略cap值，那么cap将默认和len一样。根据我们第一段的描述，slice和array大致关系可以表示为下图：

![](http://res.cloudinary.com/dxdsd8err/image/upload/c_scale,w_528/v1521356443/slice_k7ubqc.jpg)

通过图示，可以看出capacity其实就是slice对应的array可以访问的末尾，到slice指向array位置之间的距离。


以下是常见的slice用法：

``` go

package main

import (
	"fmt"
	"strings"
)

func main() {
	s := []int{1, 4, 52, 3, 5, 4, -2}
	fmt.Println(len(s)) //7
	fmt.Println(cap(s)) //7

	n1 := s[:4]
	fmt.Println(n1)      //[1 4 52 3]
	fmt.Println(cap(n1)) //7 = 7-0

	n2 := s[3:]
	fmt.Println(n2)      //[3 5 4 -2]
	fmt.Println(cap(n2)) //4 = 7-3

	n3 := s[1:4]
	fmt.Println(n3)      //[4 52 3]
	fmt.Println(len(n3)) //3
	fmt.Println(cap(n3)) //6 = 7 -1

	n4 := s[1:4:5]       //input[low:high:max], max defines where the limits that slice can reach.
	fmt.Println(n4)      //[4 52 3]
	fmt.Println(len(n4)) //3
	fmt.Println(cap(n4)) //4 = 5-1

	//join string list
	strList := []string{"This", "is", "simple"}
	var aggregateStr string
	aggregateStr = strings.Join([]string(strList), "@")
	fmt.Println(aggregateStr) //This@is@simple
	
    //append
	ap := []int{1, 4, 5}
	ap = append(ap, 2, 3, 4, 4)
	fmt.Println(ap) //[1 4 5 2 3 4 4]
	
	//matrix or 2D array
	mat := make([][]int, 3)
	count := 0
	for i := 0; i < 3; i++ {
		mat[i] = make([]int, 2)
		for j := 0; j < 2; j++ {
			mat[i][j] = count
			count++
		}
	}
	fmt.Println(mat) //[[0 1] [2 3] [4 5]]
}

```

### copy
slice的复制，可以直接使用内置的函数copy。

``` go
func copy(dst, src []Type) int
```

copy将src里面的内容，复制到dst里面，并返回复制的长度。复制长度是src和dst之间的最小值。

``` go
	origin := []string{"bfs", "dfs", "dp"}
	new1 := make([]string, 3) //just copy the all 3 elements
	copy(new1, origin)
	fmt.Println(new1) //[bfs dfs dp]

	new2 := make([]string, 2) //just copy the first 2 elements
	copy(new2, origin)
	fmt.Println(new2) //[bfs dfs]

	new3 := make([]string, 5) //just copy the all 3 elements, and rest of dst will be empty.
	copy(new3, origin)
	fmt.Println(new3) //[bfs dfs dp  ]
```

## Take away
这篇Blog主要讲述了我入坑Golang的心路历程和我的初步印象，我的Golang环境，数组array和切片slice， 外加一个built-in copy。接下来近期会记录map， struct, pointer和interface的概念和使用。

## Reference:
[1] [The Go Blog: Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)