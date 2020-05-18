# Go语言实战【11-17】

内容源自“飞雪无情”的公众号《go语言实战》系列。

第11节至第17节的笔记，主要为并发相关的内容。

## goroutine

go语言中并发指的是让某个函数独立于其他函数运行的能力，一个goroutine就是一个独立的工作单元，Go的runtime（运行时）会在**逻辑处理器**上调度这些goroutine来运行，一个**逻辑处理器**绑定一个操作系统线程，所以说goroutine不是线程，它是一个协程，也是这个原因，它是由Go语言运行时本身的算法实现的。

这里我们总结下几个概念：

| 概念         | 说明                                          |
| ------------ | --------------------------------------------- |
| 进程         | 一个程序对应一个独立程序空间                  |
| 线程         | 一个执行空间，一个进程可以有多个线程          |
| 逻辑处理器   | 执行创建的goroutine，绑定一个线程             |
| 调度器       | Go运行时中的，分配goroutine给不同的逻辑处理器 |
| 全局运行队列 | 所有刚创建的goroutine都会放到这里             |
| 本地运行队列 | 逻辑处理器的goroutine队列                     |

当我们创建一个goroutine的后，会先存放在`全局运行队列`中，等待Go运行时的`调度器`进行调度，把他们分配给其中的一个`逻辑处理器`，并放到这个逻辑处理器对应的`本地运行队列`中，最终等着被`逻辑处理器`执行即可。

```go
func main() {
    var wg sync.WaitGroup
    wg.Add(2)    
    
    go func(){
            defer wg.Done()        
            for i:=1;i<100;i++ {
                fmt.Println("A:",i)
            }
    }()    
       
    go func(){
            defer wg.Done()
            for i:=1;i<100;i++ {
                fmt.Println("B:",i)
            }
    }()
    wg.Wait()
}
```

### 并发资源竞争

有并发，就有资源竞争，如果两个或者多个goroutine在没有相互同步的情况下，访问某个共享的资源，比如同时对该资源进行读写时，就会处于相互竞争的状态，这就是并发中的资源竞争。

**所以我们对于同一个资源的读写必须是原子化的，也就是说，同一时间只能有一个goroutine对共享资源进行读写操作**。

共享资源竞争的问题，非常复杂，并且难以察觉，好在Go为我们提供了一个工具帮助我们检查，这个就是`go build -race`命令。我们在当前项目目录下执行这个命令，生成一个可以执行文件，然后再运行这个可执行文件，就可以看到打印出的检测信息。

```go
// 实例中，新声明了一个互斥锁mutex sync.Mutex，这个互斥锁有两个方法，一个是mutex.Lock(),一个是mutex.Unlock(),这两个之间的区域就是临界区，临界区的代码是安全的。
// 临界区中的代码只能有一个goroutine进行访问
mutex sync.Mutex
mutex.Lock()
// your code
mutex.Unlock()
```

## Go 通道

在多个goroutine并发中，我们不仅可以通过原子函数和互斥锁保证对共享资源的安全访问，消除竞争的状态，还可以通过使用通道，在多个goroutine发送和接受共享的数据，达到数据同步的目的。

通道，他有点像在两个routine之间架设的管道，一个goroutine可以往这个管道里塞数据，另外一个可以从这个管道里取数据。

```go
// 声明
ch:=make(chan int)

ch <- 2 //发送数值2给这个通道
x:=<-ch //从通道里读取值，并把读取的值赋值给x变量
<-ch //从通道里读取值，然后忽略
close(ch)	//关闭通道
```

### 无缓冲通道

无缓冲的通道指的是通道的大小为0，也就是说，这种类型的通道在接收前没有能力保存任何值，它要求发送goroutine和接收goroutine同时准备好，才可以完成发送和接收操作。

从上面无缓冲的通道定义来看，发送goroutine和接收gouroutine必须是同步的，同时准备后，如果没有同时准备好的话，先执行的操作就会阻塞等待，直到另一个相对应的操作准备好为止。这种无缓冲的通道我们也称之为`同步通道`。

```go
func main() {
    ch := make(chan int)
    go func() {
      var sum int = 0
      for i := 0; i < 10; i++ {
        sum += i
      }
      ch <- sum
    }()

    fmt.Println(<-ch)
}
```

在计算sum和的goroutine没有执行完，把值赋给`ch`通道之前，`fmt.Println(<-ch)`会一直等待，所以`main`主goroutine就不会终止，只有当计算和的goroutine完成后，并且发送到`ch`通道的操作准备好后，同时`<-ch`就会接收计算好的值，然后打印出来。

使用多个通道，能实现管道的效果。我们使用通道也可以做到管道的效果，我们只需要把一个通道的输出，当成下一个通道的输入即可。

### 有缓冲通道

有缓冲通道，其实是一个队列，这个队列的最大容量就是我们使用`make`函数创建通道时，通过第二个参数指定的。

```go
ch := make(chan int, 3)

func mirroredQuery() string {
    responses := make(chan string, 3)
    go func() { responses <- request("asia.gopl.io") }()
    go func() { responses <- request("europe.gopl.io") }()    
    go func() { responses <- request("americas.gopl.io") }()    
    return <-responses // return the quickest response
}
func request(hostname string) (response string) { /* ... */ }
```

当队列满的时候，发送操作会阻塞；当队列空的时候，接受操作会阻塞。有缓冲的通道，不要求发送和接收操作时同步的，相反可以解耦发送和接收操作。

这是Go语言圣经里比较有意义的一个例子，例子是想获取服务端的一个数据，不过这个数据在三个镜像站点上都存在，这三个镜像分散在不同的地理位置，而我们的目的又是想最快的获取到数据。

所以这里，我们定义了一个容量为3的通道`responses`，然后同时发起3个并发goroutine向这三个镜像获取数据，获取到的数据发送到通道`responses`中，最后我们使用`return <-responses`返回获取到的第一个数据，也就是最快返回的那个镜像的数据。

### 单向通道

有时候，我们有一些特殊场景，比如限制一个通道只可以接收，但是不能发送；有时候限制一个通道只能发送，但是不能接收，这种通道我们称为单向通道。

定义单向通道也很简单，只需要在定义的时候，带上`<-`即可。

```go
var send chan<- int //只能发送
var receive <-chan int //只能接收
```

## Go并发实例

通过一个例子，演示使用通道来监控程序的执行时间，生命周期，甚至终止程序等。我们这个程序叫runner，我们可以称之为`执行者`，它可以在后台执行任何任务，而且我们还可以控制这个执行者，比如强制终止它等。

```go
package runner

import (
	"errors"
	"os"
	"os/signal"
	"time"
)

var ErrTimeOut = errors.New("执行者执行超时")
var ErrInterrupt = errors.New("执行者被中断")

//一个执行者，可以执行任何任务，但是这些任务是限制完成的
//该执行者可以通过发送终止信号终止它
type Runner struct {
	tasks []func(int)	//要执行的任务
	complete chan error	//用于通知任务全部完成
	timeout <-chan time.Time	//这些任务在多久内完成
	interrupt chan os.Signal	//可以控制强制终止的信号
}

//初始化一个Runner
func NewRunner(tm time.Duration) *Runner {
	return &Runner{
		complete:  make(chan error),
		timeout:   time.After(tm),	//在tm时间后，会同伙一个time.Time类型的只能接收的单向通道，来告诉我们已经到时间了
		interrupt: make(chan os.Signal, 1),
	}
}

//将需要执行的任务，添加到Runner里
func (r *Runner) AddTasks(tasks ...func(int)) {
	r.tasks = append(r.tasks, tasks...)
}

//检查是否接收到了中断信号
func (r *Runner) isInterrupt() bool {
	select {
	case <-r.interrupt:
		signal.Stop(r.interrupt)
		return true
	default:
		return false
	}
}

//执行任务，执行的过程中接收到中断信号时，返回中断错误
//如果任务全部执行完，还没有接收到中断信号，则返回nil
func (r *Runner) run() error {
	for id, task := range r.tasks {
		if r.isInterrupt() {
			return ErrInterrupt
		}
		task(id)
	}
	return nil
}

//开始执行所有任务，并且监视通道事件
func (r *Runner) StartRunner() error {
	//希望接收哪些系统信号
	signal.Notify(r.interrupt, os.Interrupt)
	go func() {
		r.complete <- r.run()
	}()
	select {
	case err := <- r.complete:
		return err
	case <-r.timeout:
		return ErrTimeOut
	}
}
```

## go并发实例 - pool

演示使用有缓冲的通道实现一个资源池，这个资源池可以管理在任意多个goroutine之间共享的资源，比如网络连接、数据库连接等，我们在数据库操作的时候，比较常见的就是数据连接池，也可以基于我们实现的资源池来实现。

```go
package runner

import (
	"errors"
	"io"
	"sync"
	"log"
)

var ErrPoolClosed = errors.New("资源池已关闭")

//一个安全的资源池，被管理的资源必须都实现io.Close接口
type Pool struct {
	m sync.Mutex	//互斥锁，这主要是用来保证在多个goroutine访问资源时，池内的值是安全的
	res chan io.Closer	//有缓冲的通道，用来保存共享的资源，这个通道的大小，在初始化Pool的时候就指定的
	factory func()(io.Closer, error)	//函数类型，它的作用就是当需要一个新的资源时，可以通过这个函数创建
	closed bool
}

//创建一个资源池
func NewPool(fn func()(io.Closer, error), size uint)(*Pool, error) {
	if size < 0 {
		return nil, errors.New("size can not < 0")
	}
	return &Pool{
		res:     make(chan io.Closer, size),
		factory: fn,
		closed:  false,
	}, nil
}

//从资源池里获取一个资源
func (p *Pool) Acquire() (io.Closer, error) {
	select {
	//不能阻塞，可以获取到就获取，不能就生成一个
	case r,ok := <-p.res:
		log.Println("Acquire: 共享资源")
		if !ok {
			return nil, ErrPoolClosed
		}
		return r, nil
	default:
		return p.factory()
	}
}

//关闭资源池，释放资源
func (p *Pool) Close()  {
	//使用了互斥锁，因为有个标记资源池是否关闭的字段closed需要再多个goroutine操作，所以我们必须保证这个字段的同步
	p.m.Lock()
	defer p.m.Unlock()
	if p.closed {
		return
	}
	//这里把关闭标志置为true
	p.closed = true
	//关闭通道，不让写入了
	close(p.res)    //关闭通道里的资源
	for r:=range p.res {
		r.Close()
	}
}

//释放资源
func (p *Pool) Release(r io.Closer) {
	//保证该操作和Close方法的操作是安全的
	p.m.Lock()
	defer p.m.Unlock()

	if p.closed{
		r.Close()
		return
	}
	select {
	case p.res <- r:
		log.Println("资源释放到池子里了")
	default:
		log.Println("资源池满了，释放这个资源吧")
		r.Close()
	}
}
```

## 读写锁

读写锁可以让多个读操作同时并发，同时读取，但是对于写操作是完全互斥的。也就是说，当一个goroutine进行写操作的时候，其他goroutine既不能进行读操作，也不能进行写操作。

```go
var rw sync.RWMutex
// 读锁
rw.RLock()
rw.RUnlock()
// 写锁
rw.Lock()
rw.Unlock()
```

# 参考文献

Go goroutine：https://mp.weixin.qq.com/s/XmJG4hQnAhLNxeIB6peuIw

Go并发资源竞争：https://mp.weixin.qq.com/s/tBtw81ciwCII7VOSSOxb3g

Go通道：https://mp.weixin.qq.com/s/2LVU8HOza_39hpTGeTqK9Q

Go并发实例：https://mp.weixin.qq.com/s/Q3U4B__7gJkFEDgdX68t7g、https://mp.weixin.qq.com/s/E6BT-QNWA8wyb6IwE0AkUg

Go读写锁：https://mp.weixin.qq.com/s/wyezW1swNlDkXi4V-ABa-w