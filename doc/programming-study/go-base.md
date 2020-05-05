# Go语言学习之路

记录一下从零开始学习go语言的过程。

https://www.runoob.com/go/go-tutorial.html

## 环境搭建

由于一直用idea进行开发，因此还是优先学习如何在idea中集成go的开发环境。

第一步当然是需要在电脑上安装go语言环境，常规操作，装好后在terminal中验证一下即可。

```shell
liyedeMacBook-Pro:~ liye$ go version
go version go1.14.2 darwin/amd64
```

其次需要注意Go的主要环境变量

```shell
liyedeMacBook-Pro:~ liye$ go env
xxxx
# GOROOT 指向系统安装路径
GOROOT="/usr/local/go" 
		--src                 <<--- Go 语言自带的源代码
    --pkg                 <<--- 编译的中间文件放在此文件夹
    --bin                 <<--- 编译的目标文件放在此文件夹
# GOPATH指向工作路径，
GOPATH="/Users/liye/go"
		--src                 <<--- 项目源代码放置在此文件夹。
        --HelloWorld      <<--- 我们项目源代码所在的文件夹。
        --vendor          <<--- 第三方开源代码文件夹
            --github.com
                --...
    --pkg                 <<--- 编译的中间文件放在此文件夹，Go编译器自动生成此文件夹
    --bin                 <<--- 编译的目标文件放在此文件夹，Go编译器自动生成此文件夹
```

第二步需要在idea中安装go相关的插件，然后新建go项目即可，参考：https://blog.csdn.net/chushoutaizhong/article/details/82220419

## 基础语法

Go 程序可以由多个标记组成，可以是关键字，标识符，常量，字符串，符号。

### 语言变量

变量定义和声明的方式如下。

```go
var identifier1, identifier2 type

// eg, 注意，在不用var声明的情况下，需要用到:=号来赋值
var a, b int = 1, 2
var s = "Hi"
s := "HiHi"

// 这样是不允许的
var a = 1
a := 2

// 如果你想要交换两个变量的值，两个变量的类型必须是相同
a, b = b, a
```

### 语言常量

常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。

```go
const identifier [type] = value

// eg
const LENGTH = 10
const WIDTH int = 5

// iota，特殊常量，可以认为是一个可以被编译器修改的常量
// iota 可理解为 const 语句块中的行索引
const (
  a = iota   //0
  b          //1
  c          //2
  d = "ha"   //独立值，iota += 1
  e          //"ha"   iota += 1
  f = 100    //iota +=1
  g          //100  iota +=1
  h = iota   //7,恢复计数
  i          //8
)
fmt.Println(a,b,c,d,e,f,g,h,i)	// 0 1 2 ha ha 100 100 7 8
```

### 条件和循环语句

```go
// if, else if, else
a := 2
if a >= 2 {
	// do something
}

// switch, fallthrough
switch a {
	case 5: // do something
  case 6: // do something
  default: // do something
}
switch {
  case a >= 5: fallthrough // do something
  default: // do something
}

// for
for init; condition; post { }
for condition { }
for { }

// for iterator
strings := []string{"google", "runoob"}
for i, s := range strings {
  fmt.Println(i, s)
}

// goto
LOOP: for a < 20 {
  if a == 15 {
    /* 跳过迭代 */
    a = a + 1
    goto LOOP
  }
  fmt.Printf("a的值为 : %d\n", a)
  a++    
}  
```

### 函数

```go
func function_name( [parameter list] ) [return_types] {
   函数体
}

// 值传递，值传递是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。go语言默认是值传递。
func swap(x int, y int) {}

// 引用传递,引用传递是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。
func swap(x *int, y *int) {}

// 闭包
func getSequence() func() int {
	i := 0
	return func() int {
		i += 1
		return i
	}
}
bibao := getSequence()
fmt.Println(bibao())	// 1
fmt.Println(bibao())	// 2
```

## 数据类型

常见的数据类型包括：数组、结构体、集合等。

### 数组

```go
var variable_name [SIZE] variable_type
var variable_name [SIZE1][SIZE2]...[SIZEN] variable_type

// 声明和初始化
var balance [10] float32
var balance = [5] float32 {1000.0, 2.0, 3.4, 7.0, 50.0}
var salary float32 = balance[9]

// 多维数组
var threedim [5][10][4] int
a = [3][4]int{  
 {0, 1, 2, 3} ,   /*  第一行索引为 0 */
 {4, 5, 6, 7} ,   /*  第二行索引为 1 */
 {8, 9, 10, 11},   /* 第三行索引为 2 */
}
```

### 指针

```go
var var_name *var-type

// &a, ip均为a变量的地址
// a, *ip均为a变量的值
var a int = 20
var ip *int
ip = &a

// 指向指针的指针
var a int
var ptr *int
var pptr **int
```

### 结构体

```go
type struct_variable_type struct {
   member definition
   member definition
   ...
   member definition
}
variable_name := structure_variable_type {value1, value2...valuen}

// 初始化
cat := new(module.Cat)
cat2 := module.Cat{"Meili2", 2}
cat3 := module.Cat{Name:"Meili3"}

// 访问结构体成员
fmt.Println(cat.Name, cat.Age)
```

### 切片（Slice）

```go
// 切片是动态可变的数组, 初始化时不指定长度
var identifier []type

// 使用make()函数来创建切片
slice1 := make([]type, len, cap)

// 初始化
s :=[] int {1,2,3}	// 直接初始化
s := arr[startIndex:endIndex] // 使用数组创建新的切片
s := make([]int, 3, 5)	// 使用make函数初始化

// 使用
numbers := []{1,2,3,4,5}
numbers[1:3]	// like python
numbers = append(numbers, 6, 7, 8)	// 追加元素
/* 创建切片 numbers1 是之前切片的两倍容量*/
numbers1 := make([]int, len(numbers), (cap(numbers))*2)
/* 拷贝 numbers 的内容到 numbers1 */
copy(numbers1,numbers)
```

### 范围（Range）

```go
// 遍历切片
nums := []int{2,3,4}
for i, num := range nums {
  fmt.Println(i, num)
}

// 遍历map
kvs := map[string]string{"a": "apple", "b": "banana"}
for k, v :=range kvs {
  fmt.Printf(k, v)
}
```

### 集合（Map）

```go
/* 声明变量，默认 map 是 nil */
var map_variable map[key_data_type]value_data_type

/* 使用 make 函数 */
map_variable := make(map[key_data_type]value_data_type)

m = make(map[string]string)
m["a"] = "a"

// 判断是否存在
a, isExist := m["a"]
if isExist {
  // doSomething
}

// 删除元素
delete(m, "a")
```

## 语言特性

### 语言类型转换

```go
var sum int = 17
var count int = 5
var mean float32
mean = float32(sum)/float32(count)
```

### 接口

```go
/* 定义接口 */
type interface_name interface {
   method_name1 [return_type]
   ...
   method_namen [return_type]
}

/* 定义结构体 */
type struct_name struct {
   /* variables */
}

/* 实现接口方法 */
/* 方法在声明时带有接收者 */
func (struct_name_variable struct_name) method_name1() [return_type] {
   /* 方法实现 */
}

type Phone interface {
    call()
}

type NokiaPhone struct {
}

func (nokiaPhone NokiaPhone) call() {
    fmt.Println("I am Nokia, I can call you!")
}
```

### 错误处理

```go
// go内置的错误接口
type error interface {
    Error() string
}

func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // 实现
}

// 调用
result, err:= Sqrt(-1)
if err != nil {
   fmt.Println(err)
}
```

### 并发

Go 语言支持并发，我们只需要通过 go 关键字来开启 goroutine 即可。

goroutine 是轻量级线程，goroutine 的调度是由 Golang 运行时进行管理的。

```go
go 函数名( 参数列表 )
// eg
go f(x, y, z)

func say(s string) {
  for i := 0; i < 5; i++ {
    time.Sleep(100 * time.Millisecond)
    fmt.Println(s)
  }
}

func main() {
  go say("world")
  say("hello")
}
```

通道（channel）是用来传递数据的一个数据结构。

通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。操作符 `<-` 用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道。

```go
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据, 并把值赋给 v

// 声明通道
ch := make(chan int)

// 默认不带缓冲区的通道使用
func sum(s []int, c chan int) {
  sum := 0
  for _, v := range s {
    sum += v
  }
  c <- sum // 把 sum 发送到通道 c
}

func main() {
  s := []int{7, 2, 8, -9, 4, 0}

  c := make(chan int)
  go sum(s[:len(s)/2], c)
  go sum(s[len(s)/2:], c)
  x, y := <-c, <-c // 从通道 c 中接收

  fmt.Println(x, y, x+y)	// -5 17 12
}

// 通道可以设置缓冲区，通过 make 的第二个参数指定缓冲区大小
ch := make(chan int, 100)
func main() {
  // 这里我们定义了一个可以存储整数类型的带缓冲通道
  // 缓冲区大小为2
  ch := make(chan int, 2)

  // 因为 ch 是带缓冲的通道，我们可以同时发送两个数据
  // 而不用立刻需要去同步读取数据
  ch <- 1
  ch <- 2

  // 获取这两个数据, 输出1,2
  fmt.Println(<-ch)
  fmt.Println(<-ch)
}

// 遍历和关闭通道
func fibonacci(n int, c chan int) {
  x, y := 0, 1
  for i := 0; i < n; i++ {
    c <- x
    x, y = y, x+y
  }
  close(c)
}

func main() {
  c := make(chan int, 10)
  go fibonacci(cap(c), c)
  // range 函数遍历每个从通道接收到的数据，因为 c 在发送完 10 个
  // 数据之后就关闭了通道，所以这里我们 range 函数在接收到 10 个数据
  // 之后就结束了。如果上面的 c 通道不关闭，那么 range 函数就不
  // 会结束，从而在接收第 11 个数据的时候就阻塞了。
  for i := range c {
    fmt.Println(i)
  }
}
```

