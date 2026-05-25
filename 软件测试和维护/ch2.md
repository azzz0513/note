## 测试流程（Test Process）
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

