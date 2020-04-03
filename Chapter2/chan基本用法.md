## Channel

Channel用于数据传递或数据共享，其本质上是一个先进先出的队列，使用Goroutine 和channel进行数据通讯简单高效，同时也线程安全，多个Goroutine可同时修改一个Channel，不需要加锁

在channel中箭头的指向是数据的流向,和map类似,channel也一个对应make创建的底层数据结构的引用

```
ch := make(chan int)
ch <- p    // 发送值p到Channel ch中
p := <-ch  // 从Channel ch中接收数据，并将数据赋值给p
```

和其它的引用类型一样,channel的零值也是nil.

chan类型的定义格式： `ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) Type .`

<-代表channel的方向。如果没有指定方向，那么Channel就是双向的，既可以接收数据，也可以发送数据

```
    chan p //可以接受发送类型为p的数据
    chan<- float64 //可以用来发送float64类型数据
    <-chan int //可以用来接收int类型数据
```

make初始化chan可以设置channel的容量，容量代表channel缓存的大小,没有缓存只有sender和receiver都准备 好了之后他们的通信才会发生。设置了缓存就可能不会发生阻塞，只有buffer满了之后send才会阻塞，之后缓存 空了之后receive才会阻塞。一个nil chan 不会通信

### 无缓冲channels

发送和接收动作是同时发生的Channels的发送和接收操作将导致两个goroutine做一次同步操作。因为这个原因，无缓存Channels有时候也被称为同步Channels 。如果没有 goroutine 读取 channel （<- channel），则发送者 (channel <-) 会一直阻塞。

### 缓冲的Channels

缓冲：缓冲 channel 类似一个有容量的队列。当队列满的时候发送者会阻塞；当队列空的时候接收者会阻塞。
使用channel之后可以进行关闭，关闭channel后,对该channel的任何发送操作都将导致panic异常。对一个已经被close过的channel之行接收操作依然可以接受到之前已经成功发送的数据;如果channel中已经没有数据的话讲产生一个零值的数据。

关闭chan close(ch)

- 重复关闭 channel 会导致 panic.
- 向关闭的 channel 发送数据会 panic.
- 从关闭的 channel 读数据不会 panic，读出channel中已有的数据之后再读就是channel类似的默认值

判断默认值胡子和关闭ok语法

```go
    ch := make(chan int, 10)
    ...
    close(ch)

    val, ok  := <-ch
    if ok == false {
        //channel closed
    }
```

### chan的经典用法

1. goroutine使用chan通信

   ```go
    func main(){
        x := make(chan int)
        go func(){
            x <- 1
        }()
        <- x
    }
   ```

2. select选择一组可能的send receive操作去处理

   ```go
    select {
        case e, ok := <-ch1:
            //TODO
        case e, ok := <-ch2:
            //TODO
        default:  
    }
    for {
        select {
            //TODO
        }
    }
   ```

3. range channel

   ```go
   使用range channel 我们可以直接取到 channel 中的值。当我们使用 range 来操作 channel 的时候，一旦 channel 关闭，channel 内部数据读完之后循环自动结束。
    func consumer(ch chan int){
        for x := range ch{
            fmt.Println(x)
        }
    }
   
    func producer(ch chan int) {
        for _,v range values {
            ch <-v
        }
    }
   ```

​      在这里面的chan的关闭要在生产者中关闭 否则会遇到异常

```go
package main

import (
	"fmt"
)

func main() {
	ch := make(chan int)
	go producer(ch)
	fmt.Println("start")
	consumer(ch)
	fmt.Println("end")
}
func consumer(ch chan int) {
	for x := range ch {
		fmt.Println(x)
	}
}
func producer(ch chan int) {
	defer close(ch)
	values := []int{1, 2, 4}
	for _, v := range values {
		ch <- v
	}
}

```



4. 超时控制

   ```go
    select如果没有case需要处理，就会一直阻塞， select可以实现超时控制
    func main(){
        chstr := make(chan string, 1)
        go func(){
            time.Sleep(time.Second * 2)
            chstr <- "retuslt"
        }()
   
        select{
            case res := <-chstr:
                fmt.Println(res)
            case <- time.After(time.Second * 1):
                fmt.Println("timeout")
        }
    }
   ```

5. chan同步
   channel 将goroutine 隔离开，并发编程的时候可以将注意力放在 channel 上

```go
    func mission(done chan bool){
        time.Sleep(time.Second)
        //通知任务完成
        done <- true
    }

    func main(){
        done := make(chan bool, 1)
        go mission(done)
        //等待mission任务完成
        <-done
    }
```