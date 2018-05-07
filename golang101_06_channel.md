# Golang 101系列网络篇——初识Channel

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1522303664/index_ifnqdy.jpg)

之前我们初次聊了一下[goroutine](https://happyisotope.herokuapp.com/golang101_routine),本文来聊聊相关的channel这个神奇的东西。channel主要用于多协程同步的情况，提供了一个交流机制，方便goroutine之间相互沟通。

## 声明
channel分三种，双向，只读，只写。他们的声明方式都是通过make来声明的，具体语法分别是：

```
ch0 := make(chan int, 3)//双向, buffered
ch1 := make(chan int)//双向, unbuffered

ch2 := make(<-chan int) //只读，且读出来的是int

ch3 := make(chan-< int) //只写，且直往里面写int
ch4 := make(chan<- chan int) //只写，且直往里面写另一个int的channel
```

当make的时候，在Heap上面分配了一个结构体的空间，并返回指向它的指针。所以其实得到的channel其实是<b>指针</b>，这也就是解释了为什么可以在函数间轻松地传来传去，而不需要指明使用指针，因为本身它就是指针。

如果声明时，指明了箭头，那么就代表要么只读，要么只写，具体看箭头方向是指向channel，还是从channel指出。同时箭头，本身也代表了读写操作。

``` go
ch <- true //写值
isDead := <-ch //读值并赋值
```

## Buffered Channel
在探讨channel之前，我们需要了解他的结构。以buffer channel为例子，它的结构体内有一个buffer队列Q，大小为buffer size, 有一个锁，还有一些Index。

每当只定了channel的size， 也就同时指定了buffer queue的size。当传入channel的个数超过队列size时，会block掉当前goroutine，直到有receiver来接收channel里面的信息，每接收一个信息，channel就会pop出该信息，然后被阻塞的goroutine即可继续运行（channel内信息数量没有超过size）。看一个例子：

``` go
func main() {
	message := make(chan string, 2) // no buffer
	count := 4

	go func() {
		for i := 0; i < count; i++ {
			fmt.Println("Send message")
			message <- fmt.Sprintf("Message %d", i)
		}
	}()

	time.Sleep(time.Second * 5)

	for i := 0; i < count; i++ {
		fmt.Println(<-message)
	}
}

```

运行结果有可能会出现：
```
Send message
Send message
Send message
Message 0
Message 1
Message 2
Send message
Message 3
```

首先往channel里面扔了3个message，还没到主进程读取，主进程在睡觉。当第四个message传入channel时，整个goroutine被堵塞，正好又恰逢主程序刚刚醒了，于是输出channel里面已有的Message内容。同时channel里面buffer空了出来，整个routine可以继续下去，channel可以接收第四个message，之后再主进程读取打印。

如果主进程需要从channel输出的信息个数大于我们从goroutine中往channel扔进去的信息个数呢？这时会出现deadlock！

## 一个死锁的例子
还有一个初学者常会误会导致的死锁问题。我们来看一个例子：

``` go
func main() {
	ch := make(chan int)

	ch <- 999 // write to a channel
	fmt.Println(<-ch)
}
```
当运行时，我们会得到"fatal error: all goroutines are asleep - deadlock!"这么一个死锁错误。原因是因为整个主程序都是单线程，当写入channel时，必须要有一个接收方在另一个协程，不然会一直block。

为了让程序work,我们可以直接在goroutine里面写，在主程序中接收：
``` go
func main() {
	ch := make(chan int)

	go func() {
		ch <- 999 // write to a channel
	}()

	println(<-ch) // read from a channel
}
```
这样就可以通过了。


## range
channel里面的内容是可以通过range来遍历的。

``` go
package main

import (
	"fmt"
	"time"
)

func getMsg(count int, message chan string) {
	for i := 0; i < count; i++ {
		fmt.Println("Send message")
		message <- fmt.Sprintf("Message %d", i)
	}
	//close a channel to indicate that no more values will be sent. 
	close(message)
}

func main() {
	message := make(chan string, 3) // no buffer

	go getMsg(cap(message), message)

	time.Sleep(time.Second * 4)

	fmt.Println("The cap of channel now is", cap(message))

	for i := range message {
		fmt.Println(i)
	}
}

```

如果我们往已经关闭的channel里面继续发送，会产生panic。比如将上面程序改成：

``` go
package main

import (
	"fmt"
	"time"
)

func getMsg(count int, message chan string) {
	for i := 0; i < count; i++ {
		fmt.Println("Send message")
		message <- fmt.Sprintf("Message %d", i)
	}
	//
	close(message)
}

func method2(message chan string) {
	message <- "ABC"
}

func main() {
	message := make(chan string, 3) // no buffer

	go getMsg(cap(message), message)

	time.Sleep(time.Second * 4)
	
	//Cause panic
	go method2(message)

	fmt.Println("The cap of channel now is", cap(message))

	for i := range message {
		fmt.Println(i)
	}

}

```


## 推荐资料
[Youtube GopherCon 2017: Kavya Joshi - Understanding Channels
](https://www.youtube.com/watch?v=KBZlN0izeiY)
