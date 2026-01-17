# Vibeloop 系统设计文档 (Design)

## 1. 系统架构 (System Architecture)

### 1.1 物理层拓扑: The Hive Cluster

系统运行在单机宿主环境上，利用文件系统隔离和容器化技术构建“蜂巢”结构。

*   **Host Environment (Orchestrator Node)**
    *   运行环境: Python / Node.js 进程
    *   职责: 运行 Architect Agent，管理 Docker 守护进程，操作 Git 命令。
*   **Storage (Persistent Volume)**
    *   由一个 Git 仓库根目录构成。
    *   作为所有 Worktree 的 Source of Truth。
*   **Isolation Units (Worktrees)**
    *   利用 `git worktree` 技术。
    *   **Mapping**: `Worktree #N` <-> `Branch #N` <-> `Docker Container #N`
    *   优势: 毫秒级环境切换，物理文件隔离，共享对象库 (.git/objects)。

### 1.2 逻辑实体: Agents

*   **The Architect**: 全局状态管理者。不写代码，只写指令。
*   **The Breaker**: 专业的破坏者。输入 Spec，输出 Test Cases。
*   **The Builder**: 坚韧的实施者。输入 Spec + Test Logs，输出 Code。
*   **The Critic**: 审美裁判。输入 Code，输出 Review/Veto。

## 2. 核心机制 (Core Mechanisms)

### 2.1 蜂巢工作流协议 (Hive Workflow Protocol)

工作流遵循 **Feature Branch -> Validation -> Rebase** 模式。

1.  **Initialization**: Architect 解析 Ticket，生成 `feat/X` (给 Builder) 和 `test/X` (给 Breaker)。
2.  **Parallel Execution**:
    *   Worktree A (Docker A): Breaker 编写测试用例。
    *   Worktree B (Docker B): Builder 编写业务逻辑。
3.  **The Encounter (交汇)**:
    *   Architect 将 `test/X` 的产物注入 Worktree B。
    *   在 Docker B 中运行测试脚本。
4.  **Feedback Loop**:
    *   **Fail**: Logs 返回给 Builder -> 重试 (goto 2)。
    *   **Pass**: 进入 Critic 审查。
5.  **Review & Merge**:
    *   Critic 审查通过 -> Architect 执行 Rebase `feat/X` to `main`。
    *   如有冲突 -> 触发 Conflict Resolution Protocol。

### 2.2 冲突解决策略 (Conflict Resolution)

*   **Strategy A (Auto-Fix)**:
    *   针对: 空白符、Import 排序、格式化差异。
    *   工具: Linter/Formatter 自动修复。
*   **Strategy B (Intelligent Merge)**:
    *   针对: 逻辑冲突。
    *   机制: Architect 创建 `merge-resolve-tmp` Worktree，调用高级 Agent 尝试语义合并。
*   **Strategy C (Abort & Retry)**:
    *   针对: 无法自动解决的深层逻辑冲突。
    *   机制: 标记 Feature 分支 `FAILED`，将其 Ticket 重新放回任务队列尾部，等待串行调度。

### 2.3 动态快照与时间锚点

*   **Snapshots**:
    *   每次成功 Merge 后自动打 Tag (`auto-vX.Y.Z`).
    *   更新 `LATEST_STABLE` 指针。
*   **Intervention (The Oracle)**:
    *   人类通过 Dashboard 注入新的 Spec 或 Feedback。
    *   支持 `System Reset`: 强制回退 Git HEAD 到指定 Tag，清理所有脏 Worktree。

## 3. 数据结构 schemas

### 3.1 GlobalState (内存对象)

```yaml
GlobalState:
  project_root: "/app/repo"
  active_spec_hash: "sha256_of_current_spec"
  worktrees:
    - id: "wt_01"
      path: "/app/repo/worktrees/wt_01"
      branch: "feat/login"
      agent_role: "Builder"
      status: "Coding" # Coding, Testing, Idle, Locked
      docker_container_id: "container_7f8a9b"
  task_queue:
    - id: 101
      type: "Feature"
      priority: High
      status: "Pending" # Pending, In-Progress, Review, Done
  history_log: []
```

### 3.2 TaskTicket (任务描述符)

```json
{
  "ticket_id": "T-1024",
  "type": "Feature",
  "goal": "Implement User Login API",
  "constraints": {
    "read_only_paths": ["tests/", "docs/", "spec/"],
    "timeout_seconds": 600
  },
  "context_refs": [
    "src/utils/auth.py",
    "docs/api_spec.md"
  ],
  "vibe_config": {
    "linter": "flake8",
    "complexity_limit": 10
  }
}
```

## 4. 风险防御 (Risk Mitigation)

| 风险点 | 防御机制 |
| :--- | :--- |
| **Spec Drift (规范漂移)** | 文件系统权限锁 (Read-only mounts for tests/spec); 合并前 Hash 校验。 |
| **Infinite Loops (死循环)** | Docker 资源硬限制 (CPU/RAM); 任务级 Max Retries 熔断机制。 |
| **Model Collapse (模型坍塌)** | 温度扰动 (Temperature Perturbation) in Retries; 异构模型对抗 (Sonnet vs GPT-4)。 |

