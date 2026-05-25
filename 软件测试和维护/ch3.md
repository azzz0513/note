## Test Principle 测试原理
### 静态和动态验证

|           | 静态验证 （Static Verification）                                                                                                               | 动态验证（Dynamic Verification）                                                                                                                                                                                                                              |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 是否需要执行代码？ | ✖                                                                                                                                        | ✅                                                                                                                                                                                                                                                       |
| 怎么做的？     | Reading through the code (straightforward)<br><br>直接通读代码                                                                                 | Executing<br><br>执行代码                                                                                                                                                                                                                                   |
| 包含步骤      | static analysis, code reviews, checks against coding standards and guidelines, and other techniques<br><br>静态分析、代码审查、根据编码标准和指南进行检查以及其他技术 | ![](https://a1npn29y3xu.feishu.cn/space/api/box/stream/download/asynccode/?code=NmEyNTJkMDE1ZTM5NGFjYzFhODFjOTYxYzdmN2U4ZDNfRjJQOExmdW9MYldUZVZEek1TRzBEUEtyUGZ2SXo1VEtfVG9rZW46SHZEd2JBUFBEb1lNWFB4RmZWZ2NEZjBjbnVkXzE3Nzk3MTA2NDE6MTc3OTcxNDI0MV9WNA) |
静态验证
a formal approach consisting of symbolic verification of the translation between the specification and the source code
一种正式的方法，包括对规范和源代码之间的转换进行符号验证

动态验证
Test Cases are created that guide the selection of suitable Test Data (consisting of Input values and Expected Output values ) / 创建测试用例以指导选择合适的测试数据（由输入值和预期输出值组成）
Input values.
Actual Outputs are compared with the Expected Outputs.

### 对比黑盒测试和白盒测试
![[images/Pasted image 20260525200604.png]]

|                               | 黑盒测试     | 白盒测试            |
| ----------------------------- | -------- | --------------- |
| 依赖内容                          | 只依赖需求说明书 | 依赖源代码实现 + 需求说明书 |
| 代码更改后<br><br>测试用例<br><br>能否重用 | 可以       | 一般情况下不可以        |
| 需要什么                          | 只需要指定规范  | 需要在测试之前写好代码     |
| 自动化测试难度                       | 较难       | 较易              |
- White box testing does **not** find faults related to **missing** functionality. These are errors of **omission**.
- 白盒测试没有发现与**功能缺失**相关的故障。这些是**遗漏错误**。
- Black box testing does **not** find faults related to **extra** functionality. These are errors of **commission**.
- 黑盒测试未发现与**额外功能**相关的故障。这些是**委托错误**。