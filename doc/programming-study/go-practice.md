# Go语言实战

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

# 参考文献

Go包管理：https://mp.weixin.qq.com/s/E5Ipd-geqcA-jURdndvyuw

Go开发工具：https://mp.weixin.qq.com/s/4hMr7FVsxfhn-0z9Y4Lm_w

Go Doc文档：https://mp.weixin.qq.com/s/7AL2sTgoG2l5wv0QEAVFsw