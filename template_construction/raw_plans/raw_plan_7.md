# 阶段七：Agent Runtime、Tool Runner 与 Hook Runner 规划（改进版）

> 目标（一句话）：在前六个阶段打好的骨架、知识、注册与脚手架基础之上，**完成系统级运行时能力的建设**——包括：统一的工具执行层（Tool Runner，替代 Ability Runtime，非常驻）、可复用的 Hook Runner 库，以及唯一常驻的 Agent Runtime（会话协调器），并定义它与外部 AI（Cursor / Claude 等）的协作契约。

---

## 1. 阶段定位与范围

### 1.1 在整体路线中的位置

- 阶段一：目录骨架与空文件清单  
- 阶段二：文档规范与 `AGENTS.md` 结构设计  
- 阶段三：核心机制知识与标准流程沉淀  
- 阶段四：各类注册的写入脚本与协同机制（Registry 层）  
- 阶段五：脚手架（Scaffolding）与 AI+人交互流程  
- 阶段六：实际注册一批知识 / 能力 / Hook，并用脚手架做小规模“实战”验证  

**阶段七**：在上述基础上，补齐“真正能跑”的**系统运行时层**，使得：

- ability_routing 中的“统一能力池 + 标准执行脚本”有对应的 Tool Runner 实现；  
- hooks 文档中定义的四类事件（PromptSubmit / PreAbilityCall / PostAbilityCall / SessionStop）通过 Hook Runner 统一调度；  
- Agent Runtime 作为唯一常驻进程，对外提供统一接口，对内协调 Hook Runner 与 Tool Runner 的调用。

### 1.2 本阶段“做 / 不做”

**本阶段要完成：**

1. 设计并实现 **Tool Runner**（工具运行库）：  
   - 以库/脚本形式暴露 `task.create / task.run / task.get / task.update / task.delete / direct_run`；  
   - 内部负责 PreAbilityCall / PostAbilityCall 事件的触发与调用 Hook Runner；  
   - 负责工具调用的统一 usage/telemetry 记录。  

2. 设计并实现 **Hook Runner**（Hook Engine）：  
   - 以库形式暴露 `run_hooks(event_type, context, blocking_only)`；  
   - 支持 4 类事件；  
   - 被 Agent Runtime 和 Tool Runner 复用调用，而不是单独常驻。  

3. 明确定义并实现 **Agent Runtime**：  
   - 作为唯一常驻进程，负责：  
     - 会话生命周期管理（Session Manager）；  
     - PromptSubmit / SessionStop 事件的触发与 Hook 调用；  
     - 将外部 AI 的“动作原语”（create/update/run task 等）转译为 Tool Runner 调用；  
     - 维护 session 内任务状态与 workdocs 更新；  
   - 定义其对外接口与内部组件结构。  

4. 在 Registry/Config 中完善能力级 Hook 配置方式：  
   - 不再使用模糊的 `guardrail` 字段；  
   - 使用 `hooks.pre_call` / `hooks.post_call` 显式绑定能力级 Pre/Post Hook。  

**本阶段不做：**

- 不强制引入消息队列或分布式任务系统；  
- 不要求实现完整的在线对话系统，只提供足够支撑 demo / 测试的最小 Agent Runtime；  
- 不把 Hook Runner 或 Tool Runner 实现为必须常驻的后台服务。

---

## 2. 角色与分层结构

### 2.1 三个核心“运行时角色”

1. **Tool Runner（工具执行库，替代 Ability Runtime）**  
   - 形式：非常驻库/脚本（例如 `scripts/runtime/tool_runner.py`）；  
   - 职责：统一执行能力/工具调用，提供任务生命周期接口；  
   - 内部：负责构造 PreAbilityCall / PostAbilityCall 上下文并调用 Hook Runner。  

2. **Hook Runner（Hook Engine）**  
   - 形式：非常驻库/脚本（例如 `scripts/runtime/hook_runner.py`）；  
   - 职责：解析 `.system/hooks/*.yaml`，按 event_type+match 规则调 handler，并合并 HookResult；  
   - 由 Agent Runtime / Tool Runner 在特定事件点调用。  

3. **Agent Runtime（会话协调器，唯一常驻进程）**  
   - 形式：长期运行的服务或进程；  
   - 职责：  
     - 管理会话（session）生命周期；  
     - 触发 PromptSubmit / SessionStop 事件并调用 Hook Runner；  
     - 对外暴露“动作原语”接口（LIST/CREATE/UPDATE/RUN/DELETE task 等）；  
     - 将外部 AI 的操作映射为 Tool Runner 调用，并负责写入 workdocs。  

### 2.2 会话级事件 vs 能力级事件

四类事件分为两类：

- **会话级事件（Session 级）**  
  - `PromptSubmit`：新任务/消息提交；  
  - `SessionStop`：一次会话或阶段结束。  
  - 触发方：Agent Runtime；  
  - Hook 配置：只在 `.system/hooks/*.yaml` 中声明为会话级 Hook。  

- **能力级事件（Ability 级）**  
  - `PreAbilityCall`：执行某个 ability 之前；  
  - `PostAbilityCall`：执行某个 ability 之后。  
  - 触发方：Tool Runner；  
  - Hook 配置：  
    - 全局 Pre/Post Hook： `.system/hooks/*.yaml` 中声明；  
    - 能力级 Hook：在 ability config 的 `hooks.pre_call` / `hooks.post_call` 中显式挂载。  

> 约定：**任何能力调用（无论来自 Agent Runtime、CI、cron 还是 CLI），都必须通过 Tool Runner**，从而统一执行 PreAbilityCall / PostAbilityCall 及能力级 Hook。

---

## 3. Registry / Config 层中的 Hook 配置

### 3.1 Registry / Config / Implement 三层回顾

- **Registry**：描述能力“是什么”——能力 ID / operation_key、意图、输入输出 schema、适用范围等；  
- **Config**：描述能力“如何执行”——脚本路径、参数模板、超时、环境 profile，以及能力级 Hook 绑定；  
- **Implement**：实际执行逻辑的脚本/服务。

### 3.2 Config Schema 中增加能力级 Hook 字段

在 `.system/registry/schemas/ability_impl.schema.yaml` 中，将原来的 guardrail 概念替换为 Hook 声明，例如：

```yaml
type: object
required: [id, impl]
properties:
  id:
    type: string          # ability_id
  impl:
    type: object
    required: [kind]
    properties:
      kind:
        type: string      # script | mcp | api
      script:
        type: object
        properties:
          path:
            type: string  # 脚本路径
          args_template:
            type: string  # 命令行参数模板
          timeout_sec:
            type: integer
          env_profile:
            type: string
      # mcp/api 配置略
  hooks:
    type: object
    properties:
      pre_call:
        type: array
        items:
          type: string    # PreAbilityCall 阶段挂载的 hook_id 列表
      post_call:
        type: array
        items:
          type: string    # PostAbilityCall 阶段挂载的 hook_id 列表
```

说明：

- 原 `guardrail` 逻辑统一下沉为若干 PreCallHook（hook_id），通过 `hooks.pre_call` 声明；  
- `hooks.post_call` 用于补充能力级 PostAbilityCall 行为（例如某个能力需要额外审计日志），而**通用 usage 统计由 Tool Runner 内部负责**。  

### 3.3 会话级 Hook 配置仍在系统层

所有 PromptSubmit / SessionStop Hook 依旧通过 `.system/hooks/*.yaml` 声明，不挂在能力配置上。例如：

```yaml
id: intent_normalizer
event_type: PromptSubmit
enabled: true
blocking: true
handler:
  kind: script
  command: ./scripts/hooks/intent_normalizer.py
```

---

## 4. Tool Runner 设计

### 4.1 角色与目标

> Tool Runner 是**唯一合法的工具/能力执行入口**。任何需要调用能力的主体（Agent Runtime / CI / cron / CLI）都应通过 Tool Runner 来创建/执行任务，以统一触发 Pre/Post Hook 与 usage 记录。

### 4.2 对外接口（CLI/API）

Tool Runner（例如 `scripts/runtime/tool_runner.py`）对外提供以下接口：

- `task.create(ability_id, input_payload) -> {task_key, status}`  
  - 主要步骤：  
    1. 根据 registry 判断 ability 是否存在、是否合法；  
    2. 根据 config 解析能力实现与能力级 Hook 配置；  
    3. 构造 `PreAbilityCallContext`（含 ability_id、caller.source 等）；  
    4. 调用 Hook Runner：`run_hooks("PreAbilityCall", ctx, blocking_only=True)`：  
       - 其中包含全局 PreAbilityCall Hook + ability.hooks.pre_call 声明的 Hook；  
    5. 解析 HookResult 得出的 guard 决策（allow / deny / need_human 等）；  
    6. 允许执行时创建 `.system/implement/<task-key>/` 目录并写入 `tool_call.md`，缓存输入与配置；  
    7. 返回 task_key 与可用性状态。  

- `task.get(task_key) -> {ability_id, input_payload, status, metadata}`  
  - 用于外部查看任务当前参数与状态。  

- `task.update(task_key, patch) -> {updated_payload}`  
  - 在执行前修改部分参数；  
  - 不默认触发 Hook，由调用方（通常是 Agent Runtime）决定何时重新 create/guard 或直接 run。  

- `task.run(task_key) -> {result}`  
  - 主要步骤：  
    1. 加载缓存参数与配置，确认实现可用；  
    2. 默认**不再执行 PreAbilityCall**，直接调用具体实现（脚本/mcp/api）；  
    3. 记录基础 usage/telemetry（能力 ID、状态、时延、错误信息等）；  
    4. 构造 `PostAbilityCallContext`；  
    5. 调用 Hook Runner：`run_hooks("PostAbilityCall", ctx, blocking_only=False)`：  
       - 包含全局 PostAbilityCall Hook + ability.hooks.post_call；  
    6. 返回标准化 result。  
  - 特殊情况：若 config 标明此能力需要在 run 阶段再次做 PreCall 检查（例如 `double_check_on_run: true`），Tool Runner 可在执行前再次跑一遍 PreAbilityCall Hook。  

- `task.delete(task_key)`  
  - 删除缓存任务及其临时文件。  

- `ability.direct_run(ability_id, input_payload) -> {result}`  
  - 一次性调用，内部可实现为 “create + run + delete” 的组合；  
  - 适用于低风险操作或测试场景。  

### 4.3 usage / telemetry 职责

- **通用 usage 记录**（如所有能力的调用次数、时延等）由 Tool Runner 容器内部统一完成；  
- PostAbilityCall Hook 为“附加行为”，仅当能力或项目有额外需求时才挂载，不负责基础统计。  

---

## 5. Hook Runner 设计

### 5.1 角色与形态

- 名称：Hook Runner / Hook Engine；  
- 形态：非常驻库/脚本；  
- 职责：  
  - 从 `.system/hooks/*.yaml` 读取 Hook 定义；  
  - 根据 `event_type` 和 `match` 规则筛选 Hook；  
  - 调用 handler 脚本（stdin 传入 context，stdout 返回 HookResult）；  
  - 合并 HookResult 并返回给调用方（Agent Runtime 或 Tool Runner）。  

### 5.2 核心接口

统一接口：

```python
def run_hooks(event_type: str, context: dict, blocking_only: bool = False) -> list[HookResult]:
    ...
```

- `event_type`：`PromptSubmit` / `PreAbilityCall` / `PostAbilityCall` / `SessionStop`；  
- `context`：事件上下文，由调用方构造；  
- `blocking_only`：若为 True，则只执行 `blocking: true` 的 Hook。  

### 5.3 事件级语义约束

- **PromptSubmit / PreAbilityCall（AI-facing，阻塞）**  
  - 调用方：Agent Runtime（PromptSubmit）/ Tool Runner（PreAbilityCall）；  
  - 必须使用 `blocking_only=True`；  
  - 其 HookResult 中的 `hook_signals` 被合并进 turn_context 或用于 guard 决策，可直接影响当前行为。  

- **PostAbilityCall / SessionStop（infra-only，非阻塞）**  
  - 调用方：Tool Runner（PostAbilityCall）/ Agent Runtime（SessionStop）；  
  - 使用 `blocking_only=False`；  
  - runtime 必须忽略这些 HookResult 中任何 AI-facing 字段，仅用于日志、缓存、指标等用途。  

> 结论：**Hook Runner 不关心调用者是谁，只依赖 event_type/context；事件何时触发、如何被解释由 Agent Runtime / Tool Runner 决定。**

---

## 6. Agent Runtime 设计（会话协调器）

### 6.1 角色与总体职责

> Agent Runtime 是**唯一常驻进程**，负责在“外部 AI / 人类”与“内部 Tool Runner / Hook Runner / workdocs”之间做协调，是“会话级 orchestrator”。

它承担三类核心职责：

1. **会话生命周期管理（Session Manager）**  
   - 创建/关闭 session（会话 ID）；  
   - 维护每个 session 的基本状态：最近输入、活跃 task 列表、最近 hook_signals 等。  

2. **Turn Pipeline（轮次管线）**  
   - 对每个用户输入：构造 PromptSubmitContext，调用 Hook Runner（PromptSubmit），合并 hook_signals，生成 turn_context；  
   - 将 turn_context 传给外部 AI。  

3. **Action Router / Command Handler（动作路由）**  
   - 接收外部 AI 在该轮返回的“动作原语”列表，而不是完整 plan：  
     - LIST_ABILITIES / INSPECT_ABILITY；  
     - CREATE_TASK / GET_TASK / UPDATE_TASK / RUN_TASK / DELETE_TASK；  
   - 对每个动作：  
     - 解析参数；  
     - 调用对应 Tool Runner 接口；  
     - 更新 Session Manager 中的状态；  
     - 必要时写入 workdocs。  

（可选）4. **Workdocs Integrator**  
   - 决定哪些状态变化需要同步到 `workdocs/plan.md`、`context.md`、`tasks.md` 或归档；  
   - 可在 SessionStop 或显式指令时写盘。  

### 6.2 Agent Runtime 对外协议（动作原语）

Agent Runtime 对外暴露的操作原语包括（可经 HTTP/RPC/CLI 暴露）：

- `OPEN_SESSION / CLOSE_SESSION`  
- `SUBMIT_INPUT(session_id, user_input)`  
  - 触发 PromptSubmit：构造 context → 调 Hook Runner → 返回 turn_context（含 hook_signals 和任务摘要）。  
- `LIST_ABILITIES / INSPECT_ABILITY`  
  - 由 Agent Runtime 读取 `ABILITY.md` 与 registry 信息，并返回给外部 AI。  
- `CREATE_TASK(session_id, ability_id, input_payload)`  
  - 调 Tool Runner 的 `task.create`；  
  - 根据 guard 结果更新 session 状态并反馈给外部 AI。  
- `GET_TASK / UPDATE_TASK / RUN_TASK / DELETE_TASK`  
  - 转发到 Tool Runner，对结果进行结构化封装并更新 session 状态。  

> 约定：**外部 AI 只与 Agent Runtime 交互，不直接调用 Tool Runner / Hook Runner。**  非会话型调用（如 CI/cron）可直接调用 Tool Runner，但须在策略文档中明确范围与风险。

### 6.3 最小主循环（Agent Runtime 视角）

以单轮交互为例：

1. `SUBMIT_INPUT(session_id, user_input)`：  
   - 构造 PromptSubmitContext；  
   - 调 `run_hooks("PromptSubmit", ctx, blocking_only=True)`；  
   - 合并 `hook_signals`，形成 turn_context；  
   - 将 turn_context（含 hook_signals、当前任务摘要等）返回给外部 AI。  

2. 外部 AI 返回动作原语列表：  
   - 如：`CREATE_TASK`、`UPDATE_TASK`、`RUN_TASK` 等；  

3. Agent Runtime 依序处理每个动作：  
   - 转发给 Tool Runner；  
   - 根据返回结果更新 session 状态；  
   - 必要时写入 workdocs。  

4. 会话结束或阶段结束时：  
   - 构造 SessionStopContext；  
   - 调 `run_hooks("SessionStop", ctx, blocking_only=False)`；  
   - 用于 build/test/总结，不再向当前 AI 回合注入信号。  

---

## 7. 系统级联调与完成标准

### 7.1 验收场景

1. **PreCall Guard 场景**  
   - 某能力在 config 中挂载了 PreCallHook；  
   - 外部 AI 通过 Agent Runtime 调用 `CREATE_TASK`：Tool Runner 在 `task.create` 内触发 PreAbilityCall、Hook 拒绝或要求人工确认，Agent Runtime 将结果返回给 AI。  

2. **能力执行 + PostCall 附加 Hook 场景**  
   - 某能力在 config 中挂载了 `hooks.post_call`；  
   - 外部 AI 创建并执行任务；  
   - 验证 Tool Runner 执行能力并记录 usage，同时 Hook Runner 在 PostAbilityCall 阶段执行附加行为（如写审计日志）。  

3. **会话级 PromptSubmit / SessionStop 场景**  
   - PromptSubmit Hook 提供 routing_hint/normalized_intent；  
   - SessionStop Hook 在会话结束时执行 build/test 汇总；  
   - 验证 Agent Runtime 正确触发这两个事件。  

### 7.2 完成判定（DoD）

当满足以下条件时，可认为 Phase 7 已完成：

1. Tool Runner 提供完整任务接口，并在 `task.create` / `task.run` 中统一触发 Pre/ Post AbilityCall Hook；  
2. Hook Runner 支持四类事件，并严格区分 AI-facing（PromptSubmit/PreAbilityCall）和 infra-only（PostAbilityCall/SessionStop）的行为；  
3. Agent Runtime 作为唯一常驻进程，实现了会话管理、PromptSubmit/SessionStop 调度和动作路由，对外暴露清晰的操作原语；  
4. Registry/Config 中使用 `hooks.pre_call / hooks.post_call` 取代 guardrail 字段，会话级 Hook 仅在 `.system/hooks/*.yaml` 中声明；  
5. 至少有若干 end-to-end 测试场景验证上述组件协作正确，并在 workdocs 中留下记录。  
