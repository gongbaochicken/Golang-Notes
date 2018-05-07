# Golang 101系列基础篇——初识Goroutine

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1522303339/gophercomplex1_duq7fh.jpg)

本文终于来到了多线程的一个主题：Goroutine。在Golang中，Goroutine相当于是一个轻量级的线程，context switch的代价小。我们来看看Goroutine的特点和基本使用，在后期的文章和学习中，我再继续探讨更深入的内容。
<!--more-->

## 特点
在Golang中，Goroutine相当于是一个轻量级的线程（协程），就是一些函数，可以同时运行（Concurrency）。当我们使用go这个关键词来运行一个Goroutine的时候，程序为他专门在Heap专门开辟一片2Kb空间（一般线程大概1Mb）,因此在程序中可以使用大量goroutine（前提是你知道你在做什么），来提高运行效率，充分利用资源。

同时，这也体现其最大的优势：在并发开发中实现对线程池的动态扩展，不会由于某个任务的阻塞而导致死锁。因为当某个goroutine被阻塞，runtime会重新开始一个线程来处理其他goroutine，直到阻塞消失。

## Example
下面给一个很常见的例子：

``` go
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 4; i++ {
		fmt.Println(s)
	}
}

func main() {
	go say("Hi")
	go say("Hola")
	fmt.Println("End")
}

```
我的目的是在主程序里面启动两个goroutine,同时分别执行打印对应的字符串，每个goroutine打印四次。

但是结果只出现了“End”，这是为什么呢?因为在两个goroutine在运行的时候，main已经运行结束了，整个程序也就终止了，所以goroutine的内容什么也没有打印出来。

这个很好办，我们让主程序先休息一下，等等goroutine再结束。同时为了看到goroutine的交叉结果（Interleaving）, 我们让goroutine在每次打印前也稍微缓一缓。

``` go
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 4; i++ {
		time.Sleep(10 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("Hi")
	go say("Hola")
	time.Sleep(100 * time.Millisecond)
	fmt.Println("End")
}

```
在某次运行后，我得到了：

```
Hi
Hola
Hola
Hi
Hi
Hola
Hola
Hi
End
```
很显然，goroutine相互之间出现了interleaving.符合我们对多线程的一般认知。

## WaitGroup
上面这个例子，通过主程序sleep的方法等待两个goroutine的执行结果，未免显得太Naive。这里我们可以引入一个更优雅的方式，使用WaitGroup。

WaitGroup属于sync这个包，这个包顾名思义，就是提供synchronization基本件的一个包。这里面有Mutex, lock, condition, WaitGroup, 读写锁等等，都是比较底层的部件。


[WaitGroup](https://golang.org/pkg/sync/#WaitGroup)是主程序用来等待goroutine执行完毕的。通过使用Add来添加需要等待goroutine的个数，通过Wait来进行阻塞等待goroutine完事，goroutine通过Done来通知主程序说我完事儿了。

于是上面的那个例子，可以进化为:

``` go
var wg sync.WaitGroup

func say(s string) {
	defer wg.Done() //Delta-1
	for i := 0; i < 4; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	wg.Add(1)
	go say("Hi")
	wg.Add(1)
	go say("Hola")
	wg.Wait() //Delta = 2, so need to wait 2 goroutines to finish.

	wg.Add(1)
	go say("Yeah")
	wg.Wait()
}
```

某次运行的结果如下：
```
Hola
Hi
Hi
Hola
Hola
Hi
Hi
Hola
Yeah
Yeah
Yeah
Yeah
```

可以看到say("Hi")和say("Hola")被添加进了第一次wg,所以wait阻塞结束是在两者都交叉运行完了之后，再继续往下执行say("Yeah")。所以我们会看到Hi, Hola交叉运行，而Yeah是单独连续print的。

可以看到WaitGroup相当于是内部有个计数器delta, Add是+1， Done是-1,当Delta重新回到0时，Wait结束。按照我的理解，画个草图相当于这样：
![](http://res.cloudinary.com/dxdsd8err/image/upload/c_scale,w_948/v1522302921/1.pic_xsnlt0.jpg)

后面我们结合上面的例子结构再讨论讨论lock, mutex相关的内容，预告里面还有channel，之后可能会深入分析一下routine源码和scheduler。