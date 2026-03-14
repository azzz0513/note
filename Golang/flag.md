### os.Args
```Go
package main

import (
	"fmt"
	"os"
)

func main() {
	// os.Args是一个[]string
	if len(os.Args) > 0 {
		for index, arg := range os.Args {
			fmt.Printf("args[%d]=%s\n", index, arg)
		}
	}
}
```
```
PS C:\code_go\flag_study> ./flag_study 1 2 3
args[0]=C:\code_go\flag_study\flag_study.exe
args[1]=1
args[2]=2
args[3]=3
```


### flag
Go语言内置的`flag`包实现了命令行参数的解析，`flag`包使得开发命令行工具更为简单。

flag包支持的命令行参数类型有bool、int、int64、uint、uint64、float、float64、string、duration。

#### 定义命令行flag参数
##### flag.Type()
基本格式如下：
```Go
flag.Type(flag名, 默认值, 帮助信息) *Type
```
例如我们要定义名称、年龄两个命令行参数，可以按如下方式定义：
```Go
name := flag.String("name", "张三", "姓名")
age := flag.Int("age", 18, "年龄")
delay := flag.Duration("d", 0, "时间间隔")
```
`name`，`age`，`delay`此时均为对应类型的指针。

##### flag.TypeVar()
基本格式如下：
```Go
flag.TypeVar(Type指针, flag名, 默认值, 帮助信息)
```
例如我们要定义名称、年龄两个命令行参数，可以按如下方式定义：
```Go
var name string
var age int
var delay time.Duration

flag.StringVar(&name, "name", "张三", "姓名")
flag.IntVar(&age, "age", 18, "年龄")
flag.DurationVar(&delay, "d", 0, "时间间隔")
```

#### flag.Parse()
通过以上两种方法定义好命令行flag参数后，需要通过调用`flag.Parse()`来对命令行参数进行解析。

支持的命令行参数格式有以下几种：
- `-flag xxx`：使用空格，一个-符号
- `--flag xxx`：使用空格，两个-符号
- `-flag=xxx`：使用等号，一个-符号
- `--flag=xxx`：使用等号，两个--符号
==其中，布尔类型的参数必须使用等号的方式指定==

Flag解析在第一个非flag参数（单个“-”不是flag参数）之前停止，或者在终止符“-”之后停止。

#### 其他参数
```Go
flag.Args() // 返回命令行参数后的其他参数，以[]string类型
flag.NArgs() // 返回命令行参数后的其他参数个数
flag.NFlag() // 返回使用的命令行参数个数
```

### 示例
```Go
package main

import (
	"flag"
	"fmt"
	"time"
)

func main() {
	// 定义命令行参数方式
	var name string
	var age int
	var married bool
	var delay time.Duration
	flag.StringVar(&name, "name", "张三", "姓名")
	flag.IntVar(&age, "age", 18, "年龄")
	flag.BoolVar(&married, "married", false, "婚否")
	flag.DurationVar(&delay, "d", 0, "时间间隔")

	// 解析命令行参数
	flag.Parse()
	fmt.Println(name, age, married, delay)
	// 返回命令行参数后的其他参数
	fmt.Println(flag.Args())
	// 返回命令行参数后的其他参数个数
	fmt.Println(flag.NArg())
	// 返回使用的命令行参数个数
	fmt.Println(flag.NFlag())
}
```