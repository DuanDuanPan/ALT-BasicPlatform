# 工业研发底座领域建模与事件附录

状态：`Draft v0.5`  
日期：`2026-03-11`  
对应修订项：`M-04`

---

## 1. 建模总规则

1. 聚合根以事务一致性、生命周期一致性和责任团队一致性划分。
2. 聚合根之间默认通过 `ID + TraceLink` 关联，不允许跨聚合直接嵌套完整对象。
3. 领域事件只表达“已提交业务事实”，不表达界面动作。
4. 跨域联动优先用事件和投影，不在领域服务之间做写时级联。

---

## 2. 研发协同与系统工程域

### `Requirement`

- 职责：维护需求、指标、约束和验证引用
- 事务边界：需求主记录、需求分解项、状态和版本在同一事务内
- 引用规则：引用 `ArchitectureDefinition`、`Project` 仅保存 ID；不嵌套任务和试验数据
- 推荐领域事件：`RequirementCreated`、`RequirementApproved`、`RequirementBaselined`、`RequirementChanged`

### `ArchitectureDefinition`

- 职责：承接需求到设计/试验/仿真的中枢架构对象
- 事务边界：架构主记录、架构状态、架构版本和基线关系在同一事务内
- 引用规则：对子架构、接口、验证、工作包、试验规范、仿真任务只保存 ID
- 推荐领域事件：`ArchitectureDefined`、`ArchitectureBaselined`、`ArchitectureLinkedToRequirement`、`ArchitectureReleased`

### `Project`

- 职责：管理研发项目上下文、阶段、负责人和主责组织
- 事务边界：项目主信息、阶段状态、关键里程碑在同一事务内
- 引用规则：不直接内嵌 `WorkPackage`、`Task` 和 `Contract`；通过映射或主线关联
- 推荐领域事件：`ProjectCreated`、`ProjectActivated`、`ProjectMilestoneReached`、`ProjectMappedToResearchProject`

### `WorkPackage`

- 职责：承载可执行工作单元、责任分派和交付边界
- 事务边界：工作包主记录、责任人、计划、状态在同一事务内
- 引用规则：引用 `Project`、`Requirement`、`ArchitectureDefinition`、`Task` 时只存 ID
- 推荐领域事件：`WorkPackagePlanned`、`WorkPackageStarted`、`WorkPackageCompleted`、`WorkPackageReplanned`

### `Task`

- 职责：承载最小协同单元、动作项和待办上下文
- 事务边界：任务主记录、动作项、责任人、状态在同一事务内
- 引用规则：任务只指向一个主上下文对象 ID；不承载结果包和文件内容
- 推荐领域事件：`TaskAssigned`、`TaskStarted`、`TaskCompleted`、`TaskBlocked`

### `Deliverable`

- 职责：承载研发交付物和版本化交付结果
- 事务边界：交付物主记录、版本、基线关系在同一事务内
- 引用规则：文件由 `File/Object Service` 管理，仅保留引用
- 推荐领域事件：`DeliverableCreated`、`DeliverableVersioned`、`DeliverableSubmittedForReview`、`DeliverableArchived`

---

## 3. 试验数据域

### `Test`

- 职责：承载单次试验活动及其执行状态
- 事务边界：试验主记录、执行状态、参与设备和主规范引用在同一事务内
- 引用规则：引用 `TestSpec`、`MeasurementSchema`、`DataSet`、`Device` 时只保存 ID
- 推荐领域事件：`TestPlanned`、`TestStarted`、`TestCompleted`、`TestAborted`

### `TestSpec`

- 职责：承载试验规范、方案模板和验证目标
- 事务边界：规范主记录、版本、审批状态在同一事务内
- 引用规则：对 `MeasurementSchema`、`Requirement` 和 `ArchitectureDefinition` 只保留 ID
- 推荐领域事件：`TestSpecDrafted`、`TestSpecApproved`、`TestSpecReleased`、`TestSpecDeprecated`

### `MeasurementSchema`

- 职责：承载测量定义、采集结构、通道规则和标准语义映射
- 事务边界：模式主记录、通道定义、标准语义引用在同一事务内
- 引用规则：量纲单位语义来自 `TODSApplicationModel` 和参照数据中心
- 推荐领域事件：`MeasurementSchemaDefined`、`MeasurementSchemaPublished`、`MeasurementSchemaMappedToTODS`

### `DataSet`

- 职责：承载试验数据集、导入批次和结果集边界
- 事务边界：数据集主记录、导入批次、状态和主文件引用在同一事务内
- 引用规则：时序明细不内嵌，`MeasurementSeries` 和对象存储按引用管理
- 推荐领域事件：`DataSetIngested`、`DataSetValidated`、`DataSetPublished`、`DataSetArchived`

### `Device`

- 职责：承载试验设备、仪器和工装视图
- 事务边界：设备主标识、站点绑定、状态在同一事务内
- 引用规则：专业资产属性通过 `EquipmentAsset` 映射，不在同一聚合中维护
- 推荐领域事件：`DeviceRegistered`、`DeviceActivated`、`DeviceBoundToSite`、`DeviceRetired`

---

## 4. 仿真与算力协同域

### `SimulationTask`

- 职责：承载仿真业务任务、输入配置和责任上下文
- 事务边界：任务主记录、输入包引用、调度策略在同一事务内
- 引用规则：不内嵌 `HPCJob` 和求解器状态，只保留 ID
- 推荐领域事件：`SimulationTaskCreated`、`SimulationTaskSubmitted`、`SimulationTaskCompleted`、`SimulationTaskFailed`

### `HPCJob`

- 职责：承载调度器作业、队列、回调状态和重试信息
- 事务边界：作业主记录、调度状态、队列、失败原因在同一事务内
- 引用规则：只引用 `SimulationTask` 和 `ResultPackage`
- 推荐领域事件：`HPCJobQueued`、`HPCJobRunning`、`HPCJobSucceeded`、`HPCJobFailed`

### `ResultPackage`

- 职责：承载仿真结果包、分层存储位置和回流引用
- 事务边界：结果包主记录、版本、分层位置和归档状态在同一事务内
- 引用规则：结果文件内容通过对象存储引用，知识沉淀通过 `KnowledgeAsset` 关联
- 推荐领域事件：`ResultPackageRegistered`、`ResultPackageTiered`、`ResultPackagePublished`、`ResultPackageArchived`

---

## 5. 管理信息化域

### `ResearchProject`

- 职责：承载科研计划、经营口径和科研项目治理语义
- 事务边界：科研项目主记录、计划状态、组织关系在同一事务内
- 引用规则：与 `Project` 通过统一映射索引和 `TraceLink` 连接，不共享事务
- 推荐领域事件：`ResearchProjectApproved`、`ResearchProjectActivated`、`ResearchProjectMapped`、`ResearchProjectClosed`

### `Contract`

- 职责：承载合同、协议和付款节点管理
- 事务边界：合同主信息、付款节点、履约状态在同一事务内
- 引用规则：只引用 `ResearchProject`、`Project` 和 `Deliverable` 的 ID
- 推荐领域事件：`ContractSigned`、`ContractChanged`、`ContractMilestoneReached`、`ContractClosed`

### `EquipmentAsset`

- 职责：承载设备资产台账、借调、维修和报废语义
- 事务边界：资产主记录和当前状态在同一事务内
- 引用规则：试验执行视图由 `Device` 聚合负责，二者通过主索引映射
- 推荐领域事件：`EquipmentAssetCreated`、`EquipmentAssetBorrowed`、`EquipmentAssetRepaired`、`EquipmentAssetScrapped`

---

## 6. 知识与智能域

### `KnowledgeAsset`

- 职责：承载结构化知识条目、经验和复用资产
- 事务边界：知识主记录、分类、版本和来源引用在同一事务内
- 引用规则：来源对象只保存 ID 和来源版本，不复制业务真源
- 推荐领域事件：`KnowledgeAssetCreated`、`KnowledgeAssetPublished`、`KnowledgeAssetReferenced`、`KnowledgeAssetArchived`

### `TraceLink`

- 职责：承载跨域对象的主线、影响和验证关系
- 事务边界：单条关系定义、关系类型、方向、版本和状态在同一事务内
- 引用规则：只引用两端对象 ID、对象类型、版本和关系类型；不承载对象快照
- 推荐领域事件：`TraceLinkCreated`、`TraceLinkUpserted`、`TraceLinkDeprecated`、`TraceLinkRebuilt`

---

## 7. 跨域领域事件目录

| 事件 | 生产者 | 消费者 | 一致性模式 |
| --- | --- | --- | --- |
| `RequirementApproved` | Requirement Service | Architecture、Test Planning、Trace | 最终一致 |
| `ArchitectureBaselined` | Architecture Service | WorkPackage、Test Planning、Simulation、Trace | 最终一致 |
| `WorkPackageCompleted` | WorkPackage Service | Deliverable、Knowledge、Trace | 最终一致 |
| `TestCompleted` | Test Execution Service | DataSet、Analysis、Knowledge、Trace | 最终一致 |
| `DataSetPublished` | DataSet Service | Search、Trace、Knowledge、MIS 汇总 | 最终一致 |
| `SimulationTaskCompleted` | Simulation Task Service | Result Package、Knowledge、Trace | 最终一致 |
| `ResearchProjectMapped` | Research Project Service | Project、Contract、Trace | 最终一致 |
| `ContractMilestoneReached` | Contract Service | Project、Deliverable、Audit | 最终一致 |
| `KnowledgeAssetPublished` | Knowledge Service | Search、Semantic Retrieval、AI Worker | 最终一致 |
| `TraceLinkUpserted` | Trace Service | Graph/Search 投影、影响分析视图 | 最终一致 |

---

## 8. 边界红线

- `Task` 不允许内嵌整个 `WorkPackage` 或 `DataSet`
- `DataSet` 不允许吞并 `MeasurementSeries` 全量内容进入关系库主表
- `ResearchProject` 不允许直接写研发域主对象状态
- `HPCJob` 不允许直接驱动业务审批状态改变，必须回流到 `SimulationTask`
- `TraceLink` 不允许由页面装配层和脚本直接绕过 `Trace Service` 写入图数据库
