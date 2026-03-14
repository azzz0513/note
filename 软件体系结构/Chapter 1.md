### ==什么是软件体系结构==（What is Software Architecture）
_The software architecture of a system is_ _the set of structures（有三种必考的重要结构）_ _needed to_ _**reason（推导）**_ _for the system, which comprise software elements, relations among them, and properties of both._ / 系统的软件架构是为系统进行推理所需的一组结构，包含软件元素、它们之间的关系以及两者的属性。

### 架构是一组软件结构（Architecture is a Set of Software Structures）
结构是一组通过关系连接在一起的元素
软件系统由许多结构组成，且没有单一结构可以声称自己是架构
有三种重要的结构类型：
- Moduel（模块）
- Component and Connector（组件和连接器）
- Allocation（分配）

#### Moduel Structures（模块结构）
Modules：**Implementation units** that could partition systems（可以把系统拆分的实现单元）
模块被分配特定的计算责任，是编程团队工作分配的基础。
在大型项目中，这些元素（模块）被细分并分配给子团队。（**前后端分离也算是模块结构的一种表现形式，而前端和后端又会根据功能的不同拆分出细致的模块**）

#### Component-and-connector Structures（组件-连接器结构）
Named as **runtime structures**（运行结构）：Focus on the way the elements interact with each other at runtime to carry out the system's functions.（关注元素在运行时如何相互作用，以执行系统的功能。）
A component is always a runtime entity.（一个组件一直都是运行实体）：
- 在面向服务架构（SOA）中，系统作为一组服务构建
- 这些服务由不同实现单元（模块）中的程序组成（编译而来）
- 假设系统将作为一组服务构建
- 服务、它们交互的基础设施，以及它们之间的同步和交互关系，构成了另一种常用于描述系统的结构

![[images/Pasted image 20251221100830.png]]
- 组件（Component）：在 Linux 内核的 TCP/IP 体系结构中，组件可以是网络协议栈中的各个模块，如网络驱动程序、IP层、TCP层、UDP层、应用程序接口（如 sockets）等。每个组件都有自己的功能，它们各自负责特定的任务（例如数据传输、地址分配、协议处理等）。
- 连接器（Connector）：连接器在这里就是这些组件如何相互交互。比如说，TCP 层和 IP 层之间是通过特定的协议接口来进行交互的。TCP 协议通过 IP 协议传输数据包，这样就形成了一个组件-连接器的互动关系。每个层次的协议都依赖于上层和下层的通信接口（连接器）来完成各自的任务。
- 服务和交互：TCP/IP 协议栈在运行时提供的服务和这些服务之间的交互是非常典型的组件-连接器结构。在实际应用中，数据的传输不仅依赖于各个组件的功能实现，还依赖于它们之间的协调与同步。例如，在 TCP 传输过程中，数据的可靠传输需要 TCP 层和应用层之间的协作，而这种协作则依赖于底层的操作系统和硬件资源。

Linux 内核的 TCP/IP 体系结构图正是通过展示网络栈中各个协议层（如 TCP、IP、网络驱动程序等）作为“组件”，它们之间通过数据流、协议接口等“连接器”互相连接，从而共同完成系统的网络功能。你可以将每个协议栈看作是一个组件，而它们之间的通信和数据交换则是连接器。通过这个结构图，我们可以知道，系统的功能并非单一的，而是由多个组件协同工作，通过各自的连接器实现复杂的交互和同步。

### Allocation Structures（分配结构）
Allocation structures **describe the mapping** from software structures to the system's environments.（分配结构描述了从软件结构到系统环境的**映射关系**）
- Organizational（组织性的）
- Developmental（发展的）
- Installation（安装）
- Execution（执行）

- Modules are assigned to **teams** to develop, and assigned to places in **a file structure** for implemantation, integration, and testing. / 模块被分配给了团队去开发，并且分配了一个文件结构进行实现、集成和测试
- Components are deployed onto **hardware** in order to execute. / 为了执行，组件都被部署到了**硬件**上

### 哪些结构是架构化的
**A structure is architectural if it supports reasoning about the system and the system's properties.**（**如果一个结构支持对系统及其属性的推理，那么它就是架构性的。**）

该推理应当围绕系统中对某些利益相关者重要的属性进行。
这些属性包括：
- functionality achieved by the system（系统实现的功能）
- the availability of the system in the face of faults（系统在故障面前的可用性）
- the difficulty of making specific changes to the system（对系统进行特定更改的难易程度）
- the responsiveness of the system to user requests（系统对用户请求的响应性）
- many others（以及许多其他属性）

### Architecture is an Abstraction（架构是一种抽象）
架构特意省略了关于元素的某些信息，这些信息对于对系统进行推理没有帮助。
架构抽象使我们能够从元素的角度观察系统，了解它们是如何排列、如何交互、如何组合等。
这种抽象对于驾驭架构的复杂性至关重要。

### Architecture includes behavior（架构包括了行为）
每个元素的行为是架构的一部分，只要该行为可以用来对系统进行推理。
这种行为体现了元素之间如何相互作用，而这显然是架构定义的一部分。

在同步异步的应用：
- 同步与异步的交互行为：
	- 同步：在同步的交互中，一个组件或服务在等待另一个组件完成操作之前不会继续执行。这种交互方式意味着组件之间的操作是顺序的，通常会影响系统的响应时间和吞吐量
	- 异步：异步交互则允许一个组件在等待其他组件完成操作时继续执行其他任务，减少了等待时间，提高了系统的并发性和响应性
- 这两种行为体现了组件如何在系统中相互作用，以及这些交互如何影响系统的整体性能、可扩展性等属性。
- **推理与架构属性：** 同步和异步行为可以作为“推理”系统架构中某些重要属性的依据。例如，如果系统的可用性和响应时间是关键属性，那么选择同步或异步的交互方式将直接影响这些属性的实现。在架构设计中，理解这些行为有助于评估系统在面对高并发请求时的表现，或者系统在发生故障时的可恢复性。
- **对关键利益相关者的重要性：** 在设计架构时，不同的利益相关者（如开发人员、运维人员、用户等）关心的可能是不同的系统属性。同步和异步的选择能够影响系统的可维护性（比如修改异步通信方式比同步方式更复杂）或系统的响应时间（异步可能更适合处理高并发的用户请求）。这种行为是架构设计中的核心，帮助各利益相关者推理和评估系统。
同步和异步行为是架构设计中的基本元素，它们展现了组件如何相互作用，并且通过这些交互，我们能够推理出系统的各种重要属性（如响应性、可用性、可扩展性等）。这种推理过程是架构设计中至关重要的一部分，因为它能够帮助我们预测系统在不同场景下的表现，从而做出更加合理的设计决策。

### Strcutures and Views（结构和视图）
Architects design structures. They document views of those structures.（架构师设计结构，并记录这些结构的视图。）
A view is a representation of a coherent set of architectural elements, as written by and read by system stakeholders.（视图是一个连贯的架构元素集合的表现形式，由系统利益相关者编写和阅读。）
A structure is the set of elements itself, as they exist in software or hardware.（结构是元素本身的集合，存在于软件或硬件中。）
In short, a view is a representation of a structure.（简而言之，视图是结构的表现形式。）
- 例如，模块结构是系统模块及其组织方式的集合。
- 模块视图是该结构的表现形式，按照选定的符号模板进行文档化，并由一些系统利益相关者使用。

#### Module Structures（模块结构）
Module structures **embody（体现了）** decisions as to how the system is to be structured as a set of code or data units.（模块结构了体现了如何让系统被组成一组代码或者数据单元的决策。）

模块被分配了功能责任区域

##### 一些有用的模块结构
###### **Decomposition Structure**（分解结构）
这些单元是通过“是一个子模块”关系相互关联的模块。
它展示了模块如何递归地被分解成更小的模块，直到模块足够小，易于理解。
模块通常会有一些产品（如接口规范、代码、测试计划等）与之关联。
分解结构在很大程度上决定了系统的可修改性，通过确保可能的变更被局部化来实现这一点。
![[images/Pasted image 20251221104334.png]]

###### Uses Structures（使用结构）
这些单元也是模块，可能是类
这些单元通过“使用关系”相互关联，这是一种特定形式的依赖关系。
如果第一个单元的正确性依赖于第二个单元的正确运行，那么第一个单元就使用了第二个单元。
![[images/Pasted image 20251221110125.png]]

###### Layer Structure（层结构）
该结构中的模块称为层。
层是一个抽象的“虚拟机”，通过受控接口提供一组有凝聚力的服务。
层允许以严格管理的方式使用其他层。
在严格的分层系统中，一个层只允许使用一个其他层。
这种结构赋予系统可移植性，即能够更换底层计算平台。

###### Class （or generalization） Structure（类结构）
该结构中的模块单元称为类。
该关系为“继承自”或“是某个实例”。
**Inheritance** is a mechanism for **code reuse** and to allow independent extensions of the original software.（继承是一种代码重用机制，允许对原始软件进行独立扩展。）
类结构使得可以推理关于重用和功能的增量添加。

###### **Data Model Structure**（数据模型结构）
数据模型通过数据实体及其关系描述静态信息结构。
例如，在银行系统中，实体通常包括账户、客户和贷款。
账户有多个属性，如账户号码、类型（储蓄或支票）、状态和当前余额。

#### Component-and-connector Structures（组件-连接器结构）
C & C structures embody decisions as to how the system is to be structured as a set of elements that have runtime behavior(components) and interactions(connectors). / C＆C 结构体现了如何将系统构建为具有运行时行为（组件）和交互（连接器）的一组元素的决策。
元素是运行时组件，如服务、对等体、客户端、服务器或其他许多类型的运行时元素。
连接器是组件之间的通信方式，如调用-返回、进程同步操作符、管道等。

组件和连接器视图可帮助我们回答以下问题：
- 主要的执行组件有哪些，它们在运行时如何交互？
- 主要的共享数据存储有哪些？
- 系统的哪些部分被复制？
- 数据如何在系统中传输？
- 系统的哪些部分可以并行运行？
- 系统的结构是否可以在执行过程中发生变化？如果可以，如何变化？

**在MapReduce的应用：**
MapReduce主要包含两个阶段：
- Map阶段：对输入数据进行处理，将其转换为键值对（key-value pairs）
- Reduce阶段：对由Map阶段输出的键值对进行合并与汇总

通过 MapReduce 框架的例子，我们可以看到其典型的组件-连接器结构：
- **组件**：Map 阶段、Shuffle 阶段、Reduce 阶段、任务调度组件等；
- **连接器**：组件之间的数据传递和协调，如 Map 和 Shuffle 之间的连接、Shuffle 和 Reduce 之间的连接等。

这些组件通过连接器进行高效的协作，共同完成大规模数据处理任务。通过这种架构设计，MapReduce 不仅支持系统功能的实现，还能满足容错、扩展性等重要属性，从而使其成为大数据处理中的重要工具。

**在CS、P2P和BS的应用：**
1. **CS 架构（Client-Server 架构）**
	- **组件**：在 CS 架构中，客户端（Client）和服务器端（Server）分别作为两个主要的组件。客户端负责发起请求，而服务器端负责响应请求并提供服务。
	- **连接器**：客户端和服务器之间的通信通常通过 **请求/响应协议**（如 HTTP、FTP）来实现。这个协议充当连接器的角色，负责在客户端和服务器之间传递信息。
1. **BS 架构（Browser-Server 架构）**
	- **组件**：在 BS 架构中，主要的组件是 **浏览器（Browser）** 和 **服务器（Server）**。浏览器作为客户端，通过网络与服务器进行交互。
	- **连接器**：浏览器与服务器之间的交互通过 **HTTP 协议**（或其他类似的协议）来完成。HTTP 协议不仅传输请求和响应的数据，也负责数据的格式化和渲染。
2. **P2P 架构（Peer-to-Peer 架构）**
	- **组件**：在 P2P 架构中，所有参与的节点都可以同时作为客户端和服务器。每个节点（Peer）既能发起请求，也能响应请求。
	- **连接器**：节点之间的通信通过 **P2P 协议**（如 BitTorrent、Skype 等）来实现。这种协议确保节点间的数据交换和共享。

##### Some useful C & C Structures
###### Service structure / 服务结构
The units are services that interoperate with each other by service coordination mechanisms such as SOAP.（这些单元是通过服务协调机制（如SOAP）相互操作的服务。）
The service structure helps to engineer a system composed of components that may have been developed anonymously and independently of each other.（服务结构有助于构建由可能匿名开发且相互独立的组件组成的系统。）

###### Concurrency structure / 并发结构
This structure helps determine opportunities for parallelism and the locations where resource contention may occur. / 该结构有助于确定并行性的机会以及可能发生资源争用的位置。
The units are components. / 单元是组件。
The connectors are their communication mechanisms. / 连接器是它们的通信机制。
The components are arranged into logical threads. / 组件被安排为逻辑线程。
![[images/Pasted image 20251221151824.png]]

#### Allocation Structures（分配结构）
**Allocation structures** show the relationship between the software elements and elements in one or more external environments **in which the software is created and executed**.（**分配结构**显示软件元素与一个或多个外部环境中元素之间的关系，这些环境是软件创建和执行的场所。）
**分配视图**帮助我们回答以下问题：
- 每个软件元素在什么**处理器**上执行？
- 每个元素在开发、测试和系统构建过程中存储在哪些目录或**文件**中？
- 每个软件元素分配给哪些开发**团队**？

##### 一些有用的分配结构
###### Deployment Structure / 部署结构
部署结构展示了软件如何分配给硬件处理和通信元素
这些元素包括软件元素（通常是C&C视图中的进程）、硬件实体（处理器）和通信通道
Relations are **allocated-to**, showing on which physical units the software elements reside, and migrates-to if the allocation is dynamic.（关系通过**“分配到”**来表示，展示软件元素所在的物理单元；如果分配是动态的，则通过“迁移到”来表示。）

This structure can be used to reason for performance, data integrity, security, and availability.
该结构可用于推理性能、数据完整性、安全性和可用性。

It is of particular interest in distributed and parallel systems.
在分布式和并行系统中，这一结构尤为重要。
![[images/Pasted image 20251221152415.png]]

###### Implementation structure / 实现结构
该结构展示了软件元素（通常是模块）如何映射到系统的开发、集成或配置控制环境中的**文件结构。**
![[images/Pasted image 20251221152456.png]]

###### Work assignment structure / 工作分配结构
This structure assigns responsibility for implementing and integrating the modules to the **teams** who will carry it out. / 该结构将模块的实现和集成责任分配给将负责执行的**团队**。

### Structures provide insight（结构提供灵感）
每个结构提供了一种视角，用于推理系统的一些相关质量属性。
例如：
- 模块结构体现了哪些模块使用了其他模块，它与系统扩展的难易程度密切相关。
- 并发结构体现了系统内部的并行性，它与系统避免死锁和性能瓶颈的难易程度密切相关。
- 部署结构与实现性能、可用性和安全性目标密切相关。

### Relating Structures to Each other（结构之间相互关联）
Elements of one structure will be related to elements of other structures, and we need to reason about these relations. / 一个结构的元素将与其他结构的元素相关，我们需要推理这些关系。
- A module in a decomposition structure may be manifested as one, part of one, or several components in one of the component-and-connector structures. / 分解结构中的模块可能表现为组件和连接器结构中的一个组件、一个组件的一部分或多个组件。
In general, mappings between strucures are many to many. / 通常，结构之间的映射是多对多的。
![[images/Pasted image 20251221153139.png]]

### Architectural Patterns / 架构模式
An **architectural pattern** presents the element types and their forms of interaction used in solving a particular problem. / 架构模式呈现了解决特定问题时所使用的元素类型及其交互形式。 A common **module type pattern** is the Layered pattern. / 一种常见的模块类型模式是分层模式。
- When the usage of software elements is strictly unidirectional, a system of layers emerges. / 当软件元素之间的使用关系严格单向时，分层系统就会出现。
- A layer is a coherent set of related functionality. / 一层是一个相关功能的连贯集合。

Common **component-and-connector type patterns**: / 常见的组件和连接器类型模式：
- Shared-data (or repository) pattern. / 共享数据（或仓库）模式。 ① This pattern comprises components and connectors that create, store, and access persistent data. / 该模式包含创建、存储和访问持久数据的组件和连接器。 ② The repository usually takes the form of a (commercial) database. / 仓库常采用（商业）数据库的形式。 ③ The connectors are protocols for managing the data, such as SQL. / 连接器用于管理数据的协议（SQL）

![[images/Pasted image 20251221153216.png]]
  Common **component-and-connector type patterns**: / 常见的组件和连接器类型模式：
- Client-server pattern. / 客户端-服务器模式。
    - The components are the clients and the servers. / 组件是客户端和服务器。
    - The connectors are protocols and messages they share among each other to carry out the system's work. / 连接器是它们之间共享的协议和消息，用于完成系统的工作。
- Peer-to-peer pattern / 点对点模式
    - E.g. Bittorrent, eMule / 例如，Bittorrent，eMule
    
Common **allocation patterns**: / 常见的分配模式：
- Multi-tier pattern / 多层模式
    - This pattern specializes in the generic deployment (software-to-hardware allocation) structure. / 该模式专门化了通用的部署（软件到硬件分配）结构。
    - Describes how to distribute and allocate the components of a system in distinct subsets of hardware and software, connected by some communication medium. / 描述如何将系统的组件分配到不同的硬件和软件子集，并通过某种通信媒介将它们连接起来。
- Competence center pattern and platform pattern / 能力中心模式与平台模式
    - These patterns specialize in a software system's work assignment structure. / 这些模式专门化了软件系统的工作分配结构。
    - In competence center, work is allocated to sites depending on the technical or domain expertise located at a site. / 在能力中心，工作根据各站点的技术或领域专长进行分配。
    - In platform, one site is tasked with developing **reusable core assets** of a software product line, and other sites develop applications that use the core assets. / 在平台模式中，一个站点负责开发软件产品线的可重用核心资产，其他站点开发使用这些核心资产的应用程序。

### Rules of Thumb makes "Good" Architecture / 经验法则创造了好的架构
#### **Process “Rules of Thumb”** / 过程“经验法则”
Architecture should be the product of a single architect or a small group of architects with an identified technical leader. / 架构应由一名架构师或一个由明确技术领导者领导的小组架构师团队完成。 
The architect (or architecture team) should base the architecture on a prioritized list of well-specified quality attribute requirements. / 架构师（或架构团队）应基于优先排序的明确质量属性需求列表来构建架构。 
The architecture should be documented using views. / 架构应通过视图文档化。 Architecture should be evaluated for its ability to deliver the system's important quality attributes. / 应评估架构在提供系统重要质量属性方面的能力。
Architecture should lend itself to incremental implementation. / 架构应便于增量实现。

#### **Structural “Rules of Thumb”** / 结构“经验法则”
  The architecture should feature well-defined modules. / 架构应具有明确定义的模块。 
  The architecture should never depend on a particular version of a commercial product or tool. / 架构不应依赖于特定版本的商业产品或工具。 
  Modules that produce data should be separate from modules that consume data. / 产生数据的模块应与消耗数据的模块分离。
- This tends to increase modifiability. / 这有助于提高可修改性。

  Don't expect a one-to-one correspondence between modules and components. / 不要期望模块与组件之间存在一一对应关系。 
  Every process should be written so that its assignment to a specific processor can be easily changed, perhaps even at runtime. / 每个进程应编写成可以轻松地分配到特定处理器，甚至在运行时改变。 
  The architecture should feature a small number of ways for components to interact. / 架构应具有少量的组件交互方式。
- The system should do the same things in the same way throughout. / 系统应始终以相同的方式执行相同的操作。


## Summary
### What is software architecture ? / 什么是软件架构？
**The software architecture** of a system is the set of structures needed to reason about the system, which comprise software elements, relations among them, and properties of both. / 系统的软件架构是为系统进行推理所需的一组结构，包含软件元素、它们之间的关系以及两者的属性。
**A structure** is a set of elements and their relations among them. / 结构是一组元素及其之间的关系。 **A view** is a representation of a coherent set of architectural elements. A view is a representation of one or more structures. / 视图是一个连贯的架构元素集合的表现形式。视图是一个或多个结构的表现形式。
  
### **Some Useful Structures** / 一些有用的结构
#### Module Structures / 模块结构
- Decomposition structure / 分解结构
- User structure -> layer pattern / 用户结构 -> 分层模式
- Class structure / 类结构
- Data model / 数据模型

#### Component-and-connector structures / 组件与连接器结构
- Service structure / 服务结构
- Concurrency structure / 并发结构

#### Allocation structures / 分配结构
- Deployment structure / 部署结构
- Implementation structure / 实现结构
- Work assignment structure / 工作分配结构

### **Architectural Patterns** / 架构模式
- Module type pattern / 模块类型模式
    - Layered pattern / 分层模式
- Component-and-connector type pattern / 组件与连接器类型模式
    - Shared data pattern / 共享数据模式
    - Client and server pattern / 客户端-服务器模式
    - Peer to peer pattern / 点对点模式
- Allocation type pattern / 分配类型模式
    - Multi-tier pattern / 多层模式
    - Competence center pattern / 能力中心模式
    - Platform pattern / 平台模式