# Skill可复用技能包
用 MewCode 久了，你会发现自己在反复向 Agent 解释同样的事情，让它帮忙做代码审查，你得说一遍团队的审查标准、要重点关注哪些维度、报告格式是什么样的，让它帮忙搭一个前端页面，你得重新描述一遍项目的组件结构、状态管理方案、样式规范，让它帮忙写测试，你得再交代一次用的是 Vitest 还是 Jest、偏好集成测试还是单元测试、mock 策略是什么。

​这些上下文每次都一样，但每个新会话都得重新说，因为 Agent 不会记得上一次你怎么交代的，这是第一个痛点：重复解释，同样的偏好和流程反复输入，浪费时间还容易遗漏。

​​第二个痛点更隐蔽，回头看看 MEWCODE.md，随着使用时间变长，你会往里面塞越来越多的东西：代码审查的标准、commit message 的格式、部署流程、API 调用的注意事项。这些内容从「事实描述」慢慢长成了「操作流程」，MEWCODE.md 变成了一个大杂烩，问题是它每一轮对话都会完整加载，你只是想问一个简单问题，那段 200 行的部署流程也跟着进了上下文，白白消耗 token。​​

第三个痛点是知识流失，团队踩过的坑往往散落在各人脑子里：某个内部 API 有个不写在文档里的限速规则，deploy 脚本在周五下午跑会触发告警，代码审查要特别留意某个历史遗留模块的并发问题，这些经验 Agent 根本不知道，每次让它帮忙你都得手动把相关上下文贴进去。​​

Skill 系统一并解决这三个问题：把重复的偏好和流程打包成独立的 Markdown 文件，只在需要时加载不浪费 token，团队的专业知识变成可版本控制、可分发的资产。

## Skill是什么：写给Agent的SOP
SOP（Standard Operating Procedure），就是标准操作流程。新人入职，给他一份SOP：`当你要部署时，按照1、2、3步骤来`。

Skill 就是写给 Agent 的 SOP。​

它告诉 Agent：「当用户要求做提交/审查/测试时，按照这个流程和标准来执行」，区别在于，SOP 给人看，人来执行，Skill 给 Agent 看，Agent 来执行。​

但有一点很关键：人类可以「领会精神」，你写得模糊一点，有经验的人也能理解你的意思。Agent 不行，它只会照字面做，你写「注意安全」，人知道要检查 SQL 注入和 XSS，你写「注意安全」给 Agent，它可能只会在回复里加一句「已注意安全问题」就完事了，所以 Skill 里的指令需要比人类 SOP 更精确、更具体。​​

Anthropic 把 Skill 定位成一个开放标准（Agent Skills），核心理念是知识固化和跨平台复用：团队踩过的坑、定下的规范不再靠口头传授，一份 SKILL.md 可以在 Claude.ai、Claude Code、API 甚至其他 AI 工具里共用。MewCode 的 Skill 设计沿用同样的思路，把可复用的操作流程和专业知识从对话里抽出来，变成可编辑、可分发的 Markdown 资产。

## Slash Command和Skill的分工
上一章实现的 Slash Command 分成两大类：`local` 和 `local-ui` 是纯本地操作，`/help` 查帮助、`/clear` 清屏、`/plan` 切模式，不走 LLM 不消耗 token，确定性百分之百，按下去就执行；`prompt` 类型则会把一段预设的 prompt 转发给 Agent 处理，`/review` 就是它的代表。​

Skill 和 `prompt` 类 Slash Command 看起来很像，都是「把一段写好的 prompt 交给 Agent 执行」。区别在于 Skill 多了三个能力。​

第一是 **Agent 能主动发现**。Slash Command 只能用户手动输入 `/xxx` 触发，Skill 可以被 Agent 根据用户意图自动匹配和加载，用户不需要记住有哪些 Skill。​

第二是**可以携带整套资源**。`prompt` 类 Slash Command 就是一段孤零零的字符串，Skill 是一个目录，可以装参考文档、示例文件、辅助脚本，Agent 按需读取。​

第三是 **inline / fork 两种执行模式**。Slash Command 一律在当前对话里执行，Skill 可以选择在独立上下文中执行，适合需要客观判断的场景（比如代码审查不应该受之前对话的影响）。

​简单判断：不需要 AI 参与的确定性操作用 Slash Command，需要 AI 判断且流程可复用的任务用 Skill，实际上在 Claude Code 的最新版本中，自定义命令已经和 Skill 合并成了同一套机制。

## Markdown就是定义文件
Skill的定义格式是Markdown，因为它同时满足三个需求：人类可以直接阅读，系统可以通过YAML frontmatter解析结构化数据，LLM可以直接理解prompt body中的指令。

一个完整的Skill文件长这样：
![[Pasted image 20260727214502.png]]
上半部分是 YAML frontmatter，即两个 `---` 之间的内容，定义了 Skill 的元信息。下半部分是 prompt body，就是实际发送给 LLM 的指令。

​这意味着什么？你要调整 commit message 的格式规范，打开这个 `.md` 文件改几行文字就行了，不用碰源代码，不用重新编译。你想加一个新 Skill，创建一个 `.md` 文件就行了。

## Frontmatter里的每个字段
YAML frontmatter 是 Skill 的「身份证」，让我们逐个看看每个字段是干什么的。

​`name` 是必填的，它是 Skill 的唯一标识符，同时也是调用它的命令名。`name: commit` 意味着用户可以通过 `/commit` 来调用这个 Skill。命名规范是小写字母、数字和连字符。注意不能和内置 Slash Command 冲突，你不能定义一个 `name: help` 的 Skill，因为 `/help` 已经被内置命令占了。​

`description` 也是必填的，一句话描述 Skill 的功能。它会出现在 `/help` 列表里，也用于自动匹配。​

`model` 是可选的，指定 Skill 使用的模型。有些任务简单（比如生成 commit message），可以用便宜一点的模型；有些任务复杂（比如代码审查），需要最强的模型。不指定就用默认模型。​

`mode` 是可选的，控制 Skill 的执行模式：`inline`（默认）或 `fork`。这个区别很重要，后面会专门讲。

​`context` 也是可选的，只在 fork 模式下生效，决定把多少主对话的上下文带进 fork 会话。可以是 `full`（完整对话的摘要，默认）、`recent`（最近 5 条消息）、`none`（完全隔离）。inline 模式本身就共享对话历史，这个字段会被忽略。

## 三个地方找Skill，谁优先级高
Skill文件可以放在三个位置，按优先级从高到低：
![[Pasted image 20260727214813.png]]
这个设计你应该不陌生，跟 npm 的包搜索路径很像：先看项目本地，再看全局，最后看内置。也跟 Git 的 `.gitconfig` 一样：项目级配置覆盖全局配置。

​同名 Skill 高优先级覆盖低优先级，这意味着几件很实用的事情。

​项目可以定义项目特有的 Skill，比如你的项目有自己的 deploy 流程，写一个 `deploy.md` 放在 `.mewcode/skills/` 下面就行。​

用户可以定义个人通用的 Skill。你喜欢 commit message 用中文？写一个自己版本的 `commit.md` 放在 `~/.mewcode/skills/` 下面，它会覆盖内置的英文版。

​项目级 Skill 可以提交到版本控制，整个团队共享。团队的最佳实践从口头传授变成了一个个可执行的 Skill 文件。新人入职，clone 仓库，Skill 就自动加载了。

​MewCode 自带几个内置 Skill，开箱即用，不需要用户手动安装。

## inline vs fork：要不要共享上下文
Skill有两种执行模式，这是理解Skill系统最重要的概念之一。

先说`inline`模式，它是默认模式。Skill的prompt被注入到当前对话中，和正常的用户消息一样走Agent Loop。Skill可以看到之前的对话上下文，执行结果也会留在对话历史中。
![[Pasted image 20260727215104.png]]
为什么 `/commit` 适合 inline？因为 Agent 可能在前面的对话中已经帮你做了一些代码修改，Skill 需要知道这些修改的上下文才能正确判断哪些文件该 commit。​

再说 `fork` 模式。Skill 在一个独立的上下文中执行，不影响也不受当前对话影响。就像开了一个新的 Agent 会话，执行完后只把结果摘要返回到主对话。
![[Pasted image 20260727215220.png]]
为什么 `/review` 适合 fork？想想看，代码审查应该是客观的。如果 Agent 之前在对话里说了「我觉得这个实现挺好的」，在 inline 模式下这句话会影响审查结果，Agent 可能倾向于给出更正面的评价。fork 模式隔离了上下文，审查结果更客观。​

在 frontmatter 中指定执行模式很简单：
![[Pasted image 20260727215259.png]]
fork 模式可以通过 `skill.context` 控制带多少主对话的上下文进去。​

代码审查这种需要客观判断的场景用 `none` 最稳妥；面试模拟这种需要看到候选人简历摘要的场景用 `full`；想要带一点背景但又不想被太多对话历史污染的场景用 `recent`。Skill 作者根据任务性质自己挑。

## 自动注册为命令：Skill和Slash Command对用户透明
Skill加载后会自动注册为Slash Command。比如MewCode自带的`skill-creator`，安装好就能通过`/skill-creator`调用，出现在`/help`列表，支持Tab补全，和内置命令的体验完全一致。

## 意图识别：让Agent自己选Skill
`/commit`这种显式调用很直观，但有个限制：用户得记住有哪些Skill、对应什么命令。如果用户说"帮我提交一下"或者`我想做个后端面试准备`，Agent能不能自动识别意图，主动加载对应的Skill？

### 两阶段加载
关键思路是把Skill的加载拆成两个阶段

**第一阶段：轻量注册**。MewCode启动时只加载每个Skill的frontmatter（name、description），不加载完整的prompt body。这些轻量信息作为系统提示的一段注入，告诉Agent有哪些Skill可用：
![[Pasted image 20260727215911.png]]

**第二阶段：按需加载**。当 Agent 判断用户意图匹配某个 Skill 时，调用 `LoadSkill` 工具，把 SKILL.md 的完整 SOP 加载到对话中。模型在下一轮迭代时就能看到完整的 SOP 指令，跟着执行。LoadSkill 是只读操作，不会触发权限确认，和 ReadFile、Grep 走同一条通道。
​
这就是渐进式披露的完整实现。Agent 平时只看到 Skill 的名称和描述，选择压力很低。一旦确定要用某个 Skill，才加载完整 SOP，注意力集中在当前任务上。

### 目录型Skill
前面介绍的单文件Skill只有一个Markdown文件，适合简单的流程指令。但有些Skill需要携带更多资源：模板文件让Agent填充、示例输出展示期望格式、辅助脚本让Agent通过Bash执行、长文档作为参考资料。一个文件装不下这些东西。所以Skill演化为了一个目录：
![[Pasted image 20260727220233.png]]
`SKILL.md` 是入口，角色和单文件 Skill 一致。目录里的其他文件是附属资源，Agent 通过已有的内置工具（ReadFile、Bash）来访问它们。SKILL.md 里需要引用这些文件，告诉 Agent 每个文件是什么、什么时候该去读。

​和单文件 Skill 的区别在于：单文件 Skill 只有 SOP 指令本身，目录型 Skill 可以携带模板、示例、脚本和参考文档。这些附属文件不会在加载 Skill 时全部塞进上下文，Agent 按需读取，只有真正需要时才消耗 token。

​目录型 Skill 还带来另一个常被低估的好处：**整套能力可打包移植**。整个 `backend-interview/` 目录是一个自包含单元，SOP、示例、脚本、参考文档全在里面。​

同事写好的 Skill 可以推到 GitHub 仓库或者打成 zip 分发，新人 clone 或解压下来扔进 `~/.mewcode/skills/`，下次启动 MewCode 就能直接用 `/backend-interview` 调用。团队内部沉淀的最佳实践不再散落在各人的 prompt 收藏夹里，变成可被代码评审、可被打版本号、可在 CI 里自动校验的一份配置资产。

## 完整流程
![[Pasted image 20260727220425.png]]
两种触发方式并存：用户可以直接打`/commit`显式调用，也可以用自然语言描述需求让Agent自动匹配。无论哪种方式触发，后面的执行流程是一样的。

