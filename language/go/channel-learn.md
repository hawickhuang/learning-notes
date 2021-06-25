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

## 源码学习

下面是一个简单的使用channel在两个协程通信的例子：

```go
package main

import (
	"fmt"
)

func main()  {
	c := make(chan int)
	go func() {
		c <- 1
	}()
	x := <-c

	fmt.Println("x=", x)
}
```

翻译后的汇编指令：

```armasm
main_pc0:
        ...
        CALL    runtime.makechan(SB)
        ...
        CALL    runtime.newobject(SB)
        ...
        CMPL    runtime.writeBarrier(SB), $0
        ...
        JMP     main_pc101
main_pc85:
        ...
        CALL    runtime.gcWriteBarrierCX(SB)
main_pc101:
        ...
        CALL    runtime.newproc(SB)
        ...
        CALL    runtime.chanrecv1(SB)
        ...
        CALL    runtime.convT64(SB)
        ...
        CALL    fmt.Fprintln(SB)
        ...
main_pc215:
        ...
        CALL    runtime.morestack_noctxt(SB)
        ...
        JMP     main_pc0
main_func1_pc0:
        ...
        CALL    runtime.chansend1(SB)
       	...
        RET
main_func1_pc47:
        ...
        CALL    runtime.morestack(SB)
        ...
        JMP     main_func1_pc0
os_File_close_pc0:
        ...
        JNE     os_File_close_pc69
os_File_close_pc29:
        ...
        CALL    os.(*file).close(SB)
        ...
        RET
os_File_close_pc52:
        ...
        CALL    runtime.morestack_noctxt(SB)
        ...
        JMP     os_File_close_pc0
os_File_close_pc69:
        ...
        JMP     os_File_close_pc29
```

上面我把`CALL`调用的都留下来了，删除了其它的指令，在上面，其实我们主要关注三个`CALL`调用：

- `runtime.makechan(SB)`: 对应我们go代码中的`make(chan int)`
- `runtime.chansend1(SB)`: 对应我们go代码中的 `c <- 1`
- `runtime.chanrecv1(SB)`: 对应我们go代码中的 `x := c <- 1`

我们现在去go的源代码中查看这三个方法的实现，路径是`src/runtime/chan.go`.

### `hchan`结构体

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

可以将`hchan`结构体中的属性大致分为三种类型：

- buffer相关属性：
  - `buf`:  存储数据的缓冲区
  - `dataqsiz`: 用户构造时指定的缓冲区大小
  - `qcount`: 缓冲区中已有的元素数量
- waitq相关属性，一种FIFO的队列：
  - `recvq`: 等待接收数据的goroutine
  - `recvx`: 缓冲区中已接收的索引位置
  - `sendq`: 等待发送数据的goroutine
  - `sendx`: 缓冲区中已发送的索引位子
- 其它属性：
  - `elemsize`: 元素的大小
  - `elemtype`: 元素的类型信息
  - `closed`:channel是否已关闭， 0表示已关闭 
  - `lock`: 锁变量

### makechan

makechan主要的功能是对参数进行合法性检测，再分配内存：

- 类型大小的检测，要小于2^16
- 对齐检测
- 申请的大小检测，是否会大于堆空间
- 分配内存
- 返回hchan指针

### chansend

chansend1函数会调用chansend，chansend是channel中发送数据的真正实现。

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	if c == nil {
		if !block {
			return false
		}
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	if debugChan {
		print("chansend: chan=", c, "\n")
	}

	if raceenabled {
		racereadpc(c.raceaddr(), callerpc, funcPC(chansend))
	}

	// Fast path: check for failed non-blocking operation without acquiring the lock.
	//
	// After observing that the channel is not closed, we observe that the channel is
	// not ready for sending. Each of these observations is a single word-sized read
	// (first c.closed and second full()).
	// Because a closed channel cannot transition from 'ready for sending' to
	// 'not ready for sending', even if the channel is closed between the two observations,
	// they imply a moment between the two when the channel was both not yet closed
	// and not ready for sending. We behave as if we observed the channel at that moment,
	// and report that the send cannot proceed.
	//
	// It is okay if the reads are reordered here: if we observe that the channel is not
	// ready for sending and then observe that it is not closed, that implies that the
	// channel wasn't closed during the first observation. However, nothing here
	// guarantees forward progress. We rely on the side effects of lock release in
	// chanrecv() and closechan() to update this thread's view of c.closed and full().
	if !block && c.closed == 0 && full(c) {
		return false
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

	lock(&c.lock)

	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}

  // 若recvq不为空，则从recvq中取出头部goroutine，并将数据发送给该goroutine
	if sg := c.recvq.dequeue(); sg != nil {
    // chan内部会做内存拷贝
    // 并调研goready，唤醒recvq中的等待接收的goroutine
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}

  // 缓冲区未满，将元素加入缓冲区
	if c.qcount < c.dataqsiz {
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			raceacquire(qp)
			racerelease(qp)
		}
		typedmemmove(c.elemtype, qp, ep)
    // 修改偏移量，环形缓冲区
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}

	if !block {
		unlock(&c.lock)
		return false
	}

	// 无缓冲channel，或缓冲已满
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// 将当前goroutine及channel中的数据放入sendq队列中
	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
	c.sendq.enqueue(mysg)
	// 将goroutine转入waiting状态，并解锁
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
	// Ensure the value being sent is kept alive until the
	// receiver copies it out. The sudog has a pointer to the
	// stack object, but sudogs aren't considered as roots of the
	// stack tracer.
	KeepAlive(ep)

	// 当前goroutine被唤醒，做异常处理；若无异常释放资源，继续执行
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if gp.param == nil {
		if c.closed == 0 {
			throw("chansend: spurious wakeup")
		}
		panic(plainError("send on closed channel"))
	}
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	mysg.c = nil
	releaseSudog(mysg)
	return true
}
```

### chanrecv

chanrecv1在内部也通过调用chanrecv，chanrecv实现了真正的接收数据的功能：

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	if debugChan {
		print("chanrecv: chan=", c, "\n")
	}

	if c == nil {
		if !block {
			return
		}
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

  // 通道未准备好
	if !block && empty(c) {		
		if atomic.Load(&c.closed) == 0 {
			return
		}
    // double check
		if empty(c) {
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

	lock(&c.lock)

	if c.closed != 0 && c.qcount == 0 {
		if raceenabled {
			raceacquire(c.raceaddr())
		}
		unlock(&c.lock)
		if ep != nil {
			typedmemclr(c.elemtype, ep)
		}
		return true, false
	}

  // sendq队列有goroutine
	if sg := c.sendq.dequeue(); sg != nil {
    // recv会先判断缓冲区是否为0，不为0从缓冲区去元素，再将sg的数据塞入缓冲区尾部
		recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true, true
	}

  // 缓冲区有元素
	if c.qcount > 0 {
		// 直接从缓冲区头部取元素
		qp := chanbuf(c, c.recvx)
		if raceenabled {
			raceacquire(qp)
			racerelease(qp)
		}
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		typedmemclr(c.elemtype, qp)
    // 管理索引
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}

	if !block {
		unlock(&c.lock)
		return false, false
	}

	// 没有可接收数据，阻塞当前goroutine
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// No stack splits between assigning elem and enqueuing mysg
	// on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	c.recvq.enqueue(mysg)
	// Signal to anyone trying to shrink our stack that we're about
	// to park on a channel. The window between when this G's status
	// changes and when we set gp.activeStackChans is not safe for
	// stack shrinking.
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

	// goroutine被唤醒，异常处理；释放资源或继续运行
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	closed := gp.param == nil
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, !closed
}

```

## 总结

channel是go开发中最常见的技术之一，其中用到的环形缓冲区、数据接收和发送的管理和加锁逻辑，对我们开发出高性能高并发的程序有很大帮助。

其中关于goroutine状态切换的源码还没看，所以未做解释，后面准备再写一篇文章来学习。

## 参考文档：

https://github.com/golang/go/blob/master/src/runtime/chan.go

https://www.cyhone.com/articles/analysis-of-golang-channel/