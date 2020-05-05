# Go语言学习笔记

Go语言实战笔记。

## 第2章：快速开始一个go程序

### point1

在Go语言里，标识符要么从包里公开，要么不从包里公开。

当代码导入一个包时，程序可以直接访问这个包中任意一个公开的标识符。这些标识符以**大写字母**开头。以**小写字母**开头的标识符是不公开的，不能被其他包中的代码直接访问。

```go
// module包中定义一个Cat的struct
type Cat struct {
	Name string
	Age int
}
// 其他包不能直接调用这个变量
small_cat := 0

// main包中调用
func main() {
  cat := new(module.Cat)
}
```

### point2

根据经验，如果需要声明初始值为零值(nil)的变量，应该使用var关键字声明变量；如果提供确切的非零值初始化变量或者使用函数返回值创建变量，应该使用简化变量声明运算符。

```go
var test
test2 := 0
test3 := some_func()
```

### point3

查找map里的键时，有两个选择：返回一个变量or两个变量。若指定了第二个值，就会返回一个布尔标志，来表示查找的key是否存在于map中。若key不存在，map会返回其**值类型的零值**作为返回值。

```go
test_map := make(map[string]string)
for index, value := range test_arr {
  // index为索引值
  index
  // value为当前值
  test_map_val, exists = test_map[value]
  if !exists {
    // do something
  }
}
```

### point4

非常推荐使用WaitGroup来跟踪goroutine的工作是否完成。WaitGroup是一个计数信号量，我们可以利用它来统计所有的goroutine是不是都完成了工作。

```go
// 初始化
var waitGroup sync.WaitGroup

// 将WaitGroup变量的值设置为将要启动的goroutine的数量
waitGroup.add(len(vals))

// 递减WaitGroup的计数
for _, val := range vals {
  go func() {
    // do something
    waitGroup.Done()
  }
}

// 等候所有任务完成，再继续运行下面的代码
waitGroup.Wait()
```

### point5

命名接口的时候，也需要遵守Go语言的命名惯例。如果接口类型只包含一个方法，那么这个类型的名字以er结尾，如Mather。如果接口类型内部声明了多个方法，其名字需要与其行为关联。

## 小结

- 每个代码文件都属于一个包，而包名应该与代码文件所在的文件夹名相同。
- 使用**指针**可以在函数间或者goroutine间共享数据，go语言的值传递使用的是copy的方式，因此需要共享数据时，必须使用指针。