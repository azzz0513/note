## MCP
MCP（Model Context Protocol）是一个开放协议，定义了AI应用如何和外部能力进行标准化通信。

在MCP之前，每个Agent想接入每个工具，都需要写专门的对接代码：
![[Pasted image 20260721211653.png]]'

有了MCP之后，Agent只需要实现一次MCP客户端，工具只需要实现一次MCP服务端，双方就可以通信：
![[Pasted image 20260721211736.png]]

所以我们只需要实现MCP Client。

## MCP由哪些部分构成
MCP至少需要从两个角度来看：运行时角色和协议层次。

运行时角色：
- Host：真正的AI应用本体。比如Claude Desktop、Code、IDE插件，或者我们正在构建的MewCode。
- Client：Host里的一个连接组件。它负责和某一个MCP Server建立连接、发送请求、接收响应。
- Server：对外暴露能力的程序。它可能运行在本地，也可能运行在远程。
**MewCode本身是Host，不是Client**，MewCode这个Host内部会创建一个或多个MCP Client，每个Client对应一个Server。
![[Pasted image 20260721212139.png]]

协议层次：
- Data Layer（数据层）：定义消息长什么样、初始化怎么握手、有哪些能力、怎么列工具、怎么调工具。
- Transport Layer（传输层）：定义消息怎么在两边之间传过去，是走`stdio`，还是走`Streamable HTTP`，属于这一层。
可以想象成 信件内容 和 送信方式 的区别。Data Layer规定信里写什么、格式是什么；Transport Layer决定是快递送，还是自己跑腿送。内容不变，传输方式可以换。

## MCP双方各提供什么
MCP最常被提到的是Server暴露的三种核心原语（primitives），也就是前面说的Tools、Resources、Prompts。

**Tools**：一个MCP Server可以暴露一组工具，每个工具有名称、描述和参数的JSON Schema定义。比如一个Github MCP Server可能提供`search_issues`、`create_issue`、`list_pull_requests`这些工具。
![[Pasted image 20260721214434.png]]

**Sources**：可以理解为可读取的数据源。比如一个数据库MCP Server可以暴露表结构作为Resource，Agent读取它就能了解有哪些表、哪些字段，不用自己去猜：
![[Pasted image 20260721214543.png]]
Agent 通过`resources/read`请求这个URL，Server就会返回对应的数据。这有点像RAG里的上下文源，用来给Agent补充背景信息。

**Prompts**：是MCP Server提供的预定义提示词模板。比如一个SQL MCP Server可以提供一个引导Agent按正确格式生成查询的Prompt：
![[Pasted image 20260721214754.png]]
Agent调用这个Prompt时传入参数，Server返回一段组装好的提示词文本，Agent拿着这段文本去生成SQL。

再看Client：
Client也可以声明自己支持什么能力：
- Roots：告诉Server当前项目的根目录或工作区目录
- Sampling：允许Server反过来请求Host帮它调用LLM
- Elicitation：允许Server请求Host向用户追问额外信息

## 传输层：stdio和Streamable HTTP
这两个标准传输的区别在于Host和Server进程之间怎么通信：
- stdio：Host把MCP Server作为子进程启动，通过`stdin/stdout`管道读写消息。Server本身可以访问任何远程服务。比如Github API、数据库、云平台，管道只管Host和Server之间那一段通信。
- Streamable HTTP：MCP Server是一个独立运行的HTTP服务，Host用HTTP POST / GET和它通信，必要时用SSE做流式消息。Server可能在本机，也可能在远端。

### stdio传输：没有网络，没有端口
MewCode启动一个子进程来运行MCP Server，然后通过这个子进程的stdin和stdout管道来通信。MewCode往子进程的stdin写请求，从子进程的stdout读响应。
![[Pasted image 20260721220426.png]]
传统的RPC是怎么做的。服务端得监听一个端口，比如`localhost:8080`，客户端连接过去。这会导致：端口可能被占用，得处理冲突。防火墙可能阻拦本地连接，得配置规则。如果同时跑多个MCP Server，每个都要占一个端口，还得管理端口分配。进程挂了端口可能不会立即释放，下次启动会报"address already in use"。

stdio传输把这些问题全部消除了。MCP Server就是一个普通的命令行程序，MewCode启动它，通过操作系统的管道通信。不需要额外的网络配置，不需要担心端口占用，不需要服务发现机制。进程启动即可用，退出即清理。进程的生命周期管理是操作系统最擅长的事情。

MCP Server的stderr不参与协议通信，它被用来输出日志和调试信息。开发者调试MCP Server的时候可以随便往stderr打日志，不会干扰协议的正常通信。

根据官方规范，stdio里的消息时UTF-8编码的JSON-RPC消息，通常以换行分隔。Server的stdout上不能混入任何非协议内容，否则Client就会解析失败。这也是为什么日志一定要走stderr，而不是stdout。

### Streamable HTTP传输：远程Server怎么接
stdio靠子进程的stdin/stdout管道收发消息，Streamable HTTP靠HTTP请求收发消息。

Client把JSON-RPC消息通过HTTP `POST`发给Server的固定端点。Server处理完后，有两种回复方式：如果结果已经准备好了，直接返回`application/json`响应；如果需要流式推送（比如长时间运行的工具），可以返回`text/event-stream`，用SSE逐步发送结果。

所以Client发请求的时候，`Accept`头必须同时声明这两种：
![[Pasted image 20260723215904.png]]

另一个区别是认证。stdio的Server是本地子进程，天然信任，不需要额外认证。远程Server通常需要API Key或者OAuth Token。所以HTTP transport要支持自定义请求头，让用户在配置里声明认证信息。

![[Pasted image 20260723220036.png]]

Manager看到`command`就创建StdioTransport，看到`url`就创建HTTPTransport。后面的MCP Client流程完全一样。

**transport只负责消息的收发，上层的initialize、tools/list、tools/call完全不关心底层用的是管道还是HTTP**

## JSON-RPC 2.0：消息格式
不管哪种传输方式，MCP的消息格式统一使用JSON-RPC 2.0。

JSON-RPC 2.0 只有三种消息类型。

**请求（Request）**：有id，有method，有params。Client发给Server，期望得到响应。
![[Pasted image 20260723220408.png]]

**响应（Response）**：有id（和请求对应），有result或error。Server发给Client。
![[Pasted image 20260723220452.png]]

**通知（Notification）**：有method，但没有id。通知不需要响应，发出去就完了。
![[Pasted image 20260723220536.png]]

通知和请求的区别就看有没有id字段。有id就是请求（期望响应），没id就是通知（不期望响应）。

在代码中，我们这样定义JSON-RPC消息结构：
![[Pasted image 20260723220753.png]]

## 一次完整的MCP会话长什么样
### 第一阶段：初始化握手
MewCode（Client）启动MCP Server子进程后，第一件事是发送`initialize`请求，声明自己的身份和能力。Server回应自己的身份和能力。
![[Pasted image 20260723220908.png]]

Client 声明自己的协议版本、支持的能力和身份信息。Server收到后回应自己的身份和能力：
![[Pasted image 20260723220940.png]]
- `capabilities`：告诉Client这个Server支持哪些能力

握手成功后，Client还要发一个`notifications/initialized`通知，告诉Server"我准备好了，可以开始工作了"。这是一个通知（没有id），不需要等待响应。

### 第二阶段：工具发现
Client 发送`tools/list`请求，获取Server提供的所有工具定义
![[Pasted image 20260723221245.png]]

请求很简单，不需要额外参数。Server返回它提供的所有工具定义：
![[Pasted image 20260723221401.png]]
拿到工具列表后，MewCode就知道这个Server有哪些工具可用了。这些工具定义会被包装成MewCode内部的Tool接口，注册到ToolRegistry里。Agent在下一轮对话中就能看到它们。

### 第三阶段：工具调用
当Agent决定使用某个MCP工具时，Client发送`tools/call`请求：
![[Pasted image 20260723221541.png]]

Server执行完工具后返回结果：
![[Pasted image 20260723221602.png]]

返回的content是一个数组，每个元素是一个内容块，可以是文本、图片等。对于大多数工具来说，就是一段文本。

整个流程就是：initialize → notifications/initialized → tools/list → （tools/call）× N
前两步只做一次，后面的工具调用可以重复多次。

## 请求-响应的异步匹配
在stdio传输中，Client和Server通过同一对管道通信。如果Client连续发了两个请求（id = 1和id = 2），Server可能先回复id = 2再回复id = 1。Client就是靠id知道哪个响应对应哪个请求。

在实现上，我们用一个字典（map）来管理等待中的请求。发请求时，创建一个等待通道放进字典；收到响应时，根据id找到对应的通道把消息发过去。

![[Pasted image 20260723222129.png]]

另一边有一个读取循环，持续从子进程的stdout读消息，根据id分发到对应的通道：
![[Pasted image 20260723222209.png]]
当读取循环退出时（说明子进程的stdout关闭了），把`alive`标记设为false。后面连接管理器会用到这个标记来判断是否需要重连。

## 把MCP工具融入MewCode
MCP Server返回的工具定义和MewCode 内部的Tool接口不是一回事，怎么让Agent统一使用？使用**适配器模式**。我们写一个MCPToolWrapper，把MCP工具包装成MewCode的Tool接口。这是经典的设计模式：两个接口不兼容，用一个中间层做转换。

![[Pasted image 20260723222524.png]]
- `description()`和`parameters()`直接透传`toolDef()`里的原始值。
- `extractText`从Server返回的`content`数组中提取所有`text`类型的块，拼接成一个字符串返回
- 工具名加了`mcp_`前缀和Server名称：因为不同的MCP Server可能提供同名的工具。一个Github Sever有search工具，一个Jira Server也有search工具，不加前缀就冲突了。

## 配置：让用户告诉MewCode该连哪些Server
项目级配置放在`.mewcode.yaml`，只在当前项目生效：
![[Pasted image 20260723223001.png]]

用户级配置在`~/.mewcode/config.yaml`里，在所有项目生效：
![[Pasted image 20260723223031.png]]

## 完整流程：从配置到调用
1. 启动：读取配置文件，获取MCP Server列表
2. 选择transport：根据Server配置选择`stdio`或`Streamable HTTP`
3. 后台连接：启动时异步连接所有配置的Server
4. 初始化：发送initialize + notifications/initialized
5. 工具发现：发送tools/list，获取工具定义
6. 包装注册：为每个工具创建MCPToolWrapper，注册到ToolRegistry
7. Agent使用：Agent在工具列表中看到MCP工具，决定是否调用
8. 工具调用：Agent调用 → MCPToolWrapper.execute → MCP Client → MCP Server
9. 结果返回：MCP Server返回结果 → MCPToolWrapper转换 → Agent处理

### 什么时候连接MCP Server
MCP协议要求调用`tools/list`才能拿到工具定义，而`tools/list`必须在连接建立之后才能发。

也就是说：不连接就不可能知道Server有哪些工具。

如果等到Agent第一次尝试调用时才连接（懒加载），Agent根本不知道有这些工具存在，也就不会去调用他们。

所以采用启动时后台连接所有Server的策略：启动后立即异步连接，拿到工具列表注册到ToolRegistry。

这样Agent第一轮（或前几轮）对话就能在system prompt中看到MCP工具。连接缓存住，后续复用；部分Server连接失败不阻止启动，只打警告。

![[Pasted image 20260725162126.png]]

## 工具延迟加载：80个工具塞不进上下文
假如用户配置了4个MCP Server，每个提供15-20个工具，加上自己内置的6个工具，工具列表一下膨胀到80个。每个工具的完整schema包含名称、描述、参数定义和类型约束，平均占100到300个token。80个就是8000到24000个token的工具定义，每一轮对话都要带着。浪费token。

token浪费只是表面问题。更深层的印象是模型的选择质量。当工具列表挤满几十个名字相似的工具，模型需要在这些选项中做决策，很干扰模型判断。

解决思路：不需要每轮都把所有工具的完整schema塞给模型。大部分MCP工具在一次对话里可能根本用不到，只会占用上下文窗口。

延迟加载分四步：
1. MCP工具在注册时标记自己为`延迟工具`。MCPToolWrapper的ShouldDefer()固定返回true，意思是`我的完整schema默认不进模型的工具列表`
2. Agent Loop每轮生成工具列表时，跳过这些延迟工具的完整schema，只在system-reminder里列出他们的名字
3. 模型看到名字列表，判断需要某个工具时，先调用ToolSearch拉取它的完整定义
4. ToolSearch在客户端的Registry里找到工具、返回完整Schema、标记为`已发现`，从下一轮开始这个工具就会出现在正常的工具列表里
![[Pasted image 20260725163722.png]]

ToolSearch支持两种查询模式：
- `select:mcp__grafana__query_prometheus`按名称精确拉取，适合模型已经知道要用哪个工具的场景
- 直接输入关键词（比如`prometheus`）则在所有延迟工具的名称和描述里做搜索，适合模型不确定具体工具名的场景

**内置工具一律常驻，MCP工具一律延迟**

## 安全考量：信任边界
1. 工具审批。MCP工具和内置工具一样，受权限系统管控。Agent想调用MCP工具时，一样需要经过权限检查。可以按工具名配置权限规则：
![[Pasted image 20260725164150.png]]
2. 命令白名单。理论上配置文件里可以写任意命令。如果对安全要求比较高，可以考虑限制允许运行的MCP Server命令，防止配置文件中的恶意命令。