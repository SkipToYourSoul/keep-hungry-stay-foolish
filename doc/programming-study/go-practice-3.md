# Go语言实战【18-27】

内容源自“飞雪无情”的公众号《go语言实战》系列。

第18节至第27节的笔记。

## log 日志

对此，Go语言为我们提供了标准的`log`包，来跟踪日志的记录。

```go
// 最简单的日志输出
log.Println("Hello world")

// 自定义设置
log.SetPrefix("【UserCenter】")
log.SetFlags(log.Ldate|log.Lshortfile)
log.Println("设置了时间和源代码行号")

const (
    Ldate         = 1 << iota     //日期示例： 2009/01/23
    Ltime                         //时间示例: 01:23:23
    Lmicroseconds                 //毫秒示例: 01:23:23.123123.
    Llongfile                     //绝对路径和行号: /a/b/c/d.go:23
    Lshortfile                    //文件和行号: d.go:23.
    LUTC                          //日期时间转为0时区的
    LstdFlags     = Ldate | Ltime //Go提供的标准抬头信息
)
```

一个可参考的第三方日志包：https://www.cnblogs.com/rickiyang/p/11074164.html

## Writer 和 Reader

Go Writer 和 Reader接口的设计遵循了Unix的输入和输出，一个程序的输出可以是另外一个程序的输入。他们的功能单一并且纯粹，这样就可以非常容易的编写程序代码，又可以通过组合的概念，让我们的程序做更多的事情。

```go
// Writer is the interface that wraps the basic Write method.
//
// Write writes len(p) bytes from p to the underlying data stream.
// It returns the number of bytes written from p (0 <= n <= len(p))
// and any error encountered that caused the write to stop early.
// Write must return a non-nil error if it returns n < len(p).
// Write must not modify the slice data, even temporarily.
//
// Implementations must not retain p.
type Writer interface {
    Write(p []byte) (n int, err error)
}

// Reader is the interface that wraps the basic Read method.
//
// Read reads up to len(p) bytes into p. It returns the number of bytes
// read (0 <= n <= len(p)) and any error encountered. Even if Read
// returns n < len(p), it may use all of p as scratch space during the call.
// If some data is available but not len(p) bytes, Read conventionally
// returns what is available instead of waiting for more.
//
// When Read encounters an error or end-of-file condition after
// successfully reading n > 0 bytes, it returns the number of
// bytes read. It may return the (non-nil) error from the same call
// or return the error (and n == 0) from a subsequent call.
// An instance of this general case is that a Reader returning
// a non-zero number of bytes at the end of the input stream may
// return either err == EOF or err == nil. The next Read should
// return 0, EOF.//// Callers should always process the n > 0 bytes returned before
// considering the error err. Doing so correctly handles I/O errors
// that happen after reading some bytes and also both of the
// allowed EOF behaviors.//// Implementations of Read are discouraged from returning a
// zero byte count with a nil error, except when len(p) == 0.
// Callers should treat a return of 0 and nil as indicating that
// nothing happened; in particular it does not indicate EOF.
//
// Implementations must not retain p.
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

## Context

控制并发有两种经典的方式，一种是WaitGroup，另外一种就是Context，今天我就谈谈Context。

WaitGroup尤其适用于，好多个goroutine协同做一件事情的时候，因为每个goroutine做的都是这件事情的一部分，只有全部的goroutine都完成，这件事情才算是完成，这是等待的方式。

在实际的业务中，我们可能会有这么一种场景：需要我们主动的通知某一个goroutine结束。比如我们开启一个后台goroutine一直做事情，比如监控，现在不需要了，就需要通知这个监控goroutine结束，不然它会一直跑，就泄漏了。

```go
func main() {
  // context.Background(): go内置的context接口实现，返回一个最顶层的Context
  // context.WithCancel(): 传递一个父Context作为参数，返回子Context，以及一个取消函数用来取消Context。
    ctx, cancel := context.WithCancel(context.Background())
  	valueCtx := context.WithValue(ctx, "key", "value")
    go watch(valueCtx)    
    go watch(valueCtx) 

    time.Sleep(10 * time.Second)
    fmt.Println("可以了，通知监控停止")
    cancel()    
    //为了检测监控过是否停止，如果没有监控输出，就表示停止了
    time.Sleep(5 * time.Second)
}
func watch(ctx context.Context) {
    for {        
        select {        
        case <-ctx.Done():
          	fmt.Println(ctx.Value("name"),"监控退出，停止了...")           
            return
        default:
            fmt.Println(ctx.Value("name"),"goroutine监控中...")
            time.Sleep(2 * time.Second)
        }
    }
}
```

示例中启动了3个监控goroutine进行不断的监控，每一个都使用了Context进行跟踪，当我们使用`cancel`函数通知取消时，这3个goroutine都会被结束。这就是Context的控制能力，它就像一个控制器一样，按下开关后，所有基于这个Context或者衍生的子Context都会收到通知，这时就可以进行清理操作了，最终释放goroutine，这就优雅的解决了goroutine启动后不可控的问题。

**Context 使用原则**

1. 不要把Context放在结构体中，要以参数的方式传递
2. 以Context作为参数的函数方法，应该把Context作为第一个参数，放在第一位。
3. 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO
4. Context的Value相关方法应该传递必须的数据，不要什么数据都使用这个传递
5. Context是线程安全的，可以放心的在多个goroutine中传递

## 单元测试

Go语言为我们提供了测试框架，以便帮助我们更容易的进行单元测试，但是要使用这个框架，需要遵循如下几点规则：

1. 含有单元测试代码的go文件必须以`_test.go`结尾，Go语言测试工具只认符合这个规则的文件
2. 单元测试文件名`_test.go`前面的部分最好是被测试的方法所在go文件的文件名，比如例子中是`main_test.go`，因为测试的`Add`函数，在`main.go`文件里
3. 单元测试的函数名必须以`Test`开头，是可导出公开的函数
4. 测试函数的签名必须接收一个指向`testing.T`类型的指针，并且不能返回任何值
5. 函数名最好是Test+要测试的方法函数名，比如例子中是`TestAdd`，表示测试的是`Add`这个这个函数

遵循以上规则，我们就可以很容易的编写单元测试了，单元测试的重点在于测试代码的逻辑，场景等，以便尽可能的测试全面，保障代码质量逻辑。

```go
// main.go
func Add(a,b int) int{
	return a+b
}

// main_test.go
func TestAdd(t *testing.T) {
	sum := Add(1,2)
	if sum == 3 {
		t.Log("the result is ok")
	} else {
		t.Fatal("the result is wrong")
	}
}
```

## 基准测试

基准测试，是一种测试代码性能的方法，比如你有多种不同的方案，都可以解决问题，那么到底是那种方案性能更好呢？这时候基准测试就派上用场了。

基准测试主要是通过测试CPU和内存的效率问题，来评估被测试代码的性能，进而找到更好的解决方案。比如链接池的数量不是越多越好，那么哪个值才是最优值呢，这就需要配合基准测试不断调优了。

```go
func BenchmarkSprintf(b *testing.B){
	num:=10
	b.ResetTimer()
	for i:=0;i<b.N;i++{
		fmt.Sprintf("%d",num)
	}
}
```

这是一个基准测试的例子，从中我们可以看出以下规则：

1. 基准测试的代码文件必须以_test.go结尾
2. 基准测试的函数必须以Benchmark开头，必须是可导出的
3. 基准测试函数必须接受一个指向Benchmark类型的指针作为唯一参数
4. 基准测试函数不能有返回值
5. `b.ResetTimer`是重置计时器，这样可以避免for循环之前的初始化代码的干扰
6. 最后的for循环很重要，被测试的代码要放到循环里
7. b.N是基准测试框架提供的，表示循环的次数，因为需要反复调用测试的代码，才可以评估性能

```shell
➜  hello go test -bench=. -run=none
BenchmarkSprintf-8      20000000               117 ns/op
PASS
ok      flysnow.org/hello       2.474s
```

函数后面的`-8`了吗？这个表示运行时对应的GOMAXPROCS的值。接着的`20000000`表示运行for循环的次数，也就是调用被测试代码的次数，最后的`117 ns/op`表示每次需要话费117纳秒。

## 反射

和Java语言一样，Go也实现运行时反射，这为我们提供一种可以在运行时操作任意类型对象的能力。比如我们可以查看一个接口变量的具体类型，看看一个结构体有多少字段，如何修改某个字段的值等等。

```go
func main() {
	u:= User{"张三",20}
	t:=reflect.TypeOf(u)
	fmt.Println(t)	// *main.User
  v:=reflect.ValueOf(u)
  fmt.Println(v)	// {张三 20}
  
  // 反射动态调用方法
  mPrint:=v.MethodByName("Print")
	args:=[]reflect.Value{reflect.ValueOf("前缀")}
	fmt.Println(mPrint.Call(args))
}

type User struct{
	Name string
	Age int
}

func (u User) Print(prfix string){
	fmt.Printf("%s:Name is %s,Age is %d",prfix,u.Name,u.Age)
}

// 遍历获取字段和方法
for i:=0;i<t.NumField();i++ {
  fmt.Println(t.Field(i).Name)
}

for i:=0;i<t.NumMethod() ;i++  {
  fmt.Println(t.Method(i).Name)
}

// struct与json互转
h:=`{"name":"张三","age":15}`
err:=json.Unmarshal([]byte(h),&u)
if err!=nil{
  fmt.Println(err)
}else {
  fmt.Println(u)
}
newJson,err:=json.Marshal(&u)
fmt.Println((string(newJson)))
```

## unsafe 包之内存布局

unsafe，顾名思义，是不安全的，Go定义这个包名也是这个意思，让我们尽可能的不要使用它，如果你使用它，看到了这个名字，也会想到尽可能的不要使用它，或者更小心的使用它。

虽然这个包不安全，但是它也有它的优势，那就是可以绕过Go的内存安全机制，直接对内存进行读写，所以有时候因为性能的需要，会冒一些风险使用该包，对内存进行操作。

Sizeof函数：`Sizeof`函数可以返回一个类型所占用的内存大小，这个大小只有类型有关，和类型对应的变量存储的内容大小无关，比如bool型占用一个字节、int8也占用一个字节。

Alignof函数：`Alignof`返回一个类型的对齐值，也可以叫做对齐系数或者对齐倍数。对齐值是一个和内存对齐有关的值，合理的内存对齐可以提高内存读写的性能。

内存对齐参考：https://zhuanlan.zhihu.com/p/53413177

Offsetof函数：`Offsetof`函数只适用于struct结构体中的字段相对于结构体的内存位置偏移量。结构体的第一个字段的偏移量都是0.

> 不同的字段顺序，最终决定struct的内存大小，**所以有时候合理的字段顺序可以减少内存的开销**。

## unsafe point

`unsafe.Pointer`是一种特殊意义的指针，它可以包含任意类型的地址，有点类似于C语言里的void*指针，全能型的。

我们看下关于`unsafe.Pointer`的4个规则。

1. 任何指针都可以转换为`unsafe.Pointer`
2. `unsafe.Pointer`可以转换为任何指针
3. `uintptr`可以转换为`unsafe.Pointer`
4. `unsafe.Pointer`可以转换为`uintptr`

`*T`是不能计算偏移量的，也不能进行计算，但是`uintptr`可以，所以我们可以把指针转为`uintptr`再进行偏移计算，这样我们就可以访问特定的内存了，达到对不同的内存读写的目的。

unsafe是不安全的，所以我们应该尽可能少的使用它，比如内存的操纵，这是绕过Go本身设计的安全机制的，不当的操作，可能会破坏一块内存，而且这种问题非常不好定位。

当然必须的时候我们可以使用它，比如底层类型相同的数组之间的转换；比如使用sync/atomic包中的一些函数时；还有访问Struct的私有字段时；该用还是要用，不过一定要慎之又慎。

# 参考文献

Go log日志：https://mp.weixin.qq.com/s/wLTlStAkRkslOYlIiFVWOQ

Go Writer和Reader：https://mp.weixin.qq.com/s/nwPXAl3xB7UEr13j3fkMwQ

Go Context：https://mp.weixin.qq.com/s/yc2ee1kBNVFPBoT_fUBckg

Go 单元测试：https://www.flysnow.org/2017/05/16/go-in-action-go-unit-test.html

Go 基准测试：https://www.flysnow.org/2017/05/21/go-in-action-go-benchmark-test.html

Go 调试：https://www.flysnow.org/2017/06/07/go-in-action-go-debug.html

Go 反射：https://www.flysnow.org/2017/06/13/go-in-action-go-reflect.html、https://www.flysnow.org/2017/06/25/go-in-action-struct-tag.html

Go unsafe包：https://www.flysnow.org/2017/07/02/go-in-action-unsafe-memory-layout.html、https://www.flysnow.org/2017/07/06/go-in-action-unsafe-pointer.html