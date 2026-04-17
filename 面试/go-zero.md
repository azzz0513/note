### Go-zero的负载均衡策略是什么
- **P2C (Power of Two Choices)**。
- **原理**：每次选取节点时，随机选两个节点，然后比较它们的负载情况（基于 lag、cpu 等指标），选择负载较轻的那个。
- **为什么不用轮询或随机？**：为了避免羊群效应，且能更好地适应节点处理能力差异。

### Go-zero的熔断器是怎么实现的
- **Google SRE 弹性熔断算法**（滑动窗口机制）。
- **区别**：传统的 Hystrix 是基于“连续错误数”或“错误率”的阈值熔断。Go-zero 采用自适应算法：  
    ```
    clientRequest=math.Max(0,(requests−k∗accepts)/(requests+1))clientRequest=math.Max(0,(requests−k∗accepts)/(requests+1))
    ```
    - 根据最近一段时间的请求成功率自动计算是否拒绝请求。
    - **好处**：不需要人工配置复杂的阈值，能自动适应流量变化。

### Go-zero如何处理高并发下的限流
- Go-zero 内置了 **TokenLimit（令牌桶）** 和 **PeriodLimit（滑动窗口/固定时间窗口）**。
- 通常使用 Redis + Lua 脚本实现分布式限流。

### go-zero中mapreduce和直接使用`sync.WaitGroup`有什么区别
|特性|sync.WaitGroup|go-zero 的 mr (MapReduce)|
|---|---|---|
|**本质**|基础同步原语，仅用于 “等待一组 goroutine 完成”|基于 goroutine 封装的**并行任务处理框架**，内置 MapReduce 逻辑|
|**核心能力**|仅提供 `Add()`/`Done()`/`Wait()` 三个方法，无任务处理逻辑|封装了 “任务拆分（Map）→ 并行执行 → 结果聚合（Reduce）” 全流程|
|**使用场景**|通用的 goroutine 等待场景（无固定任务模式）|批量任务并行处理 + 结果聚合的场景（如批量查询、数据计算）|
|**底层依赖**|纯原生同步原语，无额外封装|底层基于 `sync.WaitGroup` + 通道（chan）实现，是对 WaitGroup 的上层封装|

示例：
`sync.WaitGroup`：
```Go
package main 
import ( 
	"fmt" 
	"sync" 
) 

func main() { 
	nums := []int{1, 2, 3, 4, 5} 
	var wg sync.WaitGroup 
	// 用于收集结果（需保证并发安全） 
	resultChan := make(chan int, len(nums)) 
	// 1. 启动 goroutine 执行任务（Map 阶段） 
	wg.Add(len(nums)) 
	for _, num := range nums { 
		go func(n int) { 
			defer wg.Done() 
			resultChan <- n * n 
			// 计算平方，发送结果 
		}(num) 
	} 
	
	// 2. 等待所有 goroutine 完成后关闭通道 
	go func() { 
		wg.Wait() 
		close(resultChan) 
	}() 
	
	// 3. 聚合结果（Reduce 阶段） 
	sum := 0 
	for res := range resultChan { 
		sum += res 
	} 
	
	fmt.Println("平方和：", sum) // 输出：55 
}
```

`mr`：
```Go
package main 
import ( 
	"fmt" 
	"github.com/zeromicro/go-zero/core/mr" 
) 

func main() { 
	nums := []int{1, 2, 3, 4, 5} 
	// 调用 mr.MapReduce，传入 Map 和 Reduce 函数 
	sum, err := mr.MapReduce( 
		// Map 函数：拆分任务，并行执行 
		func(source chan<- int) { 
			// 把待处理的数写入 source 通道（任务输入） 
			for _, num := range nums { 
				source <- num 
			} 
		}, 
		// Map 阶段的处理函数：计算平方 
		func(item int, writer mr.Writer[int], cancel func(error)) { 
			writer.Write(item * item) 
			// 输出每个数的平方 
		}, 
		// Reduce 函数：聚合结果 
		func(pipe <-chan int, cancel func(error)) (int, error) { 
			total := 0 
			for res := range pipe { 
				total += res 
			} 
			return total, nil 
		}, 
	) 

	if err != nil { 
		panic(err) 
	} 
	
	fmt.Println("平方和：", sum) // 输出：55 
}
```
只需关注三个核心函数：
- 第一个函数：生产待处理的任务
- 第二个函数：并行处理单个任务
- 第三个函数：聚合所有任务的结果

- **错误处理能力**：
    - `sync.WaitGroup` 无错误处理机制，需要你自己在 goroutine 中捕获错误、传递错误；
    - `mr` 内置 `cancel` 函数，支持任务执行中快速终止所有 goroutine，并返回错误，容错性更强。
- **资源控制**：
    - `sync.WaitGroup` 不限制 goroutine 数量，若批量启动大量 goroutine 可能导致资源耗尽；
    - `mr` 底层默认限制并发数（可通过参数调整），避免 goroutine 爆炸，更适合大规模任务。
- **使用成本**：
    - `sync.WaitGroup` 是 “基础工具”，灵活但需要手写大量模板代码（如 chan 管理、并发安全）；
    - `mr` 是 “成品工具”，封装度高，减少重复代码，专注业务逻辑即可。

### 真并行和假并行
- 真并行：多个 goroutine 同时利用 CPU 多核执行任务，任务执行时间接近 “单个任务耗时 / CPU 核心数”（理想情况）。
- 假并行：看似启动了多个 goroutine 执行任务，但实际上这些 goroutine 因**资源限制**（如 CPU 核心不足、goroutine 数量过多）或**代码写法问题**，变成了 “串行 / 交替执行”，最终总耗时接近 “单个任务耗时 × 任务数”，没有体现出并行的效率提升。

#### `sync.WaitGroup`导致伪并行的核心场景
##### 无限制创建goroutine，引发调度竞争（最常见）
`sync.WaitGroup` 不限制 goroutine 数量，若你为**大量任务**（比如 10 万 +）直接启动等量 goroutine，会导致：
1. Go 运行时的调度器需要频繁切换 goroutine（上下文切换开销）；
2. CPU 核心数有限（比如 8 核），绝大多数 goroutine 处于 “等待调度” 状态；
3. 最终任务执行的总耗时，远高于 “理想并行耗时”，甚至接近串行，看似并行实则伪并行。

##### 任务本身是 CPU 密集型，且 goroutine 数远超 CPU 核心
如果任务是纯 CPU 计算（无 IO 等待），而你通过 WaitGroup 启动的 goroutine 数远大于 CPU 核心数，此时：
- CPU 核心被占满，多余的 goroutine 只能排队等待；
- 调度器频繁切换 goroutine，反而增加开销，最终总耗时接近 “串行耗时”，体现不出并行优势。

##### 错误的同步逻辑，导致 goroutine 串行执行
比如在 goroutine 中使用了全局互斥锁（`sync.Mutex`），且锁的粒度覆盖了整个任务，此时即使通过 WaitGroup 启动多个 goroutine，也会因 “锁竞争” 变成串行执行，完全是伪并行

#### go-zero的mr如何避免伪并行
go-zero 的 `mr` 底层基于 `WaitGroup`，但做了**并发数限制**（核心优化），从根源避免伪并行：
1. `mr` 会默认限制并发 goroutine 数（通常等于 CPU 核心数），或允许你手动指定（比如 `WithWorkers(n)`）；
2. 只启动 “与 CPU 核心匹配” 的 goroutine，避免调度竞争和上下文切换开销；
3. 让任务真正跑在 CPU 多核上，实现**真并行**


- **`MapReduce`**：**任务并行 + 结果收集 + 聚合**（有返回值）
- **`FinishVoid`**：**纯并行执行任务，不需要返回值**（无返回值）
- **`Finish`**：**并行执行任务 + 收集结果**（有返回值）

## grpc
### 为什么有HTTP还要有RPC（Remote Procedure Call，远程过程调用）
RPC就是想像调用本地方法一样调用远端方法，大部分RPC底层使用的都是TCP协议，但是改用UDP和HTTP也是可以的。

- **HTTP (面向资源)**：它的设计哲学是 REST，通过 URL 定位资源，用 GET、POST 等方法对资源进行操作（增删改查）。它非常适合构建公开的 API，让浏览器、App 或第三方系统访问你的服务。
- **RPC (面向动作)**：它的目标是让调用远程服务像调用本地函数一样简单。你只需要关心函数名和参数，完全不用管底层的网络通信细节（如 URL 拼接、请求头设置等）。它更适合处理复杂的业务逻辑调用。


==**服务发现**==：要向某个服务器发起请求，得先建立连接，而建立连接的前提是要直到ip地址和端口。这个找到服务对应的IP端口的过程，就是服务发现。在HTTP中，只要知道服务的域名，就可以通过DNS服务去解析得到IP地址，默认80端口。而RPC就有区别，一般会有专门的中间服务去保存服务名和IP信息（如etcd、consul，甚至Redis），想要访问某个服务，就去这些中间服务去获得IP和端口信息


传输的内容：基于TCP传输的二进制流信息，无非就是`消息头（header）+消息体（body）`
- header适用于标记一些特殊信息，其中最重要的就是消息体长度
- body则是放真正需要传输的内容，而这些内容只能是二进制01串

序列化：将结构体转换为二进制数组的过程就叫做**序列化**，将二进制数组复原为结构体的过程就叫做**反序列化**

gRPC通常使用protobuf或其他序列化协议去保存结构体数据

RPC在内部通信中取代HTTP的最主要原因：

|对比维度|HTTP (通常指 HTTP/1.1 + JSON)|RPC (以 gRPC + Protobuf 为例)|
|:--|:--|:--|
|数据格式|文本格式 (如 JSON)：人类可读，方便调试，但数据冗余多，体积大，解析慢。|二进制格式 (如 Protobuf)：机器可读，数据紧凑，体积小，序列化和反序列化速度极快。|
|传输开销|每次请求都带有冗长的 HTTP 头（Header），在高频调用下是巨大的浪费。|协议头非常精简，专注于传输有效数据，网络开销小。|
|连接管理|HTTP/1.1 虽有长连接，但在高并发下仍有性能瓶颈。|RPC 框架通常内置高效的连接池，能更好地复用连接，显著提升吞吐量。|
|通信模式|主要是“一问一答”的请求-响应模式。|支持更丰富的模式，如双向流式通信，客户端和服务器可以同时、持续地发送数据，非常适合实时推送、直播等场景。|
- **开发体验**：使用 RPC 框架，开发者通过接口定义语言（IDL）定义好服务接口后，可以**自动生成**客户端和服务端的代码。调用远程服务就像调用本地方法一样，类型安全，极大提升了开发效率和代码可维护性。而使用 HTTP，开发者需要手动处理 URL、参数、请求头等，更容易出错。
- **服务治理**：成熟的 RPC 框架（如 Dubbo, gRPC）不仅仅是通信协议，更是一套完整的服务治理解决方案。它们通常**内置**了服务发现、负载均衡、熔断降级、链路追踪等企业级功能。而 HTTP 要实现这些能力，通常需要依赖 Nginx、服务注册中心（如 Nacos）等大量外部组件，架构更复杂。

在现代微服务架构中，一个非常流行且推荐的模式是**混合使用 HTTP 和 RPC**，做到“内外有别”。
1. **对外暴露 HTTP 接口**  
    通过 **API 网关**向外部（如前端 App、浏览器、第三方合作伙伴）提供标准的、基于 HTTP 的 RESTful API。这保证了接口的通用性、安全性和易用性，任何系统都能轻松接入。
2. **内部使用 RPC 通信**  
    在微服务集群**内部**，各个服务之间采用高性能的 RPC 框架（如 gRPC, Dubbo）进行通信。这保证了内部调用的高效率、低延迟和强类型约束。

### gRPC是什么
gRPC是一个高性能RPC框架，可以让你像调用本地函数一样调用远程服务

例：
```Go
user, err := client.GetUser(ctx, &GetUserRequest{Id: 1})
```
实际发生的是：
```
你的进程 → 网络 → 远程服务器 → 执行 → 返回结果
```

#### 核心组成
##### Protocol Buffers（数据协议）
Protocol Buffers是gRPC默认使用的序列化协议

##### HTTP/2（传输层）
HTTP/2是gRPC的底层协议

为什么用HTTP/2：
因为其支持
- 多路复用（关键）：
```
一个 TCP 连接
同时跑多个请求
```
- 双向流：
```
client ↔ server 可以同时发数据
```
- Header压缩：减少网络开销

##### RPC框架本身
负责：
- 请求编码/解码
- 连接管理
- 超时/重试
- 负载均衡

#### gRPC调用流程（完整链路）
1. client调用：`调用stub方法`
2. 序列化：`protobuf → 二进制`
3. 发送（HTTP/2）：`封装成 HTTP/2 frame`
4. server接收：`解码 → 调用 handler`
5. 执行业务逻辑：`你的 Go 函数`
6. 返回结果：`序列化 → 网络 → client`

#### 四种通信模式
##### Unary（最常用）
一问一答
```proto
rpc GetUser(Request) returns (Response)
```

##### Server Streaming
```
client 请求一次
server 返回多个
```

##### Client Streaming
```
client 发多个
server 返回一个
```

##### Bidirectional Streaming（最强）
双方可以不断发数据
```
WebSocket，但更规范
```