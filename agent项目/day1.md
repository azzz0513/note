## API长什么样？
在写客户端代码之前，建议先翻一遍Anthropic官方的Messages API文档：
> https://docs.anthropic.com/en/api/messages

文档中有完整的参数说明、请求示例和响应结构，随着API更新也会同步。

项目同时兼容Anthropic和OpenAI两套协议，先以Claude的Messages API为你。OpenAI协议的适配在封装层处理。

先用一个curl命令直观感受API的请求和响应。可以在终端里直接跑这条命令（把`$ANTHROPIC_API_KEY`更换为你自己的真实Key）：
```Bash
curl -v https://api.deepseek.com/anthropic/v1/messages ^
  -H "Content-Type: application/json" ^
  -H "anthropic-version: 2023-06-01" ^
  -H "x-api-key: sk-xxxxxxxx" ^
  -d "{\"max_tokens\":1000,\"model\":\"deepseek-v4-pro\",\"messages\":[{\"role\":\"user\",\"content\":[{\"type\":\"text\",\"text\":\"hello, world!\"}]}]}"
```
对应的响应（省略了部分字段）：
```json
{
	"id": "70d76341-4b3c-4967-8bb4-6c49a815c860",
	"type": "message",
	"role": "assistant",
	"model": "deepseek-v4-pro",
	"content": [
		{
			"type": "thinking",
			"thinking": "We need to respond to \"hello, world!\" in a helpful manner. The user is just greeting. I could respond with a friendly greeting and offer assistance. The instruction doesn't specify any particular role. I'll respond as a helpful AI assistant.",
			"signature": "70d76341-4b3c-4967-8bb4-6c49a815c860"
		},
		{
			"type": "text",
			"text": "Hello! It's great to see you. How can I help you today?"
		}
	],
	"stop_reason": "end_turn",
	"stop_sequence": null,
	"usage": {
		"input_tokens":8,
		"cache_creation_input_tokens": 0,
		"cache_read_input_tokens": 0,
		"output_tokens": 67,
		"service_tier": "standard"
	}
}
```
一个HTTP POST，发了一段JSON过去，拿了一段JSON回来。本质上所有LLM应用都在干这种事，只是上面包了不同的壳。

### Message格式
先看请求中的`messages`。它是一个数组，每条消息都有`role`和`content`两个字段。
- `role`：只有两个值：`user`（用户说的话）和`assistant`（模型的回复）

Claude的模型是按user和assistant交替对话的模式训练的，所以messages数组里最好保持两个角色交替出现，第一条通常是user。

不过这不是硬性限制。如果连续放了两条user消息，API不会报错，它会自动把它们合并成一条。

但在实际写Agent的时候，保持交替仍然是一个好习惯。在实现工具调用时会碰到一个坑：LLM返回一个工具调用请求，这算assistant的消息。你执行完工具拿到了结果，需要把结果发回去——这个结果要作为user消息发送，因为从API的视角看，所有你发给模型的东西都归user，模型返回的都归assistant。如果搞错了角色，把工具结果也当成assistant发出去，就会出现两条assistant连在一起，API会直接拒绝。

![[images/Pasted image 20260606163734.png]]

再看响应中的`content`字段。响应中的content永远是一个数组，不是一个字符串（虽然发送请求时content可以用字符串简写，但响应里永远是数组格式）

为什么是数组？因为模型的一次回复可能包含多种内容。它可能想说一段话，然后请求调用一个工具，甚至一次调多个工具。

每种内容是一个独立的`content block`，类型可能是`text`、`tool_use`等等
![[images/Pasted image 20260606164022.png]]

## 流式响应：不是可选的，是必须的
前面的curl示例用的是普通请求：模型生成完所有内容，一次性返回。这在调试的时候没问题，但在产品里完全不能用。

因为Claude生成一段长回复可能需要10到30秒。用户不可能长时间等待一个空白屏幕。

所以必须使用流式响应。流式的意思是：模型一边生成一边推送给你，你一边收到一边显示。用户会看到文字一个一个蹦出来，就像有人在实时打字。

流式响应基于**SSE**（Server-Sent Events）协议，只需直到它本质是一个长连接的HTTP响应。服务器往里面持续写数据。[[../面试/计算机网络#SSE]]

Claude的流式时间是有固定顺序的：
![[images/Pasted image 20260606165724.png]]
你的代码需要在不同事件上做不同的事情：
- `message_start`到达时记录输入Token数
	- `content_block_delta`每来一个就把文字增量推给UI显示
- `message_delta`里提取输出Token数
- 最后`message_stop`做收尾

有一个容易踩的坑：一次响应可能有多个content_block。比如模型先输出一段文字，再请求调用一个工具，这就是两个block。你的解析代码不能假设只有一个文本块。

### 不同语言的流式处理模型
流式处理的核心需求是一样的：生产者持续产生事件，消费者逐个处理。但不同语言有不同的惯用模式：


| 语言     | 流式原语              | 消费方式                        |
| ------ | ----------------- | --------------------------- |
| Go     | `channel`         | `for event := range ch`     |
| Python | `async generator` | `async for event in stream` |
| Java   | `Iterable<Event>` | `for (Event e : stream)`    |
不管用哪种，核心就是让生产者和消费者各干各的、互不阻塞。流跑完了连接要自动关掉，不能漏资源。用户按Ctrl+C的时候也得能干净退出，不能挂在那里。

### 其他不同的信息字段
- `system`参数放的是角色设定和环境信息。告诉模型你是谁、该怎么行为，也告诉它当前工作目录是什么、操作系统是什么。你总不希望模型在Linux上给你建议用PowerShell把。这部分在一次会话内相对固定，不随对话变化。
- `messages`数组放的是对话历史和动态上下文。用户和模型之间你一句我一句地对话，以及后续会讲到的项目指令、动态提醒等信息，都放在这里。
- `tools`参数放的是工具描述。你的Agent有哪些工具可以用、每个工具的参数是什么格式、返回什么结果。这就是Function Calling。

### Token
Token是LLM的计费单位。粗略来说，英文每个单词大约1~2个token，中文每个字大约1~2个token。具体取决于模型使用的tokenizer，你不需要精确计算，只需要知道它是衡量输入输出量和计费的基本单位。

可以看前面响应示例中的`usage`字段：
```json
"usage": {
	"input_tokens":8,
	"cache_creation_input_tokens": 0,
	"cache_read_input_tokens": 0,
	"output_tokens": 67,
	"service_tier": "standard"
}
```
Claude API的计费分两部分：
- `input_tokens`是你发给模型的所有内容，包括system prompt、messages和tools描述。
- `output_tokens`是模型生成的回复

想象一下多轮对话的场景：每一轮请求，你都要把完整的对话历史发过去。如果你跟模型聊了20轮，第21轮请求会包含前20轮的所有消息。`input_tokens`会随着对话轮次线性增长。

粗略计算。假设每次请求的固定开销（system prompt、环境信息、工具描述等）加起来有1000tokens，每轮用户输入50tokens，模型回复500tokens。

到第20轮，仅`input_tokens`就是1000+20×(50+500) = 12000tokens。而第1轮只需要1050tokens。差了10倍还多。

这就是为什么需要上下文压缩的原因。

## Extended Thinking：让模型先想再说
Claude支持Extended Thinking，让模型在正式回复之前先进行一轮内部推理。开启后响应的content数组会多一个`thinking`类型的内容块，排在`text`块之前
```json
{
	"type": "thinking",
	"thinking": "We need to respond to \"hello, world!\" in a helpful manner. The user is just greeting. I could respond with a friendly greeting and offer assistance. The instruction doesn't specify any particular role. I'll respond as a helpful AI assistant.",
	"signature": "70d76341-4b3c-4967-8bb4-6c49a815c860"
},
```

对Agent开发来说，只需要记住两件事：
- thinking的Token算在`output_tokens`里，是有成本的，本质上是用钱换更准确的工具调用决策。Agent场景下这通常值得，因为一次准确的工具调用可以节省好几轮纠错的开销
- thinking内容不能放进后续请求的messages里，维护对话历史时必须把thinking块过滤掉，只保留text和tool_use块发送给API，否则API会报错

## 封装的核心原则
上层代码只认你自己定义的类型：消息、流式事件、Token用量。至于底下到底调的是Claude还是GPT，上层完全不关心。

配置上只需要四个字段就能覆盖所有主流供应商：
- `protocol`决定走哪家的API协议
- `model`指定模型
- `base_url`指定端口地址
- `api_key`做认证

封装层干的事说白了就是翻译：往外发请求的时候，把你的统一类型翻译成对应供应商的格式。收到响应的时候，再翻译回来。

用伪代码来说就是这样：
```Go
// 你自己定义的类型（上层代码只用这些）
type Message struct {
	role
	content
}
type StreamEvent struct {
	type
	text
	usage
	error
}
type Usage struct {
	inputTokens
	outputTokens
}

type LLMClient struct {
	protocol
	model
	baseURL
	apiKey
}

func Constructor(protocol, model, baseURL, apiKey) *LLMClient 

func (this *LLMClient) streamChat(systemPrompt, messages) []byte {
	// 1. 把自定义Message转成对应供应商的格式
	// 2. 调用对应的流式API
	// 3. 把供应商的事件转成自定义StreamEvent
	// 4. 通过异步流返回给调用方
}
```
封装层内部怎么折腾是它自己的事，调用方完全不需要知道。从调用方的视角看，用起来就这么几行：
```GO
events := client.streamChat(systemPrompt, messages)
for _, event := range events {
	if event.type == "text" {
		fmt.Print(event.text)
	} else if event.type == "done" {
		fmt.Print(event.usage) // 显示token用量
	} else if event.type == "error" {
		handleError(event.error) // 处理错误
	}
}
```

## 从单轮到多轮
到目前位置，我们讨论的都是单词API调用：发一个请求，拿一个回复，结束。

但对于一个Coding Agent来说，多轮对话是基本能力。用户描述一个需求，Agent问几个澄清问题，然后开始执行。这个过程天然就是多轮的。

没有上下文记忆的Agent，每次都要用户把需求从头说一遍，根本没法用。

那么多轮对话时怎么实现的？
每次调API，就把完整的对话历史发过去。

Claude API没有什么会话ID让服务器记住之前的对话。每次你发请求，都要把从第一轮到最新一轮的所有消息打包发送。模型靠这些历史消息来理解上下文。

每一轮请求都包含之前所有轮次的完整内容。你需要在客户端维护完整的消息列表，每次用户发消息、模型回复，都要记录下来。

## 消息模型：两层设计
