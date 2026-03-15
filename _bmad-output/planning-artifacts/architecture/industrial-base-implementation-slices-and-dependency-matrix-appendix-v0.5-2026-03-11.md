# 工业研发底座实施切片与依赖矩阵附录

状态：`Draft v0.6`
日期：`2026-03-15`（基于 v0.5 2026-03-11 更新）
对应修订项：`M-07`、`M-08`
变更说明：v0.6 同步——MVP Core 补入 Graph Service；Deferred 中 Plugin Lifecycle 改为 Extension Unit Lifecycle 并移至 Wave 2+；纵切 A/B 依赖补入 Graph Service；服务依赖拓扑对齐新口径。

---

## 1. 依赖原则

1. 先共享控制面，后业务纵切，再工业深水区能力。
2. 纵切必须显式列出最小依赖服务，不允许“默认底座都会有”。
3. `§5.1` 的优先级、`§12.2` 的拆分波次和 `§13` 的纵切验收必须使用同一套命名。

---

## 2. 波次与服务矩阵

| 波次 | 服务 |
| --- | --- |
| `MVP Core` | Identity、Master Data、Data Source Management、Object Model Registry、File/Object、Workflow、Task Orchestration、ACL Policy、Classification & Secrecy Policy、Crypto Policy、Meta-model Extension、Rule、Trace、Graph Service、Audit、Search、Integration Gateway、Config & Release、Industry Template Package、Tenant Bootstrap |
| `Wave 1` | Requirement、Architecture、Function/Logical/Physical Architecture、Interface Definition、Project、WorkPackage、Task Collaboration、Review & Change、Test Planning、Test Execution、DataSet、Analysis、Site & Device、Research Project、Contract、Approval、Model Automation |
| `Wave 2` | Simulation Task、HPC Job Proxy、Result Package、Workbench Session、Toolflow Orchestration、CAD/CAE Runtime、Storage Tiering、TODS Model Registry、ATF/XML Exchange、Knowledge、Semantic Retrieval、Worker Executor |
| `Wave 2+` | Extension Unit Lifecycle（ServerUnit 注册/沙箱/进程管理、ClientUnit 注册/加载）、FED 可视化编辑工作台 |
| `Deferred` | AI Assistant 高级能力、Advanced Dashboard、生态开放组件 |

---

## 3. 纵切 A

### 3.1 目标

验证工业语义主线最小闭环：

`Requirement -> ArchitectureDefinition -> TestSpec -> MeasurementSchema -> DataSet -> TraceLink`

### 3.2 最小依赖服务

| 类别 | 服务 |
| --- | --- |
| 身份与安全 | Identity、ACL Policy、Classification & Secrecy Policy、Audit |
| 模型与对象 | Object Model Registry、Meta-model Extension、Rule |
| 业务与对象 | Requirement、Architecture、Test Planning、Test Execution、DataSet、File/Object |
| 主线与检索 | Trace、Graph Service、Search |
| 发布与环境 | Config & Release |

### 3.3 出口验收

- 能创建需求并关联架构定义
- 能基于架构引用创建 `TestSpec` 和 `MeasurementSchema`
- 能导入一个 `DataSet`
- `TraceLink` 至少形成 `Requirement -> ArchitectureDefinition`、`ArchitectureDefinition -> TestSpec`、`Test -> DataSet`
- 新对象和关系可被权限、审计和搜索识别

---

## 4. 纵切 B

### 4.1 目标

验证复制交付与柔性建模最小闭环：

`IndustryTemplatePackage -> SchemaVersion -> InitializationPlan -> TODSApplicationModel -> MeasurementSchema -> Data Source Management`

### 4.2 最小依赖服务

| 类别 | 服务 |
| --- | --- |
| 身份与安全 | Identity、Audit |
| 模型与注册 | Object Model Registry、Meta-model Extension |
| 复制交付 | Industry Template Package、Tenant Bootstrap、Config & Release |
| 数据源 | Data Source Management |
| 检索与治理 | Search、Graph Service、Rule |

### 4.3 出口验收

- 能装载一个 `ITP`
- 能发布一个 `SchemaVersion`
- 能生成 `InitializationPlan` 并完成租户初始化
- 能把 `TODSApplicationModel` 映射到 `MeasurementSchema`
- 升级失败时能按模板包粒度回滚

---

## 5. 纵切 A/B 集成点

| 集成点 | 说明 |
| --- | --- |
| `SchemaVersion` | 纵切 B 发布的模型版本必须被纵切 A 的业务对象消费 |
| `MeasurementSchema` | 纵切 A 使用的试验定义必须能引用纵切 B 的标准语义 |
| `权限/审计/搜索注册` | 纵切 B 完成安装后自动注册纵切 A 所需的权限点、审计类别和搜索索引 |
| `TraceLink` | 纵切 A 业务变更产生的关系应落到纵切 B 已安装的对象类型体系内 |

---

## 6. 服务依赖拓扑

| 服务 | 前置依赖 |
| --- | --- |
| Requirement | Identity、ACL、Object Model Registry、Audit |
| Architecture | Requirement、Object Model Registry、Trace、Audit |
| Test Planning / Test Execution | Architecture、Rule、Object Model Registry、Audit |
| DataSet | Test Execution、File/Object、Audit |
| Trace | Audit、Search |
| Graph Service | Trace（写入源）、Audit、ACL Policy |
| Meta-model Extension | Object Model Registry、Config & Release、Audit |
| Industry Template Package | Meta-model Extension、Data Source Management、Tenant Bootstrap、Config & Release |
| Tenant Bootstrap | Identity、Master Data、Config & Release、Audit |
| Simulation Task | WorkPackage/TestSpec、Task Orchestration、Audit |
| HPC Job Proxy | Simulation Task、Integration Gateway、Audit |

---

## 7. 治理 SLA

| 治理对象 | 责任组 | 普通 SLA | 紧急 SLA |
| --- | --- | --- | --- |
| `SchemaVersion` 发布 | 模型治理委员会 | `5` 个工作日 | `24h` |
| 模板包准入/升级 | 产品复制治理小组 | `3` 个工作日 | `24h` |
| 涉密导出/跨网交换 | 安全与保密治理小组 | `2` 个工作日 | `8h` |
| 例外发布/紧急回滚 | 架构与发布治理小组 | `2` 个工作日 | `4h` |

---

## 8. 实施建议节拍

### Sprint 1

- 建立 `MVP Core`
- 跑通纵切 A 的最小命令链

### Sprint 2

- 跑通纵切 B 的模板包与模型发布链
- 打通 A/B 集成点

### Sprint 3

- 补齐 `Wave 1`
- 完成主线、搜索、审计和发布对账

### Sprint 4+

- 进入 `Wave 2` 的 HPC、Workbench 和知识增强
- `Wave 2+`：在编辑态和扩展治理过渡方案验证充分后，交付 Extension Unit Lifecycle 和 FED 可视化编辑工作台的完整运行时基础设施
