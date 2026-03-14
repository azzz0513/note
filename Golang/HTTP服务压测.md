在项目正式上线之前，我们通常需要通过压测来评估当前系统能够支撑的请求量、排查可能存在的隐藏bug，同时了解了程序的实际处理能力能够帮我们更好的匹配项目的实际需求，节约资源成本。

### 压测相关术语
- 响应时间（RT）：指系统对请求作出响应的时间
- 吞吐量（Throughout）：指系统在单位时间内处理请求的数量
- QPS每秒查询量（Query Per Second）：“每秒查询量”，是一台服务器每秒能够响应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准
- TPS（TransactionPerSecond）：每秒钟系统能够处理的交易或事务的数量
- 并发连接数：某个时刻服务器所接受的请求总数

### 压力测试工具
#### ab
ab全称Apache Bench，是Apache自带的性能测试工具。使用这个工具，只需指定同时连接数、请求数以及URL，即可测试网站或网站程序的性能。

通过ab发送请求模拟多个访问者同时对某一个URL地址进行访问，可以得到每秒传送字节数、每秒处理请求数、每请求处理时间等统计数据。

命令格式：
```
ab [options] [http://]hostname[:post]/path
```
常用参数如下：
```
-m requests 总请求数
-c concurrency 一次产生的请求数，可以理解为并发数
-t timelimit 测试所进行的最大秒数，可以当作请求的超时时间
-p postfile 包含了需要POST的数据的文件
-T content-type POST数据所使用的Content-type头信息
```

例如测试某个GET请求接口：
```
ab -n 10000 -c 100 -t 10 "http://127.0.0.1:8080/api/v1/posts?size=10&page=1"
```
测试POST请求接口：
```
ab -n 10000 -c 100 -t 10 -p post.json -T "appliction/json" "http://127.0.0.1:8080/api/v1/post"
```

#### go-wrk
wrk是一款开源的HTTP性能测试工具，它和上面提到的`ab`同属于HTTP性能测试工具，它比`ab`功能更加强大，可以通过编写lua脚本来支持更加复杂的测试场景。

`go-wrk`是Go语言版本的`wrk`，Windows环境下可以使用它来测试，使用如下命令来安装`go-wrk`：
```
go get github.com/adeven/go-wrk
```
使用方法同`wrk`类似，基本格式如下：
```
go-wrk [flags] url
```
常用的参数：
```
-H="User-Agent: go-wrk 0.1 bechmark\nContent-Type: text/html;"：由'\n'分隔的请求头
-c=100：使用的最大连接数
-k=true：是否禁用keep-alives
-i=false：if TLS security checks are disabled
-m="GET"：HTTP请求方法
-n=1000：请求总数
-t=1：使用的线程数
-b=""：HTTP请求体
-s=""：如果指定，他将计算响应中包含搜索到的字符串s的频率
```