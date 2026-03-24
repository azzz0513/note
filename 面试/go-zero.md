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
