# 阶段五：脚手架与交互式流程规划

> 目标（一句话）：在前四个阶段的目录、规范、知识与写入脚本基础上，**为模板中的各类关键场景补齐“可跑通的脚手架”**，通过 AI + 人交互的方式完成，从需求到变更落盘的完整流程。

---

## 1. 阶段定位与范围

### 1.1 在整体路线中的位置

- 阶段一：固定目录与空文件清单 —— 解决“东西在哪儿”。  
- 阶段二：文档规范与 `AGENTS.md` 结构 —— 解决“AI 到每个目录后能看懂规则和入口”。  
- 阶段三：核心机制知识与标准流程 —— 解决“模块 / 能力 / 内容路由这些大决策怎么做”。  
- 阶段四：注册写入脚本与协同机制 —— 解决“如何安全地把最终决定写进各类 registry 文档”。  
- **阶段五：脚手架与交互式流程 —— 解决“如何让 AI 在与人协作下，从需求出发，一路跑到调用脚本、写回 registry 与骨架文件”。**

### 1.2 本阶段“做 / 不做”

**本阶段要做：**

1. 为模板中的关键场景设计并落地**脚手架知识文档**（伪 agent），统一放在 `knowledge/scaffolding/`（项目初始化除外的运行资产仍在 `PROJECT_INIT/`）。  
2. 为各脚手架编写**编排脚本（orchestrator）**：  
   - 接收由 AI+人确认好的参数；  
   - 调用阶段四的注册写入脚本和相关文件操作；  
   - 输出结构化结果。  
3. 在相关 `AGENTS.md` 中写清楚**“如何发起、如何交互、如何记录”**，让 AI 能主导脚手架流程。  
4. 规范脚手架过程的 `workdocs/` 使用方式，保证每次脚手架执行都有可追溯“流水账”。  

**本阶段暂不做：**

- 不实现复杂的 DevOps 真实部署/CI 流程（可以先只生成配置骨架）；  
- 不实现复杂 UI/服务，只通过 CLI + 文档驱动；  
- 不为所有可能场景都写脚手架，只优先覆盖模板中的“主干场景”。  

---

## 2. 脚手架场景矩阵（优先范围）

### 2.1 项目级脚手架（全部资产在 `PROJECT_INIT/` 下）

1. **项目初始化脚手架**  
   - 作用：从一个空/半空仓库，初始化基础模块/registry/配置骨架。  
   - 运行资产：  
     - `PROJECT_INIT/AI/AGENTS.md`  
     - `PROJECT_INIT/scripts/init_project.py`（orchestrator）  
     - `PROJECT_INIT/workdocs/`（记录初始化全过程）  
   - 知识文档：  
     - `knowledge/scaffolding/project_init.md`（流程说明，供 AI 加载）。  

> 约束：与“项目初始化”相关的脚手架脚本与 workdocs，只能放在 `PROJECT_INIT/` 目录下。

### 2.2 模块 / 能力 / 路由 / Hook 脚手架

统一视为“开发期”脚手架：

1. **新建模块实例脚手架**  
   - 场景：根据业务需求创建新的 `modules/<module_id>/`。  
   - 涉及：模块 registry、模块目录骨架、模块级 `AGENTS.md` 与 `workdocs/` 初始化。  

2. **新增低级能力（low-level ability）脚手架**  
   - 场景：为已有模块新增一个底层能力，并注册到 `.system/registry/low-level/`。  

3. **新增高级能力 / workflow / agent 脚手架**  
   - 场景：基于已有低级能力与策略，注册一个高阶 workflow/agent 到 `.system/registry/high-level/`。  

4. **新增内容路由（content route）脚手架**  
   - 场景：为新的 scope/topic 或需求场景新增/调整路由规则，更新 `modules/routes/*.yaml`。  

5. **新增 Hook 注册脚手架**  
   - 场景：在 `.system/hooks/*.yaml` 中新增 Hook 条目，挂接到某个能力/阶段。  
   - 注意：本阶段仍只处理“注册”，不实现具体 handler 逻辑。  

### 2.3 DevOps 脚手架（本阶段以“生成配置骨架”为主）

1. **新 HTTP 服务的打包脚手架**  
   - 目标：在 `ops/packaging/services/<service_name>/` 下生成 `Dockerfile` + build 脚本骨架。  

2. **新 Job 的打包脚手架**  
   - 目标：在 `ops/packaging/jobs/<job_name>/` 下生成打包配置骨架。  

3. **新 HTTP 服务的部署脚手架**  
   - 目标：在 `ops/deploy/http_services/<service_name>/<env>.yaml` 下生成配置骨架。  

4. **新 Job 的部署脚手架**  
   - 目标：在 `ops/deploy/workloads/<job_name>/<env>.yaml` 下生成配置骨架。  

> DevOps 脚手架在本阶段“写好配置骨架 + 调整知识文档”，具体跟真实基础设施集成可以放到后续阶段。  

---

## 3. 工作块 A：脚手架知识文档（伪 agent）设计

**目标：**为每一个脚手架场景提供一份独立的“说明书”，AI 只要加载对应文档就能知道：要问什么问题、调用哪些工具、何时让人确认。

### 3.1 目录结构

```text
knowledge/
  scaffolding/
    project_init.md              # 项目初始化脚手架说明（运行资产仍在 PROJECT_INIT/ 下）
    new_module.md                # 新建模块脚手架说明
    new_ability_low.md           # 新增低级能力脚手架说明
    new_ability_high.md          # 新增高级能力/agent 脚手架说明
    new_route.md                 # 新增内容路由脚手架说明
    new_hook.md                  # 新增 Hook 注册脚手架说明
    devops_packaging_service.md  # 新 HTTP 服务打包脚手架说明
    devops_packaging_job.md      # 新 Job 打包脚手架说明
    devops_deploy_service.md     # 新 HTTP 服务部署脚手架说明
    devops_deploy_job.md         # 新 Job 部署脚手架说明
```

> 项目初始化的运行逻辑（AGENTS, scripts, workdocs）仍在 `PROJECT_INIT/` 下，这里只放“知识”。

### 3.2 每份脚手架文档的推荐结构

统一使用以下骨架（由 AI 填充内容）：

1. **Purpose & Scope**  
   - 该脚手架解决什么问题，不解决什么问题。  

2. **Inputs & Preconditions**  
   - AI 在开始执行前需要从人类拿到哪些关键信息；  
   - 仓库中需要已经存在哪些前置状态（例如 Phase 4 的写入脚本可用）。  

3. **Step-by-step Flow（AI + 人交互）**  
   - 分步列出：  
     - AI 先做什么（分析 / 生成方案）；  
     - 需要向人确认/补充的节点；  
     - 确认后调用哪些脚本 / 写哪些文件。  

4. **Tools & Scripts to Use**  
   - 对应的 orchestrator 脚本路径（见工作块 B）；  
   - 可能会调用的底层注册写入脚本（Phase 4 的成果）。  

5. **Outputs & Side Effects**  
   - 会新增/修改哪些 registry YAML；  
   - 会创建哪些目录 / 文件骨架；  
   - 对 `workdocs/` 的记录要求。  

6. **Safety & Rollback Notes**  
   - 在发现问题或执行中断时，建议的回滚方式或手动修复步骤。  

**分工：**

- AI：根据 Phase 3 的标准流程与 Phase 4 的能力，生成上述文档内容初稿；  
- 人：快速检查流程是否符合实际需求，不满意的地方交给 AI 重写。  

---

## 4. 工作块 B：脚手架 orchestrator 脚本

**目标：**为每个脚手架提供一个“流程入口”，把多步操作串起来，并与 Phase 4 的写入脚本良好对接。

### 4.1 目录布局与特殊约束

1. **项目初始化脚手架脚本** —— 必须放在 `PROJECT_INIT/` 下：

```text
PROJECT_INIT/
  scripts/
    init_project.py          # 初始化 orchestrator（AI + 人交互参数 → 调用写入脚本）
```

2. **其他脚手架脚本** —— 可统一放在 `scripts/scaffold/` 下：

```text
scripts/
  scaffold/
    new_module.py
    new_ability_low.py
    new_ability_high.py
    new_route.py
    new_hook.py
    devops_packaging_service.py
    devops_packaging_job.py
    devops_deploy_service.py
    devops_deploy_job.py
```

> 这些 orchestrator 一般不直接跟用户互动，而是接收由 AI 整理好的参数，并返回结构化结果。与人的对话主要由 AI 在 `AGENTS.md` 中定义的模式负责。

### 4.2 orchestrator 的通用行为模式

以 `new_module.py` 为例，其行为模式可参考：

1. 接收参数（如通过 CLI、JSON 文件或环境变量）：  
   - `module_id`  
   - `module_type`  
   - `description`  
   - 其他必要元信息。  

2. 逻辑步骤：  
   - 校验参数是否符合 schema（可借用 Phase 4 的 `validate_entry` 等通用函数）；  
   - 调用 Phase 4 的模块注册写入函数，更新：  
     - `modules/overview/instance_registry.yaml`  
     - 其他相关 registry（如 `instance_maturity.yaml`）；  
   - 在 `modules/<module_id>/` 下创建目录骨架：  
     - `AGENTS.md`（空或最小模板）  
     - `MANIFEST.yaml`（按 schema 填入基本字段）  
     - `ROUTING.md`、`ABILITY.md`（空骨架）  
     - `workdocs/` 目录结构。  

3. 输出结果：  
   - 返回一个结构化的结果（如 JSON），包含：  
     - 成功/失败状态；  
     - 新增/修改文件路径列表；  
     - 供 AI 写入 workdocs 的摘要信息。  

其他 orchestrator（new_ability / new_route / new_hook / DevOps 系列）同理：

- 输入参数根据各自 schema 与脚手架文档定义；  
- 内部调用 Phase 4 的各类 registry 写入与校验函数；  
- 在对应目录下创建/更新配置骨架（DevOps 场景重点是生成配置文件骨架）；  
- 返回可供 AI 写入 workdocs 的结果。  

---

## 5. 工作块 C：在相关的 `AGENTS.md` 中（不要一次性把所有内容扔给AI，在需要时加载）定义 AI+人交互流程

**目标：**让 AI 清楚知道：

- 什么时候建议用户用脚手架；  
- 用哪一份 `knowledge/scaffolding/*.md` 做参考；  
- 采用怎样的对话节奏与确认步骤；  
- 在执行结束后如何记录和总结。

### 5.1 需要更新的 `AGENTS.md` 列表

至少包括：

- 根级 `AGENTS.md`：  
  - 新增 “Scaffolding Overview” 小节，列出主要脚手架及其入口 scope。  

- `modules/AGENTS.md`：  
  - 描述：  
    - 新建模块脚手架；  
    - 新增能力脚手架；  
    - 新增内容路由脚手架。  

- `PROJECT_INIT/AI/AGENTS.md`：  
  - 详细说明项目初始化脚手架的交互流程；  
  - 指向 `knowledge/scaffolding/project_init.md` 与 `PROJECT_INIT/scripts/init_project.py`。  

- `ops/packaging/AGENTS.md` 与 `ops/deploy/AGENTS.md`：  
  - 描述 DevOps相关脚手架（打包/部署）的使用方式与限制。  

### 5.2 AGENTS 中的交互模式建议

以 “新建模块脚手架” 为例，在相应 `AGENTS.md` 中约定：

1. **收集需求阶段**  
   - AI 提问：模块作用域、依赖关系、是否有相似模块、期望的模块类型等；  
   - 人回答后，AI 将关键信息整理成结构化摘要。  

2. **生成方案与确认阶段**  
   - AI 基于 `knowledge/scaffolding/new_module.md` 与 Phase 3 的标准流程，生成执行 plan（包括要修改的 registry、要创建的文件）；  
   - 将 plan 展示给人类，请求确认或修改。  

3. **执行阶段**  
   - 经确认后，AI 根据 plan 组织 orchestrator 的输入参数；  
   - 调用 `scripts/scaffold/new_module.py`；  
   - 解析输出结果。  

4. **记录与总结阶段**  
   - 将本次执行过程写入 `modules/<module_id>/workdocs/active/<task_id>/`；  
   - 向人总结变更内容与后续建议。  

其他脚手架场景的 `AGENTS.md`，可沿用相同 pattern 并结合具体细节。  

---

## 6. 工作块 D：脚手架执行与 `workdocs/` 记录规范

**目标：**保证每一次脚手架执行都有清晰、统一的“流水账”，便于追溯与复盘。

### 6.1 场景与 workdocs 位置对应

- 项目初始化脚手架：  
  - `PROJECT_INIT/workdocs/` 下按任务/时间组织；  

- 模块/能力/路由/Hook 脚手架：  
  - 主要记录在对应模块或全局 scope 的 `workdocs/` 下，例如：  
    - 模块相关：`modules/<module_id>/workdocs/`  
    - 只影响全局路由：可放在与路由相关的场景工作区（如后续新增的 `modules/routes/workdocs/` 或其他约定路径）；  

- DevOps 脚手架：  
  - `ops/packaging/workdocs/`、`ops/deploy/workdocs/`。  

> workdocs的写入应该由AI完成，一般情况下，脚手架包含的项目脚本不要随意改动workdocs文档

### 6.2 内容规范（建议写进 `DOCS_CONVENTION.md` 和相关 `AGENTS.md`）

每个脚手架 task 至少应包含：

1. 任务元信息：触发时间、场景类型（new_module/new_route/...）、发起人；  
2. 初始需求描述（人类输入 + AI 归纳）；  
3. 关键决策（包括 human override 的地方）；  
4. 调用的 orchestrator 脚本及其参数摘要（非敏感部分）；  
5. 脚本输出的总结（新增/修改的文件与 registry 条目）；  
6. 如有异常或暂未完成的部分，说明 TODO 与风险点。  

当任务完成后：

- 将 `active` 下的记录移动到 `archive`；  
- 可按月份/任务类型组织 archive。  

---

## 7. 阶段五完成判定标准（Definition of Done）

满足以下条件时，可认为 Phase 5 已完成：

1. `knowledge/scaffolding/` 下已有所有优先场景的脚手架说明文档，并通过至少一轮人工审核。  
2. 对以下场景，均实现了可调用的 orchestrator 脚本：  
   - 项目初始化（`PROJECT_INIT/scripts/init_project.py`）；  
   - 新建模块；  
   - 新增低级能力；  
   - 新增高级能力/agent；  
   - 新增内容路由；  
   - 新增 Hook 注册；  
   - 新 HTTP 服务/Job 的打包与部署配置骨架。  
3. 相关 scope 的 `AGENTS.md`（根级、modules、PROJECT_INIT、ops/packaging、ops/deploy）中：  
   - 清晰描述了如何发起脚手架；  
   - 明确了 AI+人交互步骤；  
   - 指向了对应的 scaffolding 文档和 orchestrator 脚本。  
4. 至少用一个真实/模拟案例，完整跑通：  
   - 项目初始化脚手架；  
   - 一条开发期脚手架（如新建模块或新增能力）。  
   运行结果在对应 `workdocs/` 中有完备记录。  
5. 整个阶段中，大部分文档与脚本由 AI 生成与修改；  
   人类主要负责：选择场景优先级、校正流程逻辑与做关键集成测试。  

