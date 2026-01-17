# Vibeloop 项目需求文档 (Requirements)

## 1. 项目概述

**项目名称**: Vibeloop
**核心愿景**: 构建一个基于 Git Worktree 和 Docker 容器集群的、具备“生成-对抗-评估”三元机制的自治编程系统。
**核心理念**: 移除人类作为实时的 "Ground Truth"，建立依赖客观标准（编译状态、对抗性测试、静态分析）的“人造真理”系统。

## 2. 设计哲学与核心原则

### 2.1 人造真理 (Synthetic Ground Truth)
系统必须依赖以下绝对客观标准，而非人类的主观判断：
*   **编译/解释器状态**: Exit Code 0 是最低标准。
*   **对抗性测试集**: 代码必须通过由 "Breaker" Agent 生成的对抗性测试用例。
*   **静态分析 (Linting)**: 代码必须符合 AST 层面的硬性规则。

### 2.2 熵减系统 (Entropy Reduction)
系统设计必须致力于控制自动化迭代产生的熵增（混乱、幻觉、死循环）。
*   **原子化提交**: 任务必须拆解为微小的、可验证的提交。
*   **频繁回滚**: 遇到不可修复的错误或死循环时，系统应能快速强制回滚 (Reset)。
*   **严格隔离**: 利用 Worktree 和 Docker 进行文件系统和运行时环境的隔离。

### 2.3 异步上帝视角 (Asynchronous Divinity)
人类角色从“驾驶员”转变为“造物主”。
*   **不干涉运行时**: 人类不直接编写代码或干涉具体的时间片。
*   **宏观调控**: 人类只通过修改 Spec/Rules 物理常数或执行时间回溯（Git Reset）来影响系统。

## 3. 功能需求

### 3.1 物理层架构 (The Hive Cluster)
*   **R1.1 宿主环境**: 系统需运行在 Python/Node.js 驱动的 Orchestrator 进程中。
*   **R1.2 存储管理**: 持久化存储单一 Git 仓库根目录。
*   **R1.3 隔离单元**: 使用 `git worktree` 在同一物理磁盘上映射多个逻辑工作区，每个工作区绑定独立的 Docker 容器实例。

### 3.2 逻辑层 Agent 角色
系统禁止 Agent 间自然语言直接对话，必须通过结构化协议交互。

#### 3.2.1 The Architect (规划者)
*   **R2.1 权限管理**: 拥有最高权限，负责管理 Git 分支、分配 Worktree、启动/销毁 Docker、合并代码。
*   **R2.2 任务编排**: 维护全局状态机，解析 Spec，将大任务拆解为 Ticket，裁决冲突。
*   **R2.3 异常处理**: 处理死循环、超时等系统级异常。

#### 3.2.2 The Breaker (破坏者)
*   **R3.1 对抗测试**: 仅根据 Spec 生成极端的单元测试和边界条件测试。
*   **R3.2 独立性**: 目标是击溃 Builder 的代码，信条是“未考虑的空输入是 Builder 的错”。

#### 3.2.3 The Builder (建设者)
*   **R4.1 代码实现**: 根据 Spec 和 Breaker 提供的失败 Test Log 编写/修改源代码。
*   **R4.2 标准遵循**: 必须致力于通过测试并满足 Critic 的代码风格要求。

#### 3.2.4 The Critic (审查者)
*   **R5.1 质量保证**: 进行 Code Review，检查 Linter Output 和圈复杂度。
*   **R5.2 否决权**: 拥有对不符合风格（Vibe）代码的强制否决权，即使功能测试通过。

### 3.3 核心工作流 (Hive Workflow)
*   **R6.1 并行开发**: 支持 Architect 同时创建 Feature 分支和 Test 分支。
*   **R6.2 交汇验证**: 实现将 Test 分支的测试代码注入 Feature 分支进行验证的机制。
*   **R6.3 循环反馈**: 建立“失败 -> Log 反馈 -> 修改 -> 重试”的自动循环。
*   **R6.4 原子合并**: 验证通过后，自动将 Feature 分支 Rebase 到主分支。

### 3.4 人类交互接口
*   **R7.1 仪表盘**: 提供简单的 Dashboard 显示当前 Spec、最新 Tag 和运行状态。
*   **R7.2 神谕注入 (Oracle Injection)**: 允许用户以 P0 级任务插入反馈或修改 Spec。
*   **R7.3 时间回溯**: 允许用户指令系统硬重置到指定的 Git Tag（如 `auto-v0.1.0`）。

## 4. 非功能需求

### 4.1 安全性与隔离
*   **R8.1 文件权限锁**: Builder 只能读取 `tests/` 和 `docs/`，禁止修改 Spec 和测试用例。
*   **R8.2 Spec完整性**: 合并前必须校验 Spec 文件的 Hash，防止漂移。

### 4.2 鲁棒性与资源控制
*   **R9.1 资源限制**: 每个 Docker 容器需限制 CPU 和 RAM 使用量（例如 1 CPU / 512MB RAM）。
*   **R9.2 超时机制**: 运行测试或任务执行需有超时自动 Kill 机制。
*   **R9.3 最大重试**: 单一任务迭代超过阈值（如 10 次）未通过，需强制终止并报告路径不可行。

### 4.3 模型多样性
*   **R10.1 温度控制**: 重试时动态调整 LLM Temperature。
*   **R10.2 异构模型**: 建议 Breaker 和 Builder 使用不同能力特性的模型以增强对抗性。

## 5. 实施路线图

*   **Phase 1: The Nucleus (单线程 MVP)**
    *   目标: 跑通 Generate -> Run -> Feedback -> Fix 循环。
    *   验证: 独立完成 FizzBuzz 或简单 CRUD。
*   **Phase 2: The Hive (多线程 Alpha)**
    *   目标: 引入 Worktree 和 Architect，支持并发开发和 Rebase。
    *   验证: 并行派发不冲突任务并成功合并。
*   **Phase 3: The Critic (全功能 Beta)**
    *   目标: 引入 Critic 和 Vibe Check，集成 Linter。
    *   验证: 系统拒绝功能正确但风格糟糕的代码。
*   **Phase 4: The Interface (异步交互)**
    *   目标: 构建人类控制台和神谕注入机制。
    *   验证: 运行时动态修改 Spec 并生效。
