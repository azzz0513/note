### swagger生成接口文档
Swagger本质上是一种用于描述使用JSON表示的RESTful API的接口描述语言。

### gin-swagger实战
想要使用`gin-swagger`为代码自动生成接口文档，一般需要下面三个步骤：
- 按照swagger要求给接口代码添加声明式注释
- 使用swag工具扫描代码自动生成API接口文档数据
- 使用gin-swagger渲染在线接口文档页面

#### 安装
`go get -u github.com/swaggo/cmd/swag`

#### 添加注释
在程序入口main函数上以注释的方式写下项目相关介绍信息。

#### 生成接口文档数据
编写完注释后，在项目根目录执行以下命令，使用swag工具生成接口文档数据。
`swag init`
执行完上述命令后，如果写的注释格式没问题，此时项目根目录下会多出来一个`docs`文件夹。

#### 浏览器访问
在生成完接口文档后，可以运行项目并在`http://ip:host/swagger/index.html`查看接口文档效果

#### 引入gin-swagger渲染文档数据
在项目代码中注册路由的地方按如下方式引入`gin-swagger`相关内容：
```Go
import (
    	_ "web_app/docs" // 记得导入上一步生成的docs

	swaggerFiles "github.com/swaggo/files"
	gs "github.com/swaggo/gin-swagger"
)
```
注册swagger api相关路由
```Go
r.GET("/swagger/*any", gs.WrapHandler(swaggerFiles.Handler))
```

`gin-swagger`同时还提供了`DisablingWrapHandler`函数，方便我们通过设置某些环境变量来禁用Swagger。
```Go
r.GET("/swagger/*any", gs.DisablingWrapHandler(swaggerFiles.Handler, "NAME_OF_ENV_VARIABLE"))
```
此时如果将环境变量`NAME_OF_ENV_VARIABLE`设置为任意值，则`/swagger/*any`将返回404响应，就像未指定路由时一样。

#### 示例
```Go
// LoginHandler 处理登录请求
// @Tags 用户管理
// @Summary 用户登录
// @Description 接收前端数据登录用户账户
// @Param request body models.ParamLogin  true  "登录凭证"
// @Router /api/v1/login [post]
// @Accept json
// @Produce json
// @Success 200 {object} ResponseData "成功响应示例：{"code":1000,"msg":"业务处理成功","data":models.LoginResponse}"
func LoginHandler(c *gin.Context) {
	// 1.获取请求参数和参数校验
	p := new(models.ParamLogin)
	if err := c.ShouldBindJSON(p); err != nil {
		// 请求参数有误，直接返回响应
		zap.L().Error("Login with invalid param", zap.Error(err))
		// 判断errs是不是validator.ValidationErrors类型
		errs, ok := err.(validator.ValidationErrors)
		if !ok {
			ResponseError(c, CodeInvalidParam)
			return
		}
		ResponseErrorWithMsg(c, CodeInvalidParam, removeTopStruct(errs.Translate(trans)))
		return
	}
	// 2.业务处理
	user, err := logic.Login(p)
	if err != nil {
		zap.L().Error("logic.Login failed", zap.Error(err))
		if errors.Is(err, mysql.ErrorUserNotExist) {
			ResponseError(c, CodeUserNotExist)
			return
		}
		ResponseError(c, CodeInvalidPassword)
		return
	}

	// 3.返回响应
	ResponseSuccess(c, &models.LoginResponse{
		UserID:   fmt.Sprintf("%d", user.UserID), // id值1<<53-1，int64类型的最大值为1<<63-1
		UserName: user.Username,
		Token:    user.Token,
	})
}
```

```Go
// UpdateUserHandler 处理修改用户数据
// @Tags 用户管理
// @Summary 用户数据修改
// @Description 接收前端数据修改用户数据
// @Param request body models.UpdateUser  true  "用户数据修改参数"
// @Router /api/v1/change_data [post]
// @Security BearerAuth
// @Accept json
// @Produce json
// @Success 200 {object} ResponseData "成功响应示例：{"code":1000,"msg":"业务处理成功","data":null}"
func UpdateUserHandler(c *gin.Context) {
	// 获取请求参数
	u := new(models.UpdateUser)
	if err := c.ShouldBindJSON(&u); err != nil {
		zap.L().Debug("c.ShouldBindJSON(l) err", zap.Any("err", err))
		zap.L().Error("UpdateUser with invalid param", zap.Error(err))
		ResponseError(c, CodeInvalidParam)
		return
	}
	// 获取用户id
	userID, err := getCurrentUserID(c)
	if err != nil {
		ResponseError(c, CodeNeedLogin)
		return
	}
	u.UserID = userID
	// 修改成员数据的具体逻辑
	if err := logic.UpdateUser(u); err != nil {
		zap.L().Error("logic.UpdateUser failed", zap.Error(err))
		ResponseError(c, CodeServerBusy)
		return
	}
	// 返回响应
	ResponseSuccess(c, nil)
}
```