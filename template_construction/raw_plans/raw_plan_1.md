# 初始模板建设规划

由于架构概述文档没有细节支撑，逐个搭建基础设施将会十分困难。相应的，我们的核心建设思路是：“打地基 → 分层往上搭”，每一层的重点都是为下一层铺路。

## 阶段一：Repo 模板骨架搭建规划

> 目标：在不纠结实现细节的前提下，**一次性固定项目的目录结构与空文件清单**，为后续文档规范、说明内容与脚本/基础设施的开发打好“地基”。

---

### 1. 阶段一范围与产出

#### 1.1 范围（做什么 / 不做什么）

**本阶段只做：**

- 确定项目根目录结构与各子目录的职责边界；
- 建立所有关键位置的 **空文件 / 占位文件**（尤其是 `AGENTS.md`、`README.md`、注册类 YAML 等）；
- 规划好 `knowledge/` 中可复用知识的分区布局。

**本阶段不做：**

- 不撰写详细策略、规范或架构说明；
- 不实现任何脚本逻辑或能力逻辑；
- 不配置 CI/CD 或环境依赖。

#### 1.2 预期产出

- 一个可初始化的 repo 模板（可直接 git init 后使用）；
- 目录树与空文件清单（即本文件中的“参考目录树”）；
- 清晰的文档角色划分约定：  
  - `AGENTS.md`：唯一面向 AI 的策略与规范文档；  
  - `README.md`：面向人的说明文档；  
  - `knowledge/`：集中存放可复用的知识文档。

---

### 2. 关键设计原则（Phase 1 固定的约束）

1. **模块是主战场，根目录只放“地图”**  
   - 真实业务开发主要落在 `modules/<module_id>/` 下；  
   - 根目录与上层目录负责说明全局结构与导航，而非承载大量业务逻辑。

2. **AGENTS / README / knowledge 的角色边界**  
   - `AGENTS.md`：  
     - 只表达“AI 在此目录范围内应如何工作”；  
     - 是策略与规范的唯一入口，其他知识文档都由它来“指路”。  
   - `README.md`：  
     - 只服务人类读者（项目概览、如何运行、如何测试等）；  
     - AI 不默认把 README 视为规范文档。  
   - `knowledge/`：  
     - 存放可复用的架构说明、模板说明、设计文档等；  
     - 由各层级的 `AGENTS.md` 进行显式引用和路由。  

3. **知识文档集中在 `knowledge/` 下管理**  
   - 包括模块模板说明（原本的 `MODULE_TEMPLATE` 概念）、架构总览、DevOps 模板等；  
   - 代码与运行配置保留在 `modules/`、`ops/`、`db/` 等业务目录中。

4. **场景工作区使用局部 `workdocs/`，不搞“全局大锅”**  
   - 初始化：`PROJECT_INIT/workdocs/`  
   - 数据库设计与演进：`db/workdocs/`  
   - 打包/部署：`ops/packaging|deploy/workdocs/`  
   - 模块内开发与迭代：`modules/<module_id>/workdocs/`  
   每个场景的 AI 工作痕迹只留在其专属的 `workdocs/` 中，避免全局混杂。

5. **基础设施集中收敛在 `.system/` 和 `scripts/`**  
   - `.system/`：声明 hooks、能力 registry、运行配置等；  
   - `scripts/`：承载调用外部系统的脚本（例如 DB、打包、部署等）。

6. **先冻结路径，后逐层填充内容**  
   - 本阶段只创建目录和空文件，不追求内容完整性；  
   - 后续阶段在此基础上补充文档规范、说明内容与脚本实现。

---

### 3. 根目录结构与职责说明

#### 3.1 根目录一级结构

```text
.
  README.md                 # 面向人类的项目简介、运行指引（后续阶段补充内容）
  AGENTS.md                 # 根级 AI 策略与导航入口
  docs/                     # 文档存放目录
  template_construction/    # 搭建repo模板的工作目录
  modules/                  # 模块系统主战场（业务模块与能力模块）
  .system/                  # AI 相关基础设施（hooks、ability registry、knowledge 等）
  PROJECT_INIT/             # 项目初始化场景专用工作区
  db/                       # 数据库相关结构与工作区
  ops/                      # 打包与部署相关结构与工作区
  scripts/                  # 通用脚本入口
```

#### 3.2 `modules/` 结构

```text
modules/
  AGENTS.md                 # 跨模块 AI 策略与导航入口
  ROUTING.md                # 模块体系的根级知识路由入口（后续补充内容）

  routes/                   # 根级 topic / scope 的路由配置（YAML）
    README.md               # 说明 routes 的使用方式（后续阶段补充内容）

  overview/                 # 模块注册与全局视图
    README.md
    type_registry.yaml      # 注册模块类型（module_type）
    instance_registry.yaml  # 注册具体模块实例（module_id -> 元信息）
    instance_maturity.yaml  # 模块成熟度 / 稳定性信息
    interfaces.yaml         # 跨模块接口与依赖关系视图

  # 具体模块实例目录（Phase 1 仅预留结构，不强制创建具体模块）
  # <module_id>/
  #   AGENTS.md
  #   MANIFEST.yaml
  #   ROUTING.md
  #   ABILITY.md
  #   workdocs/
  #   docs/
  #   ...
```

在阶段一中，**可以不创建任何真实模块实例**，只需确保 `modules/` 自身结构完善、注册文件存在且为空或占位。

#### 3.3 `knowledge/` 结构（含模块模板知识）

```text
knowledge/
  README.md                 # 对整个知识库分区的说明

  architecture/             # 全局架构类知识
    overview.md             # 对应 ARCHITECTURE_OVERVIEW.md 的可复用版本或细化内容

  modules/
    template/               # 原“MODULE_TEMPLATE”概念的知识化形态
      AGENTS.md             # 指导 AI 如何脚手架新模块实例
      layout.md             # 标准模块目录结构说明
      manifest_fields.md    # MANIFEST.yaml 字段说明与约束
      registration_flow.md  # 模块注册流程说明
      examples/
        homework_system.md  # 示例模块拆分与脚手架案例（占位）

  devops/                   # DevOps 相关通用知识
    packaging_template.md   # 打包场景的通用模板说明（占位）
    deploy_template.md      # 部署场景的通用模板说明（占位）
```

说明：

- `knowledge/modules/template/` 取代了原本的顶层 `MODULE_TEMPLATE/` 目录；  
- 所有内容在 Phase 1 仅需以空文件或极简占位描述存在，详细内容留待后续阶段补充。

#### 3.4 `.system/` 结构

```text
.system/
  hooks/                    # Hook 声明文件（YAML 等）
    AGENTS.md
    README.md               # 描述支持的 hook 类型与文件结构（占位）

  registry/                 # 能力与配置的集中注册中心
    AGENTS.md
    low-level/
      README.md             # 说明低级能力注册的约定（占位）
    high-level/
      README.md             # 说明高级能力（workflow/agent）注册的约定（占位）
    config/
      README.md             # 运行配置类注册（占位）

  logs/                     # 日志输出目录（建议仅放 .gitignore 或空）
    .gitignore
  cache/                    # 缓存目录（建议仅放 .gitignore 或空）
    .gitignore
```

说明：

- Phase 1 只需要建立目录和 README/.gitignore 文件，不需要任何实际配置内容。  
- 后续阶段在此处增加 hooks 配置文件、能力注册 YAML 等。

#### 3.5 `PROJECT_INIT/` 结构

```text
PROJECT_INIT/
  doc/
    README.md               # 面向人的初始化说明（占位）

  AI/
    AGENTS.md               # 初始化阶段 AI 的策略与工作流程说明（占位）
    prompts.md              # 初始化相关的 prompt 片段（占位）

  workdocs/
    README.md               # 说明初始化阶段 workdocs 使用方式（占位）
```

说明：

- `PROJECT_INIT/` 仅用于项目初始化场景，不参与后续模块开发；  
- 通过单独的 `AGENTS.md` 和 `workdocs/`，将初始化阶段的 AI 工作与其他场景隔离。

#### 3.6 `db/` 结构

```text
db/
  AGENTS.md

  schema/
    README.md               # 模型/表结构相关说明（占位）

  migrations/
    README.md               # 迁移脚本说明与组织方式（占位）

  config/
    README.md               # 数据库连接/环境相关的配置结构说明（占位）

  samples/
    README.md               # 示例数据的组织方式说明（占位）

  workdocs/
    README.md               # DB 设计与演进的 AI 工作区说明（占位）
```

#### 3.7 `ops/` 结构

```text
ops/
  packaging/                # 打包场景
    AGENTS.md               # 打包相关 AI 策略与流程说明（占位）
    scripts/
    workdocs/

  deploy/                   # 部署场景
    AGENTS.md               # 部署相关 AI 策略与流程说明（占位）
    scripts/
    workdocs/
```

#### 3.8 `scripts/` 结构

```text
scripts/
  hooks/
    README.md               # hook 处理脚本说明（占位）
    AGENTS.md               # hook 处理脚本说明（占位），面向AI阅读

  devops/
    README.md               # 通用 DevOps 脚本说明（占位）
    AGENTS.md               # 通用 DevOps 脚本说明（占位），面向AI阅读

  # 其他脚本入口在后续阶段按需增加
```

#### 3.9 `docs` 结构

```text
docs/
  AGENTS.md               # hook 处理脚本说明（占位），面向AI阅读
  human/
  ai/
    ARCHITECTURE_OVERVIEW.md  # 项目总体架构概览（后续阶段补充内容）
    DOCS_CONVENTION.md        # 文档与命名规范说明（后续阶段补充内容）
```

---

### 4. 阶段一任务清单（可直接用于执行）

#### 4.1 必做任务

1. **创建根目录及一级文件**
   - 建立：`README.md`, `AGENTS.md`, `ARCHITECTURE_OVERVIEW.md`, `DOCS_CONVENTION.md`（内容可为空或简单占位）。

2. **创建一级子目录**
   - 创建：`modules/`, `knowledge/`, `.system/`, `PROJECT_INIT/`, `db/`, `ops/`, `scripts/`。

3. **在各子目录下创建结构与空文件**
   - 按“根目录结构与职责说明”部分中的示例结构，逐一创建目录与占位文件。  
   - 所有 `AGENTS.md` 允许仅包含一两行占位文字（例如“TODO: fill AI strategy for XXX scope”）。  
   - 各 README 仅需说明该目录的职责或留空。  
   - `.system/logs/` 与 `.system/cache/` 建议仅创建 `.gitignore`，内容可为 `*` 或注释。

4. **保证目录结构在版本库中可见**
   - 对仅含空目录的路径，通过增加 `README.md` 或 `.gitkeep`/`.gitignore` 的方式，使其被版本控制系统跟踪。
  
将以上信息整理并记录。

#### 4.2 后续准备

整理以下内容

- `DOCS_CONVENTION.md` 初步说明：  
  - `AGENTS.md` / `README.md` / `knowledge/` 三者的角色差异；  
  - `workdocs/` 的使用原则（仅用于当前场景，后续沉淀为文档或脚本）。  
- 不同子目录下 `AGENTS.md` 的存在意义；  
- 如何从根级导航到 `modules/AGENTS.md` 和 `knowledge/` 中的关键知识。

---

### 5. 参考目录树（包含空文件清单）

> 本节可以作为创建 Phase 1 初始 commit 的直接参考。实际实现时可使用脚本批量创建。

```text
.
├── README.md
├── AGENTS.md
├── ARCHITECTURE_OVERVIEW.md
├── DOCS_CONVENTION.md
├── modules
│   ├── AGENTS.md
│   ├── ROUTING.md
│   ├── routes
│   │   └── README.md
│   └── overview
│       ├── README.md
│       ├── type_registry.yaml
│       ├── instance_registry.yaml
│       ├── instance_maturity.yaml
│       └── interfaces.yaml
├── knowledge
│   ├── README.md
│   ├── architecture
│   │   └── overview.md
│   ├── modules
│   │   └── template
│   │       ├── AGENTS.md
│   │       ├── layout.md
│   │       ├── manifest_fields.md
│   │       ├── registration_flow.md
│   │       └── examples
│   │           └── homework_system.md
│   └── devops
│       ├── packaging_template.md
│       └── deploy_template.md
├── .system
│   ├── hooks
│   │   └── README.md
│   ├── registry
│   │   ├── low-level
│   │   │   └── README.md
│   │   ├── high-level
│   │   │   └── README.md
│   │   └── config
│   │       └── README.md
│   ├── logs
│   │   └── .gitignore
│   └── cache
│       └── .gitignore
├── PROJECT_INIT
│   ├── doc
│   │   └── README.md
│   ├── AI
│   │   ├── AGENTS.md
│   │   └── prompts.md
│   └── workdocs
│       └── README.md
├── db
│   ├── schema
│   │   └── README.md
│   ├── migrations
│   │   └── README.md
│   ├── config
│   │   └── README.md
│   ├── samples
│   │   └── README.md
│   └── workdocs
│       └── README.md
├── ops
│   ├── packaging
│   │   ├── AGENTS.md
│   │   ├── scripts
│   │   │   └── README.md
│   │   └── workdocs
│   │       └── README.md
│   └── deploy
│       ├── AGENTS.md
│       ├── scripts
│       │   └── README.md
│       └── workdocs
│           └── README.md
└── scripts
    ├── hooks
    │   └── README.md
    └── devops
        └── README.md
```

---

### 6. 阶段一完成判定标准

当满足以下条件时，可认为 Phase 1 已完成：

1. 按本规划建立了完整的目录结构和空文件清单；  
2. 所有提到的 `AGENTS.md` 与关键 README 文件均已存在（哪怕只是简单的 `TODO` 占位）；  
3. Git 仓库中能完整看到上述目录树；  
4. 后续阶段在此基础上可以直接开始：  
   - 填写 `DOCS_CONVENTION.md`；  
   - 在各级 `AGENTS.md` 中补充策略与导航；  
   - 在 `knowledge/` 中逐步增加模块模板与架构说明。

完成这些后，整个项目的“地基”即算打好，可以进入**阶段二：文档规范骨架与 AGENTS 结构设计**。
