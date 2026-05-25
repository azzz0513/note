## Introduction
### Validation
- **Validation** : The process of evaluating software at the end of software development to ensure **compliance with intended usage**.
- 确认：在软件开发结束时评估软件以确保**符合预期用途**的过程。
- 换句话说：我们生产的产品是正确的吗？（强调结果）

### Verification
- **Verification** : The process of determining whether the products of a given phase of the software development process **fulfill the requirements** established during the previous phase.
- 验证：确定软件开发过程给定阶段的产品是否**满足上一阶段建立的需求**的过程。
- 换句话说：我们正确地生产了产品吗？（强调过程）

### Specification（规范）
- Specifications play a key role. / 规范起着关键作用
- Detailed specifications provide the correct behavior of the software. / 详细规范提供了软件的正确行为
- They must describe normal and error behavior. / 它们必须描述正常和错误行为

#### **==规范和BUG的关系==**
The software does not do something that the specification says it should do.
The software does something that the specification says it should not do.
The software does something that the specification does not mention.
The software does not do something that the product specification does not mention but should.
The software is difficult to understand, hard to use, slow ...
规范应该做的没做，不该做的做了，未提及的做了，默认做的没做
软件太慢太难用/理解

### 如何区分Verification（验证）与Validation（确认）
Verification 和 Validation 虽然都以 "V" 开头，但核心差异在于**焦点和目标**。以下是基于 ISO 标准和实际场景的清晰区分方法：
#### 核心区别：过程vs结果

| **维度**   | **Verification（验证）**  | **Validation（确认）**    |
| -------- | --------------------- | --------------------- |
| **焦点**   | **过程正确性**（是否按规范做）     | **结果正确性**（是否满足用户需求）   |
| **问题**   | “我们是否正确地构建了产品？”       | “我们构建的是正确的产品吗？”       |
| **验证对象** | 设计规范、代码逻辑、技术文档等       | 用户实际需求、场景适用性          |
| **阶段**   | **开发过程中**（如代码审查、单元测试） | **开发完成后**（如用户测试、临床评估） |
| **类比**   | 按菜谱检查食材用量和火候          | 品尝菜品是否符合食客口味          |

### 测试公理（Software Testing Axioms）
![[images/Pasted image 20260525153830.png]]
