# 工业研发底座模型化架构专题讨论纪要

状态：`Discussion Summary v5.0`  
日期：`2026-03-11`  
参与者：`DuanDuanPan（产品架构）`、`Winston（架构师）`  
用途：记录围绕"模型化"主题的深入讨论结论，作为蓝图 `v0.5` 模型化章节的输入。

注：本版已按 `2026-03-11` 的架构分析结论，和 `v0.5` 主蓝图同步模型继承命名轴、集成职责边界、运行时服务落点与复用冲突规则。

---

## 1. 为什么需要模型化

### 1.1 产品形态转变

```
现状：通用开发平台 + 每次从 0 建模 + 项目制交付
理想：领域专业平台 + ITP 导入 + 开箱即用 + 受控差异化
```

### 1.2 四个层次

| 层次 | 模型化带来的变化 |
|---|---|
| 产品交付形态 | ITP 导入即可用，交付周期月级→周级 |
| 平台工程效率 | 注册对象 → 自动获得存储、权限、搜索、审计、主线追踪 |
| 多领域兼容演进 | SchemaVersion + 兼容性规则 + 差量升级 |
| 标准语义互操作 | 标准语义可计算映射，自动生成交换格式 |

### 1.3 定义

> **模型化 = 把业务概念从硬编码提升为平台可注册、可版本化、可自动联动、可差异化扩展、可标准化交换的一等架构资产。**

---

## 2. 模型化是工业知识的积累点

### 甲方五类核心需求

| 需求 | 关切 | 架构影响 |
|---|---|---|
| 知识主权 | 归属、导出、不锁定 | 开放格式，区分 `ownerType` |
| 知识保护 | 租户隔离、密级、导出审批 | 安全治理覆盖模型对象 |
| 知识演进 | 版本回溯、灰度验证 | `SchemaVersion` 分层继承 + `localDelta` 差量管理 |
| 知识复用 | 模板库、跨项目引用 | 客户自建模板包 |
| 知识治理 | 变更控制、影响分析 | 审批 + TraceLink |

---

## 3. 模型继承轴与实例关系

### 3.1 `M` 轴四级继承

| 层级 | 定义 | 控制者 | 变更频率 |
|---|---|---|---|
| M0 平台内核模型 | Test, DataSet, Workflow... | 平台团队 | 年级 |
| M0.5 公共业务对象 | Project, Organization, Device... | 平台团队 | 半年级 |
| M1 领域业务模型 | 风洞/协同研发/MIS ITP | 行业团队 | 季度级 |
| M2 客户差异化模型 | 扩展字段、规则、流程、本地差量 | 甲方 | 月级 |

### 3.2 实例数据不属于继承层

- 实例数据是 `M0-M2` 发布后的运行时产物，不是模型继承的一层。
- 实例对象在创建、导入或升级时必须绑定生效 `schemaVersionRef`，用于追溯、兼容性判断和影响分析。
- 模型版本回滚不等于实例数据回滚；已正式提交的实例数据只能通过迁移、修复或失效标记处理，不能无痕回到上一个模型版本。

```text
模型继承轴（M）：M0 平台内核 -> M0.5 公共业务 -> M1 领域 ITP -> M2 客户差异
运行时服务轴（L）：L1 体验 -> L2 应用 -> L3 领域服务 -> L4 共享服务 -> L5 数据 -> L6 集成 -> L7 治理
实例数据：运行时对象，绑定 schemaVersionRef，不进入 M 轴
```

### 3.3 演进策略

阶段一快照复制 → 阶段二提炼标准 → 阶段三完整继承。**标准 ITP 从项目中提炼，不预先设计。**

---

## 4. 模型包（ITP）统一结构

### 4.1 包结构（三域通用）

```
{domain}-itp-v{version}/
├── manifest.yaml
├── schema/
│   ├── object-models/           ← 领域对象模型（引用共享语义资产，不内嵌副本）
│   ├── data-organization/       ← 数据组织模型
│   └── semantic-refs/           ← 共享语义资产引用清单（MeasurementQuantity / PhysicalUnit / Environment / TODS）
├── templates/
│   ├── lifecycle-processes/     ← 全生命周期流程（FCSDP）
│   ├── activities/              ← 活动模板（ICOM）
│   ├── toolflows/               ← 工具流模板
│   ├── data-definitions/        ← 数据引擎定义
│   ├── control-processes/       ← 管控流程（BPMN/Activiti）
│   └── event-chains/            ← 事件联动链
├── behavior/
│   ├── views/                   ← 视图与页面模板（含完整 UI Schema 和 FED 扩展声明）
│   ├── server-units/            ← 后端扩展单元（替代原 plugins/，见蓝图 v0.6 §4.6）
│   └── client-units/            ← 前端扩展单元（替代原 plugins/，见蓝图 v0.6 §4.6）
└── runtime/
    ├── master-data/             ← 主数据规格与就绪要求
    ├── connectors/              ← 集成需求声明与连接器契约
    ├── compliance/              ← 合规与审计规则声明
    └── seed-data/               ← 种子数据/初始化数据
```

### 4.2 三域填充重心

| 区域 | TDM | CDM | MIS |
|---|---|---|---|
| `schema/` | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| `templates/activities` | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| `templates/control-processes` | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| `templates/event-chains` | ⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| `behavior/plugins` | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| `runtime/connectors` | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| `runtime/compliance` | ⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### 4.3 四条铁律

只声明不硬编码 / 引用优先于复制 / 可增量可回滚 / 可单独验证

---

## 5. 建模工具的对象引用能力

三种来源（平台🔒/共享📎/自定义✏️）、三种命名空间（`platform://` / `shared://` / `local://`）、四个要求（可浏览/引用非复制/完整性校验/版本绑定）。

---

## 6. 协同研发域：ICOM 模板化体系

### 核心思路

不按行业切 ITP，用通用模板框架 + 内容填充。活动模板是 ICOM 核心调度点，但不单独落成新的业务主状态机服务。

```
编排主控：Task Orchestration Service
        ├── 持有：活动实例状态 / 生命周期阶段状态 / 事件链执行状态
        ├── 调用：Workflow Service（Activiti）处理人工审批子流程
        ├── 调用：Toolflow Orchestration 处理工具执行
        ├── 调用：Object Model Registry 做模式/引用校验
        └── 调用：Trace Service 写 TraceLink

领域聚合：Requirement / WorkPackage / Test / Contract ...
        └── 继续持有各自业务真相状态，不交给编排服务托管
```

活动模板（ICOM）负责声明输入输出对象、工具流使能、审批挂接和主线要求；Task Orchestration 负责解释和执行这些声明。

---

## 7. 数字化运营域：规则驱动 + 事件联动

### 7.1 核心特征

对象简单但规则极度个性化，模型化重心在规则+联动+合规。

### 7.2 外部集成：ITP 声明 + 平台承载 + 受控扩展

- ITP 负责在 `runtime/connectors` 中声明所需外部能力、数据源范围和映射契约。
- 标准对接由平台集成能力承载：`Integration Gateway` 负责 `API/Event/File` 接入，`Data Source Management` 负责数据源注册和读写范围，`Standard Exchange` 负责 `TODS/ATF/XML` 等标准交换。
- 只有标准集成能力覆盖不到的客户特有逻辑，才进入受控扩展路径；优先采用连接器扩展，插件框架可用时再走插件挂载。
- 交付模型：平台 + ITP + 客户适配扩展，而不是“平台 + ITP + 所有集成都做成插件包”。

---

## 8. 事件联动链架构

### 8.1 与 Workflow/Toolflow 的分工

`Workflow Service` 管"人与人的审批子流程"，`Task Orchestration` 管"活动、阶段、事件链等编排实例状态"，`Toolflow Orchestration` 管"工程工具步骤执行"。审批通过→发事件→联动链接手；Task Orchestration 汇总编排状态，但不改变 `L3` 聚合的业务真相状态。

### 8.2 五个设计要点

条件分支短路 / 混合同步异步 / 每步独立失败策略 / 链路嵌套互斥 / 全链路可追溯

### 8.3 实现

基于 Task Orchestration Service 升级，内化三类能力：活动模板解析、生命周期阶段编排、事件触发与联动步骤执行。

### 8.4 状态归属

- `L3` 领域聚合持有业务对象真相状态。
- `Task Orchestration` 持有活动实例、阶段实例和联动执行实例状态。
- `Workflow Service / Activiti` 仅持有审批子流程状态。
- `Toolflow Orchestration` 仅持有工具执行运行时状态。

---

## 9. 模型化体系全景图

### 9.1 六个维度

| 维度 | 问题 | 回答 |
|---|---|---|
| 模型内容 | 是什么？ | 四区统一包，三域不同重心 |
| 模型继承 | 怎么分层？ | `M0→M0.5→M1→M2`，实例数据不在继承轴内 |
| 建模工具 | 怎么造？ | 六类工具 + 引用机制 |
| 运行时引擎 | 怎么跑？ | 四个正式服务组 + `Trace Service` 协同能力 |
| 模型治理 | 怎么管？ | 版本/兼容/差量/合规/审批/影响分析 |
| 业务域覆盖 | 覆盖什么？ | TDM/CDM/MIS 统一框架 |

### 9.2 正式服务落点与能力收敛

| 概念能力 | 实施落点 | 状态 | 主责状态/能力 |
|---|---|---|---|
| 元模型注册中心 | `Object Model Registry + Meta-model Extension` | 升级 | 模型定义、`SchemaVersion`、模式校验 |
| 活动模板引擎 | 收敛到 `Task Orchestration` | 收敛 | 活动实例状态 |
| 生命周期引擎 | 收敛到 `Task Orchestration` | 收敛 | 阶段实例状态 |
| 事件联动链引擎 | 收敛到 `Task Orchestration` | 收敛 | 联动执行状态 |
| 管控流程引擎 | `Workflow Service / Activiti` | 不变 | 审批子流程状态 |
| 工具流引擎 | `Toolflow Orchestration` | 升级 | 工具执行状态 |
| 数据引擎 | `Object Model Registry + Trace Service` 协同 | 协同能力 | 模式校验、`TraceLink` 投影与主线计算 |

正式运行时服务组可收敛为 `Object Model/Meta-model`、`Task Orchestration`、`Workflow/Activiti`、`Toolflow Orchestration`，由 `Trace Service` 作为横切主线能力协同。

### 9.3 十一项关键架构决策

| 编号 | 决策 |
|---|---|
| AD-1 | ITP 包结构三域统一，内容重心不同 |
| AD-2 | 模型继承采用 `M` 轴（`M0/M0.5/M1/M2`），实例数据移出继承体系并绑定 `schemaVersionRef` |
| AD-3 | 标准 ITP 从项目中提炼 |
| AD-4 | 测量语义由共享语义资产层治理，对象模型只引用不复制 |
| AD-5 | 建模工具三种引用命名空间 |
| AD-6 | 协同研发用 ICOM 模板框架 |
| AD-7 | `Task Orchestration` 持有编排实例状态，`Workflow Service` 仅管理审批子流程 |
| AD-8 | 活动模板、生命周期和事件联动能力收敛到 `Task Orchestration`，不另起业务主状态机 |
| AD-9 | ITP 声明集成需求，平台集成能力承载标准对接，客户特有适配逻辑走受控扩展 |
| AD-10 | 复用叠加必须采用显式 `overrides` 和阻断式冲突检查 |
| AD-11 | `runtime/` 与 `semantic-refs/` 声明必须进入依赖图谱和影响分析 |

---

## 10. 多级复用体系

### 10.1 四级复用粒度

| 级别 | 粒度 | 示例 |
|---|---|---|
| Level 1 单个模板 | 单个活动/流程/对象/事件链 | 数据质量检查活动模板 |
| Level 2 模板组 | 少量紧密关联的模板 | 检查活动 + 检查规则 + 异常处理链 |
| Level 3 能力模块 | 一组完整业务能力 | 质量管理 = 计划+检验+NCR+CAPA+看板 |
| Level 4 ITP 组合 | 多个 ITP/模块叠加 | 基础研发 + MBSE扩展 + 航天特化 |

### 10.2 实现机制

- 每个模板有独立可寻址标识，支持跨包引用
- 引用语法：`{package}://{type}/{id}@{version}`
- 支持 `extends` + `overrides`（引用标准模板后只覆盖需调整部分）
- 能力模块独立于 ITP 定义，ITP 通过 `includes` 引用模块

### 10.3 合并优先级与冲突规则

- 合并优先级（从低到高）：`M0 平台内核 < M0.5 公共业务 < capability module < M1 ITP 自有 < M2 客户差量`
- 优先级由 `manifest` 和层级决定，不依赖文件加载顺序。
- 同一逻辑 `id` 默认阻断，只有显式声明 `overrides` 才允许覆盖。
- 菱形依赖中若同一模板出现多个版本，装载时阻断，必须指定胜出版本。
- `M2` 不能放宽 `M1` 已发布的 `required`、`securityLevel` 等安全/合规约束。
- 任何会使既有实例失效的 `cardinality` 变化，都必须附迁移计划和兼容性报告。
- 能力模块回滚以模块为粒度，卸载定义与注册项，但不自动删除已产生实例数据；实例数据标记为“模型已卸载/待迁移”。

```yaml
# 能力模块定义
capabilityModule:
  id: "quality-management"
  contains:
    objectModels: ["QualityPlan", "NCR", "CAPA"]
    activities: ["quality-planning", "inspection-execution"]
    controlProcesses: ["ncr-handling-flow"]
    eventChains: ["ncr-to-capa-propagation"]
  dependencies:
    requiredObjects: ["platform://Project", "platform://Deliverable"]

# ITP 引用能力模块
package:
  id: "tdm-windtunnel"
  includes:
    - "capability://quality-management@1.0"
    - "capability://equipment-management@1.0"
  ownTemplates:
    - "windtunnel-test-activity"
```

---

## 11. 关联约束与影响分析

### 11.1 模板与运行时声明依赖类型

```
活动模板 ──使能──→ 工具流模板
活动模板 ──输入/输出──→ 数据对象
活动模板 ──依赖──→ 主数据声明
活动模板 ──控制──→ 管控流程
生命周期模板 ──步骤──→ 活动模板
事件联动链 ──触发/操作──→ 数据对象
事件联动链 ──回调──→ 管控流程
事件联动链步骤 ──调用──→ 连接器声明
视图模板 ──绑定──→ 数据对象
视图模板 ──选用──→ 壳模板
视图模板 ──包含──→ 容器节点 / 组件节点
组件节点 ──绑定──→ 对象字段
模板变体 ──命中──→ 阶段 / 角色 / 任务类型
模板 patch ──覆盖──→ 视图节点 / 字段绑定 / 动作绑定
动作绑定 ──调用──→ 动作模板 / 提交命令
动作绑定 ──引用──→ 规则集
上下文字段映射 ──读取──→ 当前页 / 当前行 / 当前表单 / 运行时上下文
管控流程/导出动作 ──约束于──→ 合规声明
对象模型 ──初始化可选项──→ 种子数据
连接器声明 ──承接于──→ 平台集成能力
semantic-refs ──引用──→ 共享语义资产
```

### 11.2 三层保护机制

**第一层：依赖图谱自动构建。** ITP 装载时平台自动扫描 `templates/`、`behavior/`、`runtime/` 和 `schema/semantic-refs/` 下的引用关系，构建模型级依赖图谱。

**第二层：删除/修改前影响分析。** 客户裁剪时平台自动展示直接影响和间接影响，提供替换、降级或取消选项。

**第三层：约束强度分级。**

| 依赖关系 | 约束强度 | 删除行为 |
|---|---|---|
| 活动/事件链/视图 → 数据对象 | **强制** | 阻止，必须先解除引用 |
| 连接器声明 → 平台集成能力 | **强制** | 阻止安装或激活 |
| `mandatory` 主数据声明 → 活动模板 | **强制** | 阻止初始化或启动 |
| `mandatory` 合规声明 → 管控流程/导出动作 | **强制** | 阻止发布、高危动作或导出 |
| 活动 → 使能工具流 | **建议** | 允许，活动降级为"手动操作" |
| 活动 → 控制流程 | **建议** | 允许，活动变为"无管控" |
| `advisory` 主数据声明 → 活动模板 | **建议** | 允许，记录告警并在启动时检查 |
| `advisory` 合规声明 → 管控流程/导出动作 | **建议** | 允许，记录审计"合规策略缺失" |
| 事件联动链步骤 → 非阻断连接器声明 | **建议** | 允许，步骤标记"外部能力不可用" |
| 生命周期 → 步骤活动 | **弱** | 允许，步骤标记"已跳过" |
| 对象模型 → 种子数据 | **弱** | 允许，无预置数据初始化 |

**原则：数据对象与平台承载能力相关=强制；主数据/合规依赖按 `mandatory|advisory` 声明解释；工具、控制、编排和种子数据可按降级或跳过处理。**

---

## 12. 模型驱动的数据主线

### 12.1 核心洞察

**模型定义了关系类型，实例数据产生 TraceLink，数据主线是 TraceLink 的图遍历结果。** 模型不只定义数据结构，还定义了数据的组织方式和洞察路径。

### 12.2 从模型到主线的推导

```
模型层（ITP 定义）：
  Requirement ─derives→ Architecture ─allocates→ Deliverable ─verifiedBy→ Test ─produces→ DataSet

实例层（运行时）：
  REQ-001 → AD-003 → DL-007 → WT-012 → DS-045 → RPT-023
  ↑ 这就是一条完整的数据主线
```

### 12.3 三种数据洞察

| 洞察类型 | 方向 | 典型问题 |
|---|---|---|
| 正向追踪 | 需求 → 数据 | 这个需求被哪些试验验证了？ |
| 反向溯源 | 数据 → 需求 | 这份不合格数据的根因是什么？ |
| 影响分析 | 任意节点扩散 | 需求变更影响哪些下游试验和数据？ |

### 12.4 实例与模型版本绑定

- 实例数据不属于 `M` 轴，但每个模型化对象在创建、导入或重大迁移时都应记录 `schemaVersionRef`。
- 主线查询、兼容性判断和回溯视图应基于实例绑定的模型版本解释关系，而不是假定“当前最新模型就是历史真相”。
- 模型回滚不自动回滚已正式提交的实例数据，只影响后续新实例的默认解释和可用视图；存量实例需通过迁移、修复或失效标记处理。

### 12.5 自动生成机制

1. ITP 对象模型的 `relationships` 定义注册为 TraceLink 模板
2. 运行时对象在提交业务事实时绑定 `schemaVersionRef`，并由 `L3` 领域服务生成 `TraceIntent`
3. `L4 Trace Service` 基于 `TraceIntent` 幂等创建或更新 TraceLink 实例
4. 数据主线服务基于 TraceLink 实例计算主线图谱

### 12.6 主线完整性治理

模型关系定义中的 `required`、`cardinality` 约束自动转化为主线完整性检查规则：

- 必须关联 → "孤立对象"告警
- 基数约束 → "覆盖不足"告警
- 活动输出定义 → 完成时检查输出对象是否已入主线

### 12.7 分层视图

ITP 可定义不同角色的主线视图粒度：管理视图（粗粒度）、工程师视图（完整链路）、数据视图（最细粒度）。

### 12.8 统一图服务

**模型级依赖图谱和实例级 TraceLink 图谱本质相同——图上的关系管理和遍历。** 可用同一个 Graph Service 支撑两者。

| 图谱 | 节点 | 边 | 用途 |
|---|---|---|---|
| 模型依赖图谱 | 模板定义 | 引用/依赖 | 复用管理、裁剪影响分析 |
| 实例数据主线 | 数据实例 | TraceLink | 追踪、溯源、变更影响 |

---

## 13. 待进一步讨论

- [ ] 活动模板标准 schema（ICOM 声明格式）
- [ ] `mandatory/advisory` 合规与主数据声明 schema
- [ ] `Task Orchestration / Workflow / Toolflow / Trace` 的详细时序与回调点
- [ ] ITP 加载、卸载与 module 级回滚机制
- [ ] 元模型运行时契约与自动联动注册细节
- [ ] `SchemaVersion` 兼容性引擎与迁移模板
- [ ] `semantic-refs` 的版本绑定与共享语义发布机制
- [ ] 模型导出与开放格式
- [ ] 将结论写入蓝图 `v0.5` / ADR / 附录

---

## 14. 证据链

- `industrial-base-technical-architecture-blueprint-2026-03-10.md`（蓝图 v0.4）
- `industrial-base-unified-object-model-2026-03-10.md`（统一对象模型 v0.4）
- `industrial-base-technical-architecture-blueprint-v0.5-revision-taskboard-2026-03-11.md`（v0.5 修订任务表）
- 产品方与架构师面对面讨论（2026-03-11）
