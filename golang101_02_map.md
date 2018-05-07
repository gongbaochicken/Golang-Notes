# Golang 101系列基础篇-Map

![](http://res.cloudinary.com/dxdsd8err/image/upload/c_scale,w_500/v1521526971/gopher1_dtlosg.jpg)

## Map
Golang中的map是用hashmap来实现的，源码在runtime/hashmap.go。

map声明的方法主要有两种，一种是直接赋值，预先塞进一些值在Map中，另一种方法是使用make先预留出memory，再一个个的key-value插入map。


``` go
	// This is wrong way to assign after 'declare' like this, which cause panic: assignment to entry in nil map
	// var m1 map[int]string
	// m1[2] = "Jeff"

	//Method 1
	m1 := map[int]string{2: "Jeff", 4: "Gao"}

	//Method 2, equivalent to m := map[int]string{}
	m2 := make(map[int]string)
	m2[2] = "Jeff"
	m2[4] = "Gao"

```

map的用法也相对比较简单，主要就是查询，插入和删除K-V值，具体如下：

``` go
	m1 := map[int]string{2: "Jeff", 4: "Gao"}

	//Retrive
	employee1 := m1[2]
	fmt.Println(employee1) //"Jeff"
	employee2 := m1[3]
	fmt.Println(employee2) //nil, because its k-v pair doesn't exist

	//Iterate
	for k, v := range m1 {
		fmt.Println("key = ", k, "value = ", v)
	}

	//Delete
	delete(m1, 2)
	fmt.Println(m1[2])   //nil
	fmt.Println(len(m1)) //1

```

其中delete是内置的函数，用来从map中删除key。如果map本身是空的或者无需要删除的key，那么就直接退出什么都不做。

## make vs new
这里想比较一下make和new的用法。

当我们声明一个变量但是没有赋值时，变量会保留默认的零值。对于引用类型，他们零值是nil。所以对于一个引用类型的变量，我们直接赋值就会出现runtime panic(如果你尝试过官网的map里面的那个例子，你就会秒懂)。

那么对于引用类型(map, slice, channel)的初始化，除了声明，我们还需要在Heap堆上面给他们分配内存空间，在Golang里面会用到new或者make。

那么他们区别是什么呢？主要是区别是使用的对象不同，make只用于slice, map, channel，返回的是引用类型的本身；而new主要是用于值类型的变量或者struct需要引用时分配内存，返回指向类型的指针。

``` go
//make
//allocates and init a hash map data structure and returns a map value that points to it.
m := make(map[string]int)

//new
var i *int
i = new(int)
*i = 6

//or var i = new(int), omit *int because it can be referred from right side 
```

## Set?
细心的你，一定很快会发现，在Golang的标准库里面没有set这种数据结构的，这让很多人感觉不太适应。Golang不支持set, 主要理由是Golang没有泛型。

好吧，那么怎么表示set这种数据结构的呢？答案是用map来代替, 要点是key-value中的value使用bool值代表key是否存在：

``` go
	s := make(map[string]bool)
	s["John"] = true
	s["Sam"] = true
	fmt.Println(s["John"]) //true
	fmt.Println(s["Sam"])  //true
	fmt.Println(s["Kim"])  //false

	delete(s, "Sam")
	fmt.Println(s["Sam"]) //false
```
