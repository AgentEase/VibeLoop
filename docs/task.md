# Vibeloop Development Tasks

This document outlines the granular development tasks for the Vibeloop project, based on `requirements.md` and `design.md`.

## Group 0: Initialization & Infrastructure
*Core foundation for the Orchestrator and Docker environment.*

- [ ] **Project Setup & Repository Initialization**
    - **Description**: Initialize the project structure and Git repository.
    - **Steps**:
        1. Initialize `vibeloop` git repo.
        2. Set up Python/Node.js environment (Orchestrator).
        3. Define `requirements.txt` or `package.json`.
        4. Create directory structure (`src`, `tests`, `docs`, `config`).
    - **Priority**: High
    - **Role**: User

- [ ] **Docker Base Image Creation**
    - **Description**: Create the standard Docker image for Agent execution environments.
    - **Steps**:
        1. Write `Dockerfile`.
        2. Include necessary compilers/interpreters (Python, Node, etc.).
        3. Include Linter/Test runners.
        4. Build and verify image `vibeloop-base`.
    - **Priority**: High
    - **Role**: AI

- [ ] **Define Data Structures (Schema)**
    - **Description**: Implement the core data structures defined in Design 3.0.
    - **Steps**:
        1. Create `GlobalState` class/schema.
        2. Create `TaskTicket` class/schema.
        3. Create `WorktreeMetadata` schema.
    - **Priority**: High
    - **Role**: Backend

## Group 1: Phase 1 - The Nucleus (Single-threaded MVP)
*Goal: Establish the basic "Generate -> Run -> Feedback -> Fix" loop.*

- [ ] **Implement Builder Agent**
    - **Description**: Create the LLM wrapper for the Builder role.
    - **Steps**:
        1. define system prompt for Builder (implementing code from Spec).
        2. Implement `generate_code(spec, feedback)` function.
        3. Handle file writing operations.
    - **Priority**: High
    - **Role**: AI

- [ ] **Implement Breaker Agent**
    - **Description**: Create the LLM wrapper for the Breaker role.
    - **Steps**:
        1. Define system prompt for Breaker (generating adversarial tests).
        2. Implement `generate_tests(spec)` function.
    - **Priority**: High
    - **Role**: AI

- [ ] **Implement Basic Executor**
    - **Description**: A simple module to run code and tests locally or in a single container.
    - **Steps**:
        1. Implement `run_test(code_path, test_path)`.
        2. Capture `stdout`, `stderr`, and Exit Code.
        3. Return structured ExecutionResult.
    - **Priority**: High
    - **Role**: Backend

- [ ] **Implement The "Nucleus" Loop Logic**
    - **Description**: Orchestrate the single-threaded loop.
    - **Steps**:
        1. Input: Simple Spec (e.g., FizzBuzz).
        2. Step 1: Breaker generates tests.
        3. Step 2: Builder generates code.
        4. Step 3: Executor runs tests.
        5. Step 4: If fail, feed output to Builder (Retry).
        6. Verify "FizzBuzz" completion.
    - **Priority**: High
    - **Role**: Backend

## Group 2: Phase 2 - The Hive (Multi-threaded & Isolation)
*Goal: Introduce Git Worktree, Docker Clusters, and the Architect.*

- [ ] **Implement Worktree Manager**
    - **Description**: Python/Node wrapper for `git worktree` operations.
    - **Steps**:
        1. `create_worktree(branch, path)`
        2. `remove_worktree(path)`
        3. `list_worktrees()`
        4. Ensure clean cleanup on exit.
    - **Priority**: High
    - **Role**: Backend

- [ ] **Implement Docker Container Manager**
    - **Description**: Manage container lifecycle mapped to Worktrees.
    - **Steps**:
        1. `spawn_container(worktree_path, mounts)`
        2. `kill_container(id)`
        3. Implement Resource Limits (R9.1: CPU/RAM).
    - **Priority**: High
    - **Role**: Backend

- [ ] **Implement The Architect Agent**
    - **Description**: The high-level decision maker and orchestrator.
    - **Steps**:
        1. Define Architect system prompt (Task breakdown).
        2. Implement `dispatch_task(ticket)`: Assigns to Worktree.
        3. Implement `manage_lifecycle()`: Monitors container status.
    - **Priority**: High
    - **Role**: Backend/AI

- [ ] **Implement Parallel Workflow (The Hive Protocol)**
    - **Description**: Enable simultaneous Feature and Test branch development.
    - **Steps**:
        1. Architect spawns Worktree A (Feature) & Worktree B (Test).
        2. Run Builder in A, Breaker in B concurrently.
        3. **Injection**: Copy tests from B to A.
        4. Execute validation in A.
    - **Priority**: High
    - **Role**: Backend

- [ ] **Implement Merge & Rebase Logic**
    - **Description**: Handle the atomic merging of features.
    - **Steps**:
        1. Implement "Squash & Rebase" strategy.
        2. Handle basic git conflicts (Strategy A: Auto-Fix).
        3. Update `LATEST_STABLE` pointer.
    - **Priority**: Medium
    - **Role**: Backend

## Group 3: Phase 3 - The Critic (Quality Assurance)
*Goal: Add Style Review, Static Analysis, and "Vibe Check".*

- [ ] **Integrate Static Analysis Tools**
    - **Description**: Setup Linters/Formatters (R5.1).
    - **Steps**:
        1. Configure `flake8` / `eslint`.
        2. Implement `run_linter(path)` in Executor.
        3. Define "Hard Fail" rules (AST compliance).
    - **Priority**: Medium
    - **Role**: Backend

- [ ] **Implement Critic Agent**
    - **Description**: The Code Reviewer Agent.
    - **Steps**:
        1. Define System Prompt for Critic (Style, Complexity, Best Practices).
        2. Input: Diff + Code + Linter Result.
        3. Output: Approve / Veto + Comments.
    - **Priority**: Medium
    - **Role**: AI

- [ ] **Implement "Veto" Workflow**
    - **Description**: Enforce Critic's decision in the pipeline.
    - **Steps**:
        1. Add Gate: After Tests Pass, trigger Critic.
        2. If Veto: Send feedback to Builder -> Retry (Loop).
        3. If Approve: Proceed to Merge.
    - **Priority**: Medium
    - **Role**: Backend

## Group 4: Phase 4 - The Interface (Human Interaction)
*Goal: Dashboard, Oracle Injection, and Safety Controls.*

- [ ] **Implement Global Dashboard API**
    - **Description**: Backend API to expose system state.
    - **Steps**:
        1. Expose `GlobalState` via HTTP/WebSocket.
        2. Show active Worktrees, moving tickets, success rates.
    - **Priority**: Low
    - **Role**: Frontend/Backend

- [ ] **Implement Console/Web UI**
    - **Description**: Simple visualization (R7.1).
    - **Steps**:
        1. Build simple Frontend (React/Vue or Terminal UI).
        2. Display real-time logs from Agents.
    - **Priority**: Low
    - **Role**: Frontend

- [ ] **Implement Oracle Injection Mechanism**
    - **Description**: Allow human intervention (R7.2).
    - **Steps**:
        1. Create "Oracle" Ticket Type (Priority P0).
        2. Implement mechanism to pause/preempt current queue.
        3. Allow editing active Spec.
    - **Priority**: Medium
    - **Role**: Backend

- [ ] **Implement Time Travel (Reset)**
    - **Description**: System-wide reset capability (R7.3).
    - **Steps**:
        1. Implement `hard_reset(tag_name)`.
        2. Kill all containers, remove all worktrees.
        3. `git reset --hard` to target tag.
    - **Priority**: Low
    - **Role**: Backend

## Group 5: Safety & Robustness (Cross-Cutting)
*Enhancements strictly required by Non-Functional Requirements.*

- [ ] **Implement File Usage Locks**
    - **Description**: Enforce Read-Only paths (R8.1).
    - **Steps**:
        1. Configure Docker Mounts: `tests/` and `docs/` as Main-Read-Only for Builder.
        2. Verify file permissions on startup.
    - **Priority**: High
    - **Role**: DevOps/Backend

- [ ] **Implement Spec Integrity Check**
    - **Description**: Anti-Drift mechanism (R8.2).
    - **Steps**:
        1. Hash Spec file at start of task.
        2. Verify Hash before Merge.
    - **Priority**: Medium
    - **Role**: Backend

- [ ] **Implement Timeout & Circuit Breaker**
    - **Description**: Prevent zombie processes (R9.2, R9.3).
    - **Steps**:
        1. Add strict timeouts to Docker `exec`.
        2. Implement "Max Retry" counter in `TaskTicket`.
        3. If > 10 retries, mark Task FAILED.
    - **Priority**: High
    - **Role**: Backend
