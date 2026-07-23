## ReAct
举个例子：
![[Pasted image 20260716222359.png]]
每一轮都是三步：模型先解释自己为什么要做这一步（Think），然后选一个工具去执行（Act），看到执行结果后（Observe），再决定下一步怎么走。

- Think就是assistant消息里的`text`内容
- Act就是`tool_use`
- Observe就是`tool_result`

### ReAct与其他范式的对比：

| 范式                | 核心思想         | 优点    | 局限               |
| ----------------- | ------------ | ----- | ---------------- |
| Chain-of-Thought  | 只推理，不行动      | 推理质量高 | 无法与环境交互          |
| Act-only          | 只行动，不推理      | 执行快   | 盲目调工具，容易出错       |
| ReAct             | 推理与行动交替      | 想清楚再做 | 每轮都要一次LLM调用，成本较高 |
| Plan-then-Execute | 先出完整计划，再逐步执行 | 全局规划好 | 计划可能过时，不如边走边看灵活  |

## Agent Loop的核心
就是一个while循环
![[Pasted image 20260716223016.png]]
一个while循环加几次API调用，这就是所有Coding Agent的核心

## while循环什么时候停下来
四种停止条件，缺一不可：
1. 模型主动说"我做完了"。Claude API返回的`stop_reason`如果是`end_turn`，并且响应里没有任何`tool_use`，就表示模型认为任务已经完成。这是最理想的停止范式，模型自然地收尾
2. 迭代上限。设一个最大循环次数，比如50次。超过之后强制停止，给用户一个提示"Agent已经执行了50步但仍未完成，已自动停止"。这是安全网。正常的编码任务很少需要超过50次工具调用，如果超了，大概率是模型陷入了某种无意义的循环
3. 用户取消。用户按Esc主动中断当前循环。注意这里是中断循环，程序本身不退出，用户还可以继续输入新问题。Ctrl+C才是真正退出整个程序
4. 异常状态检测。如果模型请求调用的工具不存在，比如工具名拼错了，或者那个工具被禁用了，返回一个错误结果让模型自己调整。如果连续好几次都请求不存在的工具，说明模型已经迷失了，可以提前终止

实现需要支持取消信号传播：用户在UI层触发取消，信号传递到Agent Loop，Loop在下一轮循环开始前检测到取消信号，干净退出。Go语言可以通过`context.Context`来实现

关键原则：每一轮循环开始前检查取消信号，如果被取消了就干净退出，释放所有资源

## AgentEvent流：让UI实时看到Agent在干什么
Agent Loop产生的事件类型有这些：

| 事件类型          | 含义          | 携带的数据           |
| ------------- | ----------- | --------------- |
| stream_text   | 模型正在输出的文字增量 | 一小段文本           |
| tool_use      | 模型请求调用工具    | 工具名、工具输入、请求ID   |
| tool_result   | 工具执行完成      | 执行结果、是否出错、耗时    |
| turn_complete | 一轮LLM调用完成   | 当前轮次序号          |
| loop_complete | 整个循环结束      | 总轮次             |
| usage         | Token用量更新   | 累计输入/输出 token 数 |
| error         | 发生错误        | 错误信息            |
和之前提到的`content_block_delta`、`messgae_start`那些不是同一个东西，之前提到的是API返回的原始SSE事件，属于LLM API层面的协议。而上面表格这些是Agent自己定义的AgentEvent，是更高层的抽象。Agent内部消费原始SSE事件，处理后转成AgentEvent给UI用。

UI层只需要从事件流里消费事件，根据事件类型更新界面就行了。收到`stream_text`，把文字追加到输出区域。收到`tool_use`，显示一个"正在执行ReadFile..."的提示。收到`tool_result`，把工具结果折叠暂时。收到`loop_complete`，整个交互结束。

Agent和UI完全解耦。Agent不知道UI长什么样，UI不知道Agent内部跑了多少轮循环。甚至可以把UI层整个换掉，换成一个Web界面或者一个纯JSON输出，Agent那边一行代码都不用改。

Go语言可以通过Channel来实现整个功能。

## 状态机思维：每轮循环只有两条路
Agent Loop的每一轮其实可以用一个非常简单的状态机来理解。模型每次响应后，只有两种可能：继续循环/终止循环
![[Pasted image 20260719145705.png]]
后续加入需要`NEED_CONFIRM`，遇到破坏性操作需要用户确认才能继续。或者`RATE_LIMITED`，被API限流时需要暂停一会再重试。如果一开始就用状态机的思维来写，后续加新状态就是加一个分支的事。

## 工具执行的分批逻辑
前面提到工具可以串行执行，但模型可能一次返回多个工具调用，比如同时ReadFile三个不同文件。串行跑就得等三次磁盘IO，完全没必要

可以按每个工具的`isConcurrencySafe`声明做分批：
- 安全的并发执行
- 不安全的串行执行

`partitionToolCalls`就负责把工具调用列表扫一遍做分区：
![[Pasted image 20260719150303.png]]
假如模型返回`[Read, Read, Edit, Read, Read]`，会被分成三批：`[Read, Read]`并发 → `[Edit]`串行 → `[Read, Read]`并发。每一批串行批只包含一个不安全调用，并发批可以包含多个安全调用

并发执行`runConcurrently`的实现很简单，每个工具调用起一个Goroutine，同时执行，等全部完成。为了防止无限并发拖垮系统，可以加一个并发上限。串行执行`runSerially`就是逐个跑。

## System Prompt与环境信息
Agent Loop每轮都需要把System Prompt传给Claude。

环境信息很容易被忽略，但是非常重要。如果模型不知道当前工作目录在哪里，它执行命令的时候就不知道该用绝对路径还是相对路径。如果不知道操作系统是什么，就可能在Linux上给里写Windows的命令

## Plan Mode
实现核心：通过Prompt指令约束模型行为。系统注入一段强指令，告诉模型当前是规划模式：
![[Pasted image 20260719171321.png]]


# 实现
## Agent结构体
![[Pasted image 20260719150949.png]]
- `Client`负责和LLM通信
- `Registry`提供工具
- `Protocol`决定工具描述的格式
- `Checker`权限系统
- `Hooks`钩子
- `onLoopComplete`记忆系统

## Agent Event：Agent和UI的通信协议
所有事件类型都实现了一个空的`AgentEvent`接口
```Go
type AgentEvent interface{ agentEvent() }

type StreamText struct{ Text string }
type ThinkingText struct{ Text string }
type ToolUseEvent struct {
    ToolID   string
    ToolName string
    Args     map[string]any
}
type ToolResultEvent struct {
    ToolID   string
    ToolName string
    Output   string
    IsError  bool
    Elapsed  time.Duration
}
type TurnComplete struct{ Turn int }
type LoopComplete struct{ TotalTurns int }
type UsageEvent struct{ InputTokens, OutputTokens int }
type ErrorEvent struct{ Message string }
type CompactEvent struct{ Message string }
type RetryEvent struct {
    Reason string
    Wait   time.Duration
}
```

权限请求事件`PermissionRequestEvent`比较特殊，它带了一个`ResponseCh chan<- PermissionResponse`：
```Go
type PermissionRequestEvent struct {
    ToolName   string
    Desc       string
    ResponseCh chan<- PermissionResponse
}
```
Agent发出权限请求后阻塞在channel上等待，UI收到事件后弹窗让用户选择允许还是拒绝，选完把结果写回channel。这是Agent和UI之间唯一的反向通信通道。Agent不知道UI长什么样，UI不知道Agent内部跑了多少循环，两边完全通过事件流解耦。

## 主循环
### 入口：`Run()`
![[Pasted image 20260719151857.png]]
`Run()`不会阻塞调用方。它创建一个容量32的buffered channel，启动一个goroutine在后台跑循环，事件通过channel推给UI，立即返回channel引用。UI那边用`for ev := range ch`消费事件，channel关闭就知道Agent结束了。

缓冲区大小32的用意：Agent可以连续发32个事件而不用等UI消费。这让Agent的执行节奏不会被UI的渲染速度拖慢。

异常兜底靠`defer close(ch)`。不管goroutine里面发生了什么，chennel最后都会被关闭，UI侧的`for range`不会永远阻塞。

### 循环骨架
![[Pasted image 20260719152221.png]]
准备上下文 → 问LLM → 看LLM回什么 → 有工具就执行 → 没工具就结束

### 调用LLM和消费流式响应
LLM调用只有一行：
```Go
events, errs := a.Client.Stream(ctx, conv, toolSchemas)
```
返回两个channel，一个出流式事件，一个出错误。然后用`for range`消费事件流：
![[Pasted image 20260719152524.png]]
`executor.Submit()`在LLM还在输出后续内容的时候就把已经解析完的工具调用提交给执行器。加入LLM返回三个工具调用，第一个解析完就立刻开始执行，不用等后面两个。工具执行和LLM输出是并行的，延迟更低。

事件分发逻辑：
- `TextDelta`：实时推给UI让用户看到流式输出
- `ToolCallComplete`：收集到列表并立刻提交执行
- `StreamEnd`：记录停止原因和Token用量

### 终止判断
LLM响应处理完之后，整个循环走到一个分岔口：
```Go
if len(toolCalls) == 0 {
	conv.AddAssistantFull(text, thinkingBlocks, nil)
	if a.FileHistory != nil {
		summary := text
		if len(summary) > 60 {
			summary = summary[:60] + "..."
		}
		a.FileHistory.MakeSnapshot(conv.Len(), summary)
	}
	ch <- LoopComplete{TotalTurns: iteration}
	if a.OnLoopComplete != nil {
		go a.OnLoopComplete(conv)
	}
	return
}
```
没有工具调用，说明LLM认为任务已经完成了，循环结束。有工具调用，继续往下走去收集工具执行结果，然后进入下一轮。

### 工具结果收集
工具在流式阶段就已经开始执行了，这里只需要等它们全部跑完：
![[Pasted image 20260719153118.png]]
工具的完整输出推给UI展示，但写进对话历史时会做体积控制。超过`MaxOutputChars`的输出完整写到磁盘文件，对话里只保留2KB预览和文件如今，模型后续可以用ReadFile读取完整内容。

### 四个停止条件
#### LLM不再调用工具
`if len(toolCalls) == 0`判断，最常见的正常退出路径，模型自然收尾。

#### 迭代次数上限
```Go
if a.MaxIterations > 0 && iteration > a.MaxIterations {
	ch <- ErrorEvent{Message: fmt.Sprintf("Agent reached maximum iterations (%d)", a.MaxIterations)}
	return
}
```

#### 连续未知工具
```Go
if consecutiveUnknown >= 3 {
	ch <- ErrorEvent{Message: "Too many consecutive unknown tool calls"}
	return
}
```
**连续**3次调用不存在的工具就终止

#### 用户取消
```Go
if ctx.Err() != nil {
	return
}
```

### 工具执行
#### StreamingExecutor：边流式边执行
每个工具调用一解析完就提交
![[Pasted image 20260719154043.png]]

### 单工具执行流程
`executingSingleTool`是工具执行的完整管线，按顺序走四关：
#### 第一关：查找工具
```Go
    tool := a.Registry.Get(tc.toolName)
    start := time.Now()
    if tool == nil {
        return toolExecResult{
            toolID:    tc.toolID,
            toolName:  tc.toolName,
            output:    fmt.Sprintf("Error: unknown tool '%s'", tc.toolName),
            isError:   true,
            elapsed:   time.Since(start),
            isUnknown: true,
        }
    }
```
在Registry里查找，找不到就返回unknown错误，同时标记`isUnknown`给连续未知工具检测用

#### 第二关：权限检查
![[Pasted image 20260719154724.png]]
- `Deny`：直接拒绝
- `Allow`：放行
- `Ask`：发事件给UI，阻塞在channel上等用户选择。用户选了`Always Allow`的话，还会往权限规则引擎里追加一条规则，后续同类操作不再询问

#### 第三关：Pre-tool Hook
![[Pasted image 20260719155136.png]]
Hook引擎可以拦截工具执行。Pre-tool Hook是整个Hook系统里唯一能"阻断"的事件，其他Hook都是通知性的

#### 第四关：真正执行
![[Pasted image 20260719155244.png]]
拿到结果后出发`post-tool Hook`，计算执行耗时，返回结果。

## 错误恢复与自愈
### 上下文过长恢复
![[Pasted image 20260719155502.png]]
当API返回上下文过长错误时，Agent不会直接崩溃。先做一次Layer 1裁剪，再把裁剪后的消息传给`ForceCompact`触发强制压缩，把对话历史摘要化，然后`continue`重试这一轮LLM调用。强制压缩成功后需要调用`conv.ClearUsageAnchor()`清零锚点，因为压缩改写了对话内容，旧的统计基线不再精确

### 限流等待重试
![[Pasted image 20260719155742.png]]
遇到限流时，解析`Retry-After`头拿到等待时间（默认5秒），发一个RetryEvent通知UI，然后用`select`同时监听定时器和context取消。定时器到期就重试，用户中途取消也能立刻响应，不会一直傻等

### 输出截断恢复（max_tokens）
![[Pasted image 20260719155912.png]]
两阶段恢复策略：
1. 第一次max_tokens截断时，先把`max_output_tokens`提升到64000上限，注入续写指令，重试。
2. 如果提升上限后还是被截断，就进入第二阶段：最多再重试3次，每次告诉LLM把工作拆小。超过3次就放弃，按正常结束处理。

## Plan Mode
不改变循环结构，只在每轮迭代开头注入一段提示词：
![[Pasted image 20260719160210.png]]
1. 告诉权限检查器Plan文件的路径，让写Plan文件成为例外，不被Plan Mode的只读限制拦住
2. 往对话里注入一段system-reminder，告诉LLM现在处于规划模式，只能思考和分析，不能执行写操作