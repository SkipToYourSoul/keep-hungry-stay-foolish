# Go语言学习之路

记录一下从零开始学习go语言的过程。

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

