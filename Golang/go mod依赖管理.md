#### go.mod文件
```go
// 指定module名
module gomod
// 指定go sdk版本
go 1.23.4
// 当前module（项目）依赖的包，通过require指定
require (
    github.com/bytedance/sonic v1.8.0 // indirect
)
// 排除第三方包，在确定第三方依赖某个版本存在bug的情况下，可以采用排除的方法
exclude (
//  dependency latest
)
// 修改依赖包的路径或者版本
// 依赖包发生偏移
// 原始包无法访问，需要使用代理时
// 使用本地包替换原始包
replace (
//  source latest => target latest
)
// 撤回
// 当前项目作为其他项目的依赖，如果某个版本出现问题则撤回该版本
retract (
    v1.0.0
    v1.0.2
)
```

#### go mod命令
- 将模块下载到本地缓存，需要指定模块路径和版本号（并不会添加依赖，仅仅只是下载当前依赖包）：`go mod download`
例：`go mod download github.com/gin-gonic/gin@v1.9.0`
- 初始化一个新的模块到当前目录：`go mod init`
例：`go mod init gomod`
- 依赖对齐：添加缺少的依赖，删除未使用的依赖：`go mod tidy`
- 通过工具或脚本编辑go.mod：`go mod edit`
例：
添加依赖项
`go mod edit -require="github.com/gin-gonic/gin@v1.9.0"`
替换路径：`old[@version]替换成new[@version]`
`go mod edit -replace="golang.org/x/crypto@v0.0.0=github.com/golang/crypto@latest"`
排除第三方依赖的某个版本
`go mod edit -exclude="github.com/gin-gonic/gin@v1.9.0"`
当前项目作为其他项目的依赖时，添加撤回版本用于排除有问题的版本
`go mod edit -retract="v1.0.0"`
`go mod edit -retract="v1.1.0"`
删除撤回版本记录
`go mod edit -dropretract="v1.0.0"`
- 根据go.mod中的依赖项制作vendor副本，有了vendor副本，项目将不再依赖本地缓存（会优先检查vendor目录中是否有依赖）：`go mod vendor`
- 检查依赖是否正确：`go mod verify`
- 返回对指定模块的依赖关系的最短路径，解释为什么依赖指定包：`go mod why`

#### go install/get/clean
- 安装可执行插件：`go install`
例：`go install github.com/google/gops@latest`
- 获取模块信息并更新go.mod文件（若本地缓存没有该模块，则下载模块，如果有则直接引用）：`go get`
例：`go get github.com/gin-gonic/gin@v1.9.0`
- 更新模块依赖，并更新go.mod：`go get github.com/gin-gonic/gin@v1.9.0`
- 清理临时目录中的文件：`go clean`
例：清理整个module下载的缓存文件：`go clean -modcache`