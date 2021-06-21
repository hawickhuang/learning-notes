# Go Channel 学习

Channel是Golang中非常重要的特性之一，它相当于一个FIFO的队列，主要用于Golang中Goroutine之间的通信。

Golang实现了CSP并发模型，有关CSP并发模型的有本书([cspbook.pdf](http://www.usingcsp.com/cspbook.pdf))；有能力的可以去拜读一下......CSP模型由并发执行的实体（线程或者进程）所组成，实体之间通过发送消息来进行通信，在这里发送消息即是通过读写channel来实现。

## Channel类型

channel的定义格式：

```go
ChannelType = ("chan" | "chan<-" | "<-chan") ElementType
```

包括关键字chan、可选的 **<-** 代表channel的读写属性，如果没有指定 **<-**,那么channel就是可读可写的。

```go
chan T // 可以接收和发送类型为T的数据
chan<- T // 只可以用来发送（写）类型为T的数据
<-chan T // 只可以用来接收（读）类型为T的数据
```

使用make关键字来初始化channel并设置容量：

```go
c := make(chan int, 3) // 表示创建一个容量为3，可读可写的数据为int类型的channel
```

容量是channel缓冲的大小，我们一般会依据容量来分为两种channel：

- 无缓冲channel：容量为0；只有在sender和receiver都准备好后才进行通信，其它情况阻塞；
- 有缓冲channel：容量大于0；sender在容量未满时可以一直写，在容量满时被阻塞；receiver在容量无数据时阻塞，其它情况可读；

## Channel操作

### 创建

1. 使用make关键字创建

   ```go
   c1 := make(chan int, 3) // 缓冲大小为3
   c2 := make(chan int) // 无缓冲
   ```

2. 只声明，对未初始化的channel进行读写，会触发deadlock，即都被阻塞，Golang运行时发现deadlock，触发fatal error

   ```go
   var c chan int
   ```

### 读写

1. 读

   ```go
   x <- c // 将channel c中的数据读到变量x中，若c中无数据会被阻塞
   x, ok <- c // 同上，不过可以用ok来判断channel c是否已close；若ok=false，则已被close且数据已被读完
   ```

2. 写

   ```go
   c <- x // 将变量x的值写入channel c中，若channel c容量已满，会被阻塞
   ```

### 关闭

关闭channel可以通过built-in函数close来实现：

```go
close(c)
```

在关闭channel时有几个注意事项：

- 关闭已被关闭的channel，会触发panic；
- 向已关闭的channel写数据，会触发panic；
- 已关闭的channel，若还有数据，则仍可以读取；可以使用前问 `读` 示例中的ok-idiom方式来判断；

## 使用方法

### Goroutine通信

如下面代码所示，主协程创建一个chan int类型channel，新起一个协程向c中写入1；主协程等待从c中读取数据并打印；

```go
func Test_channel1(t *testing.T) {
	c := make(chan int)
	go func() {
		c <- 1
	}()
	t.Logf("x = %d", <-c) // 这里读会阻塞，知道可以读取到数据
  close(c) // 日常使用用要习惯去关闭channel
}
```

### Select

Select 可以实现对多个channel的监听，但每次运行只会选择任意一个满足可读的case执行；如下面代码所示，创建了两个协程，并分别新起协程进行写入；在select中监听两个channel；运行的话，只会运行某一个case；

```go
func Test_channel2(t *testing.T) {
	c1 := make(chan int)
	c2 := make(chan string)

	go func() {
		c1 <- 1
	}()
	go func() {
		c2 <- "string"
	}()

	select {
	case data, ok := <-c1:
		t.Logf("receive from c1, data: %d, ok: %t", data, ok)
	case data, ok := <-c2:
		t.Logf("receive from c2, data: %s, ok: %t", data, ok)
	}
}
```

如果要持续运行select对多个channel持续监听，则可以在select外层加一个for循环；但要定义for循环的退出条件，不然会触发deadlock；下面代码通过添加一个定时器来触发timeout，也可以定义多一个channel来，不过这个时候可以将for循环包装进go func()中，原理跟定时器一样。

```go
func Test_channel2(t *testing.T) {
	c1 := make(chan int)
	c2 := make(chan string)

	go func() {
		c1 <- 1
	}()
	go func() {
		c2 <- "string"
	}()

	for {
		select {
		case data, ok := <-c1:
			t.Logf("receive from c1, data: %d, ok: %t", data, ok)
		case data, ok := <-c2:
			t.Logf("receive from c2, data: %s, ok: %t", data, ok)
		case <-time.After(time.Second*3):
			t.Logf("timeout")
			return
		}
	}
}
```

### Range 

for range 语句也可以用来处理channel，当channel被关闭时，range会退出迭代循环。如果不关闭close，for-range会一直被阻塞；在这里会产生deadlock，如果把for-range包装进协程中，则在1秒后主协程退出，其它协程也会退出；

```go
func Test_channel3(t *testing.T) {
	c := make(chan int)
	go func() {
		for i:=0; i<10;i++{
			c <- i
		}
		close(c)
	}()

	for i := range c {
		t.Logf("data: %d", i)
	}
	time.Sleep(time.Second)
}
```

## 源码拜读



