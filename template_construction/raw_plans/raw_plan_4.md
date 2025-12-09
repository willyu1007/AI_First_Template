# 阶段四：注册写入脚本与协同机制规划

> 目标（一句话）：为路由、能力、Hook 等各类注册信息提供**统一的脚本写入能力**，并明确不同注册之间的**协同与一致性机制**，但暂不实现完整脚手架流程。

---

## 1. 阶段定位与范围

### 1.1 在整体路线中的位置

- 阶段一：目录骨架与空文件清单 —— 解决“东西在哪儿”。  
- 阶段二：文档规范与 AGENTS 结构 —— 解决“AI 到每个目录后能看懂规则和入口”。  
- 阶段三：核心机制知识与标准流程 —— 解决“模块 / 能力 / 内容路由这些大决策怎么做”。  
- **阶段四：注册写入脚本与协同机制 —— 解决“如何安全地把最终决定写进各类 registry 文档”。**  

### 1.2 本阶段的两大目标

1. **完成各类注册信息的“最终写入”脚本**  
   - 支持对以下类型的注册进行**添加 / 更新**：  
     - 模块注册（type / instance / maturity / interfaces）；  
     - 内容路由（content routing）；  
     - 能力注册（低级能力 / 高级 workflow/agent）；  
     - Hook 注册。  
   - 写入脚本只负责：  
     - 按 schema 校验给定条目；  
     - 合并入对应 YAML 文件；  
     - 保持结构、排序与去重；  
     - 写回文件。  
   - **不负责**生成条目内容（即不做交互式问答或复杂脚手架）。  

2. **明确注册之间的协同机制与一致性规则**  
   - 当新增/修改某一注册（如能力）时，需要检查 / 提示哪些其它注册与文档（如模块实例、路由）；  
   - 用文档 + 代码接口的形式定义这些协同机制，**为下一阶段的脚手架搭建打好基础**。  

---

## 2. 目录与职责约定（Schema vs Registry 内容）

### 2.1 Schema 统一放置在 `.system/`

**约束：**所有“注册类型的 Schema YAML”统一放在：

```text
.system/
  registry/
    schemas/
      module_type.schema.yaml
      module_instance.schema.yaml
      module_maturity.schema.yaml
      module_interface.schema.yaml
      route.schema.yaml
      ability_low.schema.yaml
      ability_high.schema.yaml
      hook.schema.yaml
      # 其他需要的 schema 可在此扩展
```

这些文件的职责：

- 定义每一类注册条目的字段结构（id、引用、描述、状态等）；  
- 用于写入脚本的结构校验；  
- 不存放具体业务条目。  

### 2.2 实际“已注册内容”的 YAML 存放位置

实际的注册内容按功能分散存放，是**查阅与写入的主入口**：

1. **模块注册**（仅内容）

```text
modules/
  overview/
    type_registry.yaml        # 已有 module_type
    instance_registry.yaml    # 已注册 module_instance
    instance_maturity.yaml    # 模块成熟度/状态
    interfaces.yaml           # 跨模块接口关系
```

2. **内容路由注册（content routing）**

```text
modules/
  routes/
    <scope_or_topic>.yaml     # 每个文件存放一组路由规则
```

3. **能力注册（ability routing 相关）**

```text
.system/
  registry/
    low-level/
      <group_or_domain>.yaml  # 低级能力条目列表
    high-level/
      <group_or_domain>.yaml  # 高级 workflow/agent 条目列表
```

4. **Hook 注册**

```text
.system/
  hooks/
    <hook_type>.yaml          # 比如 PromptSubmit.yaml / PreAbilityCall.yaml 等
```

**查阅 vs 管理：**

- 日常查看“当前有哪些东西” → 看上述 registry 内容 YAML；  
- 更新 / 新增条目 → 通过 **阶段四要实现的写入脚本** 完成；  
- Schema 只在写入时校验使用，不直接作为查阅入口。  

---

## 3. 工作块 A：注册类型盘点与 Schema 固化

**目标：**用文档和 schema 文件把“有哪些 registry、结构长什么样”一次性说明白。

### 3.1 任务 A1：注册类型清单确认

在 `.system/registry/README.md`中：

- 列出所有注册类型：  
  - 模块类型（module_type）  
  - 模块实例（module_instance）  
  - 模块成熟度（module_maturity）  
  - 模块接口（module_interface）  
  - 内容路由（route）  
  - 低级能力（ability_low）  
  - 高级能力 / workflow/agent（ability_high）  
  - Hook（hook）  
- 对每一类给出：  
  - **schema 文件路径**（`.system/registry/schemas/*.schema.yaml`）；  
  - **内容文件路径**（如 `modules/overview/instance_registry.yaml`）；  
  - 简短说明该 registry 的用途。  

### 3.2 任务 A2：Schema 文件定义与对齐

由 AI 根据阶段三的知识文档，生成各类型的 schema 文件初稿：

- 每个 schema 至少包括：  
  - 条目主键字段（如 `id` / `module_id` / `ability_id`）；  
  - 与其他实体的关联字段（如 `module_id`、`ability_ids` 等）；  
  - 基本描述字段（`description`、`status`、`tags` 等）；  
  - 枚举字段与必填/可选约束。  

需要给人工确认的内容：

- 确认字段命名和引用关系与整体架构一致；  
- 不亲自写 YAML，只提供修正意见，交由 AI 更新 schema。  

**完成标志（A 块）：**

- `.system/registry/schemas/*.schema.yaml` 已创建并通过人工审核；  
- 有一份文档清晰列出“schema 文件 ↔ registry 内容文件”的映射关系。  

---

## 4. 工作块 B：最小注册写入脚本（只做最终写入）

**目标：**实现一组脚本，使得给定**已经准备好的条目数据**时，可以安全地将其写入相应的 registry 内容 YAML。

### 4.1 任务 B1：通用写入库

建议在 `scripts/devops/registry/` 下建立通用库：

```text
scripts/
  devops/
    registry/
      __init__.py
      common.py
      modules.py
      routes.py
      abilities.py
      hooks.py
```

- `common.py`：  
  - `load_yaml(path) / dump_yaml(path, data)`  
  - 通用的 `insert_or_update_entry(data, key_field, new_entry)`  
  - 基础排序与去重逻辑  
  - 读取 schema 并进行结构校验的函数：  
    - `validate_entry(entry, schema_path)`  

- `modules.py`：  
  - 根据 module 相关 schema 与 `modules/overview/*.yaml` 实现：  
    - 新增/更新 module_type / module_instance / module_maturity / module_interface 条目。  

- `routes.py`：  
  - 根据 route schema 与 `modules/routes/*.yaml` 实现路由条目的新增/更新。  

- `abilities.py`：  
  - 依据 ability_low / ability_high schema 与 `.system/registry/low-level|high-level` 实现能力条目的新增/更新。  

- `hooks.py`：  
  - 根据 hook schema 与 `.system/hooks/*.yaml` 实现 Hook 条目的新增/更新。  

### 4.2 任务 B2：简单 CLI 入口（可选但推荐）

在 `scripts/devops/registry/` 或顶层 `scripts/devops/` 下添加若干入口脚本，例如：

- `registry_add_entry.py`  
  - 参数示例：  
    - `--type module_instance`  
    - `--entry-file path/to/entry.yaml` 或 `--entry-json '{...}'`  

脚本逻辑：

1. 解析 `--type`，决定调用哪个子模块（modules/routes/abilities/hooks）；  
2. 从 entry 文件 / JSON 载入条目；  
3. 使用对应 schema 校验；  
4. 调用写入函数更新目标 registry 内容文件。  

**注意：**

- 本阶段只要求：**如果条目数据合法，就能正确写入**；  
- 条目数据如何生成、交互式收集，留给下一阶段脚手架来做。  

**完成标志（B 块）：**

- 对每一类注册（模块 / 路由 / 能力 / Hook），都有可调用的写入函数；  
- 手工构造一两条测试 entry，可以通过脚本写入对应 YAML，且不破坏原有结构。  

---

## 5. 工作块 C：协同机制设计与一致性校验接口

**目标：**明确各类注册之间需要维持的“一致性关系”和“不变量”，并在代码中提供基础校验接口，为未来的脚手架与自动化流程提供依赖。

### 5.1 任务 C1：协同关系文档化

在 `.system/registry/registry_coordination.md` 中，描述：

1. **能力 ↔ 模块实例**  

- 新增能力时应检查：    
  - 是否需要在模块实例的能力路由中引入此能力（此阶段可只提示，不强制写入）。  

2. **路由 ↔ 模块实例**  

- 新增 content route 时应检查：  
  - 是否需要在模块实例的文档路由相关文件中引入此能力（此阶段可只提示，不强制写入）。  
  - 是否与现有 route 冲突（例如 scope/topic 完全重叠且无优先级说明）。  

3. **Hook ↔ 能力 / 路由**  

- 新增 Hook 注册时应检查：  
  - 绑定的能力、阶段名称合法且存在；  
  - 若某 Hook 声称与特定 routing 阶段绑定，该阶段在当前 routing 模型中是合法值。  

4. （其他协同关系可继续补充）

每条协同关系需给出：

- “当 A 变更时，应检查 B/C”；  
- 推荐的处理方式：阻塞写入 / 给出 Warning / 允许但记录。  

### 5.2 任务 C2：基础一致性校验接口

在 `scripts/devops/registry/*.py` 中，为未来脚手架留下可调用的校验函数，例如：

- `modules.py`：  
  - `check_module_instance_consistency(new_instance_entry) -> list[Problem]`  

- `abilities.py`：  
  - `check_ability_addition_consistency(new_ability_entry) -> list[Problem]`  
    - 检查 module 是否存在、是否与已有能力冲突等。  

- `routes.py`：  
  - `check_route_addition_consistency(new_route_entry) -> list[Problem]`  
    - 检查目标模块/能力是否存在、是否造成明显冲突。  

- `hooks.py`：  
  - `check_hook_addition_consistency(new_hook_entry) -> list[Problem]`  

**阶段四要求：**

- 这些函数至少能：  
  - 读取当前 registry 内容（模块 / 路由 / 能力 / hook）；  
  - 根据协同规则做基本检查；  
  - 返回问题列表（字符串或结构化对象）。  
- 写入脚本可以选择性地在写入前调用这些校验：  
  - 默认开启，遇到 **严重问题** 时中止写入；  
  - 提供 `--no-check` 选项允许在特殊情况下跳过。  

### 5.3 人 / AI 分工

- AI：  
  - 主导编写 `registry_coordination.md` 初稿；  
  - 实现各校验函数的初版逻辑。  
- 人：  
  - 调整协同规则（哪些是 hard error，哪些是 warning）；  
  - 用少量实际场景验证校验逻辑是否符合预期。  

**完成标志（C 块）：**

- 有一份清晰的协同关系文档；  
- 对主要注册类型存在对应的 `check_*_consistency` 函数；  
- 写入脚本在默认模式下会调用这些检查。  

---

## 6. 阶段四完成判定标准（Definition of Done）

满足以下条件时，可认为 Phase 4 已完成：

1. **Schema 与 registry 映射清晰稳定**  
   - `.system/registry/schemas/*.schema.yaml` 已存在且结构明确；  
   - 有一份文档列出每个 schema 对应的实际 registry 内容文件位置。  

2. **所有主要注册类型都有“最终写入脚本”**  
   - 模块 / 路由 / 能力 / Hook 的注册 YAML，均可以通过脚本安全地新增/更新条目；  
   - 手工构造的测试条目可以顺利写入，且不破坏原有结构和排序。  

3. **协同机制有文档 & 校验实现**  
   - `registry_coordination.md` 中写明：新增/修改某一 registry 时需要检查哪些其他 registry；  
   - 代码层面存在对应的 `check_*_consistency` 函数，写入脚本可调用。  
