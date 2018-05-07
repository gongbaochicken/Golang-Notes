# Golang 101系列基础篇——Useful functions

在本篇笔记中，我将总结一些比较有用的函数，有些是原生内置(built-in),有些则是需要引用外库。这些函数就像是建造一栋楼的基石或者水泥一样，能够完成一些很重要的功能。

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1521526970/de4daf20b7e43fc4bca3450d86a1a32c_alreuy.png)


## Copy
在前面的几期中，我们使用到了一个copy函数，今天我们更深入地来探究一下。copy函数的形式如下：

> func copy(dst, src []Type) int 

主要作用是将src slice里面的元素复制到dst slice里面，返回值是成功复制的元素个数。当src比dst长时，只复制前面部分信息，超过的部分不进行复制。我们来看一个例子：


``` go
package main

import (
	"fmt"
)

func main() {
	d := make([]int, 4)
	s := []int{3, 4, 5}
	fmt.Println("Before copying: ", d) //Before copying:  [0 0 0 0]
	fmt.Println("Copy length is: ", copy(d, s)) //Copy length is:  3
	fmt.Println("After copying: ", d) //After copying:  [3 4 5 0]

	d = make([]int, 4)
	s = []int{3, 4, 5, 6, 7, 2}
	fmt.Println("Before copying: ", d) //Before copying:  [0 0 0 0]
	fmt.Println("Copy length is: ", copy(d, s)) //Copy length is:  4
	fmt.Println("After copying: ", d) //After copying:  [3 4 5 6]
}

```

看起来非常Easy，但是如果两个slice是相互交叉的,又会是什么情况呢。下面是两个例子：

``` go
package main

import (
	"fmt"
)

func main() {
	//Example 1
	d := []int{1, 2, 3}
	s := d[1:]
	fmt.Println("Before src:", s, "dest:", d) //Before src: [2 3] dest: [1 2 3]
	fmt.Println("Copy length is: ", copy(d, s)) //Copy length is:  2
	fmt.Println("After src:", s, "dest:", d) //After src: [3 3] dest: [2 3 3]

	//Example 2
	s = []int{1, 2, 3}
	d = s[1:]
	fmt.Println("Before src:", s, "dest:", d) //Before src: [1 2 3] dest: [2 3]
	fmt.Println("Copy length is: ", copy(d, s)) //Copy length is:  2
	fmt.Println("After src:", s, "dest:", d) //After src: [1 1 2] dest: [1 2]

```
注意slice其实都是对应array的reference，所以我们可以画一个图来表示：

![](http://res.cloudinary.com/dxdsd8err/image/upload/c_scale,q_70,w_861/v1522050620/2_q2cdyc.jpg)

在第一个例子中，dest slice指向array的头array[0]，src slice指向array[1],这时要求src往dest slice里面复制，则其实是让
> array[0] = array[1], arrray[1] = array[1]

于是array变成了[2, 3, 3]，所以dest slice就变成了[2,3,3], src slice从array[1]开始算，即[3，3]。

同理可以推论第二个例子，再次就不再赘述。

## Sort

实用的sort函数来自于Golang中的一个sort包，通过import来引用。

首先来看两个简单的例子，分别递增排序和降序排列string slice和int slice。

``` go
package main

import (
	"fmt"
	"sort"
)

func main() {
	//sort string slice
	s := []string{"shcool", "cats", "apple", "beats"}
	sort.Strings(s)
	fmt.Println(s)                                   //[apple beats cats shcool]
	fmt.Println("Sorted:", sort.StringsAreSorted(s)) //Sorted: true

	//Deceding sort string slice
	sort.Sort(sort.Reverse(sort.StringSlice(s)))
	fmt.Println(s)                                   //[shcool cats beats apple]
	fmt.Println("Sorted:", sort.StringsAreSorted(s)) //Sorted: false

	//sort int slice
	i := []int{1, 44, 2, 32, 25, -1}
	sort.Ints(i)
	fmt.Println(i)                                //[-1 1 2 25 32 44]
	fmt.Println("Sorted:", sort.IntsAreSorted(i)) //Sorted: true

	//Deceding sort int slice
	sort.Sort(sort.Reverse(sort.IntSlice(i)))
	fmt.Println(i)                                //[44 32 25 2 1 -1]
	fmt.Println("Sorted:", sort.IntsAreSorted(i)) //Sorted: false
}
```


在[源码](https://golang.org/src/sort/sort.go)中, 可以通过实现以下interface来自定义一些排序方法，需要实现的方法有Len, Less, Swap这三个函数，完成之后则既是实现了该接口。
``` go
// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package. The methods require that the
// elements of the collection be enumerated by an integer index.

type Interface interface {

	// Len is the number of elements in the collection.
	Len() int

	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool

	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

下面是一个简单的自定义排序例子，通过结构体中的age来排序。

``` go
//Define people struct
type People struct {
	name string
	age  int
}

//Define interface as slice of People
type NameBook []People

func (nb NameBook) Len() int { return len(nb) }

func (nb NameBook) Less(i, j int) bool { return nb[i].age < nb[j].age }

func (nb NameBook) Swap(i, j int) { nb[i], nb[j] = nb[j], nb[i] }

//Sort by age
nb := []People{
	People{"Tony", 24},
	People{"Staney", 29},
	People{"Witz", 11},
}

fmt.Println(nb) //[{Tony 24} {Staney 29} {Witz 11}]
sort.Sort(NameBook(nb))
fmt.Println(nb) //[{Witz 11} {Tony 24} {Staney 29}]

```

## defer
defer在Golang里还挺常见的，主要作用根据名字就能猜到，可以用来延迟函数的执行。实际用处上，主要是用在刚申请完资源（lock, connection等）后，使用defer清理过程来保证在程序结束后，一定会执行清理资源，从而避免因为程序员的疏忽 OR 程序提前中断而没有释放资源。

第一个例子是拿到锁之后，立马defer锁的开锁，避免在程序后面，因为各种原因忘记释放锁。

``` go
package main

import (
	"fmt"
	"sync"
)

var mu sync.Mutex
var m = make(map[string]int)

//Use mutex to prevent condition race, so no one else can write, when this get method is reading.
func read(key string) int {
	mu.Lock()
	defer mu.Unlock() //defer the lock, in case exception
	return m[key]
}

func main() {
	m["Jack"] = 24
	fmt.Println(read("Jack")) //24
}

````

另一个例子是在TCP连接中，当TCP建立后，一个listener监听在特定端口，那么在拿到listener之后，一般会选择使用
> defer listener.Close()
来避免忘记关链接。


当有多个defer同时存在时，defer延迟函数执行的顺序是跟stack一样的“后入先出“，请看下面这个小例子：

``` go
package main

import "fmt"

func main() {
	defer fmt.Println("1")
	defer fmt.Println("2")
	defer fmt.Println("3")
}
```

上述代码所得到的结果是：
```
3
2
1
```