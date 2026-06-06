## API长什么样？
在写客户端代码之前，建议先翻一遍Anthropic官方的Messages API文档：
> https://docs.anthropic.com/en/api/messages

文档中有完整的参数说明、请求示例和响应结构，随着API更新也会同步。

项目同时兼容Anthropic和OpenAI两套协议，先以Claude的Messages API为你。OpenAI协议的适配在封装层处理。

先用一个curl命令直观感受API的请求和响应。可以在终端里直接跑这条命令（把`$ANTHROPIC_API_KEY`更换为你自己的真实Key）：
```Bash
curl https://api.deepseek.com/anthropic \
	-H 'Content-Type': application/json \
	-H "X-API-Key: sk-f46afd404a544f5486619ba65249d1d0" \
	-d '{
		"max_tokens": 1000,
		"model": "deepseek-v4-pro",
		"message": [
		{
			"role": "user",
			"content": [
			{
				"type": "text",
				"text": "hello, world!"
			}]
		}]
	}'
```