### Gin的路由算法是什么
Radix Tree（基数树）
- 基于 httprouter 实现。
- 利用公共前缀（Common Prefix）压缩树的高度。
- 每个节点存储的是 URL 的一段路径。
- 相比于传统的正则表达式匹配或 Map 遍历，Radix Tree 查找复杂度是 O(k)（k 是 URL 长度），且内存占用极低。

### Gin的中间件是如何实现的
**洋葱模型**、HandlersChain。
- Gin 的 Context 中有一个 handlers 数组（`type HandlersChain []gin.HandlerFunc`）。
- 当你注册中间件时，它们和业务逻辑 Handler 一起被按顺序放入这个数组。
- **Next() 方法**：
    - 调用 c.Next() 会移动索引（index++），执行数组中的下一个 Handler。
    - 当下一个 Handler 执行完返回后，代码执行权会回到 c.Next() 之后。这构成了“请求进去，响应出来”的洋葱模型。
- **Abort() 方法**：
    - 设置索引为最大值（abortIndex），阻止后续 Handler 执行。

###  Gin 的 c.ShouldBind 和 c.Bind 有什么区别？
- Bind：校验失败时，会自动设置 Response 状态码为 400 并写入 Header，通常会打断请求流程。
- ShouldBind：校验失败时，只返回 error，由开发者自己决定如何处理（**推荐使用这个**，更灵活）。
- 底层都利用了 Go 的 reflect 包进行结构体 tag 解析。

### Gin的Context是并发安全的吗
- **不是**。
- **原因**：Gin 为了高性能，使用了 sync.Pool 来复用 Context 对象。
- **陷阱**：如果你在 Goroutine 中使用 c（比如 go func(c *gin.Context){...}），必须使用 c.Copy() 创建一个副本，否则原来的 Context 可能已经被回收到池里或者被分配给新的请求使用了，导致数据错乱。

### 在Gin的中间件和Handler之间，如何传递数据
使用`c.Set(key, value)`和`c.Get(key)`
通常在Auth中间件解析完Token后，把UserId塞入Context，后续业务逻辑直接取

