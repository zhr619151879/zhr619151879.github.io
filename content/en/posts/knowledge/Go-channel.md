---
title: "Golang Channel那些事~"
date: 2021-03-07 10:38:34
description: 揭秘Golang channel

tags:
-
series:
-
categories:

- Golang

image: images/feature1/wave.png
---



> Go 语言中的一大利器那就是能够非常方便的使用 go 关键字来进行各种并发，而并发后又必然会涉及通信。
> Channel 自然而然就成为了 Go 语言开发者中必须要明白明了的一个 “东西” 了，更别提实际工程应用和日常面试了，属于必知必会。





### 什么是 channel



> 在 Go 语言中，channel 可以称其为通道，也可以叫管道。channel 主要常见于与 goroutine+select 搭配使用，再结合语录的描述。可以知道 channel 就是用于 goroutine 的数据通信：



```go
func main(){
  ch := make(chan string)
  
  // 新建协程
  go func(){
    ch <- "煎鱼"
  }()
  // 接受channel
  msg := <-ch
  fmt.Println(msg)
}
```

* 在 goroutine1 中写入 到变量 ch 中，goroutine2 监听变量 ch，并`阻塞`等待读取到值最终返回，结束流程。





channel 承载着一个衔接器的桥梁:



{{< img src="/images/posts/Go-channel/channel-1.jpg" title="" caption="" alt="image alt" width="700px" position="center" >}}



这也是 channel 的经典思想了，**不要通过共享内存来通信，而是通过通信来实现内存共享**（Do not communicate by sharing memory; instead, share memory by communicating）。



> 从模式上来看，其就是在多个 goroutine 借助 channel 来传输数据，实现了跨 goroutine 间的数据传输，多者独立运行，不需要强关联，更不影响对方的 goroutine 状态。不存在 goroutine1 对 goroutine2 进行直传的情况。
> 这里思考一个问题，那 goroutine1 和 goroutine2 又怎么互相知道自己的数据 ”到“ 了呢？



### channel 基本特性



>  在 Go 语言中，channel 的关键字为 chan，数据流向的表现方式为 <-，代码解释方向是从左到右，据此就能明白通道的数据流转方向了。



channel 共有两种模式：双向和单向

三种表现方式：

* 声明双向通道：chan T
* 声明只允许发送的通道：chan <- T
* 声明只允许接收的通道：<- chan T







channel 中还分为 “无缓冲 channel” 和 “缓冲 channel”

```go
// 无缓冲
ch1 := make(chan int)

// 缓冲区为3
ch2 := make(chan int, 3)
```





#### 无缓冲 channel



无缓冲的 channel（unbuffered channel），其缓冲区大小则默认为 0。在功能上其接受者会阻塞等待并阻塞应用程序，直至收到通信和接收到数据。
这种常用于两个 goroutine 间互相同步等待的应用场景：



{{< img src="/images/posts/Go-channel/channel-2.jpg" title="" caption="" alt="image alt" width="700px" position="center" >}}



#### 缓冲 channel



有缓存的 channel（buffered channel），其缓存区大小是根据所设置的值来调整。在功能上，若缓冲区未满则不会阻塞，会源源不断的进行传输。当缓冲区满了后，发送者就会阻塞并等待。而当缓冲区为空时，接受者就会阻塞并等待，直至有新的数据：

{{< img src="/images/posts/Go-channel/channel-3.jpg" title="" caption="" alt="image alt" width="700px" position="center" >}}







### channel  本质



channel 听起来实现了一个非常酷的东西，也是日常工作中常常会被面试官问到的问题。
但其实 channel 并没有那么的 "神秘"，就是一个环形队列的配合。
接下来我们一步步的剖开 channel，看看里面到底是什么，怎么实现的跨 goroutine 通信，数据结构又是什么，两者又如何实现数据传输的？



#### 基本原理

本质上 channel 在设计上就是环形队列。其包含发送方队列、接收方队列，加上互斥锁 `mutex` 等结构。
channel 是一个有锁的环形队列：

{{< img src="/images/posts/Go-channel/channel-4.jpg" title="" caption="" alt="image alt" width="700px" position="center" >}}



#### 数据结构



hchan 结构体是 channel 在运行时的具体表现形式

```go
// src/runtime/chan.go
type hchan struct {
 qcount   uint      
 dataqsiz uint     
 buf      unsafe.Pointer 
 elemsize uint16
 closed   uint32
 elemtype *_type 
 sendx    uint  
 recvx    uint  
 recvq    waitq  
 sendq    waitq  

 lock mutex
}
```



- qcount：队列中的元素总数量。
- dataqsiz：循环队列的长度。
- buf：指向长度为 dataqsiz 的底层数组，仅有当 channel 为缓冲型的才有意义。
- elemsize：能够接受和发送的元素大小。
- closed：是否关闭。
- elemtype：能够接受和发送的元素类型。
- sendx：已发送元素在循环队列中的索引位置。
- recvx：已接收元素在循环队列中的索引位置。
- recvq：接受者的 sudog 等待队列（缓冲区不足时阻塞等待的 goroutine）
- sendq：发送者的 sudog 等待队列



在数据结构中，我们可以看到 `recvq` 和 `sendq`，其表现为等待队列，其类型为 `runtime.waitq` 的双向链表结构：

```go
type waitq struct{
  first *sudog
  last *sudog
}
```



且无论是 `first` 属性又或是 `last`，其类型都为 `runtime.sudog` 结构体：

```go
type sudog struct {
 g *g

 next *sudog
 prev *sudog
 elem unsafe.Pointer
 ...
}
```

- g：指向当前的 goroutine。
- next：指向下一个 g。
- prev：指向上一个 g。
- elem：数据元素，可能会指向堆栈。



>sudog 是 Go 语言中用于存放协程状态为阻塞的 goroutine 的双向链表抽象，可以直接理解为一个正在等待的 goroutine 就可以了



后续基本围绕着上述数据结构进行大量的讨论



### channel 实现原理



在了解了 channel 的基本原理后，我们进入到与应用工程中更紧密相关的部分，那就是 channel 的四大块操作，分别是：“创建、发送、接收、关闭”。



#### 创建 chan



创建 channel 的演示代码:

```go
ch := make(chan string)
```

其在编译器翻译后对应 `runtime.makechan` 或 `runtime.makechan64`方法：



```go
// 通用创建方法
func makechan(t *chantype, size int) *hchan

// 类型为 int64 的进行特殊处理
func makechan64(t *chantype, size int64) *hchan
```



> 通过前面我们得知 channel 的基本单位是 `hchan` 结构体，那么在创建 channel 时，究竟还需要做什么是呢？



* 来分析一下 `makechan` 方法~

源码如下:

```go
// src/runtime/chan.go
// 创建 channel 的逻辑主要分为三大块：
func makechan(t *chantype, size int) *hchan {
 elem := t.elem
 mem, _ := math.MulUintptr(elem.size, uintptr(size))

 var c *hchan
 switch {
// 当前 channel 不存在缓冲区，也就是元素大小为 0 的情况下，就会调用 mallocgc 方法分配一段连续的内存空间。
 case mem == 0:
  c = (*hchan)(mallocgc(hchanSize, nil, true))
  c.buf = c.raceaddr()

// 当前 channel 存储的类型存在指针引用，就会连同 hchan 和底层数组同时分配一段连续的内存空间。
 case elem.ptrdata == 0:
  c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
  c.buf = add(unsafe.Pointer(c), hchanSize)
 
// 通用情况，默认分配相匹配的连续内存空间。
 default:
  c = new(hchan)
  c.buf = mallocgc(mem, elem, true)
 }

 c.elemsize = uint16(elem.size)
 c.elemtype = elem
 c.dataqsiz = uint(size)
 lockInit(&c.lock, lockRankHchan)

 return c
}
```



> 需要注意到一块特殊点，那就是 channel 的创建都是调用的 `mallocgc`方法，也就是 channel 都是创建在堆上的。因此 channel 是会被 GC 回收的，自然也不总是需要 `close` 方法来进行显示关闭了。
>
> 从整体上来讲，`makechan` 方法的逻辑比较简单，就是创建 `hchan` 并分配合适的 `buf` 大小的堆上内存空间。





#### 发送数据



演示代码:

```go
go func(){
  ch <- "Hello"
}
```



其在编译器翻译后对应 `runtime.chansend1` 方法：

```go
func chansend1(c *hchan, elem unsafe.Pointer) {
 chansend(c, elem, true, getcallerpc())
}
```

其作为编译后的入口方法，实则指向真正的实现逻辑，也就是 `chansend`方法。



##### 前置处理



在第一部分中，我们先看看 chan 发送的一些前置判断和处理：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
  // 一开始 chansend 方法在会先判断当前的 channel 是否为 nil。若为 nil，在逻辑上来讲就是向 nil channel 发送数据，就会调用 gopark 方法使得当前 Goroutine 休眠，进而出现死锁崩溃，表象就是出现 panic 事件来快速失败。
 if c == nil {
  if !block {
   return false
  }
  gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
  throw("unreachable")
 }
 

  // 紧接着会对非阻塞的 channel 进行一个上限判断，看看是否快速失败。(full方法)
 if !block && c.closed == 0 && full(c) {
  return false
 }

 // 省略一些调试相关
 ...
}

func full(c *hchan) bool {
  // 若非阻塞且未关闭，同时底层数据 dataqsiz 大小为 0（缓冲区无元素），则会返回失败
 if c.dataqsiz == 0 {
  return c.recvq.first == nil
 }

  // 若是 qcount 与 dataqsiz 大小相同（缓冲区已满）时，则会返回失败。
 return c.qcount == c.dataqsiz
}
```



##### 上互斥锁



在完成了 channel 的前置判断后，即将在进入发送数据的处理前，channel 会进行上锁：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
 ...
 lock(&c.lock)
}
```

上锁后就能保住并发安全。另外我们也可以考虑到，这种场景会相对依赖单元测试的覆盖，因为一旦没考虑周全，漏上锁了，基本就会出问题。



##### 直接发送



在正式开始发送前，加锁之后，会对 channel 进行一次状态判断（是否关闭）：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
 ...
 if c.closed != 0 {
  unlock(&c.lock)
  panic(plainError("send on closed channel"))
 }

 if sg := c.recvq.dequeue(); sg != nil {
  send(c, sg, ep, func() { unlock(&c.lock) }, 3)
  return true
 }
}
```

这种情况是最为基础的，也就是当前 channel 有正在阻塞等待的接收方，那么只需要直接发送就可以了。



##### 缓冲发送



非直接发送，那么就考虑第二种场景，判断 channel 缓冲区中是否还有空间：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
 ...
// 会对缓冲区进行判定（qcount 和 dataqsiz 字段），以此识别缓冲区的剩余空间。紧接进行如下操作：
 if c.qcount < c.dataqsiz {
   // 调用 chanbuf 方法，以此获得底层缓冲数据中位于 sendx 索引的元素指针值。
  qp := chanbuf(c, c.sendx)
   // 调用 typedmemmove 方法，将所需发送的数据拷贝到缓冲区中。
  typedmemmove(c.elemtype, qp, ep)
// 数据拷贝后，对 sendx 索引自行自增 1。同时若 sendx 与 dataqsiz 大小一致，则归 0（环形队列）。
  c.sendx++
  if c.sendx == c.dataqsiz {
   c.sendx = 0
  }
// 自增完成后，队列总数同时自增 1。解锁互斥锁，返回结果。
  c.qcount++
  unlock(&c.lock)
  return true
 }

 if !block {
  unlock(&c.lock)
  return false
 }
}
```

至此针对缓冲区的数据操作完成。但若没有走进缓冲区处理的逻辑，则会判断当前是否阻塞 channel，若为非阻塞，将会解锁并直接返回失败。



##### 阻塞发送

在进行了各式各样的层层筛选后，接下来进入阻塞等待发送的过程：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
 ...
// 调用 getg 方法获取当前 goroutine 的指针，用于后续发送数据。
 gp := getg()
// 调用 acquireSudog 方法获取 sudog 结构体，并设置当前 sudog 具体的待发送数据信息和状态。
 mysg := acquireSudog()
 mysg.releasetime = 0
 if t0 != 0 {
  mysg.releasetime = -1
 }

 mysg.elem = ep
 mysg.waitlink = nil
 mysg.g = gp
 mysg.isSelect = false
 mysg.c = c
 gp.waiting = mysg
 gp.param = nil
// 调用 c.sendq.enqueue 方法将刚刚所获取的 sudog 加入待发送的等待队列。
 c.sendq.enqueue(mysg)

 atomic.Store8(&gp.parkingOnChan, 1)
// 调用 gopark 方法挂起当前 goroutine（会记录执行位置），状态为 waitReasonChanSend，阻塞等待 channel
 gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
// 调用 KeepAlive 方法保证待发送的数据值是活跃状态，也就是分配在堆上，避免被 GC 回收。
 KeepAlive(ep)
}
```



{{< img src="/images/posts/Go-channel/channel-5.jpg" title="" caption="" alt="image alt" width="700px" position="center" >}}



在当前 goroutine 被挂起后，其将会在 channel 能够发送数据后被唤醒：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
 ...
 // 从这里开始唤醒，并恢复阻塞的发送操作
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



唤醒 goroutine（调度器在停止 g 时会记录运行线程和方法内执行的位置）并完成 channel 的阻塞数据发送动作后。进行基本的参数检查，确保是符合要求的（纵深防御），接着开始取消 mysg 上的 channel 绑定和 sudog 的释放。

至此完成所有类别的 channel 数据发送管理。



#### 接收数据



channel 接受数据的演示代码：

```go
msg := <-ch

msg, ok := <-ch
```

两种方法在编译器翻译后分别对应 `runtime.chanrecv1` 和 `runtime.chanrecv2` 两个入口方法，其再在内部再进一步调用 `runtime.chanrecv`方法：

需要注意，发送和接受 channel 是相对的，也就是其核心实现也是相对的。因此在理解时也可以结合来看。




##### 前置处理



```go
// 一开始时 chanrecv 方法会判断其是否为 nil channel。
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
// 若 channel 是 nil channel，且为阻塞接收则调用 gopark 方法挂起当前 goroutine。
 if c == nil {
// 若 channel 是非阻塞模式，则直接返回。
  if !block {
   return
  }
  gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
  throw("unreachable")
 }
```



而接下来对于非阻塞模式的 channel 会进行快速失败检查，检测 channel 是否已经准备好接收。

```go
// 其分以下几种情况：
// 无缓冲区：循环队列为 0 及等待队列 sendq 内没有 goroutine 正在等待。
// 有缓冲区：缓冲区数组为空。
// 随后会对 channel 的 closed 状态进行判断，因为 channel 是无法重复打开的，需要确定当前 channel 是否为未关闭状态。再确定接收失败，返回。
// 但若是 channel 已经关闭且不存在缓存数据了，则会清理 ep 指针中的数据并返回。
if !block && empty(c) {
  if atomic.Load(&c.closed) == 0 {
   return
  }

  if empty(c) {
   if ep != nil {
    typedmemclr(c.elemtype, ep)
   }
   return true, false
  }
 }
 ...
}
```



##### 直接接收



当发现 channel 上有正在阻塞等待的发送方时，则直接进行接收：

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {

 lock(&c.lock)

 if sg := c.sendq.dequeue(); sg != nil {
  recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
  return true, true
 }
 ...
}
```



##### 缓冲接受



当发现 channel 的缓冲区中有元素时：

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {

 if c.qcount > 0 {
// 调用 chanbuf 方法根据 recvx 的索引位置取出数据，找到要接收的元素进行处理。若所接收到的数据和所传入的变量均不为空，则会调用 typedmemmove 方法将缓冲区中的数据拷贝到所传入的变量中。
  qp := chanbuf(c, c.recvx)
  if ep != nil {
   typedmemmove(c.elemtype, ep, qp)
  }
// 最后数据拷贝完毕后，进行各索引项和队列总数的自增增减，并调用 typedmemclr 方法进行内存数据的清扫。
  typedmemclr(c.elemtype, qp)
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
 ...
}
```





##### 阻塞接收



当发现 channel 上既没有待发送的 goroutine，缓冲区也没有数据时。将会进入到最后一个阶段阻塞接收：

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {

 gp := getg()
 mysg := acquireSudog()
 mysg.releasetime = 0
 if t0 != 0 {
  mysg.releasetime = -1
 }

 mysg.elem = ep
 mysg.waitlink = nil
 gp.waiting = mysg
 mysg.g = gp
 mysg.isSelect = false
 mysg.c = c
 gp.param = nil
 c.recvq.enqueue(mysg)

 atomic.Store8(&gp.parkingOnChan, 1)
 gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)
 ...
}
```



> 这一块接收逻辑与发送也基本类似，主体就是获取当前 goroutine，构建 sudog 结构保存当前待接收数据（发送方）的地址信息，并将 sudog 加入等待接收队列。最后调用 `gopark` 方法挂起当前 goroutine，等待唤醒。



```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {

 // 被唤醒后从此处开始
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









#### 关闭 chan



关闭 channel 主要是涉及到 `close` 关键字：

```go
close(ch)
```

其对应的编译器翻译方法为 `closechan` 方法：

```go
func closechan(c *hchan)
```



##### 前置处理



```go
func closechan(c *hchan) {
 if c == nil {
  panic(plainError("close of nil channel"))
 }

 lock(&c.lock)
 if c.closed != 0 {
  unlock(&c.lock)
  panic(plainError("close of closed channel"))
 }

 c.closed = 1
 ...
}
```

基本检查和关闭标志设置，保证 channel 不为 nil 和未关闭，保证边界。



##### 释放接收方



在完成了异常边界判断和标志设置后，会将接受者的 sudog 等待队列（recvq）加入到待清除队列 glist 中：

```go
func closechan(c *hchan) {

 var glist gList
 for {
  sg := c.recvq.dequeue()
  if sg == nil {
   break
  }
  if sg.elem != nil {
   typedmemclr(c.elemtype, sg.elem)
   sg.elem = nil
  }
  if sg.releasetime != 0 {
   sg.releasetime = cputicks()
  }
  gp := sg.g
  gp.param = nil
  if raceenabled {
   raceacquireg(gp, c.raceaddr())
  }
  glist.push(gp)
 }
 ...
}
```

所取出并加入的 goroutine 状态需要均为 `_Gwaiting`，以保证后续的新一轮调度。



##### 释放发送方



同样，与释放接收方一样。会将发送方也加入到到待清除队列 glist 中：

```go
func closechan(c *hchan) {

 // release all writers (they will panic)
 for {
  sg := c.sendq.dequeue()
  if sg == nil {
   break
  }
  sg.elem = nil
  if sg.releasetime != 0 {
   sg.releasetime = cputicks()
  }
  gp := sg.g
  gp.param = nil
  if raceenabled {
   raceacquireg(gp, c.raceaddr())
  }
  glist.push(gp)
 }
 unlock(&c.lock)
 ...
}
```



##### 协程调度



将所有 glist 中的 goroutine 状态从 `_Gwaiting` 设置为 `_Grunnable` 状态，等待调度器的调度：

```go
func closechan(c *hchan) {

 // Ready all Gs now that we've dropped the channel lock.
 for !glist.empty() {
  gp := glist.pop()
  gp.schedlink = 0
  goready(gp, 3)
 }
}
```



后续所有的 goroutine 允许被重新调度后。若原本还在被动阻塞的发送方或接收方，将重获自由，后续该干嘛就去干嘛了，再跑回其所属的应用流程。







### channel send/recv 分析



#### send



`send` 方法承担向 channel 发送具体数据的功能：

```go
func send(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
 if sg.elem != nil {
  sendDirect(c.elemtype, sg, ep)
  sg.elem = nil
 }
 gp := sg.g
 unlockf()
 gp.param = unsafe.Pointer(sg)
 if sg.releasetime != 0 {
  sg.releasetime = cputicks()
 }
 goready(gp, skip+1)
}

func sendDirect(t *_type, sg *sudog, src unsafe.Pointer) {
 dst := sg.elem
 typeBitsBulkBarrier(t, uintptr(dst), uintptr(src), t.size)
 memmove(dst, src, t.size)
}
```

- 调用 `sendDirect` 方法将待发送的数据直接拷贝到待接收变量的内存地址（执行栈）。

- - 例如：`msg := <-ch` 语句，也就是将数据从 `ch` 直接拷贝到了 `msg` 的内存地址。

- 调用 `sg.g` 属性， 从 sudog 中获取等待接收数据的 goroutine，并传递后续唤醒所需的参数。

- 调用 `goready` 方法唤醒需接收数据的 goroutine，期望从 `_Gwaiting` 状态调度为 `_Grunnable`。



#### recv



`recv` 方法承担在 channel 中接收具体数据的功能：

```go
func recv(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
 if c.dataqsiz == 0 {
  if ep != nil {
   recvDirect(c.elemtype, sg, ep)
  }
 } else {
  qp := chanbuf(c, c.recvx)
  if ep != nil {
   typedmemmove(c.elemtype, ep, qp)
  }
  typedmemmove(c.elemtype, qp, sg.elem)
  c.recvx++
  if c.recvx == c.dataqsiz {
   c.recvx = 0
  }
  c.sendx = c.recvx // c.sendx = (c.sendx+1) % c.dataqsiz
 }
 sg.elem = nil
 gp := sg.g
 unlockf()
 gp.param = unsafe.Pointer(sg)
 if sg.releasetime != 0 {
  sg.releasetime = cputicks()
 }
 goready(gp, skip+1)
}
```

> 该方法在接受上分为两种情况，分别是直接接收和缓冲接收：
>
> - 直接接收（不存在缓冲区）：
>
> - - 调用 `recvDirect` 方法，其作用与 `sendDirect` 方法相对，会直接从发送方的 goroutine 调用栈中将数据拷贝过来到接收方的 goroutine。
>
> - 缓冲接收（存在缓冲区）：
>
> - - 调用 `chanbuf` 方法，根据 `recvx` 索引的位置读取缓冲区元素，并将其拷贝到接收方的内存地址。
>   - 拷贝完毕后，对 `sendx` 和 `recvx` 索引位置进行调整。
>
> 最后还是常规的 goroutine 调度动作，会调用 `goready` 方法来唤醒当前所处理的 sudog 的对应 goroutine。那么在下一轮调度时，既然已经接收了数据，自然发送方也就会被唤醒。



### 总结



在本文中我们针对 Go 语言的 channel 进行了基本概念的分析和讲解，同时还针对 channel 的设计原理和四大操作（创建、发送、接收、关闭）进行了源码分析和图示分析。

初步看过一遍后，再翻看。不难发现，Go 的 channel 设计并不复杂，记住他的数据结构就是带缓存的环形队列，再加上对称的 sendq、recvq 等双向链表的辅助属性，就能勾画出 channel 的基本逻辑流转模型。

在具体的数据传输上，都是围绕着 “边界上下限处理，上互斥锁，阻塞/非阻塞，缓冲/非缓冲，缓存出队列，拷贝数据，解互斥锁，协程调度” 在不断地流转处理。在基本逻辑上也是相对重合的，因为发送和接收，创建和关闭总是相对的。

