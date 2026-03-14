### viper
Viper是适用于Go应用程序的完整配置解决方案。它被设计用于在应用程序中工作，并且可以处理所有类型的配置需求和格式。

其支持以下特性：
- 设置默认值
- 从JSON、TOML、YAML、HCL、envfile、Java properties格式的配置文件读取配置信息
- 实时监控和重新读取配置文件
- 从环境变量中读取
- 从远程配置系统（etcd或Consul）读取并监控配置变化
- 从命令行参数读取配置
- 从buffer读取配置
- 显式配置值

#### 安装
`go get github.com/spf13/viper`

#### 提供的操作
- 查找、加载和反序列化JSON、TOML、YAML、HCL、envfile、Java properties格式的配置文件
- 提供一种机制为你的不同配置选项设置默认值
- 提供一种机制来通过命令行参数覆盖指定选项的值
- 提供别名系统，以便在不破坏现有代码的情况下轻松重命名参数
- 当用户提供了与默认值相同的命令行或配置文件时，可以很容易地分辨出它们之间的区别

#### 优先级
每个项目的优先级都高于它下面的项目：
- 显示调用Set设置值
- 命令行参数（flag）
- 环境变量
- 配置文件
- key/value存储
- 默认值

==目前Viper配置的键（Key）是大小写不敏感的==


#### 把值存入Viper
##### 建立默认值
一个好的配置系统应该支持默认值。键不需要默认值，但如果没有通过配置文件、环境变量、远程配置或命令行标志（flag）设置键，则默认值非常有用。

```Go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

func main() {
	// 设置默认值
	viper.SetDefault("fileDir", "./test1/")

	// 读取配置文件
	viper.SetConfigName("config")                // 配置文件名称（无扩展名）
	viper.SetConfigType("yaml")                  // 如果配置文件的名称中没有扩展名，则需要配置此项
	viper.AddConfigPath(".")                     // 查找配置文件所在的路径
	//viper.AddConfigPath("$HOME/.config/code/go") // 多次调用以添加多个搜索路径
	//viper.AddConfigPath(".")                     // 还可以在工作目录中查找配置

	err := viper.ReadInConfig() // 查找并读取配置文件
	if err != nil {
		// 处理读取配置文件的错误
		panic(fmt.Errorf("Fatal error config file: %s \n", err))
	}
}
```

还可以像下面这样处理找不到配置文件的特定情况：
```Go
if err := viper.ReadInConfig(); err != nil {
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {
        // 配置文件未找到错误；如果需要可以省略
    } else {
        // 配置文件被找到，但产生了另外的错误
    }
}

// 配置文件找到并成功解析
```

##### 写入配置文件
从配置文件中读取配置文件是有用的，但是有时想要存储在运行时所做的所有修改。为此，可以使用下面一组命令，每个命令都有自己的用途：
- WriteConfig：将当前的viper写入预定义的路径并覆盖（如果存在的话）。如果没有预定义的路径，则报错。
- SafeWriteConfig：将当前的viper配置写入预定义的路径。如果没有预定义的路径，则报错。如果存在，将不会覆盖当前的配置文件。
- WriteConfigAs：将当前的viper配置写入给定的文件路径。将覆盖给定的文件（如果存在的话）。
- SafeWriteConfigAs：将当前的viper配置写入给定的文件路径。不会覆盖给定的文件（如果存在的话）。

标记为`safe`的所有方法都不会覆盖任何文件，而是直接创建（如果不存在），而默认行为是创建或截断。

```Go
viper.WriteConfig() // 将当前配置写入“viper.AddConfigPath()”和“viper.SetConfigName”设置的预定义路径
viper.SafeWriteConfig()
viper.WriteConfigAs("/path/to/my/.config")
viper.SafeWriteConfigAs("/path/to/my/.config") // 因为该配置文件写入过，所以会报错
viper.SafeWriteConfigAs("/path/to/my/.other_config")
```

#### 监控并重新读取配置文件
Viper支持在运行时实时读取配置文件的功能。
viper驱动的应用程序可以在运行时读取配置文件的更新，而不会错过任何信息。
只需告诉viper实例watchConfig。可选的，可以为Viper提供一个回调函数，以便在每次发生更改时运行。

```Go
package main

import (
	"fmt"
	"github.com/fsnotify/fsnotify"
	"github.com/gin-gonic/gin"
	"github.com/spf13/viper"
	"net/http"
)

func main() {
	// 设置默认值
	viper.SetDefault("fileDir", "./test1/")

	// 读取配置文件
	viper.SetConfigName("config") // 配置文件名称（无扩展名）
	viper.SetConfigType("yaml")   // 如果配置文件的名称中没有扩展名，则需要配置此项
	viper.AddConfigPath(".")      // 查找配置文件所在的路径
	//viper.AddConfigPath("$HOME/.config/code/go") // 多次调用以添加多个搜索路径
	//viper.AddConfigPath(".") // 还可以在工作目录中查找配置

	err := viper.ReadInConfig() // 查找并读取配置文件
	if err != nil {
		// 处理读取配置文件的错误
		panic(fmt.Errorf("Fatal error config file: %s \n", err))
	}
	// 实时监控配置文件的变化
	viper.WatchConfig()
	// 当配置变化之后调用的一个回调函数
	viper.OnConfigChange(func(e fsnotify.Event) {
		// 配置文件发生变更之后会调用的回调函数
		fmt.Println("Config file changed:", e.Name)
	})

	r := gin.Default()
	r.GET("/viper", func(c *gin.Context) {
		c.String(http.StatusOK, viper.GetString("version"))
	})
	r.Run(":8080")
}
```

###### 确保在调用`WatchConfig()`之前添加了所有的配置路径
```Go
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event)) {
    // 配置文件发生变更之后会调用的回调函数
    fmt.Println("Config file changed:", e.Name)
}
```

#### 从io.Reader读取配置
Viper预先定义了许多配置源，如文件、环境变量、标志和远程K/V存储，但并不受其约束。还可以实现自己所需的配置源并将其提供给viper。
```Go
viper.SetConfigType("yaml") // 或者viper.SetConfigType("YAML")

// 任何需要将此配置添加到程序中的方法
var yamlExample = []byte(`
Hacker: true
name: steve
hobbies:
- skateboarding
- snowboarding
- go
clothing:
	jacket: leather
	trousers: denim
age: 35
eyes: brown
beard: true
`)

viper.ReadConfig(bytes.NewBuffer(yamlExample))

viper.Get("name") // 这里会得到"steve"
```


#### 覆盖设置
这些可能来自命令行标志，也可能来自我们自己的应用程序逻辑。
```Go
viper.Set("Verbose", true)
viper.Set("LogFile", LogFile)
```

#### 注册和使用别名
别名允许多个键引用单个值
```Go
viper.RegisterAlias("loud", "Verbose") // 注册别名（此处loud和Verbose建立了别名）

viper.Set("verbose", true) // 结果与下一行相同
viper.Set("loud", true) // 结果与前一行相同

viper.GetBool("loud") // true
viper.GetBool("verbose") // true
```

#### 使用环境变量
Viper完全支持环境变量。这使`Twelve-Factor App`开箱即用。有五种方法可以帮助与ENV协作：
- AutomaticEnv()
- BindEnv(string...) : error
- SetEnvPrefix(string)
- SetEnvKeyReplacer(string...) *strings.Replacer
- AllowEmptyEnv(bool)
使用ENV变量时，务必要意识到Viper将ENV变量视为区别大小写。

Viper提供了一种机制来确保ENV变量时唯一的。通过使用`SetEnvPrefix`，可以告诉Viper在读取环境变量时使用前缀。`BindEnv`和`AutomaticEnv`都将使用这个前缀。

`BindEnv`使用一个或两个参数。第一个参数是键名称，第二个是环境变量的名称。环境变量的名称区分大小写。如果没有提供ENV变量名，那么Viper将自动假设ENV变量与以下格式匹配：前缀+“-”+键名全部大写。当显式提供ENV变量名（第二个参数）时，它不会自动添加前缀。例如，如果第二个参数是“id”，Viper将查找环境变量“ID”。

在使用ENV变量时，需要注意的一件重要事情是，每次访问该值时都将读取它。Viper在调用`BindEnv`时不固定值。

`AutomaticEnv`是一个强大的助手，尤其是与`SetEnvPrefix`结合使用时。调用时，Viper会在发出`viper.Get`请求时随时检查环境变量。它将应用以下规则。它将检查环境变量的名称是否与键匹配（如果设置了`EnvPrefix`）。

`SetEnvKeyReplacer`允许你使用`strings.Replacer`对象在一定程度上重写Env键。如果你希望在`Get()`调用中使用`-`或者其他什么符号，但是环境变量里使用`_`分隔符，那么这个功能是非常有用的。可以在`viper_test.go`中找到它的使用示例。

或者，你可以使用带有`NewWithOptions`工厂函数的`EnvKeyReplacer`。与`SetEnvKeyReplacer`不同，它接受`StringReplacer`接口，允许你编写自定义字符串替换逻辑。

默认情况下，空环境变量被认为是未设置的，并将返回到下一个配置源。若要将空环境变量视为已设置，可以使用`AllowEmptyEnv`方法。
```Go
SetEnvPrefix("spf") // 将自动转为大写
BindEnv("id")

os.Setenv("SPF_ID", "13") // 通常是在应用程序之外完成的

id := Get("id") // 13
```


#### 使用Flags
Viper具有绑定到标志的能力。Viper支持Cobra库中使用的`Pflag`。

与`BindEnv`类似，该值不是在调用绑定方法时设置的，而是在访问该方法时设置的。这意味着我们可以根据需要尽早进行绑定，即使在`init()`函数中也是如此。

对于单个标志，`BindPFlag()`方法提供此功能。
```Go
serverCmd.Flags().Int("port", 1138, "Port to run Application server on")
viper.BindPFlag("port", serverCmd.Flags().Lockup("port"))
```


#### 从Viper获取值
在Viper中，有几种方法可以根据值的类型获取值：
- `Get(key string) : interface{}`
- `GetBool(key string) : bool`
- `GetFloat64(key string) : float64`
- `GetInt(key string) : int`
- `GetIntSlice(key string) : []int`
- `GetString(key string) : string`
- `GetStringMap(key string) : map[string]interface{}`
- `GetStringMapString(key string) : map[string]string`
- `GetStringSlice(key string) : []string`
- `GetTime(key string) : time.Time`
- `GetDuration(key string) : time.Duration`
- `IsSet(key string) : bool`
- `AllSettings() : map[string]interface{}`
==每一个Get方法在找不到值时都会返回对应类型零值。为了检查给定的键是否存在，提供了`IsSet()`方法==

##### 访问嵌套的键
访问器方法也接受深度嵌套键的格式化路径。例如，如果加载下面的JSON文件：
```json
{
	"host": {
		"address": "localhost",
		"port": 5799
	},
	"datastore": {
		"metric": {
			"host": "127.0.0.1",
			"port": 3099
		},
		"warehouse": {
			"host": "198.0.0.1",
			"pory": 2112
		}
	}
}
```
Viper可以通过传入`.`分割的路径来访问嵌套字段：
```Go
Getstring("datastore.metric.host") // 返回“127.0.0.1”
```

##### 提取子树
从Viper中提取子树。

例：viper实例现在代表了以下配置：
```Go
app:
	cache1:
		max-items: 100
		item-size: 64
	cache2:
		max-items: 200
		item-size: 80
```
执行后：
```Go
subv := viper.Sub("app.cache1")
```
`subv`现在就代表：
```Go
max-items: 100
item-size: 64
```

##### 反序列化
可以选择将所有或特定的值解析到结构体、map等。
- `Unmarshal(rawVal interface{}) error`
- `UnmarshalKey(key string, rawVal interface{}) error`

```Go
type config struct {
	Port int
	Name string
	PathMap string `mapstructure:"path_map"`
}

var C config

err := viper.Unmarshal(&C)
if err != nil {
	t.Fatalf("unable to decode into struct, %v", err)
}
```
如果想要解析的那些键本身就包含`.`（默认的键分隔符）的配置，就需要修改分隔符
```Go
v := viper.NewWithOptions(viper.KeyDelimiter("::"))
```

##### 序列化成字符串
可能需要将viper中保存的所有设置序列化到一个字符串中，而不是将它们写入到一个文件中。可以将自己喜欢的格式的序列化器与`AllSettings()`返回的配置一起使用。
```Go
import (
	yaml "gopkg.in/yaml.v2"
	// ...
)

func yamlStringSettings() string {
	c := viper.AllSettings()
	bs, err := yaml.Marshal(c)
	if err != nil {
		log.Fatalf("unable to marshal config to YAML: %v", err)
	}
	return string(bs)
}
```