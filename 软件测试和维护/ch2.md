## ==测试流程（Test Process）==
### Waterfall Model（瀑布模型）
- 所有的计划在一开始就完成了，一旦被创建了以后就无法被改变
- There is **no overlap** between any of the subsequent phases.任何后续阶段之间都没有重叠
- Often anyone’s first chance to “see” the program is at the very end once the testing is complete.通常任何人第一次“看到”该计划的机会是在测试完成后的最后
![[images/whiteboard_exported_image.png]]

| 优势  | 1. If time is spent early on making sure that the requirements and design are absolutely correct, then this will save much time and effort later.（如果早期设计是**完全正确**的，那么之后能节约很多时间）<br>    <br>2. There is an emphasis on documentation which keeps all knowledge in a central repository and can be referenced easily by new members joining the team.（重点是文档，它将所有知识保存在一个中央存储库中，并且可以很容易地被加入团队的新成员引用。）                                                                                                                                                                                                                                                                                                                                                                                                                |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 不足  | 1. Few visible signs of **progress** until the end of the project （直到项目结束，几乎没有明显的进展迹象）<br>    <br>2. It is not flexible to **changes** （针对变化不灵活）<br>    <br>3. Time-consuming to produce all the **documentation**（文档生成耗时）<br>    <br>4. **Tests** are only carried out at the end – this could mean a compromise if time or budgetary constraints exist（测试只在最终环节被实施：如果存在时间或预算限制，这可能意味着妥协）<br>    <br>5. Having to test the program as a **whole** could result in incomplete testing （将程序作为一个整体进行测试会导致不完全的测试）<br>    <br>6. If testing does identify a fault that suggests a redesign it may be ignored because of the trouble involved（如果测试确实发现了建议重新设计的故障，则可能会因为涉及的问题而被忽略）<br>    <br>7. If the customer is unhappy it may **incur a long maintenance** phase resolving their issues（顾客不满意的话，需要长时间的维护需解决他们的诉求） |

### Spiral Model螺旋模型
开发 → 迭代：对**每一个迭代模型进行评审以及验证**
- **风险驱动**的开发流程
- 组合了**瀑布模型**以及**快速原型迭代模型**
- 从**目标设计**开始，从客户回顾流程结束

1. 决定目标
2. 识别风险
3. 开发 & 测试
4. 计划下一轮迭代
![[images/Pasted image 20260525161100.png]]
![[images/Pasted image 20260525161107.png]]
优势：功能更改可适当推后、成本估计简单、风险管理有效、开发快、有用户反馈空间
缺点：可能无法按时+平账、只适用于大项目、管理严格、文档更多、对小项目不可取

### V Model
![[images/Pasted image 20260525162134.png]]
这是瀑布模型的扩展
通过 **标记每一个阶段的生命周期以及测试活动** 来强调 Verification & Validation
一旦编码完成，测试就随之开始了
**从单元测试开始，然后测试层级逐步提高 ，直到验收测试完成。**
![[images/Pasted image 20260525162636.png]]

| 优势  | 1. It is simple and **easy to manage** due to the rigidity of the model. （由于模型的刚性，它简单易管理）<br>    <br>2. It encourages **verification and validation** at all phases. （它鼓励在所有阶段进行验证和校验）<br>    <br>3. Each phase has specific deliverables and a review process. （每个阶段都有**特定的可交付成果和审查流程**）<br>    <br>4. It gives **equal weight to testing** alongside development rather than treating it as an afterthought at the end.（它将**测试与开发同等重视**，而不是在最后将其视为事后的想法） |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 不足  | 1. Like the Waterfall model , there is no working **software** produced until **late** during the life cycle.（和瀑布模型一样，直到生命周期后期才生产出工作软件）<br>    <br>2. It is **unsuitable** where the requirements are at a moderate to **high risk of changing**. （不适合需求处于中度到高度变更风险的情况。）<br>    <br>3. The tight link between test, debug and change tasks during the test phase is not clear.（测试阶段的测试、DEBUG 和变更任务是不清晰的）                                                   |

### W Model
![[images/Pasted image 20260525163024.png]]
别名：V 模型拓展 / 双 V 模型
测试并不是在编码完成后进行的，而是和开发过程**平行（PARALLEL）**
强调开发和测试的协作 **（CO-OPERATION）** 
测试不只是构建，还包括执行和评估

### Agile Model——XP敏捷模型：极限编程
![[images/Pasted image 20260525182148.png]]

#### 极限编程
- 迭代和增量开发
	- 将大项目拆分为小的周期，每个交付一个功能增量以迭代细化特征
- 以消费者为中心的价值交付
	- 经常交付工作软件以适应不断变化的需求
- 拥抱变化
	- 将需求变更视为改进的机会，通过持续反馈调整优先级，而不是严格遵守最初的计划
- 自组织的团队和组织
	- 赋能跨职能团队自我管理，强调面对面沟通

#### 极限编程的价值
![[images/Pasted image 20260525182437.png]]

##### Communication
- XP programmers communicate with their customers and fellow programmers
沟通：
极限编程的程序员与他们的客户和其他程序员交流

##### Simplicity
- they keep their design simple and clean
简单：
他们保持设计简单干净

##### Feedback
- Get feedback by software testing from the start
反馈： 
从一开始就通过软件测试获得反馈

##### Courage
- Deliver the system to customers as early as possible 
- Implement changes as suggested, responding with courage to changing requirements
勇气：
尽早将系统交付给客户 
按照建议实施变更，勇于响应不断变化的需求

#### 极限编程的软件测试
##### 生命周期测试
软件生命周期的测试：
- 左移测试：在需求阶段过程中开始测试，和开发相互平行
- 持续测试：将测试集成到每一个迭代上，从而保证每一次增量发布的质量稳步提升

##### ==TDD==
定义：TDD（Test-Driven Development）测试驱动开发
- 先写测试——然后编译——接下来跑测试——写代码——跑测试——测试通过再重构
- 在编码之前写测试用例：帮助开发者思考接口设计以及边界条件
- 先书写单元测试——然后开发代码去通过测试——最后重构代码
- 通过测试定义需求去保证代码符合预期并且保持可维护性
  开始 —— 编写测试 —— 编译 —— 修复编译错误 —— 运行代码观察其失败 —— 编写代码 —— 运行代码观察其通过 —— 根据需求重构代码 —— 重复编写测试
![[images/Pasted image 20260525195436.png]]

###### 实现细节
- 仅在自动化测试失败时编写代码
- 如果你通过其他方式找到bug，先写一个失败的测试，然后修复bug
	- Bug以后不会重新出现
- 尽可能经常地运行测试，理想情况下每次更改代码时都要运行测试
	- 拥有全面的单元测试可以让您自信地重构代码
	- 没有单元测试，代码很脆弱——更改可能会破坏客户端

###### 好处
- 单元测试实际上被编写了
- 程序员的满意能让测试用例编写变得更有持续性
- 让接口和行为的细节更加清晰
- 可证明、可重复、自动化的验证
- 为程序员提供重构的自信

###### 验收测试
- **Acceptance Testing Aligned with User Stories**
Define acceptance criteria for each user story, with tests designed around these criteria.

- 与用户故事对齐验收测试
为每个用户故事定义验收标准，并围绕这些标准设计测试。

###### CI（持续集成）
- **Continuous Feedback and Improvement**
Use daily builds and continuous integration (CI 持续集成) to automate tests

- 持续反馈和改进
使用每日构建和持续集成（CI持续集成）来自动化测试

###### 测试金字塔
- Automation as the Backbone
测试金字塔（自顶向下）UI 测试 → 集成测试 → 单元测试
![[images/Pasted image 20260525200220.png]]

