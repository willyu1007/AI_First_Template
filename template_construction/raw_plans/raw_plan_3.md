# 阶段三：核心机制知识沉淀与标准流程规划

> 目标（一句话）：在阶段一的目录骨架与阶段二的规范基础上，**系统化沉淀核心机制的说明内容**，让代码大模型在“模块 / 能力 / 内容路由”三个关键层面有可依赖的知识基础，但暂不进入 hooks / DevOps / 示例模块的详细设计与实现。

---

## 1. 阶段定位与范围

### 1.1 在整体路线中的位置

- 阶段一：固定目录与空文件清单 → 解决“东西在哪儿”。  
- 阶段二：文档规范与 AGENTS 结构 → 解决“AI 到了某个目录后，知道自己能做什么、看什么、改什么”。  
- **阶段三：沉淀核心机制的说明内容 → 解决“AI 在做关键设计决策时，有统一、可靠的知识依据”。**

### 1.2 本阶段任务

**本阶段聚焦在：**

1. 将份架构说明文档、模块化开发文档、以及hook机制的说明内容，重组到 `knowledge/` 体系中；  
2. 为 **模块 / 能力 / 内容路由 / hook进程** 核心机制建立“标准工作流”说明；  
3. 建立术语表与常见误用说明，统一概念。  
 
---

## 2. 工作块 A：架构文档 → 核心概念

**目标：**  
把当前的模板架构文档：

- `modular_mannual.md`  
- `ability_routing.md`  
- `content_routing.md`  
- `agents_strategy_guide.md`  
- `hooks.md`  
- `devops_extension_guide.md`  

拆解为若干 **位置明确、可复用的知识文件**，从面向人类阅读转为面向AI阅读的风格，侧重保留高层理念和边界，主要落在：

- `knowledge/architecture/`：将直接在根级AGENTS.md被索引
- `knowledge/modules/template/` ：将用于模块初始化和模块的根级策略文档 
- `knowledge/devops/`：按需调用  

### 2.1 映射草稿与约束

1. 写一份“章节映射草稿”（放在 `template_construction/`目录下），粗略说明：  
   - modular 文档中的“模块目录结构 / 生命周期 / 注册信息”等分别映射到：  
     - `knowledge/modules/template/layout.md`  
     - `knowledge/modules/template/manifest_fields.md`  
     - `knowledge/modules/template/registration_flow.md`  
   - ability_routing 文档中的核心概念与约束，映射到：  
     - `knowledge/architecture/overview.md` 中的“能力路由”章节；  
   - content_routing 文档中的路由模型与阶段说明，映射到：  
     - `knowledge/architecture/overview.md` 中的“内容路由”章节；  
   - hooks 机制的说明以及AI如何使用hook机制，映射到：
     - `knowledge/architecture/overview.md` 中的“hook进程”章节；  
   - devops 文档：  
     - 仅提取“概念 / 目标 / 与系统其它部分的边界”，暂时汇总到合适的 architecture/devops 知识文件。  

2. 给出简短约束：  
   - 不在多个文件重复同一大段说明；  
   - DevOps 只保留概念，不写操作步骤。  

### 2.2 拆分与重写

根据映射草稿执行：

1. 对原始 6 份文档做内容拆分 / 重组，按照文档规范输出以下文件：  
   - `knowledge/architecture/overview.md`  
   - `knowledge/modules/template/layout.md`  
   - `knowledge/modules/template/manifest_fields.md`  
   - `knowledge/modules/template/registration_flow.md`  
   - （可选）少量 devops 高层理念的归档文档。  

2. 在保证内容不矛盾的前提下，对语言进行适度简化和统一：  
   - 去重、删历史背景；  
   - 保留关键概念、约束、抽象流程。  
   - 确保AI友好

### 2.3 人类抽查与反馈

- 按文件维度抽查：  
  - 是否包含映射草稿中应该出现的关键内容；  
  - 是否放错了目录（例如模块级内容不应出现在全局 architecture 中）。  
- 对问题点给出高层反馈，再由 AI 修订对应文件。  

**完成标志（A 块）：**  
- 核心内容都可以在 `knowledge/` 下找到归宿；  
- 原始文档可以只保留为人类阅读文档，在仓库中降级为次要资料。  

---

## 3. 工作块 B：典型开发场景的工作流

**目标：**  
让AI知道基础设施的在不同情况下的使用流程、hook机制如何运作。按照情景编写若干工作流说明，每份文档可以作为知识被引用、

### 3.1 确认情景与工作流

- 在`template_construction/`目录下生成典型的开发情景
- 简单概述开发情景和基础设施/hooks进程的配合
- 人工确认是否需要新增或修改
- 将文档中的内容落实成知识文档，并确认不同情景的落点，后续阶段需要形成这些知识文档的路由

### 3.2 检查点

- 流程是否与整体架构理念一致（不违背 modular 思路与路由模型）；  
- 流程是否可被 AI 直接执行为一套“规划步骤”；  
- 若有疑虑，由 AI 调整流程文档，不由人亲自重写。  

**完成标志（B 块）：**  
- 对模块 / 能力 / 内容路由至少各有一份“标准说明”文档；  
- 相关 AGENTS 能显式引用这些流程作为建议路径。  

---

## 4. 工作块 C：术语表与常见误用说明

**目标：**  
为代码大模型与未来的维护者提供一份统一的概念参考，避免在关键术语和行为模式上产生理解偏差。

### 4.1 术语表（Glossary）

建议文件位置：`knowledge/architecture/glossary.md`。

**内容要点：**

- 对以下关键术语给出简明定义 + 示例：  
  - module / module_type / module_instance；  
  - ability / low-level ability / high-level workflow / agent；  
  - hook（仅概念层面）；  
  - workdocs / knowledge / template / overview / registry；  
  - routing / content routing / ability routing；  
  - scope / topic / route / doc set 等。  
- 每个条目形式：  
  - 名称；  
  - 一句定义；  
  - 一句话示例（尽量结合本项目的目录结构）。  

### 4.2 常见误用与反例（Pitfalls / FAQ）

建议文件位置：`knowledge/architecture/pitfalls.md`。

**内容要点：**

- 列出最重要的“不要这样做”的情况，例如：  
  - 不要把长期知识只放在 `workdocs/`，不沉淀到 `knowledge/` 或模块内 docs；  
  - 不要在 `README.md` 中定义 AI 的行为规范；  
  - 不要直接创建未注册的模块目录并开始开发，而不更新 `instance_registry.yaml`；  
  - 不要跨模块随意修改他人模块的 `AGENTS.md`。  
- 每条使用“错误做法 vs 正确做法”的对比形式：  
  - **错误**：描述一个常见的错误模式；  
  - **正确**：给出符合当前架构思路的替代做法。  

### 4.3 人 / AI 分工

- AI 负责写第一版术语表与常见误用说明；  
- 人只需要从整体设计哲学的角度检查：  
  - 定义是否偏离原设想；  
  - 反例是否抓住了真正“危险 / 成本高”的行为；  
- 必要时由 AI 根据反馈重写部分条目。  

**完成标志（C 块）：**  
- `glossary.md` 以及 `pitfalls.md`存在、内容完整、且符合面向AI的文档规范；  

---

## 5. 第二、三阶段的衔接点与依赖

在执行 Phase 3 前，建议确认：

1. 阶段一：目录结构与空文件已经就绪；  
2. 阶段二：  
   - `DOCS_CONVENTION.md` 已存在并说明文档类型/路径基本规则；  
   - 根级及关键位置的 `AGENTS.md` 已有基础结构（至少包含 Scope / Allowed Operations / Knowledge Routing / Workdocs Rules / Safety）；  
   - `knowledge/` 下的分区与 README 已创建。  

Phase 3 的工作将基于上述基础设施，把架构说明“拆块化”和“场景化”，而不再改动底层骨架与规范结构。  

