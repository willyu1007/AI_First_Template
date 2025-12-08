# 阶段二：文档规范与 AGENTS 结构设计规划

> 目标（一句话）：在阶段一目录骨架的基础上，由 **AI 负责产出主要文档内容，人只作为检查与纠偏角色**，完成文档规范初稿和各层级 `AGENTS.md` 的结构与基础内容。

---

## 1. 阶段定位与范围

### 1.1 阶段定位

- 阶段一：解决“**东西在哪儿**”，固定目录与空文件清单；  
- **阶段二：解决“AI 到了某个目录后，**知道自己能做什么、要看什么、能改什么**”**。

具体来说：

- 由 AI 批量生成：  
  - `DOCS_CONVENTION.md` 的规范初稿；  
  - 各级 `AGENTS.md` 的结构和基础内容；  
  - `knowledge/` 下关键 `README.md` 与模块模板说明文档的初稿。  
- 人类主要职责：  
  - 按 checklist 审查 AI 产出的文档，发现问题后交回 AI 修订。  

### 1.2 本阶段“做 / 不做”

**做：**

- 给出一套统一的文档规范（放在 `DOCS_CONVENTION.md`）；  
- 设计并由 AI 填充通用版 `AGENTS.md` 模板，并在关键目录中落地；  
- 为 `knowledge/` 下的核心分区提供职责说明与基础骨架；  
- 确立“AI 写 → 人审 → AI 修”的协作流程，并写入根级规范。  

**不做：**

- 不要求文档内容已经非常详尽（允许后续阶段逐步补充）；  
- 不编写具体的业务设计文档 / 详细架构分析（只搭骨架、定原则）；  
- 不实现脚本逻辑、不接入实际 hooks 或 CI/CD。

---

## 2. 工作拆分：四个主块

### 块 A：`DOCS_CONVENTION.md` 规范文档

**目标：**  
`DOCS_CONVENTION.md` 中，以 **AI 可执行、可检查** 的形式，定义文档类型、命名与放置规则。

#### A.1 检查角色

- 写一份简短的“DOCS 规范验收清单”（可以直接写在 `DOCS_CONVENTION.md` 中的一个章节里，或者前期放在 workdocs）。  
  建议至少包含：  
  1. 必须区分并说明：  
     - `AGENTS.md`：唯一面向 AI 的策略与规范入口；  
     - `README.md`：只面向人类；  
     - `knowledge/**`：可复用知识文档的集中区域；  
     - `workdocs/**`：AI 的临时工作区与中间过程。  
  2. 必须说明以下路径与边界：  
     - `modules/<module_id>/` 与 `knowledge/modules/template/` 各自的职责；  
     - `PROJECT_INIT/`、`db/`、`ops/packaging/`、`ops/deploy/` 专属的 `AGENTS`/`workdocs` 用法；  
  3. 禁止内容：  
     - 禁止在 README 中写 AI 行为规范；  
     - 禁止将长期稳定知识只放在 `workdocs/` 中而不沉淀。  

#### A.2 AI工作

- 根据阶段一的目录结构 + 验收清单，由 AI 生成 `DOCS_CONVENTION.md` 的初稿，建议章节：  
  1. 文档类型与角色  
  2. 目录与路径约定  
  3. 链接与引用规范（如 `AGENTS.md` 如何引用 `knowledge/`）  
  4. `workdocs/` 使用与升级规则  
  5. 人工 Review 与 AI 修订流程  
- 人按 checklist 进行审查：  
  - 若不符合规范，由 AI 根据反馈进行重写或增补；  
  - 确认后，标记本阶段 A 块完成。  

---

### 块 B：`AGENTS.md` 通用结构与实例化

**目标：**  
为项目中多个关键位置提供 **统一结构** 的 `AGENTS.md`，并由 AI 自动填充基础内容，确保 AI 在任何 scope 下都有明确的行为边界与导航。

#### B.1 通用 `AGENTS.md` 结构（需要人确认字段后，AI 填内容）

建议定义通用 section 清单（作为模板）：

1. **Scope & Responsibility**  
   - 当前目录范围内的职责边界；  
   - 可以修改的内容与禁止修改的内容；  
   - 与上级 / 下级 `AGENTS`（例如workdocs等） 的关系。  

2. **Allowed Operations**  
   - 在本 scope 下 AI 可以主动发起的典型任务列表（例如如何使用知识/能力路由）；  
   - 每类任务关联的目录 / 脚本入口（即使脚本尚未实现，也固定名称）。  

3. **Safety & Escalation**  
   - 不确定时的默认行为（例如：只读、不写代码、先写 plan 等）；  
   - 需要向上级 `AGENTS` 或人工确认的行为类型；  

> 在AI完成每一节的内容后，需要提醒人工审查，确定这些 section 名称与简要说明的正确性。

#### B.2 需要落地的 `AGENTS.md` 实例

由 AI 基于通用结构生成以下文件的内容初稿：

1. 根目录：`AGENTS.md`  
   - 负责说明全局视角、不同子目录 AGENTS 的分工。  

2. `modules/AGENTS.md`  
   - 负责说明模块体系（模块类型、模块实例、overview/、routes/ 等）整体的 AI 使用方式。  

3. 模块模板：`knowledge/modules/template/AGENTS.md`  
   - 专门用于指导 AI 如何创建/演化 `modules/<module_id>/`。  

4. 初始化场景：`PROJECT_INIT/AI/AGENTS.md`  
   - 负责项目初始化阶段的流程与边界。  

5. DevOps 场景：  
   - 打包：`ops/packaging/AGENTS.md`  
   - 部署：`ops/deploy/AGENTS.md`  

6. 其他场景：
   - `db/AGENTS.md`：若希望对 DB 变更流程进行单独约束，可在此阶段一并生成初稿。  

#### B.3 检查方式

- 在`template_construction`中写一份统一的“验收 checklist
- 按照checklist进行审查，坚持“AI 改到合格为止”，人不直接大段重写。  

---

### 块 C：`knowledge/` 分区规范与基础骨架

**目标：**  
让未来所有架构说明、模块模板说明、DevOps 模板等内容，都有明确的归宿和边界。

#### C.1 重点分区

结合阶段一的目录：

```text
knowledge/
  architecture/
  modules/
    template/
  devops/
```

#### C.2 定义约束

分别给每个分区写一条“职责一句话”，作为 AI 写内容的硬约束，例如：

- `knowledge/architecture/`：  
  > 只存放与全局架构相关、跨模块复用的说明，不写具体模块内部细节。  

- `knowledge/modules/template/`：  
  > 只存放关于如何设计 / 创建 / 注册模块实例的知识，用于指导 AI 搭建 `modules/<module_id>/`。  

- `knowledge/devops/`：  
  > 只存放跨项目通用的打包 / 部署 / 环境规范，不放具体项目实例配置。  

#### C.3 AI 工作

在上述约束下，AI 完成：

1. 各分区 `README.md`、`AGENTS.md` 初稿：  
   - 描述该分区的用途、目标读者（AI / 人）、大致内容类型。  

2. 模块模板知识的基础文件：  
   - `knowledge/modules/template/layout.md`：描述标准模块目录结构；  
   - `knowledge/modules/template/manifest_fields.md`：描述 `MANIFEST.yaml` 字段及约束；  
   - `knowledge/modules/template/registration_flow.md`：描述创建模块、更新 registries 的过程；  
   - `knowledge/modules/template/examples/homework_system.md`：保留占位，可先写“TODO 示例”。  

3. DevOps 模板知识：  
   - `knowledge/devops/packaging_template.md` / `deploy_template.md` 初稿：说明将来脚本应遵守的抽象流程与约束（不写具体命令）。  

人只从两个角度检查：  
- 是否违反一句话职责（信息越界）；  
- 是否与阶段一目录结构发生明显冲突。  

---


## 3. 第二阶段涉及的文件清单

以下文件在 Phase 2 中 **必须被创建或更新**（允许内容为初稿）：

- 根目录：  
  - `DOCS_CONVENTION.md`  
  - `AGENTS.md`  

- `modules/`：  
  - `modules/AGENTS.md`  

- `knowledge/`：  
  - `knowledge/README.md`  
  - `knowledge/architecture/overview.md`（可为简要初稿）  
  - `knowledge/modules/template/AGENTS.md`  
  - `knowledge/modules/template/layout.md`  
  - `knowledge/modules/template/manifest_fields.md`  
  - `knowledge/modules/template/registration_flow.md`  
  - `knowledge/modules/template/examples/homework_system.md`（可为 TODO）  
  - `knowledge/devops/packaging_template.md`  
  - `knowledge/devops/deploy_template.md`  

- 场景目录：  
  - `PROJECT_INIT/AI/AGENTS.md`  
  - `ops/packaging/AGENTS.md`  
  - `ops/deploy/AGENTS.md`  
  - （可选）`db/AGENTS.md`  

---

## 4. 阶段二完成判定标准（Definition of Done）

满足以下条件时，可以认为 Phase 2 已完成：

1. `DOCS_CONVENTION.md` 已存在且：  
   - 明确说明了 `AGENTS` / `README` / `knowledge` / `workdocs` 的角色；  
   - 对主要目录结构与路径约定有基本说明；  
   - 描述了 AI↔人协作流程（AI 写、人审、AI 改）。  

2. 根 `AGENTS.md` 与上述所有需要的 `AGENTS.md` 文件：  
   - 均采用统一的 section 结构；  
   - 对本 scope 的允许操作、知识路由与 workdocs 用法有清晰描述；  
   - 指向了正确的 `knowledge/` 文档。  

3. `knowledge/` 下的各主要分区：  
   - 具备基础 README 与模板说明文档；  
   - 各文档内容没有越出分区的一句话职责；  
   - 与阶段一的目录结构一致，无明显冲突。  

4. 所有上述文档：  
   - 都经过至少一轮人工基于 checklist 的 Review；  
   - 所有修改均由 AI 完成，确保“AI 主写、人检查”的工作模式已经跑通。  

在满足这些条件后，项目可以进入 **阶段三：补充关键说明内容与示例**，由 AI 在现有规范与模板下逐步扩充架构说明、模块设计文档与 DevOps 说明细节。
