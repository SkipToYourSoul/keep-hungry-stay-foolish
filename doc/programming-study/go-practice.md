# Go语言实战【1-10】

内容源自“飞雪无情”的公众号《go语言实战》系列。

第1节至第10节的笔记。

## Go的包管理

在go语言里，同时要满足`main`包和包含`main()`函数，才会被编译成一个可执行文件。

go的包名使用全路径导入的形式，如：import "net/http"。

go的包搜索路径如下：对于包的查找，是有优先级的，编译器会优先在`GOROOT`里搜索，其次是`GOPATH`,一旦找到，就会马上停止搜索。如果最终都没找到，就报编译异常了。

远程包导入，如：**import** "github.com/spf13/cobra"

包重命名导入，如：

```go
package main
import (
    "fmt"
    myfmt "mylib/fmt"
)
func main() {
    fmt.Println()
    myfmt.Println()
}
```

## Go文档查看

在线浏览文档

```shell
lib godoc -http=:6060
```

第三方文档和包的集合库：https://gowalker.org/

## Go数组

数组是长度固定的数据类型，必须存储一段相同类型的元素，而且这些元素是连续的。我们这里强调固定长度，可以说这是和切片最明显的区别。

```go
// 声明
var array [5]int
// 创建初始化
array:=[5]int{1,2,3,4,5}
array:=[...]int{1,2,3,4,5}
// 只给固定索引初始化
array:=[5]int{1:1,3:4}

// 遍历数组
for i, v := range array {
  fmt.Printf("索引:%d,值:%d\n", i, v)
}
```

函数间传递数组，建议用指针的方式。

```go
func main() {
    array := [5]int{1: 2, 3:4}
    modify(&array)
    fmt.Println(array)
}
func modify(a *[5]int){
    a[1] =3
    fmt.Println(*a)
}
```

这里注意，数组的指针和指针数组是两个概念，数组的指针是`*[5]int`,指针数组是`[5]*int`，注意`*`的位置。

## Go切片

切片也是一种数据结构，它和数组非常相似，因为他是围绕动态数组的概念设计的，可以按需自动改变大小，使用这种结构，可以更方便的管理和使用数据集合。

```go
// 声明，指定切片长度为5
slice:=make([]int,5)
// 指定切片底层数组容量是10
slice:=make([]int,5,10)
// 类似数组一样的声明
slice:=[]int{1,2,3,4,5}
slice:=[]int{4:1}
// 基于原切片创建新切片，新的切片和原切片共用的是一个底层数组
newSlice := slice[1:3]

// 添加元素
slice=append(slice,10,20,30)
```

> 一般我们在创建新切片的时候，最好要让新切片的长度和容量一样，这样我们在追加操作的时候就会生成新的底层数组，和原有数组分离，就不会因为共用底层数组而引起奇怪问题,因为共用数组的时候修改内容，会影响多个切片。

```go
func main() {
    slice := []int{1, 2, 3, 4, 5}
    fmt.Printf("%p\n", &slice)
    modify(slice)
    fmt.Println(slice)
}
func modify(slice []int) {
    fmt.Printf("%p\n", &slice)
    slice[1] = 1
}
// 输出如下
0xc420082060
0xc420082080
[1 10 3 4 5]
```

仔细看，这两个切片的地址不一样，所以可以确认切片在函数间传递是复制的。而我们修改一个索引的值后，发现原切片的值也被修改了，说明它们共用一个底层数组。

## Go Map

Map是一种数据结构，是一个集合，用于存储一系列无序的键值对。它基于键存储的，键就像一个索引一样，这也是Map强大的地方，可以快速快速检索数据，键指向与该键关联的值。

```go
// 声明和初始化
dict:=make(map[string]int)
dict["张三"] = 43
dict := map[string]int{"张三":43,"李四":50}
// 这样声明的map是nil
var dict map[string]int

// 判断key是否存在
var dict map[string]int

// 删除一个kv对
delete(dict,"张三")

// 遍历map
for key, value := range dict {
    fmt.Println(key, value)
}

// 函数间传递
func main() {
    dict := map[string]int{"王五": 60, "张三": 43}
    modify(dict)
    fmt.Println(dict["张三"])
}
func modify(dict map[string]int) {
    dict["张三"] = 10
}
```

上面这个例子输出的结果是`10`,也就是说已经被函数给修改了，可以证明传递的并不是一个Map的副本。

## Go 类型

### 基本类型

基本类型是Go语言自带的类型，比如数值类型、浮点类型、字符类型以及布尔类型，他们本质上是原始类型，也就是不可改变的，所以对他们进行操作，一般都会返回一个新创建的值，所以把这些值传递给函数时，其实传递的是一个值的副本。

### 引用类型

引用类型和原始的基本类型恰恰相反，它的修改可以影响到任何引用到它的变量。在Go语言中，引用类型有切片、map、接口、函数类型以及`chan`。

> 本质上，我们可以理解函数的传递都是值传递，只不过引用类型传递的是一个指向底层数据的指针，所以我们在操作的时候，可以修改共享的底层数据的值，进而影响到所有引用到这个共享底层数据的变量。

### 结构类型

结构类型是用来描述一组值的，比如一个人有身高、体重、名字和年龄等,本质上是一种聚合型的数据类型。

```go
// 定义
type person struct {
    age int
    name string
}
// 初始化
jim := person{name:"Jim",age:10}
```

### 自定义类型

Go语言支持我们自定义类型，比如刚刚上面的结构体类型，就是我们自定义的类型，这也是比较常用的自定义类型的方法。

```go
type Duration int64
var i Duration = 100
var j int64 = 100
```

有时候，大家会迷茫，已经有了`int64`这些类型了，可以表示，还要基于他们创建新的类型做什么？其实这就是Go灵活的地方，我们可以使用自定义的类型做很多事情，比如添加方法，比如可以更明确的表示业务的含义等等。

## 函数方法

在Go语言中，函数和方法不太一样，有明确的概念区分。函数是指不属于任何结构体、类型的方法，也就是说，函数是没有接收者的；而方法是有接收者的，我们说的方法要么是属于一个结构体的，要么属于一个新定义的类型的。

### 函数

函数的定义声明没有接收者，所以我们直接在go文件里，go包之下定义声明即可。

```go
/*
 提供的常用库，有一些常用的方法，方便使用
*/package lib

// 一个加法实现// 返回a+b的值
// 函数名大写，相当于public
func Add(a, b int) int { 
   return a + b
}
```

### 方法

方法的声明和函数类似，他们的区别是：方法在定义的时候，会在`func`和方法名之间增加一个参数，这个参数就是接收者，这样我们定义的这个方法就和接收者绑定在了一起，称之为这个接收者的方法。

```go
type person struct {
    name string
}
// 值接收者
func (p person) String() string{ 
   return "the person name is "+p.name
}
// 指针接收者
func (p *person) modify(){
    p.name = "李四"
}
```

Go语言里有两种类型的接收者：值接收者和指针接收者。我们上面的例子中，就是使用值类型接收者的示例。

使用值类型接收者定义的方法，在调用的时候，使用的其实是值接收者的一个副本，所以对该值的任何操作，不会影响原来的类型变量。

> 在调用方法的时候，传递的接收者本质上都是副本，只不过一个是这个值副本，一是指向这个值指针的副本。指针具有指向原有值的特性，所以修改了指针指向的值，也就修改了原有的值。我们可以简单的理解为值接收者使用的是值的副本来调用方法，而指针接收者使用实际的值来调用方法。

不管是使用值接收者，还是指针接收者，一定要搞清楚类型的本质：对类型进行操作的时候，是要改变当前值，还是要创建一个新值进行返回？这些就可以决定我们是采用值传递，还是指针传递。

## Go接口

接口是一种约定，它是一个抽象的类型，和我们见到的具体的类型如int、map、slice等不一样。具体的类型，我们可以知道它是什么，并且可以知道可以用它做什么；但是接口不一样，接口是抽象的，它只有一组接口方法，我们并不知道它的内部实现，所以我们不知道接口是什么，但是我们知道可以利用它提供的方法做什么。

```go
type animal interface {
    call()
}

type Cat struct {
	Name string
	Age int
}

func (cat Cat) call()  {
	fmt.Println("I am " + cat.Name)
}

// 以指针实现接口
func (cat *Cat) call()  {
	fmt.Println("I am " + cat.Name)
}
```

接口的值是一个两个字长度的数据结构，第一个字包含一个指向内部表结构的指针，这个内部表里存储的有`实体类型`的信息以及相关联的方法集；第二个字包含的是一个指向存储的`实体类型`值的指针。所以接口的值结构其实是两个指针，这也可以说明接口其实一个引用类型。

**实体类型以值接收者实现接口的时候，不管是实体类型的值，还是实体类型值的指针，都实现了该接口**。

**实体类型以指针接收者实现接口的时候，只有指向这个类型的指针才被认为实现了该接口。**

## 嵌入类型

嵌入类型，或者嵌套类型，这是一种可以把已有的类型声明在新的类型里的一种方式，这种功能对代码复用非常重要。

在其他语言中，有继承可以做同样的事情，但是在Go语言中，没有继承的概念，Go提倡的代码复用的方式是组合，所以这也是嵌入类型的意义所在，组合而不是继承，所以Go才会更灵活。

```go
type user struct {
    name string
    email string
}
type admin struct {
    user
    level string
}

func main() {
    ad:=admin{user{"张三","zhangsan@flysnow.org"},"管理员"}
    fmt.Println("可以直接调用,名字为：",ad.name)
    fmt.Println("也可以通过内部类型调用,名字为：",ad.user.name)
    fmt.Println("但是新增加的属性只能直接调用，级别为：",ad.level)
}
```

嵌入后，被嵌入的类型称之为内部类型、新定义的类型称之为外部类型，这里`user`就是内部类型，而`admin`是外部类型。

外部类型也可以声明同名的字段或者方法，来覆盖内部类型的。

嵌入类型的强大，还体现在：如果内部类型实现了某个接口，那么外部类型也被认为实现了这个接口。

```go
func main() {
    ad:=admin{user{"张三","zhangsan@flysnow.org"},"管理员"}
    sayHello(ad.user)//使用user作为参数
    sayHello(ad)//使用admin作为参数
}
type Hello interface {
    hello()
}
func (u user) hello(){
    fmt.Println("Hello，i am a user")
}
func sayHello(h Hello){
    h.hello()
}
```

这个例子原来的结构体类型`user`和`admin`的定义不变，新增了一个接口`Hello`,然后让`user`类型实现这个接口，最后我们定义了一个`sayHello`方法，它接受一个`Hello`接口类型的参数，最终我们在main函数演示的时候，发现不管是`user`类型，还是`admin`类型作为参数传递给`sayHello`方法的时候，都可以正常调用。

# 参考文献

Go包管理：https://mp.weixin.qq.com/s/E5Ipd-geqcA-jURdndvyuw

Go开发工具：https://mp.weixin.qq.com/s/4hMr7FVsxfhn-0z9Y4Lm_w

Go Doc文档：https://mp.weixin.qq.com/s/7AL2sTgoG2l5wv0QEAVFsw

Go数组：https://mp.weixin.qq.com/s/-VSjc6sdic69MDDjdl8Yqg

Go切片：https://mp.weixin.qq.com/s/jadOBnBUo6D-CwqUoVzUhw

Go Map：https://mp.weixin.qq.com/s/oE0qlTm-3npq-zNqskgJNg

Go类型：https://mp.weixin.qq.com/s/cawYDcrgTkLtX1R4hLqvnQ

Go函数方法：https://mp.weixin.qq.com/s/3sjlDFSthVK3-E54TqqUqw

Go接口：https://mp.weixin.qq.com/s/LQcn1WKSYQE7vm8y0GQmig

Go嵌入类型：https://mp.weixin.qq.com/s/wcCRbXsP7DnENCvDtaKAaA