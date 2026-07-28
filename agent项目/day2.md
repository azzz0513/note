## Function Calling
模型是怎么调用工具的？
你可能以为模型能直接访问你的文件系统、执行你的命令。但是其实模型永远只做一件事：生成文本。它不能直接操作任何东西。

那怎么让它动手？答案是一个协作协议，我们叫它Function Calling（也叫Tool Use）。整个流程分四步：
### 告诉模型有哪些工具
在调API的时候，通过`tools`参数告诉模型：`你现在有这些工具可以用`。每个工具都有名字、描述、参数格式。
![[images/Pasted image 20260715213351.png]]

### 模型决定调用工具
当模型认为需要使用某个工具时，它会在回复中输出一个结构化的请求：我想调这个工具，参数是这些。
![[images/Pasted image 20260715213457.png]]
注意两个细节：
- 模型可以在同一条回复里既输出文本又请求调用工具，甚至同时请求调用多个工具
- 每个`tool_use`都有一个唯一的`id`，后面返回结果的时候要用
**模型输出了`tool_use`并不意味着工具已经被执行了。它只是一个建议，或者说一个请求。实际执行与否，由我们的代码决定。**

### 你执行工具，把结果告诉模型
你的代码拿到`tool_use`请求后，在本地执行对应的操作（比如读文件），然后把结果作为`tool_result`发回给模型：
![[images/Pasted image 20260715213822.png]]
`tool_use_id`必须和上面的`id`对应。这样模型才知道这是哪个工具调用的结果。
**`tool_result`是以`user`角色发送的。因为从对话协议的角度，工具执行是用户侧做的事情，结果是你反馈给模型的。模型只负责思考和决策，不负责执行。**

### 模型继续
模型收到工具结果后，有两种可能：
- 它觉得信息够了，直接给出最终回复
- 它觉得还需要更多信息，再请求调用另一个工具

## Function Calling的本质
整个Function Calling的本质可以用一句话概括：**模型负责决策，你负责执行，结果反馈回模型**。模型永远不直接操作你的系统。

## 工具描述是最值得打磨的部分
工具描述的质量直接决定了模型的工具使用行为，包括什么时候调、调哪个、参数怎么传。

因为模型在决定是否使用某个工具时，主要依据就是工具描述。如果描述不清楚，模型要么不知道什么时候该用这个工具，要么用错工具，要么传错参数。
![[images/Pasted image 20260715215623.png]]

一个好的工具描述应该包含这些信息：

| 信息       | 说明      |
| -------- | ------- |
| 做什么      | 工具的核心功能 |
| 什么时候该用   | 典型使用场景  |
| 什么时候不该用  | 反模式提示   |
| 参数约束     | 输入的限制条件 |
| 返回格式     | 输出长什么样  |
| 与其他工具的配合 | 工作流建议   |

## 工具接口设计：为什么不能只有名字和执行
想象一些场景：ReadFile是只读操作，模型随便调，不用征求用户同意。但WriteFile会修改文件，是不是应该先让用户确认一下？Bash可能执行`rm -rf /`，是不是需要更严格的审批？
如果接口里没有`只读`、`破坏性`这样的元信息，你的权限系统怎么判断是否应该放行？

还有：你需要把工具定义发给API（名称、描述、参数Schema），这些信息如果不在接口里，你就得在别的地方维护一份，很容易和实际实现不一致。

所以一个生产级的工具接口应该包含这些能力：
![[images/Pasted image 20260715220301.png]]
每个方法都有明确的职责。
- `name`、`description`、`inputSchema`用来生成API请求里的工具定义
- `isReadOnly`、`isDestructive`用来让权限系统自动判断是否需要用户确认
- `isConcurrencySafe`标记这个工具能否和其他安全工具并发执行。它接收工具的输入参数，返回true表示可以并发，false表示必须串行，默认false，保守优先
- `category`用来做工具分类，比如file、shell、search，方便UI展示和批量管理
- `validateInput`在执行前校验参数，该拒绝的早拒绝，别等到执行一半才报错。校验失败时，把错误信息包装成isError：true的ToolResult返回给模型，让模型能调整参数重试，而不是直接抛出程序异常中断整个Agent Loop

## 执行结果：错误也是有价值的信息
![[images/Pasted image 20260715221314.png]]
这里有一个关键设计：`isError`字段

当工具执行失败时，你把失败信息包装成一个`isError: true`的ToolResult返回给模型。
例：用户说"帮我读一下config.yaml"，模型调了ReadFile，但文件不存在。

如果你把整个当成程序错误处理，Agent循环可能直接中断，给用户弹一个`内部错误`。但如果你把`文件不存在`作为ToolResult返回给模型，模型拿到整个信息之后可能会说"config.yaml 不存在，让我找找看有没有类似的配置文件"，然后调Glob搜索。

工具执行失败时对模型来说是有价值的反馈信息，它会引导模型调整策略。只有真正的系统级错误（如内存不足、程序崩溃）才应该作为程序级error上报。

`metadata`是给UI层用的额外信息，比如文件的修改时间、命令的执行耗时。这些信息不发给模型，避免浪费token，但可以在界面上展示。

## 通用基础实现
如果每个工具都从零实现一遍，代码会很重复。所以可以使用一个基础工具作为通用实现：
伪代码：
![[images/Pasted image 20260715222015.png]]

这样每个工具只需要写一个工厂函数，填入自己的名称、描述、Schema和执行函数就行了：
![[images/Pasted image 20260715222052.png]]


## 工具注册中心（Registry）
它是工具的统一管理入口，把工具的创建和使用解耦。

注册中心的能力很直观：注册工具、按名称启用或禁用、获取单个或所有启用的工具。最关键的是一个`toAPIFormat`方法，它遍历所有启用的工具，把每个工具的名称、描述、参数Schema组装成Claude API要求的格式。每次调API前调用它，告诉模型当前可用的工具列表。

注册中心支持条件启用，你可以根据配置灵活控制：
![[images/Pasted image 20260715221124.png]]

## 六个关键工具的设计全景
一个典型的工作流是：Grep搜索关键词 → 发现目标文件 → ReadFile读取完整内容 → EditFile修改 → Bash编译测试

| 工具        | 分类     | 只读  | 破坏性 | 典型场景              |
| --------- | ------ | --- | --- | ----------------- |
| ReadFile  | file   | 是   | 否   | 参看文件内容、读取配置       |
| WriteFile | file   | 否   | 否   | 创建新文件、覆盖写入        |
| EditFile  | file   | 否   | 否   | 精确修改文件某几行，节省token |
| Bash      | shell  | 否   | 是   | 编译、测试、安装依赖、执行命令   |
| Glob      | search | 是   | 否   | 了解项目结构、查找特定类型文件   |
| Grep      | search | 是   | 否   | 搜索代码中的函数定义、变量引用   |

## 集成到LLM客户端：处理流式tool_use
### 请求侧
每次调API时，从注册中心拿到当前启用的工具列表，转成工具定义放进请求参数

### 响应侧：内容类型扩展
之前我们处理的内容块只有`text`类型，现在要支持`tool_use`和`tool_result`。
内容块需要扩展的字段：

| 内容类型        | 新增字段        | 说明             |
| ----------- | ----------- | -------------- |
| tool_use    | id          | 工具调用的唯一标识      |
|             | name        | 工具名称           |
|             | input       | 调用参数（JSON）     |
| tool_result | tool_use_id | 对应的tool_use id |
|             | content     | 执行结果文本         |
|             | is_error    | 是否为错误结果        |
流式事件也要扩展，新增一种`工具调用`事件类型，携带id、name、input三个字段

## 流式tool_use解析：拼JSON碎片
在流式响应中，文本内容是一段一段的，你直接追加就行。但tool_use的输入参数也是一段一段到的，而且是JSON碎片。我们需要将碎片拼起来，最后解析成完整的JSON。

流式事件的顺序是这样的：
![[images/Pasted image 20260716212424.png]]
`content_block_start`告诉我们一个新的tool_use块开始了，给你id和name。然后一系列的`content_block_delta`给我们JSON的碎片。最后`content_block_stop`告诉这个块结束了。

处理逻辑：
1. 收到`content_block_start`且type为`tool_use`时，记下id和name，初始化一个字符串缓冲区。
2. 后续每个`content_block_delta`到达，把`partial_json`追加到缓冲区。
3. 最后`content_block_stop`时，把缓冲区里的完整JSON解析出来，发送一个ToolUse事件。

## 消息管道的变化
工具调用引入了一种新的消息模式。之前的对话是简单的user → assistant → user → assistant交替。现在变成了：
![[images/Pasted image 20260716212857.png]]
- `tool_result`是以user角色发送的。
- 一条assistant消息可能包含text和tool_use两种内容块。
- 如果模型在一次回复中请求了多个工具调用（比如ReadFile和Grep），所有tool_use块在同一条assistant消息里，所有tool_result在同一条user消息里，通过id配对

# 实现
## 核心类型
### Tool接口
所有工具都实现同一个接口：
```Go
type Tool interface {
    Name() string
    Description() string
    Category() ToolCategory
    Schema() map[string]any
    Execute(ctx context.Context, args map[string]any) ToolResult
}
```
- `Name`和`Description`告诉LLM这个工具是什么、怎么用
- `Schema`定义参数格式让LLM知道该传什么
- `Category`标记工具的读写属性，给权限系统用
- `Execute`是真正干活的地方

### ToolResult
```Go
type ToolResult struct {
    Output  string // 执行结果或错误信息
    IsError bool // 标记是否出错
}
```
`IsError`不是让程序panic的错误，而是告诉LLM"这次工具调用没成功"，LLM收到后可以换个思路再试。

### ToolCategory
```Go
type ToolCategory string

const (
    CategoryRead    ToolCategory = "read" // 只读，不改文件
    CategoryWrite   ToolCategory = "write" // 写操作
    CategoryCommand ToolCategory = "command" // 执行命令
)
```
把工具分成三类。权限系统会根据这个分类做不同的检查策略：read类工具通常默认放行，write和command类工具需要用户授权

### Registry 注册中心
```Go
type Registry struct {
    tools          map[string]Tool // 按名称存储所有工具
    discoveredTools map[string]bool // 记录哪些延迟工具已被发现
}
```
所有工具注册到这里，Agent Loop从这里获取工具列表和Schema，执行时也从这里查找工具。
`Register()`就一行：
```Go
func (r *Registry) Register(t Tool) {
    r.tools[t.Name()] = t
}
```
查找使用`Get(name)`，返回对应的Tool实例

`GetAllSchemas(protocol)`负责把所有工具的Schema收集起来，发给LLM API。它做了两件事：过滤掉未被发现的延迟工具（省token），以及根据protocol参数适配Anthropic/OpenAI两种API格式

### 主流程
#### 第一步：注册
```Go
func CreateDefaultToolsWithWorkDir(workDir string) DefaultTools {
    fsc := NewFileStateCache()
    wf := &WriteFileTool{FileStateCache: fsc}
    ef := &EditFileTool{FileStateCache: fsc}
    reg := NewRegistry()
    reg.Register(&ReadFileTool{FileStateCache: fsc})
    reg.Register(wf)
    reg.Register(ef)
    reg.Register(&BashTool{WorkDir: workDir})
    reg.Register(&GlobTool{})
    reg.Register(&GrepTool{})
    return DefaultTools{Registry: reg, WriteFile: wf, EditFile: ef}
}
```
是整个工具系统的启动入口

#### 第二步：生成Schema
以EditFile为例，它的`Schema()`返回的就是一个标准的JSON Schema：
```Go
func (t *EditFileTool) Schema() map[string]any {
    return map[string]any{
        "name":        t.Name(),
        "description": t.Description(),
        "input_schema": map[string]any{
            "type": "object",
            "properties": map[string]any{
                "file_path":  map[string]any{"type": "string", "description": "Path to the file to edit"},
                "old_string": map[string]any{"type": "string", "description": "The exact string to find and replace (must be unique in file)"},
                "new_string": map[string]any{"type": "string", "description": "The replacement string"},
            },
            "required": []string{"file_path", "old_string", "new_string"},
        },
    }
}
```
- `properties`定义每个参数的类型和含义
- `required`标记哪些是必填的
LLM会根据这些信息生成符合格式的调用参数

#### 第三步：执行
当LLM返回`tool_use`时，Agent Loop通过`Registry.Get(name)`查找工具，然后调用`executeSingleTool()`：
```Go
func (a *Agent) executeSingleTool(ctx context.Context, eventCh chan AgentEvent, tc toolCallInfo) toolExecResult {
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
    // ...权限检查...
```
找不到工具不会崩溃，而是返回错误让LLM自行调整。通过权限检查后再真正执行`tool.Execute`，执行结果包装成`toolExecResult`返回。
最终Agent把结果写回对话你是：`conv.AddToolResultsMessage(toolResults)`，每个结果通过`ToolUseID`和对应的`tool_use`配对。错误结果也发回模型，不抛异常。

### ToolSearch与延迟加载
```Go
type DeferrableTool interface {
    ShouldDefer() bool
}
```
如果一个工具实现了这个接口并返回`true`，它就不会出现再默认的Schema列表里。LLM需要通过ToolSearch按名称或关键词搜索才能拿去它的完整定义。

搜索有两种模型：`select:Name1,Name2`按名称精确查找，普通关键词按名称和描述模糊匹配，搜到之后调用`Registry.MarkDiscovered(name)`，后续的`GetAllSchemas()`就会把这个工具的Schema带上了

### 流式tool_use解析
流式响应中，`tool_use`的JSON参数不是一次性到达的，而是被拆成多个`input_json_delta`分片。代码用三个事件类型组成了一条解析管道：
```Go
type ToolCallStart struct{ ToolName, ToolID string }
type ToolCallDelta struct{ Text string } // JSON碎片
type ToolCallComplete struct {
    ToolID    string
    ToolName  string
    Arguments map[string]any // 解析后的完整参数
}
```

在Anthropic的流式处理循环中，`content_block_start`出现`tool_use`类型时，记下工具名称和ID，同时初始化一个空字符串`jsonAccum`开始累积。每收到一个`InputJSONDelta`，就把碎片拼接上去。等到`content_block_stop`事件到达，对完整的JSON字符串做一次性`json.Unmarshal`
