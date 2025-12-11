# 阶段八：能力与知识补全、文档层完善与质量体系建设规划（改进版）

> 目标（一句话）：在前七阶段完成架构与运行时建设的基础上，**补齐模板内“基础设施能力族”和关键知识文档，搭建清晰的人类说明层，通过系统测试与 CI 把模板打磨到可复用、可信赖的工程资产水平，并为 Agent Runtime 提供良好的启动与使用体验**；斜杠命令等作为可选但高价值的体验增强，与 Agent Runtime 协同使用。

---

## 1. 阶段定位与总体目标

### 1.1 在整体路线中的位置回顾 

- **阶段 1–3**：搭建目录骨架、文档规范与核心机制说明（模块 / 能力 / 内容路由 / Hook / 模块化规范）；  
- **阶段 4–5**：实现注册写入脚本与脚手架（Scaffolding），让“新增模块 / 能力 / Route / Hook / 场景”有标准流程；  
- **阶段 6**：用真实的小规模样本注册知识 / 能力 / Hook，验证脚手架工作闭环；  
- **阶段 7**：实现 Agent Runtime + Tool Runner + Hook Runner，使架构文档中“执行”和“联动”具备可运行实现。  

**阶段八**不再增加新机制，而是聚焦三类工作：  

1. **补内容**：补齐一组与模板强相关的“基础设施能力族”和少量开发/调试辅助能力；  
2. **建说明层**：构建面向人类的说明/索引文档，让人类有清晰的参与入口；  
3. **建质量线**：通过单测/契约测试/集成测试 + CI + 示例，将模板打磨成可复用、可信赖的工程资产。  

同时，规划中还需要：  

- 考虑 Agent Runtime 的**进程启动与使用方式**（dev 环境如何启动、如何连上外部 AI）；  
- 为斜杠命令与 Agent Runtime 的结合预留接口（可选增强）。  

---

## 2. 工作块 A：基础设施能力族与开发/调试辅助能力补全

### 2.1 工作块目标

> **为模板补齐一组与 repo 特性紧密相关的“基础设施能力族”和少量“开发/调试辅助能力”，使后续任何基于模板生成的项目，都能直接复用这些能力进行自检、自修、测试与调试，尤其方便搭建 e2e 场景。**

### 2.2 能力族范围

维持“infra 优先”的定位，在此基础上略微扩展到与模板关系紧密的 dev/debug 场景。

#### 2.2.1 模板健康检查能力

- 检查模块目录与 `modules/overview/*.yaml` 是否一致；  
- 检查 registry/config/hooks 中是否存在 dangling 引用；  
- 校验 `AGENTS.md`、`ROUTING.md` 等文件引用的路径是否存在；  
- 输出结构化“问题列表”，方便脚手架或 CI 使用。

#### 2.2.2 结构 / 文档同步能力

- 基于 registry / MANIFEST 变化，自动更新：  
  - 模块内 `AGENTS.md` 的能力列表片段；  
  - Tools / Runtime 概览文档中某些列表/表格段落；  
- 确保“代码 / 元数据 / 人类说明文档”三者尽量一致。

#### 2.2.3 运行时自检能力

- 对 Tool Runner / Hook Runner / Agent Runtime 做轻量自检：  
  - config 能否找到实现脚本；  
  - Hook handler 是否存在并可执行；  
  - 在 dev 环境跑一轮 smoke test（不对外产生破坏性效果）。  

#### 2.2.4 测试与 CI 辅助能力

- 一键触发模板测试套件（见工作块 C）：  
  - unit tests / contract tests / integration tests / scenario tests；  
- 汇总最近一次测试结果，生成供人类阅读的 Test Summary（可用于 README 或 workdocs）。

#### 2.2.5 开发与调试辅助能力（限模板工程侧）

在不引入业务逻辑的前提下，补充少量与模板强绑定的开发/调试能力，方便 e2e 场景构建：

- **后端开发/测试能力：**  
  - `backend.run_unit_tests`：按模块/目录运行后端单测；  
  - `backend.run_contract_tests`：对接口运行 contract tests；  
  - `backend.run_integration_tests`：按 `modules/integration/scenarios.yaml` 中的场景运行集成测试；  
  - `backend.start_local_service`：在 dev 环境拉起某服务（仅本地/非生产）。  

- **前端开发/测试能力（如适用）：**  
  - `frontend.run_unit_tests`：前端单测；  
  - `frontend.run_lint`：前端 lint/format；  
  - `frontend.build_bundle`：构建前端 bundle。  

- **调试/诊断能力：**  
  - `debug.collect_logs`：从日志镜像或 dev 环境收集指定服务的近期日志；  
  - `debug.dump_env_info`：导出当前 dev 环境的关键信息（版本、特性开关、依赖状态）；  
  - `debug.reproduce_scenario`：针对某个 integration scenario 复现一次失败路径（在测试环境）。  

> 约束：以上能力仅在 dev/test 环境下可用，真正联动外部系统的行为仍应通过 DevOps 脚本/Hook 控制，避免 AI 直接操作生产环境。

### 2.3 实施方式与约束

- 所有能力均应通过既有能力生命周期与脚手架落地：  
  - 使用 Phase 5 的能力脚手架（new_ability_low/high）；  
  - 使用 Phase 4/6 的 registry/config 写入脚本完成注册；  
  - 确保 Tool Runner 可以执行这些能力，并通过 Hook Runner 做 guard。  

- 每个能力必须：  
  - 在 `.system/registry/low-level|high-level` 注册；  
  - 在 `.system/registry/config/abilities.yaml` 有执行配置；  
  - 在对应模块的 `ABILITY.md` 有说明；  
  - 有至少一条测试覆盖（unit 或 integration）。  

---

## 3. 工作块 B：面向人类的说明层与“参与入口”

### 3.1 工作块目标

> **构建一层面向人类的简明说明层，使非模板作者的人也能快速理解当前 repo 的整体结构、关键能力与 Hook 配置，从而更好参与检验与运维。**

这里有两个侧重点：

1. 让人类快速理解“这是个什么模板、怎么用”；  
2. 让人类能看到**当前版本真实注册的 hooks / abilities / 脚手架等关键信息**，而不必阅读大量 YAML/MD。

### 3.2 README / Handbook / Tools 文档（整体说明）

延续原有规划，对文档层保持“少而精”的结构：

1. **根级 `README.md`（入门总览）**  
   - 说明项目模板的目标和使用场景；  
   - 提供目录结构鸟瞰图；  
   - 指出“人类第一次使用”和“外部 AI 第一次使用”应该看哪些文档。

2. **Template Handbook（如 `PROJECT_INIT/HANDBOOK.md`）**  
   - 系统化讲解：模块体系、运行时组件（Agent Runtime / Tool Runner / Hook Runner）、DevOps 目录（PROJECT_INIT/db/ops 等）；  
   - 描述典型工作流（初始化项目、加模块、加能力/Hook、跑测试）。  

3. **Tools & Abilities Overview（人类视角的工具地图）**  
   - 以类别分组列出常用 infra 能力和 dev/debug 能力；  
   - 每项只列：`ability_id`、一句话说明、调用入口（脚本/命令）；  
   - 不重复 registry/ABILITY.md 的技术细节。

4. **Tools Routing（需求→工具指引）**  
   - 从“我要做 X”出发，指向相关 AGENTS / 能力 / 脚手架；  
   - 例如，“新增模块”、“检查模板健康”、“为能力添加 Hook 保护”、“运行某个集成场景”等。  

5. **Runtime & Hooks Overview**  
   - 用人话解释：Agent Runtime / Tool Runner / Hook Runner 之间如何协作；  
   - 哪些 Hook 会影响 AI 行为（PromptSubmit/PreAbilityCall + blocking），哪些只是日志/统计；  
   - 外部 AI 和人类如何通过命令行或斜杠命令与 Agent Runtime 交互。

### 3.3 “人类参与入口”：运行时 & Hook 索引文档

除了以上整体说明，建议增加一份“面向运维/维护者的索引文档”，例如：  

- `RUNTIME_REFERENCE.md` 或 `HUMAN_RUNTIME_OVERVIEW.md`  

职责：

1. **模块/场景总览（简略）**  
   - 模块数量、重要 DevOps 场景（PROJECT_INIT/db/ops…）。  

2. **能力清单（按类别）**  
   - Infra 能力列表（健康检查/结构同步/自检/测试辅助）；  
   - Dev/Debug 辅助能力列表（后端/前端/调试相关能力）；  
   - 每项：`ability_id` + 一句话说明 + 调用入口（如 `tool_runner` 命令或斜杠协议）。  

3. **Hook 清单**  
   - 表格列出：`id / event_type / enabled / blocking / summary`；  
   - 标注哪些 Hook 是 AI-facing + blocking 的（例如 PromptSubmit/PreAbilityCall + blocking=true）。  

4. **脚手架与常用脚本**  
   - 列出主要脚手架能力/命令（新增模块/能力/Hook/场景）；  
   - 列出常用 Make 目标与 CI pipeline 入口。  

> 目标：人类只要看这一份，就能快速了解当前 repo 的“运行时配置状态”，不必深入 YAML/代码。

### 3.4 自动更新索引文档的脚本/能力

为避免手动维护索引，Phase 8 中应实现一个**自动更新脚本**，可同时注册成一个能力，例如：

- 脚本路径示例：`scripts/devops/update_runtime_reference.py`  
- 对应能力：`runtime.update_human_overview`  

行为：

1. 扫描 `.system/hooks/*.yaml`：  
   - 读取 `id / event_type / enabled / blocking / summary`；  
   - 生成 Hook 概览表格（Markdown）。  

2. 扫描能力 registry/config：  
   - 读取已注册的能力（low-level/high-level），结合 config 中的分类信息（如 infra/dev/debug）；  
   - 生成按类别分组的能力清单。  

3. 可选：扫描脚手架脚本、Agent Runtime 的斜杠命令约定（见第 5 节），生成“常用命令”列表。  

4. 更新 `RUNTIME_REFERENCE.md`：  
   - 使用标记区（例如 `<!-- AUTO-GENERATED: HOOKS_TABLE_START -->` / `END`）来替换自动生成区域；  
   - 保留人工维护的上下文说明和结构。  

5. 集成：  
   - 建议在 CI 中增加一步校验：  
     - 运行 `update_runtime_reference`，若有 diff，提示开发者本地运行并提交更新；  
   - 或由 AI 在合适的维护任务中主动调用该能力。

---

## 4. 工作块 C：测试体系与 CI（质量线建设）

### 4.1 工作块目标

> **为模板建立可复用的测试与 CI 体系，保证模板本身“没有明显 bug / 配置错误”，并支持后续项目直接继承这条质量线。**

### 4.2 测试维度

1. **单元测试（Unit Tests）**  
   - 覆盖：  
     - registry 写入脚本；  
     - Tool Runner（task.* 调度逻辑）；  
     - Hook Runner（事件分发与 handler 调用）；  
     - 核心 infra 能力和 dev/debug 能力。  

2. **契约测试（Contract / Schema Tests）**  
   - 验证：  
     - 所有 registry/config/hooks YAML 满足各自 schema；  
     - 所有跨引用（ability ↔ hooks ↔ scripts、AGENTS ↔ knowledge 路径）不存在明显 dangling。  

3. **集成测试（Integration Tests）**  
   - 覆盖典型完整链路：  
     - “新增能力 → 注册 → Tool Runner 调用 → Hook 生效”；  
     - “新增 Hook → 能力调用时生效 Guard/审计”；  
     - “脚手架 → 注册 → Runtime 调用”闭环。  

4. **场景测试（Scenario / Golden Tests）**  
   - 针对特定 integration scenario（例如简单 e2e 流程）构建“黄金流程”：  
     - 保证在模板演进时核心路径不会被破坏；  
     - 在 `knowledge/examples` 中记录这些流程作为正向示例。

5. **轻量性能 / 稳定性测试**  
   - 对一些核心路径（如 Tool Runner 批量调用、Hook Runner 大量无匹配 Hook 情况）做简单压力/稳定性测试，避免明显的性能/资源问题。

### 4.3 CI 集成

- 为模板仓库增加 CI pipeline：  
  - 安装依赖；  
  - 运行 unit + contract tests；  
  - 运行关键 integration / scenario tests；  
  - 可选：运行 `update_runtime_reference` 并比对是否有未提交变更。  

- CI 应该默认阻止在测试未通过的情况下合并变更；  
- 后续项目 clone 模板时可以直接继承这套 CI 配置作为默认质量线。

---

## 5. 工作块 D：Examples 作为经验知识库

### 5.1 工作块目标

> **将测试与实战过程中遇到的典型情况（正例与反例）沉淀为知识文档，帮助 AI 与人类在未来快速理解“正确用法”和“常见陷阱”。**

### 5.2 形式与落点

- 在 `knowledge/examples/` 下维护若干案例文档：  
  - 正例：某种能力/Hook/脚手架使用得很顺畅的 e2e 流程；  
  - 反例：某次配置错误或设计问题导致的 bug 及其修复。  

- 每个案例文档包含：  
  - 背景描述（关联的模块/能力/Hook/场景）；  
  - 操作步骤（可加上链接到相关 tests 和 workdocs 任务）；  
  - 问题与解决方案（若是反例）；  
  - 推荐实践与“以后应该怎么做”。  

### 5.3 与测试/CI 的关系

- 每一个 scenario test / golden test 对应至少一个案例文档；  
- 反向：当发现一个严重问题时，应：  
  - 写测试用例防止回归；  
  - 写 example 文档记录教训与推荐实践。

---

## 6. 工作块 E：斜杠命令与 Agent Runtime 结合（可选增强）

### 6.1 工作块目标

> **提供一套轻量、统一的“斜杠命令协议”，作为人类与外部 AI 使用 Agent Runtime 的友好入口，同时保持架构的一致性与可控性。**

### 6.2 斜杠命令的三层分解

1. **表示层（Representation）**  
   - 在对话/编辑器中，以 `/.*` 形式表达常见操作，例如：  
     - `/.scaffold module <module_id>`  
     - `/.run ability <ability_id> [--json <payload>]`  
     - `/.inspect hooks`  

2. **协议层（Protocol → Agent Runtime Action）**  
   - 斜杠命令解析后，对应到一组 Agent Runtime 的动作原语：  
     - 如 `/.run ability A` → `CREATE_TASK(A)` + `RUN_TASK(task_key)`；  
     - `/.scaffold module` → 调用脚手架能力。  

3. **执行层（Execution）**  
   - Agent Runtime 收到解析后的 Action 列表：  
     - 调用 Tool Runner / Hook Runner；  
     - 更新 session 状态 / workdocs；  
     - 返回结果给外部 AI / 人类。  

> Tool Runner 与 Hook Runner **不理解斜杠命令**，只接受结构化动作。斜杠协议由 Agent Runtime 层解析并执行。

### 6.3 在文档中的表现形式

- 在根 `AGENTS.md` 中增加路由指向一个专门的文档，在其中定义“斜杠命令语法 + 语义”：  
  - 描述每条斜杠的含义、对应的 Agent Runtime 动作以及适用/禁用场景；  
  - 明确哪些命令只可用于 dev/test，哪些需要人工确认。  

- Agent Runtime 对外暴露一个统一的 `slash_command` 接口：  
  - 如 HTTP：`POST /slash_command`；  
  - 或 CLI：`agent_runtime slash "/.run ability A" --session S-123`。  

- 可以在 `RUNTIME_REFERENCE.md` 中自动生成一节 “Supported Slash Commands”，方便人类查看。

> 斜杠命令本身为可选特性，但建议在模板中预留接口与文档结构，后续视项目需要逐步实现。

---

## 7. Agent Runtime 的进程启动与使用体验

### 7.1 工作块目标

> **让 Agent Runtime 作为唯一常驻进程的启动和使用方式清晰可控，便于在本地开发和 CI 环境中快速启动并接入外部 AI。**

### 7.2 启动方式与命令

在模板中应提供统一的启动方式，例如：

- CLI 命令：  
  - `scripts/runtime/start_agent_runtime.py` 或  
  - `make agent-runtime`（封装启动）  

- 启动参数：  
  - 端口 / 监听模式（HTTP/RPC）  
  - 运行环境（dev/test）  
  - 外部 AI 连接方式（如通过某个代理/桥接）  

在 Handbook / Runtime Overview 中需要说明：

- 如何在本地启动 Agent Runtime；  
- 如何确认其健康状态（例如 healthcheck endpoint）；  
- 外部 AI 或开发者如何连接到它（例如 API 地址、认证方式）。

### 7.3 与 CI / 自动化的关系

- 在 CI 环境下可以选择：  
  - 启动一个简化版 Agent Runtime，仅用于运行 scenario tests；  
  - 或仅使用 Tool Runner 与 Hook Runner，而不必启动 Agent Runtime（针对纯非交互场景）。  

- 策略：  
  - 若 CI 偏重“模拟真实使用”，建议在 pipeline 中临时启动 Agent Runtime，运行少量回归场景；  
  - 若 CI 偏重快速验证核心逻辑，可以只调用 Tool Runner + Hook Runner，省略 Agent Runtime。

### 7.4 策略文档中的说明

- 在根 `AGENTS.md` 中应：  
  - 明确 Agent Runtime 是“唯一常驻协调器”；  
  - 建议外部 AI 只通过 Agent Runtime 的接口来操作 repo（避免直接绕过 Tool Runner / Hook Runner）；  
  - 对 CI/cron 直接调用 Tool Runner 的场景给出范围与注意事项。

---

## 8. 完成判定（DoD）

当满足以下条件时，可认为 Phase 8 已完成：

1. **能力与知识层**：  
   - 基础设施能力族和少量 dev/debug 能力已实现并注册，可通过 Tool Runner 调用；  
   - 核心知识域（architecture / modules template / devops / scaffolding / runtime）有对应知识文档，并在 AGENTS/ROUTING 中被引用。  

2. **人类说明层**：  
   - 根 README + Handbook + Tools/Runtime 概览文档存在且结构清晰；  
   - 至少有一份“运行时 & Hook 索引”文档（如 RUNTIME_REFERENCE.md），其中部分区域由脚本自动维护。  

3. **测试与 CI**：  
   - 模板仓库通过单元、契约、关键集成与场景测试；  
   - CI pipeline 集成这些测试并默认阻止测试失败的合并。  

4. **Examples**：  
   - 有若干正/反例案例文档与对应的 tests/workdocs 关联，形成可供 AI 和人类参考的经验库。  

5. **斜杠命令与 Agent Runtime（如实施）**：  
   - 斜杠命令规范与 Agent Runtime 的 slash 接口定义清晰；  
   - 至少支持若干基础命令（如 scaffold、run ability、inspect hooks），并在文档中有说明。  

6. **Agent Runtime 启动与使用体验**：  
   - 本地环境下一条命令即可启动 Agent Runtime，并在文档中清楚说明接入方式；  
   - 在 CI/自动化场景下，Agent Runtime 与 Tool Runner/Hook Runner 的使用路径与策略已被明确。  

当这些目标达到时，repo 模板不仅“结构完整、机制齐全”，而且**能力丰富、文档友好、质量可控**，既适合 AI 深度参与，也为人类的检验与运维提供了清晰入口。  
