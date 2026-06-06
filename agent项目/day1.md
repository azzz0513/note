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
前面的curl示例用的是普通请求：模型生成完所有内容，一次性返回。这在调试的时候没问题，但在产品里完全bu'nen'yo
