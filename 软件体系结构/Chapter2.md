### **Inhibiting or Enabling a System's Quality Attributes** / 抑制或促进系统的质量属性
Whether a system will be able to exhibit its desired (or required) quality attributes is substantially determined by its architecture. / 系统是否能够展现其期望（或要求的）质量属性在很大程度上取决于其架构。
- Performance / 性能
- Modifiability / 可修改性
- Security / 安全性
- Scalability / 可扩展性
- Reusability / 可重用性

About 80 percent of a typical software system's total cost occurs after initial deployment: accommodate new features, adapt to new environments, fix bugs, and so forth. / 典型软件系统的总成本中约80％发生在初始部署之后：包括适应新功能、适应新环境、修复缺陷等。

Every architecture partitions possible changes into three categories: / 每个架构将可能的变化分为三类：
- A _local change_ can be accomplished by modifying a single element. / _本地变更_ 可以通过修改单个元素来实现。
- A _nonlocal change_ requires multiple element modifications but leaves the underlying architectural approach intact. / _非本地变更_ 需要修改多个元素，但不会改变底层的架构方法。
- _Architectural change_ affects the fundamental ways in which the elements interact with each other and will require changes all over the system. / _架构变更_ 影响元素之间相互作用的基本方式，必须对整个系统进行变更。

**Obviously, local changes are the most desirable. / 显然，本地变更是最理想的。**
A good architecture is one in which the most common changes are local, and hence easy to make. / 一个好的架构是那些最常见的变更是局部的，因此容易实现的架构。

### **Predicting System Qualities** / 预测系统的质量
When we examine an architecture, we can confidently predict that the architecture will exhibit the associated qualities. / 当我们审查架构时，可以自信地预测该架构将表现出相关的质量属性。 
The earlier you can find a problem with your design, the cheaper, easier, and less disruptive it will be to fix. / 你越早发现设计中的问题，修复起来就会越便宜、越容易，且对系统的影响越小。

### **Enhancing Communication Among Stakeholders** / 增强利益相关者之间的沟通
- The architecture — or at least parts of it — is sufficiently abstract that most non-technical stakeholders can understand. / 架构——或至少是其部分——具有足够的抽象性，足以让大多数非技术利益相关者理解。
- Most of the system's stakeholders can use it as a basis for creating mutual understanding, negotiating, forming consensus, and communicating with each other. / 系统的大多数利益相关者可以将其作为建立共同理解、协商、形成共识并相互沟通的基础。
- Each stakeholder of a software system is concerned with different characteristics of the system. / 软件系统的每个利益相关者关注系统的不同特性。
    - Users, clients, managers, architects / 用户、客户、经理、架构师

### **Earliest Design Decisions** / 最早的设计决策
Software architecture is a manifestation of the earliest design decisions about a system. / 软件架构是系统最早设计决策的体现。 These early decisions affect the system's remaining development, its deployment, and its maintenance life. / 这些早期决策影响系统的后续开发、部署和维护生命周期。 What are these early design decisions? / 这些早期设计决策是什么？
- Will the system run on one processor or be distributed across multiple processors? / 系统将在一个处理器上运行，还是分布在多个处理器上？
- Will the software be layered? If so, how many layers will there be? / 软件是否采用分层结构？如果是，有多少层？
- What will each one do? / 每一层将执行什么功能？
- Will components communicate synchronously or asynchronously? / 组件将同步还是异步通信？
- What communication protocol will we choose? / 我们将选择什么通信协议？
- Will the system depend on specific features of the operating system or hardware? / 系统是否依赖于操作系统或硬件的特定功能？
- Will the information that flows through the system be encrypted or not? / 流经系统的信息是否会加密？

### **Defining Constraints on an Implementation** / 定义对实现的约束
An implementation exhibits an architecture if it conforms to the design decisions prescribed by the architecture. / 如果实现遵循架构所规定的设计决策，则该实现体现了架构。
- The implementation must be implemented as the set of prescribed elements. / 实现必须作为规定元素的集合来实施。
- These elements must interact with each other in the prescribed fashion. / 这些元素必须以规定的方式相互作用。

Each of these prescriptions is a constraint on the implementer. / 这些规定中的每一项都是对实现者的约束。

### **Influencing the Organizational Structure** / 影响组织结构
Architecture prescribes the structure of the system being developed. / 架构规定了正在开发的系统的结构。 
The architecture is typically used as the basis for the work-breakdown structure. / 架构通常作为工作分解结构的基础。

### **Enabling Evolutionary Prototyping** / 启用演化原型开发
Once an architecture has been defined, it can be prototyped as a skeletal system. / 一旦架构被定义，它就可以作为骨架系统进行原型开发。 
A skeletal system is one in which at least some of the infrastructure is built before much of the system's functionality has been created. / 骨架系统是在系统的许多功能尚未创建之前，至少构建了部分基础设施的系统。 
The fidelity of the system increases as prototype parts are replaced with complete versions of these parts. / 随着原型部分被完整版本替换，系统的保真度提高。 
This approach aids the development process because the system is executable early in the product's life cycle. / 这种方法有助于开发过程，因为系统在产品生命周期早期就可以执行。 
This approach allows potential performance problems to be identified early in the product's life cycle. / 这种方法允许在产品生命周期的早期识别潜在的性能问题。 
These benefits reduce the potential risk in the project. / 这些好处降低了项目的潜在风险。

### **Improving Cost and Schedule Estimates** / 改进成本和进度估算
Architecture is used to help the project manager create cost and schedule estimates early in the project life cycle. / 架构用于帮助项目经理在项目生命周期早期制定成本和进度估算。
Top-down estimates are useful for setting goals and apportioning budgets. / 自上而下的估算有助于设定目标和分配预算。 
Bottom-up understanding of the system's pieces are typically more accurate than those that are based purely on top-down system knowledge. / 对系统各部分的自下而上的理解通常比纯粹依赖自上而下系统知识的估算更为准确。 
The best cost and schedule estimates will typically emerge from a consensus between the top-down estimates (created by the architect and project manager) and the bottom-up estimates (created by the developers). / 最佳的成本和进度估算通常来自自上而下估算（由架构师和项目经理制定）与自下而上估算（由开发人员制定）之间的共识。

### **Transferable, Reusable Model** / 可转移、可重用的模型
Reuse of architectures provides tremendous benefits for systems with similar requirements. / 架构的重用为具有相似需求的系统提供了巨大的好处。
- Not only can code be reused, but also can the requirements that led to the architecture in the first place. / 不仅代码可以重用，最初导致架构设计的需求也可以重用。
- When architectural decisions can be reused across multiple systems, all of the early-decision consequences are also transferred. / 当架构决策可以在多个系统中重用时，所有早期决策的后果也会随之转移。

### **Using Independently Developed Components** / 使用独立开发的组件
Architecture-based development often focuses on components that are likely to have been developed separately, even independently. / 基于架构的开发通常关注那些可能已被单独开发，甚至独立开发的组件。 
Commercial off-the-shelf components, open source software, publicly available apps, and networked services are examples of interchangeable software components. / 商用现成组件、开源软件、公开可用的应用程序和网络服务是可替换软件组件的例子。 The payoff can be: / 收益可能是：
- Decreased time to market / 缩短上市时间
- Increased reliability / 提高可靠性
- Lower cost / 降低成本
- Flexibility / 灵活性

### **Restricting Design Vocabulary** / 限制设计词汇
As useful architectural patterns are collected, we see the benefit in restricting ourselves to a relatively small number of choices of elements and their interactions. / 随着有用的架构模式的收集，我们发现限制选择较少的元素及其相互作用具有很大的好处。
- We minimize the design complexity of the system we are building. / 我们最小化正在构建的系统的设计复杂度。
- Enhanced reuse / 增强重用性
- More regular and simpler designs that are more easily understood and communicated / 更规则、更简单的设计，更易于理解和沟通
- More capable analysis / 更强的分析能力
- Shorter selection time / 更短的选择时间
- Greater interoperability / 更强的互操作性

### **Basis for Training** / 培训基础
Architecture can serve as the first introduction to the system for new project members. / 架构可以作为新项目成员对系统的首次介绍。
- Module views show someone the structure of a project: / 模块视图向人员展示项目的结构：
    - Who does what, which teams are assigned to which parts of the system, and so forth. / 谁做什么，哪些团队被分配到系统的哪些部分，等等。
- Component-and-connector explains how the system is expected to work and accomplish its job. / 组件与连接器解释了系统如何预期工作并完成其任务。

### Summary / 总结
软件架构重要总共有十三点原因
**英文原文：**
1. Architecture will inhibit or enable a system's driving quality attributes. / 架构将抑制或促进系统的关键质量属性。
2. The decisions made in architecture allow you to reason about and manage change as the system evolves. / 架构中的决策使您能够在系统发展过程中推理并管理变化。
3. The analysis of architecture enables early prediction of a system's qualities. / 对架构的分析使得能够提前预测系统的质量。
4. A documented architecture enhances communication among stakeholders. / 文档化的架构增强了利益相关者之间的沟通。
5. Architecture is a carrier of the earliest and hence most fundamental, hardest-to-change design decisions. / 架构承载了最早且最根本、最难更改的设计决策。
6. An architecture defines a set of constraints on subsequent implementation. / 架构定义了一组对后续实现的约束。
7. The architecture dictates the structure of an organization, or vice versa. / 架构决定了组织的结构，反之亦然。
8. An architecture can provide the basis for evolutionary prototyping. / 架构可以为演化原型提供基础。
9. Architecture is the key artifact that allows the architect and project manager to reason about cost and schedule. / 架构是架构师和项目经理推理成本和进度的关键产物。
10. Architecture can be created as a transferable, reusable model that forms the heart of a product line. / 架构可以作为一个可转移、可重用的模型创建，并成为产品线的核心。
11. Architecture-based development focuses attention on the assembly of components, rather than simply on their creation. / 基于架构的开发将重点放在组件的组装上，而不仅仅是它们的创建。
12. By restricting design alternatives, architecture channels the creativity of developers, reducing design and system complexity. / 通过限制设计选项，架构引导开发者的创造力，减少设计和系统复杂性。
13. Architecture can be the foundation for training a new team member. / 架构可以作为培训新团队成员的基础

