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