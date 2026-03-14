### 路由
在Web开发中，路由是指根据HTTP请求的 URL路径 和 方法（GET/POST等），将请求分发到对应的处理函数（Handler）的机制。

#### 路由表（Web框架中）
定义URL模式与处理函数的映射关系
```Go
// Gin框架示例
router := gin.Default()
router.GET("/users", listUsers)      // GET /users → listUsers函数
router.POST("/users", createUser)   // POST /users → createUser函数
router.GET("/users/:id", getUser)   // 动态路由参数 :id
```
- 静态路由：精确匹配路径（如 /about）。
- 动态路由：通过占位符捕获参数（如 /users/:id）。
- 通配符路由：匹配多级路径（如 /files/*filepath）。
例：当用户访问`https://example.com/users/42`时：
1.Web服务器解析请求路径为 /users/42，方法为 GET。
2.路由匹配到 /users/:id 模式，提取参数 id=42。
3.调用getUser处理函数，返回ID为42的用户信息。


### gin hello world
```Go
package main

import "github.com/gin-gonic/gin"

func main() {
	// 创建一个默认的路由（Handler）
	router := gin.Default()

	// 绑定路由规则和路由函数，访问/index的路由，将由对应的函数去处理
	router.GET("/index", func(context *gin.Context) {
		context.String(200, "hello world")
	})

	// 启动监听，gin会把web服务运行到本机的0.0.0.0:8080端口上（0.0.0.0通常用来表示一个设备或计算机网络接口上所有IPv4地址）
    // 修改ip为内网ip：router.Run("0.0.0.0:8080")
	router.Run(":8080")
    // 启动方式2
    //http.ListenAndServer(":8080", router)
}
```

### 响应
服务器处理完请求后返回的“回答”，包含客户端需要的数据或操作结果。
#### 响应字符串
```Go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.GET("/", func(c *gin.Context) {
		// 状态码：200表示正常响应，等价于http.StatusOK
		c.String(200, "你好")
	})
	router.Run(":8080")
}
```

#### 响应json
```Go
package main

import "github.com/gin-gonic/gin"

func main() {
	router := gin.Default()
	router.GET("/", func(c *gin.Context) {
		// json响应结构体
		type UserInfo struct {
			Name     string `json:"user_name"`
			Age      int    `json:"user_age"`
			Password string `json:"-"` // 忽略转换为json
		}
		//user := UserInfo{"张三", 20, "123456"}
		//c.JSON(200, user)

		// json响应map
		//userMap := map[string]string{
		//	"user_name": "张三",
		//	"age":       "20",
		//}
		//c.JSON(200, userMap)

		// 直接响应json
		c.JSON(200, gin.H{"user_name": "张三", "age": 20})
	})
	router.Run(":8080")
}
```

#### 响应XML
```Go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.GET("/", func(c *gin.Context) {
		c.XML(200, gin.H{"user": "zhangsan", "message": "hello world", "status": http.StatusOK})
	})
	router.Run(":8080")
}
```

#### 响应html
返回结构体：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<header>你好啊 姓名：{{ .Name }} 年龄：{{ .Age }}</header>
</body>
</html>
```
```Go
package main

import "github.com/gin-gonic/gin"

func main() {
	router := gin.Default()
	// 加载模板目录下的所有文件
	router.LoadHTMLGlob("templates/*")
	type Person struct {
		Name string
		Age  int
	}
	var user Person = Person{"张三", 20}
	router.GET("/", func(c *gin.Context) {
		c.HTML(200, "index.html", user)
	})
	router.Run(":8080")
}
```

返回gin.H{}：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<header>你好啊 姓名：{{ .username }}</header>
</body>
</html>
```
```Go
package main

import "github.com/gin-gonic/gin"

func main() {
	router := gin.Default()
	// 加载模板目录下的所有文件
	router.LoadHTMLGlob("templates/*")
	router.GET("/", func(c *gin.Context) {
		c.HTML(200, "index.html", gin.H{"username": "张三"})
	})
	router.Run(":8080")
}
```


#### 文件响应
前端请求后端接口，然后唤起浏览器下载：
```Go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	// 用于浏览器直接请求这个接口唤起下载
	// 要设置Content-Type，唤起浏览器下载
	// 只能是GET请求
	router.GET("/download_file", func(c *gin.Context) {
		// 表示是文件流，唤起浏览器下载，一般设置了这个，就要设置文件名
		c.Header("Content-Type", "application/octet-stream")
		// 用来指定下载下来的文件名
		c.Header("Content-Disposition", "attachment; filename=test_file.go")
		c.File("test_file.go")
	})
	router.Run(":8080")
}
```

前端唤起浏览器下载的本质：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>浏览器下载</title>
</head>
<body>
<a href="./abc" download="abc文件名.txt">文件下载</a>
</body>
</html>
```

#### 静态文件
```Go
package main

import "github.com/gin-gonic/gin"

func main() {
	router := gin.Default()
	// 第一个参数为别名，第二个参数是实际路径
	router.Static("st", "./static")
	// 静态文件的路径不能再被路由使用
	// 如下面语句后不能再GET /abc
	router.StaticFile("abc", "./static/abc")
	router.Run(":8080")
}
```


#### 重定向
```Go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	router.GET("/baidu", func(c *gin.Context) {
		// 状态码为301，永久重定向
		c.Redirect(301, "http://www.baidu.com")
	})
	router.LoadHTMLGlob("templates/*")
	type Person struct {
		Name string
		Age  int
	}
	var user Person = Person{"张三", 20}
	router.GET("/html", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.html", user)
	})
	router.GET("/png", func(c *gin.Context) {
		// 状态码为302，临时重定向
		c.Redirect(302, "/html")
	})
	router.Run(":8080")
}
```


### 请求
客户端向服务器发送的“要求”，目的是获取资源（如网页、图片）或提交数据（如登录表单）。
#### 查询参数Query
不是GET请求专属的

```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.GET("/query", func(c *gin.Context) {
		// 返回指定的查询参数
		// GET /path?id=1234&name=Manu&value=
		// c.Query("id") == "1234"
		// c.Query("name") == "Manu"
		// c.Query("value") == ""
		// c.Query("wtf") == ""
		user := c.Query("user")
		// 返回一个string和一个bool判断是否有对应的查询参数
		//	GET /?name=Manu&lastname=
		//	("Manu", true) == c.GetQuery("name")
		//	("", false) == c.GetQuery("id")
		//	("", true) == c.GetQuery("lastname")
		fmt.Println(c.GetQuery("user"))
		// 拿到多个相同的查询参数
		// GET /?user=张三&user=李四 == [张三 李四]
		fmt.Println(c.QueryArray("user"))
		// 返回map名为指定字符串的查询参数的map
		// GET /?user[name]=张三&user[id]=2
		// c.QueryMap("user") == map[id:2 user:张三]
		fmt.Println(c.QueryMap("user"))
		// 如果用户没传，就使用默认值
		fmt.Println(c.DefaultQuery("addr", "广东省"))
		fmt.Println(user)
	})
	router.Run(":8080")
}
```


#### 动态参数Param
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.GET("/param/:user_id/:name", func(c *gin.Context) {
		// GET /param/123/张三
		fmt.Println(c.Param("user_id")) // 123
		fmt.Println(c.Param("name")) // 张三
	})
	router.Run(":8080")
}
```


#### 表单参数PostForm
可以接收`multiple/form-data`和`application/x-www-form-urlencoded`这些MIME类型（媒体类型）
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	// 建议使用POST请求，GET请求无法访问表单
	router.POST("/form", func(c *gin.Context) {
		fmt.Println(c.PostForm("name"))
		fmt.Println(c.PostFormArray("name"))
		// 通过ok判断是否有传入数据
		age, ok := c.GetPostForm("age")
		fmt.Println(age, ok)
		// 如果用户没传，就使用默认值
		fmt.Println(c.DefaultPostForm("addr", "广东省"))
		forms, err := c.MultipartForm() // 接收所有的form参数，包括文件
		fmt.Println(forms, err)
	})
	router.Run(":8080")
}
```

#### 文件上传
##### 单文件上传
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.POST("/upload", func(c *gin.Context) {
		fileHeader, err := c.FormFile("file")
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(fileHeader.Filename) // 文件名
		fmt.Println(fileHeader.Size)     // 文件大小，单位为字节

		//file, _ := fileHeader.Open()
		//byteData, _ := io.ReadAll(file)
		//err = os.WriteFile("../static/xxx.png", byteData, 0666)
		//fmt.Println(err)

		err = c.SaveUploadedFile(fileHeader, "../static/"+fileHeader.Filename)
		fmt.Println(err)
	})
	router.Run(":8080")
}
```

##### 多文件上传
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.POST("/upload", func(c *gin.Context) {
		form, err := c.MultipartForm()
		if err != nil {
			fmt.Println(err)
			return
		}
		for _, headers := range form.File {
			for _, header := range headers {
				err = c.SaveUploadedFile(header, "../static/upload"+header.Filename)
				if err != nil {
					fmt.Println(err)
				}
			}
		}
	})
	router.Run(":8080")
}
```


#### 原始内容RawData
body阅后即焚问题解决：
```Go
byteData, _ := io.ReadAll(c.Request.Body)
fmt.Println(string(byteData))
c.Request.Body = io.NopCloser(bytes.NewReader(byteData))
```

```Go
// form-data
----------------------------721212574584856236307076
Content-Disposition: form-data; name="name"

abc
----------------------------721212574584856236307076
Content-Disposition: form-data; name="age"

20
----------------------------721212574584856236307076--

// x-www-form-urlencoded
name=abc&age=20

// json
{
    "name": "abc",
    "age": 20
}
```

```Go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.POST("/raw", func(c *gin.Context) {
		body, _ := c.GetRawData()
		contentType := c.GetHeader("Content-Type") // 获取对应的MIME类型
		switch contentType {
		case "application/json":
			// json解析到结构体
			type User struct {
				Name string `json:"name"`
				Age  int    `json:"age"`
			}
			var user User
			err := json.Unmarshal(body, &user)
			if err != nil {
				fmt.Println("json err:", err)
				return
			}
			fmt.Println(user) // {abc 20}
		}
		fmt.Println(string(body))
	})
	router.Run(":8080")
}
```

#### binding
##### 参数绑定
###### 查询参数
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.GET("", func(c *gin.Context) {
		type User struct {
			Name string `form:"name"`
			Age  int    `form:"age"`
		}
		var user User
		err := c.ShouldBindQuery(&user)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(user)
	})
	router.Run(":8080")
}
```

###### 路径参数
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.GET("/users/:age/:name", func(c *gin.Context) {
		type User struct {
			Name string `uri:"name"`
			Age  int    `uri:"age"`
		}
		var user User
		err := c.ShouldBindUri(&user)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(user)
	})
	router.Run(":8080")
}
```

###### 表单参数
==不能解析x-www-form-urlencoded的格式==
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.POST("/form", func(c *gin.Context) {
		type User struct {
			Name string `form:"name"`
			Age  int    `form:"age"`
		}
		var user User
		err := c.ShouldBind(&user)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(user)
	})
	router.Run(":8080")
}
```

###### json参数
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.POST("/json", func(c *gin.Context) {
		type User struct {
			Name string `json:"name"`
			Age  int    `json:"age"`
		}
		var user User
		err := c.ShouldBindJSON(&user)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(user)
	})
	router.Run(":8080")
}
```

###### header参数
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.POST("/header", func(c *gin.Context) {
		type User struct {
			Name string `header:"Name"`
			Age  int    `header:"Age"`
		}
		var user User
		err := c.ShouldBindHeader(&user)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(user)
	})
	router.Run(":8080")
}
```

##### 内置规则
```Go
package main

import "github.com/gin-gonic/gin"

func main() {
	/*
				// 不能为空，并且不能没有这个字段
				required：必填字段，如：binding:"required"

				// 针对字符串的长度
				min 最小长度，如：binding:"min=4"
				max 最大长度，如：binding:"max=10"
				len 长度，如：binding:"len=6"

				// 针对数字的大小
				eq 等于，如：binding:"eq=3"
				ne 不等于，如：binding:"ne=12"
				gt 大于
				gte 大于等于
				lt 小于
				lte 小于等于

			// 针对同级字段
			eqfield 等于其他字段的值，如：PassWord string `binding:"eqfield=PassWord"`
			nefield 不等于其他字段的值

			- 忽略字段，如：binding:"-" 或者不写

			// 枚举	只能是red或green，如：binding:"oneof=red green"

		// 字符串
		contains 包含指定的字符串
		excludes 不包含
		startswith 字符串前缀
		endswith 字符串后缀

		// 数组
		dive dive后面的验证就是针对数组中的每一个元素

		// 网络验证
		ip
		ipv4
		ipv6
		uri 在于I（Identifier）是统一资源标识符，可以唯一标识一个资源
		url 在于Locater，是统一资源定位符，提供找到该资源的确切路径

		// 日期验证 2006年1月2日下午3时4分5秒
		datetime


	*/
	router := gin.Default()
	router.POST("/json", func(c *gin.Context) {
		type User struct {
			//Name string `json:"name" binding:"required, min=3, min=5"`
			//Age  int    `json:"age"`
			Pwd   string `json:"pwd" binding:"required"`
			RePwd string `json:"re_pwd" binding:"eqfield=Pwd"`
		}
		var user User
		err := c.ShouldBindJSON(&user)
		if err != nil {
			c.String(200, err.Error())
			return
		}
		c.JSON(200, user)
	})
	router.Run(":8080")
}
```

##### 错误信息显示为中文
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/locales/zh"
	"github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"
	zh_translations "github.com/go-playground/validator/v10/translations/zh"
	"net/http"
	"strings"
)

var trans ut.Translator

func init() {
	// 创建翻译器
	uni := ut.New(zh.New())
	trans, _ = uni.GetTranslator("zh")

	// 注册翻译器
	v, ok := binding.Validator.Engine().(*validator.Validate)
	if ok {
		_ = zh_translations.RegisterDefaultTranslations(v, trans)
	}
}

func ValidateErr(err error) string {
	errs, ok := err.(validator.ValidationErrors)
	if !ok {
		return err.Error()
	}
	var msgList []string
	for _, err := range errs {
		msgList = append(msgList, err.Translate(trans))
	}
	return strings.Join(msgList, ";")
}

type User struct {
	Name  string `json:"name" binding:"required"`
	Email string `json:"email" binding:"required,email"`
}

func main() {
	r := gin.Default()
	// 注册路由
	r.POST("/user", func(c *gin.Context) {
		var user User
		if err := c.ShouldBindBodyWithJSON(&user); err != nil {
			// 参数验证失败
			c.String(200, ValidateErr(err))
			return
		}

		// 参数验证成功
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("Hello, %s! Your email is %s.", user.Name, user.Email),
		})
	})

	// 启动HTTP服务器
	r.Run(":8080")
}
```

#### 四大请求方式
Restful风格指的是网络应用中资源定位和资源操作的风格，不是标准也不是协议。
- GET：从服务器取出资源（一项或多项）
- POST：在服务器新建一个资源
- PUT：在服务器更新资源（客户端提供完整资源数据）
- PATCH：在服务器更新数据（客户端提供需要修改的资源数据）
- DELETE：从服务器删除数据


```Go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/gin-gonic/gin"
)

type ArticleModel struct {
	Title   string `json:"title"`
	Content string `json:"content"`
}
type Response struct {
	Code int    `json:"code"`
	Data any    `json:"data"`
	Msg  string `json:"msg"`
}

// 文章列表页面
func _getList(c *gin.Context) {
	articleList := []ArticleModel{
		{"Go语言入门", "这篇文章是《Go语言入门》"},
		{"Python语言入门", "这篇文章是《Python语言入门》"},
		{"Java语言入门", "这篇文章是《Java语言入门》"},
	}
	c.JSON(200, Response{0, articleList, "成功"})
}

// 文章详情页面
func _getDetail(c *gin.Context) {
	// 获取param中的id
	fmt.Println(c.Param("id"))
	article := []ArticleModel{
		{"Go语言入门", "这篇文章是《Go语言入门》"},
	}
	c.JSON(200, Response{0, article, "成功"})
}

// 创建文章
func _create(c *gin.Context) {
	// 接收前端传递来的json数据
	var article ArticleModel
	body, _ := c.GetRawData()
	contentType := c.GetHeader("Content-Type")
	switch contentType {
	case "application/json":
		err := json.Unmarshal(body, &article)
		if err != nil {
			fmt.Println(err.Error())
			return
		}
	}

	c.JSON(200, Response{0, article, "添加成功"})
}

// 编辑文章
func _update(c *gin.Context) {
	fmt.Println(c.Param("id"))
	var article ArticleModel
	body, _ := c.GetRawData()
	contentType := c.GetHeader("Content-Type")
	switch contentType {
	case "application/json":
		err := json.Unmarshal(body, &article)
		if err != nil {
			fmt.Println(err.Error())
			return
		}
	}
	c.JSON(200, Response{0, article, "修改成功"})
}

// 删除文章
func _delete(c *gin.Context) {
	fmt.Println(c.Param("id"))
	c.JSON(200, Response{0, gin.{}, "删除成功"})
}

func main() {
	router := gin.Default()
	router.GET("/articles", _getList)       // 文章列表
	router.GET("/articles/:id", _getDetail) // 文章详情
	router.POST("/articles", _create)       // 添加文章
	router.PUT("/articles/:id", _update)    // 编辑文章
	router.DELETE("/articles/:id", _delete) // 删除文章

	router.Run(":8080")
}
```

**路由分组**：把一类api划分到一个组，可以给这个组加上统一的中间件
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func UserView(c *gin.Context) {
	path := c.Request.URL
	fmt.Println(c.Request.Method, path)
}

func main() {
	router := gin.Default()
	apiGroup := router.Group("/api")
	UserGroup(apiGroup)
	router.Run(":8080")
}

func UserGroup(r *gin.RouterGroup) {
	r.GET("/users", UserView)    // GET /api/users
	r.POST("/users", UserView)   // POST /api/users
	r.PUT("/users", UserView)    // PUT /api/users
	r.DELETE("/users", UserView) // DELETE /api/users
}
```


### 中间件
![alt text](images/image-13.png)
#### 局部中间件
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func Home(c *gin.Context) {
	fmt.Println("home")
	c.String(200, "home")
}

func M1(c *gin.Context) {
	fmt.Println("M1 请求部分")
	c.Next()
	// 拦截（会原路返回 ）
	//c.Abort()
	// 拦截并返回响应
	//c.Abort()
	//c.String(200, "M1返回的响应")
	//return
	fmt.Println("M1 响应部分")
}
func M2(c *gin.Context) {
	fmt.Println("M2 请求部分")
	c.Next()
	fmt.Println("M2 响应部分")
}

func main() {
	router := gin.Default()
	// M1请求部分-->M2请求部分-->Home-->M2响应部分-->M1响应部分
	router.GET("/middleware", M1, M2, Home)
	router.Run(":8080")
}
```

#### 全局中间件
全局也就是路由组
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func Home2(c *gin.Context) {
	fmt.Println("Home")
	c.String(200, "Home")
}

func GM1(c *gin.Context) {
	fmt.Println("GM1 请求部分")
	c.Next()
	fmt.Println("GM1 响应部分")
}

func GM2(c *gin.Context) {
	fmt.Println("GM2 请求部分")
	c.Next()
	fmt.Println("GM2 响应部分")
}

func main() {
	router := gin.Default()
	g := router.Group("/api")
	g.Use(GM1, GM2)
	g.GET("/users", Home2)
	router.Run(":8080")
}
```

#### 中间件传递参数
```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

type User struct {
	Name string
}

func Home2(c *gin.Context) {
	fmt.Println("Home")
	fmt.Println(c.Get("GM1"))
	fmt.Println(c.Get("GM2"))
	_user, ok := c.Get("user")
	if ok {
		// 类型断言
		user, ok := _user.(User)
		if ok {
			fmt.Println(user.Name)
		}
	}
	c.String(200, "Home")
}

func GM1(c *gin.Context) {
	fmt.Println("GM1 请求部分")
	var user User = User{
		Name: "张三",
	}
	// Set方法可以传递Go中的所有数据类型
	// 在后续的阶段中都可以读到：GM2请求部分、Home、GM2响应部分、GM1响应部分
	c.Set("GM1", "GM1")
	c.Set("user", user)
	c.Next()
	fmt.Println("GM1 响应部分")
}

func GM2(c *gin.Context) {
	fmt.Println("GM2 请求部分")
	c.Set("GM2", "GM2")
	c.Next()
	fmt.Println("GM2 响应部分")
}

func main() {
	router := gin.Default()
	g := router.Group("/api")
	g.Use(GM1, GM2)
	g.GET("/users", Home2)
	router.Run(":8080")
}
```


### 优雅关机
在服务端关机命令发出后不是立即关机，而是等待当前还在处理的请求全部处理完毕后再退出程序，是一种对客户端友好的关机方式。而执行`ctrl+C`关闭服务端时，会强制结束进程导致正在访问的请求出现问题。

可以通过`http.Server`内置的`Shutdown()`方法实现优雅关机：
```Go
package main

import (
	"github.com/gin-gonic/gin"
	"golang.org/x/net/context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	r := gin.Default()
	r.GET("/", func(c *gin.Context) {
		time.Sleep(5 * time.Second)
		c.String(http.StatusOK, "Hello Gin")
	})

	srv := &http.Server{
		Addr:    ":8080",
		Handler: r,
	}

	go func() {
		// 开启一个goroutine启动服务
		// 不开启goroutine会导致后续代码无法执行，程序一直在ListenAndServer中循环
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("listen: %s\n", err)
		}
	}()

	// 等待中断信号来优雅地关闭服务器，为关闭服务器操作设置一个5秒的超时
	quit := make(chan os.Signal, 1) // 创建一个接收信号的通道
	// kill 默认会发送syscall.SIGTERM信号
	// kill -2 发送syscall.SIGINT信号，我们常用的Ctrl+C就是触发系统SIGINT信号
	// kill -9 发送syscall.SIGKILL信号，但是不能被捕获，所以不需要添加他

	// signal.Notify把收到的syscall.SIGINT或syscall.SIGTERM信号转发给quit
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM) // 此处不会阻塞
	<-quit                                               // 在此处堵塞，当接收到上述两种信号时才会往下执行
	log.Println("Shutdown Server ...")
	// 创建一个5秒超时的context
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	// 5秒内优雅关闭服务（将未处理完的请求处理完再关闭服务），超过5秒就超时退出
	if err := srv.Shutdown(ctx); err != nil {
		log.Fatal("Server Shutdown:", err)
	}
	log.Println("Server exiting")
}
```

### 使用Air实现gin框架实时重新加载
在修改了项目代码之后，程序能够自动重新加载并执行（live-reload）。
Air支持以下特性：
- 彩色日志输出
- 自定义构建或二进制命令
- 支持忽略子目录
- 启动后支持监听新目录
- 更好的构建过程

#### 安装Air
##### Go
`go get -u github.com/air-verse/air@latest`

##### Windows
`curl -fLo air.exe https://github.com/cosmtrek/air/releases/latest/download/air_win_amd64.exe`

##### Docker
```Dockerfile
# 选择你想要的版本，>= 1.16
FROM golang:1.23-alpine

WORKDIR /app

RUN go install github.com/cosmtrek/air@latest

COPY go.mod go.sum ./
RUN go mod download

CMD ["air", "-c", ".air.toml"]
```

```yaml
version: "3.8"
services:
  web:
    build:
      context: .
      # 修改为你的 Dockerfile 路径
      dockerfile: Dockerfile
    ports:
      - 8080:3000
    # 为了实时重载，将代码目录绑定到 /app 目录是很重要的
    volumes:
      - ./:/app
```


#### 使用Air
为了敲代码方便，可以将`alias air='~/.air'`加到你的`.bashrc`或`.zshrc`中。

首先进入项目目录：
`cd /path/to/your_project`
最简单的用法就是直接执行下面的命令：
```conf
# 首先在当前目录下查找`.air.conf`配置文件，如果找不到就使用默认的
air -c .air.conf
```
推荐的使用方式：
```conf
# 1.在当前目录创建一个新的配置文件.air.conf
touch .air.conf

# 2.复制`air_example.conf`中的内容到这个文件，然后根据需要进行修改

# 3.使用你的配置运行air，如果文件名为`air.conf`，只需执行`air`
air
```

##### air_example.conf
```conf
# 工作目录
# 使用 . 或绝对路径，请注意`tmp_dir`目录必须在`root`目录下
root = "."
tmp_dir = "tmp"

[build]
# 只需要写平常编译使用的shell命令。也可以使用`make`
cmd = "go build -o ./tmp/main.exe"
# 由`cmd`命令得到的二进制文件名
bin = "tmp/main"
# 自定义的二进制，可以添加额外的编译标识例如添加GIN_MODE=release
full_bin = "./tmp/main ./conf/config.yaml"
# 监听以下文件扩展名的文件
include_ext = ["go", "tpl", "tmpl", "html", "yaml"]
# 忽略这些文件扩展名或目录
exclude_ext = ["assets", "tmp", "vendor", "frontend/node_modules", "*.log"]
# 监听以下指定目录的文件
include_dir = []
# 排除以下文件
exclude_file = []
# 如果文件更改过于频繁，则没有必要在每次更改时都触发构建。可以设置触发构建的延迟时间
delay = 1000 # ms
# 发生构建错误时，停止运行旧的二进制文件
stop_on_error = true
# air的日志文件名，该日志文件被放置在“tmp_dir”中
log = "air_errors.log"

[log]
# 显示日志时间
time = true

[color]
# 自定义每个部分现实的颜色。如果找不到颜色，使用原始的应用程序日志
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"

[misc]
# 退出时删除tmp目录
clean_on_exit = true
```