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

MCP Server的stderr不参与协议通信，它被用来输出日志和调试信息。开发者调试MCP Server的时候可以随便往stderr打日志，不会干扰协议的正常通信。

根据官方规范，stdio里的消息时UTF-8编码的JSON-RPC消息，通常以换行分隔。Server的stdout上不能混入任何非协议内容，否则Client就会解析失败。这也是为什么日志一定要走stderr，而不是stdout。