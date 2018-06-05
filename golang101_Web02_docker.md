# Golang 101系列网络篇——Docker化一个go web app

本文尝试使用两种Dockerfile讲一个简单的Golang Web app打包成Container运行。

## A basic go web app

首先先写一个最简单的HTTP go app，通过HTTP监听，让http response writer写入一个hardcode信息。通过登录http://localhost:8080可以看到该信息。该文件命名为main.go.

```
package main

import (
	"fmt"
	"net/http"
)

//Anyting has ServeHTTP method is a Handler interface responds to an HTTP request.
type anyInterface int

//ServeHTTP should write reply headers and data to ResponseWriter and then return
func (a anyInterface) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Any thing server writes back to response writer")
}
func main() {
	var notSure anyInterface
	http.ListenAndServe(":8080", notSure)
}

```
再确认无误之后，我们可以开始Docker化该app.

## Use Onbuild Dockerfile
主要步骤就是先写Dockerfile, 然后Docker build, 最后在Docker run。

Dockerfile一定使用的是FROM开头，意味着Docker必须构建于已经存在的Docker架构之上。对于这种简单的应用和我们这里的情况，我们直接可以使用一个单纯的Golang的Dockerfile来build。这里Dockerfile有各种不同版本的选择。

我们可以先使用一种最简单的Dockerfile来玩玩。

```
# docker file test
From golang:1.8-onbuild
```

Onbuild这种版本假设我们的Go App是按照最标准的结构写的，因此wrap了一些必要的额操作，当我们build之后，可以直接使用。

Docker build:
![](http://res.cloudinary.com/dxdsd8err/image/upload/v1528180519/docker1_tqpcvr.jpg)

Docker run:
![](http://res.cloudinary.com/dxdsd8err/image/upload/v1528180519/docker2_clsrmi.jpg)

通过浏览器登陆本地8080端口，可以看到如下信息，证明我们的Docker化已经成功。
![](http://res.cloudinary.com/dxdsd8err/image/upload/v1528180519/docker3_r2c9kt.jpg)

这里使用了 -p A:B 指定了端口A是机器和浏览器中的等待的端口，而B是程序APP服务的端口。比如，如果使用了 -p 3000:8080, 因为main.go程序里面指明了8080，而这时打开浏览器，可以在localhost:3000找到我们期待的页面。


## Use Standard Dockerfile
对于标准的Dockerfile,可以从官网上找到对应所需的版本号，在第一行进行FROM引用。同时需要创建一个新的workdir, 并需要在Dockerfile中指明编译（当然也可以在执行时，而不是在Docker build时）。最后需要指明Docker run时，使用的运行命令。

```
From golang:1.9-stretch
MAINTAINER jiazhuo0528@gmail.com
 
RUN mkdir /app 
ADD . /app/ 
WORKDIR /app 
RUN go build -o main . 
CMD ["/app/main"]
```
其他步骤跟之前一样。

## alpine vs stretch
在众多Dockerfile版本中，你会发现有一个系列的标记为alpine。Alpine是一个轻量级的Linux发行版，用来减小系统的体积和运行时资源消耗，因此得到开源社区越来越多的青睐。

我们可以直接采用golang:1.9-alpine3.7和我们之前使用的golang:1.9-stretch来编译Dockerfile来进行比较，可发现alpine真的减少了很多！

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1528180519/docker4_zvwpny.jpg)

这种体积的精简，意味着可以在更精简的Linux server（比如CoreOS）上面运行，对于实际生产中的作用不言而喻。

## 小结
通过Dockerfile，我们可以将应用打包成一个个Docker image，每个image可以通过指定使Container运行在某一个端口。在实际生产中，部分的Container可以运行在不同的server，实现了Microservices的基础结构。