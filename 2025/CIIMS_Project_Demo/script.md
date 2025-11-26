# CIIMS Smart Inventory Management System Script

各位技术同仁，大家好。今天我为大家介绍 CIIMS，校园智能库存管理系统的技术架构。
Hello everyone, my fellow tech colleagues. Today I'm here to introduce the technical architecture of CIIMS, the Campus Intelligent Inventory Management System.

技术栈采用 PySide6 构建桌面前端，MySQL 作为数据存储，核心亮点是嵌入式 AI 助手。系统定位很明确，是基于 AI 的桌面端库存管理解决方案。核心模块包括登录认证、主控制台、借阅归还、以及 AI 助手四大模块。整个系统采用单体应用架构，确保部署简单、维护成本低。
The tech stack uses PySide6 to build the desktop frontend, MySQL for data storage, with the core highlight being an embedded AI assistant. The system's positioning is clear - it's an AI-based desktop inventory management solution. Core modules include login authentication, main console, borrowing/returning, and AI assistant. The entire system adopts a monolithic application architecture, ensuring simple deployment and low maintenance costs.

接下来，请看系统的实际运行演示。
Next, let's watch the actual system demonstration.

系统采用轻量化启动，支持账号密码及人脸识别双重认证，确保校园资产管理的安全性与便捷性。进入用户端，核心借阅流程极度简化。用户只需选择物品、设定归还日期，即可一键完成借阅申请，交互逻辑清晰直观。归还操作同样高效。系统实时同步库存状态，确保每一次资产流转都有据可查，实现了借还业务的数字化闭环。在个人中心，用户可实时查看借阅历史与逾期状态，并支持人脸数据的自助更新。
The system features lightweight startup, supporting dual authentication with password and facial recognition to ensure security and convenience for campus asset management. In the user interface, the core borrowing process is extremely simplified. Users only need to select items, set return dates, and complete borrowing applications with one click, with clear and intuitive interaction logic. Return operations are equally efficient. The system synchronizes inventory status in real-time, ensuring every asset transfer is traceable, achieving a digital closed loop for borrowing and returning. In the personal center, users can view borrowing history and overdue status in real-time, and support self-service facial data updates.

这是系统的核心亮点，CIMS Sherlock 智能助手。区别于传统搜索，它支持自然语言交互。无论是询问你是谁，还是模糊查询现在有什么书可以借，AI 都能通过意图识别与数据库检索，精准返回带有库存数量的实时结果，极大降低了用户的检索门槛。
This is the system's core highlight, the CIMS Sherlock intelligent assistant. Unlike traditional search, it supports natural language interaction. Whether asking 'who are you' or making fuzzy queries like 'what books are available to borrow now', the AI can accurately return real-time results with inventory quantities through intent recognition and database retrieval, greatly lowering the user's search threshold.

切换至管理员视角。我们提供了高效的库存录入功能，支持自定义分类与批量管理，快速响应物资入库需求。在管理面板，管理员可对全校成百上千件资产进行可视化查阅与编辑。同时，集成了完整的用户权限管理与全校借阅记录审计，数据一目了然。AI 助手同样赋能于管理端。面对复杂的数据查询需求，比如列出所有用户信息，管理员无需编写 SQL，只需一句指令，AI 即可自动调取底层数据并整理呈现。CIIMS，让校园库存管理更智能，更高效。
Switching to the administrator perspective. We provide efficient inventory entry functionality, supporting custom classification and batch management to quickly respond to material storage needs. In the management panel, administrators can visually view and edit hundreds or thousands of campus assets. Additionally, it integrates complete user permission management and campus-wide borrowing record auditing, with data一目了然. The AI assistant also empowers the management side. When facing complex data query needs, such as listing all user information, administrators don't need to write SQL - just one command, and the AI can automatically retrieve and present the underlying data. CIIMS makes campus inventory management smarter and more efficient.

演示过后，让我们深入探讨一下背后的技术实现。
After the demonstration, let's dive deep into the technical implementation behind it.

AI 助手是系统的技术核心，采用嵌入式设计而非独立服务。让我重点讲一下请求生命周期：用户输入首先被捕获，然后调用模型进行意图分类。这里的关键是双路径路由：普通对话走通用通道，数据库查询走专用通道。整个流程基于 QThread 异步处理，通过 Signal Slot 机制确保 UI 响应性。这种设计避免了传统 CS 架构的网络延迟问题。
The AI assistant is the technical core of the system, using embedded design rather than independent services. Let me focus on the request lifecycle: user input is first captured, then the model is called for intent classification. The key here is dual-path routing: general conversations go through the general channel, database queries go through the dedicated channel. The entire process is based on QThread asynchronous processing, ensuring UI responsiveness through the Signal Slot mechanism. This design avoids the network latency issues of traditional CS architecture.

这是整个系统的技术难点。我们采用 LLM 进行意图分类，关键在于提示词工程。我们强制 LLM 返回结构化 JSON，包含意图类型、置信度和推理过程三个字段。置信度评分完全由 LLM 自评估，范围 0.0 到 1.0。当分类失败时，系统采用保守策略，默认回退到通用问答路径。这种设计确保了系统的鲁棒性，宁可答错，不能越权。值得注意的是，我们使用 response format 强制 JSON 输出，避免了自然语言解析的复杂性。
This is the technical challenge of the entire system. We use LLM for intent classification, with the key being prompt engineering. We force the LLM to return structured JSON containing three fields: intent type, confidence score, and reasoning process. The confidence score is completely self-assessed by the LLM, ranging from 0.0 to 1.0. When classification fails, the system adopts a conservative strategy, defaulting to the general Q&A path. This design ensures system robustness - better to answer incorrectly than to overstep authority. Notably, we use response format to force JSON output, avoiding the complexity of natural language parsing.

模糊处理是另一个技术亮点。传统关键词匹配无法理解语义，我们的方案是在提示词层面解决。比如用户说我想借书，系统会生成包含模糊匹配的 SQL。从自然语言到 SQL 的转换分为四步：上下文构建、SQL 提案生成、SQL 提取验证、以及安全执行。整个过程确保每一步都可控、可审计。实体抽取完全依赖 LLM 的语义理解，不需要额外的 NLP 库，降低了系统复杂度。
Fuzzy processing is another technical highlight. Traditional keyword matching cannot understand semantics, and our solution addresses this at the prompt level. For example, when a user says 'I want to borrow books', the system generates SQL with fuzzy matching. The conversion from natural language to SQL is divided into four steps: context construction, SQL proposal generation, SQL extraction and verification, and secure execution. The entire process ensures each step is controllable and auditable. Entity extraction completely relies on the LLM's semantic understanding, requiring no additional NLP libraries, reducing system complexity.

安全是数据库操作的底线。我们实现了三层防护：第一层，SQL 类型检查，仅允许 SELECT 语句；第二层，关键词拦截，DELETE、UPDATE 等操作直接拒绝；第三层，表级权限验证，普通用户无法访问 users 表。所有 SQL 执行前都会经过函数校验，任何越权请求都会被拦截并记录日志。这种接口级控制确保了数据安全。
Security is the bottom line for database operations. We implement three-layer protection: first layer, SQL type checking, only allowing SELECT statements; second layer, keyword interception, directly rejecting operations like DELETE, UPDATE; third layer, table-level permission verification, ordinary users cannot access the users table. All SQL executions undergo function validation before execution, and any unauthorized requests are intercepted and logged. This interface-level control ensures data security.

部署方面，AI 集成采用 OpenRouter 兼容协议，支持本地化部署。通信层使用 httpx 客户端，设置 60 秒超时和快速失败策略。这种设计既保证了响应速度，又避免了资源占用。
In terms of deployment, AI integration uses the OpenRouter compatible protocol, supporting localized deployment. The communication layer uses the httpx client with a 60-second timeout and fast-fail strategy. This design ensures both response speed and avoids resource occupation.

总结一下，这个架构的技术优势在于：嵌入式设计、双路径处理、严格安全验证。但我们也面临性能瓶颈，比如 LLM 调用延迟、同步 SQL 执行以及内存占用问题。下一步优化方向很明确：引入意图分类缓存、异步 SQL 执行和流式响应实现。
To summarize, the technical advantages of this architecture are: embedded design, dual-path processing, and strict security verification. However, we also face performance bottlenecks, such as LLM call latency, synchronous SQL execution, and memory usage issues. The next optimization directions are clear: introduce intent classification caching, asynchronous SQL execution, and streaming response implementation.

这个项目展示了如何将 AI 能力深度集成到传统桌面应用中，为智能化办公系统提供了新的技术思路。
This project demonstrates how to deeply integrate AI capabilities into traditional desktop applications, providing new technical approaches for intelligent office systems.

谢谢大家！
Thank you everyone!
