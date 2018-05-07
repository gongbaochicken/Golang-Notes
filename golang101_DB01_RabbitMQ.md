# Golang和RabbitMQ

![](http://res.cloudinary.com/dxdsd8err/image/upload/c_scale,w_517/v1522740717/rabbitmq_ezbhgc.png)

在微服务称霸的今日，Message Broker和消息队列被各大公司广泛应用于业务解耦、可靠投递、广播等等。主流的消息队列有Kafka, ActiveMQ, RabbitMQ, RocketMQ, NSQ等等。本期主要总结一些RabbitMQ的一些基本理论和Golang的基本操作。

我最早接触RabbitMQ其实使用的是CloudAMQP，一套RabbitMQ云服务。云服务的好处是它帮你做好了host等工作，你只需要掌握相关API，并专注于自己的业务逻辑即可，使用非常方便。当时我使用它（同时使用的Python和Pika）的目的是，将短时间大量产生的消息，存入消息队列，等待某些服务从队列中Consume，这样一来可以解耦不同的服务，二来可以为消耗方减缓系统压力。

CloudAMQP是基于RabbitMQ的, 而RabbitMQ则是基于AMQP(Advanced Message Queuing Protocol)，一套由Chase银行制定和研发的国际化信息通道传递的标准。这套标准设计了关于消息队列的安全、可靠性、交互性等等一些基本特质，是消息队列的一个重要里程碑。CloudAMQP可以在之后专门再总结一下，本文还是专注于RabbitMQ和Golang。

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1522740717/camqp_ymawip.png)

## 特点
RabbitMQ有以下主要特点：

* 支持多语言开发，支持多工具部署。
* 支持多协议，包括不同版本的AMQP，MQTT, HTTP。
* 支持分布式和高可用。
* 支持企业级和云端服务。
* 良好的工具链和监控平台。

而作为Message Broker, RabbitMQ有以下几种信息交换的模式：

* Direct Exchange: 直接通过信息中携带的routing key对应的binding key来决定传向具体的某个队列。
* Fanout: 复制接收到的消息，并传入所有绑定了的队列中，而无视routing key。
* Topic Exchange: 消息将初入一个或者多个队列，通过消息的routing key的模式匹配（比如定义了前缀）。这样一个消息将会被同一个主题的多个队列所消耗。
* Headers: 消息通过头部信息进行分类。

关于交换方式，你可以在[这里](https://www.cloudamqp.com/blog/2015-09-03-part4-rabbitmq-for-beginners-exchanges-routing-keys-bindings.html)找到一个更详细的教程。

## Mac安装
使用万能的homebrew:
> brew install rabbitmq

成功之后，找到安装目录下的sbin文件夹（比如我的位置在：/usr/local/Cellar/rabbitmq/3.7.4/sbin）。启动server之后，默认是运行在http://localhost:15672/。

通过浏览器，你会进入登陆界面，默认用户名和密码都是guest。成功登陆之后，你会看到整个RabbitMQ的管理界面，里面你可以看到所选的exchange模式，channel具体的内容等等。

![](http://res.cloudinary.com/dxdsd8err/image/upload/v1522743577/64.pic_jqexc7.jpg)

## Go的第一个例子
第一次使用Golang来操作RabbitMQ，有点小激动，哈哈。直接研究了一个官方例子，并附上一些必要的注解。这个是非常基础的例子：

``` go
package main

import (
	"log"

	"github.com/streadway/amqp"
)

//Helper dealing with error message
func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	//Dial creates a connection
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	//conn.Channel creates a new channel, which may create multiple queues later
	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	//Make a new queue, with name as "New Queue" and durable.
	q, err := ch.QueueDeclare(
		"New Queue", // name
		true,        // durable
		false,       // delete when unused
		false,       // exclusive
		false,       // no-wait
		nil,         // arguments
	)
	failOnError(err, "Failed to declare a queue")
	log.Println("Now we create a new queue:", q.Name)

	//Publish sends a Publishing from the client to an exchange on the server.
	body := "hello2"
	for i := 0; i < 5; i++ {
		err = ch.Publish(
			"",     // exchange
			q.Name, // routing key
			false,  // mandatory
			false,  // immediate
			amqp.Publishing{
				ContentType: "text/plain",
				Body:        []byte(body),
			})
		log.Printf(" [x] Sent %s", body)
		failOnError(err, "Failed to publish a message")
	}
	log.Println("Now we have", q.Messages, "messages in our queue.")
}
```

其中Queue比较简单,只有三个成员。
``` go
type Queue struct {
    Name      string // server confirmed or generated name
    Messages  int    // count of messages not awaiting acknowledgment
    Consumers int    // number of consumers receiving deliveries
}
```

得到的log结果：

```
jiazhuos-MBP:rabbitMQ jiazhuo$ go run main.go
2018/04/03 23:51:16 Now we create a new queue: New Queue
2018/04/03 23:51:16  [x] Sent hello2
2018/04/03 23:51:16  [x] Sent hello2
2018/04/03 23:51:16  [x] Sent hello2
2018/04/03 23:51:16  [x] Sent hello2
2018/04/03 23:51:16  [x] Sent hello2
2018/04/03 23:51:16 Now we have 10 messages in our queue.
```
同时查看之前启动的RabbitMQ的Dashboard，我们可以查到新建的queue已经成功，同时已经有多个Message已经准备好（Ready状态）。
![](http://res.cloudinary.com/dxdsd8err/image/upload/c_scale,w_1208/v1522825653/1.pic_m080sq.jpg)
