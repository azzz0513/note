### 什么是系统测试？
![[images/Pasted image 20260527155526.png]]
使用黑盒测试
- 根据顶级要求协调软件
- 测试源于需求中的具体用例

### 系统测试的类型
按照测试的目的做区分，而不是它的范围或者机制（mechanism）
- 功能测试：确保功能需求符合预期
- 非功能测试：确保非功能需求符合预期
- 回归测试

#### ==功能测试==
测试的默认假设：
- 功能测试验证软件的行为是否符合预期
- 它也包括了测试坏的输入，去校验隐含假设（implicit assumptions）
	- 例：即使给了一个离谱的输入，程序也应该做出合理的回复
- 削减所有级别的测试，但**重在单元测试**
    - 系统测试使用的接口：用户接口、网络接口、专用硬件接口等

![[images/Pasted image 20260527160350.png]]
基于黑盒测试（等价类划分、边界值分析、场景测试、错误估计、决策表测试、随机测试等）=> 直接
类似单元测试，测试用例和测试数据都使用正在使用的技术进行选择。
唯一不同的是，不是调用指定参数的方法，而是要**视情况将数据输入接口，然后从接口收集结果**。

#### 非功能测试
测试软件的质量 "-ilities"
![[images/Pasted image 20260527200842.png]]

#### 回归测试
Making sure that code changes haven’t broken existing functionality, performance, security, etc.（确保代码更改没有破坏现有功能、性能、安全性等。）

回归测试的必要性是：无论是修复过往的 BUG 还是新增某个特性，引入新 BUG 是很常见的。

在实际应用中，这意味着在代码发生变更以后**要重新跑测试用例**。

With good **test automation** and good **unit/integration/system/**etc. tests. This is literally running tests again after a change.

通过良好的**测试自动化**和良好的**单元/集成/系统**/等测试。这实际上是在更改后再次运行测试。

#### 测试自动化
![[images/Pasted image 20260527201216.png]]

##### 测试自动化的优缺点
![[images/Pasted image 20260527201244.png]]
###### **优点**
- **高投资回报率（ROI） & 加快产品上市速度**
    - 支持重复测试用例的执行
    - 支持大规模测试矩阵的覆盖测试
    - 支持并行执行（如多环境同步测试）
    - 支持无人值守执行（自动化脚本可定时运行）
    - 提高准确性，减少人为错误
    - 节省时间和成本
###### **缺点**
- 自动化工具通常成本高昂；
- 无法有效评估应用的用户体验（如界面友好性、交互流畅度）；
- 必须掌握编程知识及相关经验。

#### 性能测试
![[images/Pasted image 20260527202939.png]]用于检查应用程序或软件**在工作负载下**在响应性和稳定性方面表现的测试类型。
性能测试的目标是从应用中识别并且移除性能瓶颈。这是**性能工程的子集**。
这一种测试主要用于检查软件的速度、可扩展性和稳定性是否符合预期需求。
**速度：** 应用的响应快不快
**可扩展性：** 软件应用能处理的最大用户负载
**稳定性：** 在不同的负载下应用是否稳定

##### 常见的性能问题
![[images/Pasted image 20260527203227.png]]
![[images/Pasted image 20260527203233.png]]

#### 性能测试指标：监控的参数
**Processor Usage –** an amount of time processor spends executing non-idle threads.
处理器使用率：处理器用于执行非空闲线程的时间量

**Memory use** **–** amount of physical memory available to processes on a computer.
内存利用率：计算机上进程的物理可用存储器量

**Disk time –** amount of time disk is busy executing a read or write request.
磁盘时间：磁盘忙于执行读或写请求的时间量

**Bandwidth –** shows the bits per second used by a network interface.
带宽：展示了一个网络接口每秒钟使用的比特位

**Response time** **–** time from when a user enters a request until the first character of the response is received.
响应时间：从用户输入请求到收到响应的第一个字符的时间。

**Throughput** **–** rate a computer or network receives requests per second.
吞吐量：每秒钟电脑或者网络获取请求的比率

**Hits per second** **–** the no. of hits on a web server during each second of a load test.
每秒命中数：在负载测试的每一秒期间 Web服务器 上的命中数

…… 其他参数

#### 性能测试样例（基线测试）
![[images/Pasted image 20260527204829.png]]