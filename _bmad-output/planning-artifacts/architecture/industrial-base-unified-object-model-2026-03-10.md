# 工业研发底座统一对象模型

状态：`Draft v0.4`  
日期：`2026-03-10`  
用途：作为《工业研发底座架构设计与治理草案》和《统一能力地图》的下一层产物，用于定义底座统一语言、对象边界、聚合根、关键关系和治理规则。

---

## 1. 文档目标

本文件回答五个问题：

1. 底座到底要统一哪些对象。
2. 哪些对象属于共享内核，哪些属于领域专属。
3. 对象之间如何形成 `需求 -> 架构 -> 项目/执行 -> 数据 -> 知识 -> 管理` 的主线关系。
4. 版本、基线、密级、谱系、审计应挂在哪些对象上。
5. 低/无代码能力允许扩展对象到什么边界。

---

## 2. 对象模型设计原则

## 2.1 一切从对象出发

流程、页面、接口、报表、权限最终都服务于对象。  
因此底座先统一对象，再统一服务和界面。

## 2.2 对象分层而不是对象堆叠

同名对象在不同系统中的语义必须被统一或映射，不能并存多个“看起来相似、实际上不同”的版本。

## 2.3 聚合根最小化

不追求把所有属性塞进一个超级对象。  
对象边界应以事务一致性、生命周期一致性和主责团队一致性为准。

## 2.4 共享内核稳定，领域对象演进

共享内核对象应尽量稳定。领域对象允许扩展，但扩展不能破坏统一标识、版本、权限和审计规则。

## 2.5 谱系是一级能力

对象关系不只是外键关系，还必须支持来源、影响、上下游、验证和知识反哺关系。

## 2.6 模型驱动优先

对象模型要支持：

- 核心对象的标准化定义
- 扩展字段和扩展视图
- 柔性对象派生与模型版本化
- 状态机和规则绑定
- 模板和装配

但不允许由低/无代码层重定义核心对象语义。

## 2.7 注册与发布分离

`MetaObjectType` 不能只是静态注册表。  
对象类型、模板包和模型版本必须先经过审批与发布，再进入运行时自动联动。

---

## 3. 统一对象分层

建议把对象统一划分为七类：

| 类别 | 说明 | 典型对象 |
|---|---|---|
| 共享内核对象 | 横切所有业务域的基础对象 | Organization、User、Role、Dictionary、CodeRule、FileAsset |
| 业务主对象 | 承载业务核心语义的对象 | Requirement、ArchitectureDefinition、Project、WorkPackage、Task、Test、Contract |
| 执行对象 | 承载流程、协同、评审、变更和待办 | WorkflowInstance、Review、ChangeRequest、ActionItem |
| 数据对象 | 承载文件、数据集、测量序列、报告、交付物 | DataSet、MeasurementSeries、Deliverable、Report |
| 资源对象 | 承载设备、站点、工具、模板、应用 | Device、Site、Tool、Template、App |
| 知识对象 | 承载知识沉淀和复用 | KnowledgeAsset、StandardItem、Taxonomy、TraceLink |
| 治理对象 | 承载版本、基线、权限、审计、发布和复制引擎 | Baseline、VersionRecord、Policy、AuditRecord、ReleasePackage、IndustryTemplatePackage |

---

## 4. 统一基础字段规范

所有一级业务对象和共享对象建议至少具备以下基础字段：

| 字段 | 说明 | 是否必需 |
|---|---|---|
| `id` | 全局唯一标识 | 是 |
| `code` | 业务编码 | 是 |
| `name` | 展示名称 | 是 |
| `objectType` | 对象类型标识 | 是 |
| `status` | 生命周期状态 | 是 |
| `version` | 当前版本号 | 是 |
| `schemaVersionRef` | 关联模型版本 | 否 |
| `baselineRef` | 所属基线引用 | 否 |
| `securityLevel` | 密级 | 是 |
| `tenantId` | 租户或客户域 | 否 |
| `ownerOrgId` | 归属组织 | 是 |
| `ownerUserId` | 归属责任人 | 否 |
| `sourceSystem` | 来源系统 | 是 |
| `createdAt / createdBy` | 创建信息 | 是 |
| `updatedAt / updatedBy` | 更新信息 | 是 |
| `tags` | 标签 | 否 |
| `extAttrs` | 扩展属性 | 否 |
| `traceRefs` | 谱系或关联引用 | 否 |

补充约束：

- `id` 与外部系统主键分离，外部系统主键进入映射表。
- `objectType` 必须受统一元模型中心治理。
- `extAttrs` 只能扩展，不允许覆盖核心语义字段。

---

## 5. 共享内核对象

## 5.1 组织与身份对象

| 对象 | 定义 | 说明 |
|---|---|---|
| `Organization` | 组织实体 | 组织、院所、部门、试验中心、项目群等 |
| `Position` | 岗位实体 | 岗位、职责岗位、角色岗位 |
| `User` | 用户实体 | 平台自然人账号主体 |
| `Role` | 角色实体 | 权限角色、业务角色、项目角色 |
| `Group` | 组实体 | 跨组织或临时协同组 |

### 关键规则

- `User` 与 `Organization` 是多对多归属，不应简化成单一部门字段。
- `Role` 分为平台角色、业务角色、项目角色三层。
- `Group` 用于项目型和任务型协同，不替代正式组织。

## 5.2 主数据与字典对象

| 对象 | 定义 |
|---|---|
| `Dictionary` | 字典项、枚举项 |
| `CodeRule` | 编码规则 |
| `MasterDataItem` | 跨系统共享主数据条目 |
| `ReferenceData` | 单位、物理量、类型、分类等基础参照数据 |
| `MeasurementQuantity` | 被测物理量、量纲、测量语义定义 |
| `PhysicalUnit` | 单位、单位组和转换规则 |
| `Environment` | 试验环境、工况与上下文 |

### 关键规则

- 物理量、单位、设备分类、试验分类等必须收敛到统一参照数据。
- `MeasurementQuantity / PhysicalUnit / Environment` 必须作为统一语义资产治理，而不是落到单个 `MeasurementSchema` 内私有维护。
- 字典不能在各产品线本地私建同义项。

## 5.3 文件与附件对象

| 对象 | 定义 |
|---|---|
| `FileAsset` | 文件元数据对象 |
| `Folder` | 文件或对象容器 |
| `AttachmentRef` | 业务对象到文件的引用关系 |
| `ObjectSnapshot` | 对象快照，用于审计、基线或归档 |

### 关键规则

- 文件本身与文件元数据分离。
- 文件权限必须继承或映射自业务对象权限。

## 5.4 租户与部署隔离对象

| 对象 | 定义 |
|---|---|
| `Tenant` | 逻辑租户或客户域 |
| `LegalEntity` | 法人或独立核算单位 |
| `IsolationPolicy` | 服务、数据、对象存储和搜索的隔离策略 |
| `DeploymentUnit` | 物理部署单元、命名空间或独立实例 |

### 关键规则

- `Tenant` 不等于 `Organization`，它是更高层的隔离边界。
- `LegalEntity` 与 `Tenant` 可能一对一，也可能多对一。
- `IsolationPolicy` 必须可审计，并能作用到数据库、对象存储、搜索、消息和边缘代理。

---

## 6. 核心业务对象

## 6.1 研发协同域对象

| 对象 | 聚合根 | 说明 |
|---|---|---|
| `Requirement` | 是 | 需求、指标、约束、研制要求 |
| `RequirementItem` | 否 | 需求分解项、验证点 |
| `Project` | 是 | 项目、型号项目、专题项目 |
| `Plan` | 是 | 项目计划、阶段计划、年度计划引用 |
| `WBSNode` | 否 | 结构分解节点 |
| `WorkPackage` | 是 | 可执行工作单元 |
| `Task` | 是 | 任务单元、待办、协同项 |
| `Deliverable` | 是 | 交付物、报告、图档、模型包 |
| `Review` | 是 | 评审对象 |
| `ChangeRequest` | 是 | 变更申请与变更控制对象 |

### 关键关系

- `Requirement -> Project`
- `Project -> Plan`
- `Project -> WorkPackage`
- `WorkPackage -> Task`
- `Task -> Deliverable`
- `Deliverable -> Review`
- `Review / Task / Requirement -> ChangeRequest`

### 聚合边界建议

- `Project` 作为项目治理聚合根，不直接包揽所有任务和数据明细。
- `WorkPackage` 作为执行编排核心聚合根。
- `Task` 独立成聚合根，便于协同、待办、审计、通知和工具流。

## 6.2 试验数据域对象

| 对象 | 聚合根 | 说明 |
|---|---|---|
| `TestPlan` | 是 | 试验计划与试验任务安排 |
| `Test` | 是 | 单次试验或试验活动 |
| `TestSpec` | 是 | 试验规范、试验方案、试验模板 |
| `MeasurementSchema` | 是 | 测量定义、采集定义 |
| `MeasurementChannel` | 否 | 通道定义 |
| `MeasurementSeries` | 是 | 测量时序或曲线数据逻辑对象 |
| `DataSet` | 是 | 数据集、文件集、结果集 |
| `AnalysisJob` | 是 | 分析任务 |
| `AnalysisResult` | 是 | 分析结果 |
| `TestReport` | 是 | 试验报告 |
| `Device` | 是 | 仪器、设备、工装 |
| `Site` | 是 | 试验站点、试验室、边缘节点 |

### 关键关系

- `TestPlan -> Test`
- `Test -> TestSpec`
- `Test -> MeasurementSchema`
- `MeasurementSchema -> MeasurementChannel`
- `Test -> DataSet`
- `DataSet -> MeasurementSeries`
- `DataSet -> AnalysisJob -> AnalysisResult`
- `Test / AnalysisResult -> TestReport`
- `Test -> Device`
- `Site -> Device`

### 聚合边界建议

- `Test` 是试验业务主聚合根。
- `DataSet` 独立成聚合根，避免被试验对象吞并。
- `MeasurementSeries` 不建议作为全局共享超级对象，而应由 `DataSet` 或时序服务管理。

## 6.3 管理信息化域对象

| 对象 | 聚合根 | 说明 |
|---|---|---|
| `AnnualPlan` | 是 | 年度科研计划 |
| `ResearchProject` | 是 | 科研项目管理对象 |
| `Contract` | 是 | 科研合同、协议 |
| `PaymentRecord` | 是 | 经费、付款、执行节点 |
| `EquipmentAsset` | 是 | 设备器材台账 |
| `BorrowRecord` | 是 | 借调记录 |
| `RepairRecord` | 是 | 维修记录 |
| `ScrapRecord` | 是 | 报废记录 |
| `ApprovalCase` | 是 | 管理审批对象 |

### 关键关系

- `AnnualPlan -> ResearchProject`
- `ResearchProject -> Contract`
- `ResearchProject -> PaymentRecord`
- `ResearchProject -> EquipmentAsset`
- `EquipmentAsset -> BorrowRecord / RepairRecord / ScrapRecord`

### 聚合边界建议

- `ResearchProject` 与 `Project` 不强制物理同一对象，但必须存在统一映射。
- `EquipmentAsset` 在 MIS 中为主对象，在 TDM 中按设备视图映射引用。

## 6.4 知识与智能域对象

| 对象 | 聚合根 | 说明 |
|---|---|---|
| `KnowledgeAsset` | 是 | 知识条目、经验、案例 |
| `StandardItem` | 是 | 标准、规范、制度条目 |
| `TaxonomyNode` | 是 | 分类体系 |
| `KnowledgeRelation` | 否 | 知识间关系 |
| `PromptTemplate` | 是 | 智能助手提示模板 |
| `SemanticIndex` | 是 | 语义索引对象 |

### 关键关系

- `KnowledgeAsset` 可关联 `Requirement / Project / WorkPackage / Test / Deliverable / Report`
- `StandardItem` 可约束 `Requirement / TestSpec / Review`

### 聚合边界建议

- 知识对象不应只做文件目录，而应支持结构化属性和可追溯来源。

## 6.5 仿真与算力协同对象

| 对象 | 聚合根 | 说明 |
|---|---|---|
| `SimulationTask` | 是 | 仿真任务对象 |
| `HPCJob` | 是 | 超算作业对象 |
| `ComputeQueue` | 是 | 计算队列或调度目标 |
| `ResultPackage` | 是 | 仿真结果包 |
| `ResultArtifact` | 否 | 结果文件、网格、曲线、报告等 |

### 关键关系

- `WorkPackage / TestSpec -> SimulationTask`
- `SimulationTask -> HPCJob`
- `HPCJob -> ResultPackage`
- `ResultPackage -> Deliverable / KnowledgeAsset`

### 聚合边界建议

- `SimulationTask` 负责业务语义和编排上下文。
- `HPCJob` 负责调度与执行状态，不承载业务审批语义。
- `ResultPackage` 负责结果归档、分层和回流。

## 6.6 系统工程与架构域对象

| 对象 | 聚合根 | 说明 |
|---|---|---|
| `ArchitectureDefinition` | 是 | 架构定义根对象，承接需求与专业设计之间的中枢 |
| `FunctionArchitecture` | 是 | 功能架构 |
| `LogicalArchitecture` | 是 | 逻辑架构 |
| `PhysicalArchitecture` | 是 | 物理架构 |
| `InterfaceSpec` | 是 | 接口定义、接口约束 |
| `ArchitectureBaseline` | 是 | 架构基线 |
| `VerificationCase` | 是 | 针对架构或需求的验证定义 |
| `TODSApplicationModel` | 是 | 试验域标准应用模型定义 |
| `ExchangePackage` | 是 | 标准交换包、ATF/XML 或结果交换描述 |

### 关键关系

- `Requirement -> ArchitectureDefinition`
- `ArchitectureDefinition -> FunctionArchitecture / LogicalArchitecture / PhysicalArchitecture`
- `ArchitectureDefinition -> InterfaceSpec`
- `ArchitectureDefinition -> VerificationCase`
- `ArchitectureDefinition -> WorkPackage / TestSpec / SimulationTask`
- `TODSApplicationModel -> MeasurementSchema`
- `TODSApplicationModel -> ExchangePackage`

### 聚合边界建议

- `ArchitectureDefinition` 是研发主线中的承上启下聚合根，不应退化成普通附件或文档。
- `TODSApplicationModel` 承载标准兼容语义，不替代业务侧 `MeasurementSchema`。
- `ExchangePackage` 负责标准交换上下文，不承载业务审批语义。

---

## 7. 平台执行与治理对象

| 对象 | 聚合根 | 说明 |
|---|---|---|
| `MetaObjectType` | 是 | 对象类型定义 |
| `SchemaVersion` | 是 | 模型版本定义 |
| `IndustryTemplatePackage` | 是 | 行业模板交付包 |
| `TemplateManifest` | 否 | 模板包清单与依赖声明 |
| `InitializationPlan` | 是 | 租户/项目初始化计划与执行记录 |
| `PersistenceProfile` | 是 | 对象持久化路由与存储策略档案 |
| `AutomationRegistration` | 是 | 模型发布后的自动联动注册对象 |
| `ModelExtensionRequest` | 是 | 模型扩展申请与审批对象 |
| `WorkflowTemplate` | 是 | 流程模板 |
| `WorkflowInstance` | 是 | 流程实例 |
| `ActionItem` | 是 | 待办、动作项 |
| `RuleSet` | 是 | 规则集 |
| `ViewTemplate` | 是 | 表单、页面、视图模板 |
| `DashboardView` | 是 | 看板与驾驶舱配置对象 |
| `AIWorkerTask` | 是 | AI Worker 执行节点对象 |
| `PromptVersion` | 是 | 提示词版本对象 |
| `KnowledgeSourceRef` | 否 | AI 任务引用的知识来源 |
| `Baseline` | 是 | 基线定义 |
| `VersionRecord` | 是 | 版本记录 |
| `TraceLink` | 是 | 对象之间的主线或谱系关系 |
| `Policy` | 是 | 安全、权限、保密、发布策略 |
| `AuditRecord` | 是 | 审计记录 |
| `ReleasePackage` | 是 | 应用、配置、模板、ITP 发布包 |

### 关键规则

- `MetaObjectType` 发布后不得裸生效，必须经过 `ReleasePackage / IndustryTemplatePackage` 链路。
- `SchemaVersion` 变更必须驱动 `AutomationRegistration` 更新搜索、审计、权限点和基础视图。
- `PersistenceProfile` 决定对象采用稳定关系模型还是 JSON/文档混合模型。
- `IndustryTemplatePackage` 必须与 `Tenant / InitializationPlan / SchemaVersion` 建立可追溯关联。

### 为什么 `TraceLink` 要单独建模

因为工业研发底座的竞争力来自数字主线，不只是数据表外键。  
建议把 `TraceLink` 作为一级对象管理：

- 支持多种关系类型：来源、派生、验证、引用、影响、替代、归档
- 支持跨域对象关联
- 支持审计和版本化

---

## 8. 统一对象关系主线

## 8.1 研发主线

`Requirement -> Project -> WorkPackage -> Task -> Deliverable -> Review -> ChangeRequest`

## 8.2 架构主线

`Requirement -> ArchitectureDefinition -> FunctionArchitecture / LogicalArchitecture / PhysicalArchitecture -> InterfaceSpec -> VerificationCase`

## 8.3 试验主线

`Requirement -> TestPlan -> Test -> DataSet -> AnalysisResult -> TestReport`

## 8.4 管理主线

`AnnualPlan -> ResearchProject -> Contract -> PaymentRecord`

## 8.5 设备资源主线

`Organization -> Site -> Device -> Test`

## 8.6 知识反哺主线

`Deliverable / TestReport / AnalysisResult / ChangeRequest -> KnowledgeAsset -> Requirement / Project / TestSpec`

## 8.7 仿真计算主线

`WorkPackage / TestSpec -> SimulationTask -> HPCJob -> ResultPackage -> Deliverable / KnowledgeAsset`

---

## 9. 跨域映射规则

## 9.1 `Project` 与 `ResearchProject`

建议语义区分：

- `Project`：偏研发执行对象，服务 `CDM`
- `ResearchProject`：偏经营与科研管理对象，服务 `MIS`

治理要求：

- 两者必须建立 `一对一` 或 `一对多` 映射关系
- 关键字段如名称、组织、负责人、阶段、密级需统一映射
- 不建议强行合并成单一表，除非业务流程已经完全同构

## 9.2 `Device` 与 `EquipmentAsset`

建议语义区分：

- `Device`：偏试验执行、采集和站点接入对象
- `EquipmentAsset`：偏资产台账、借调、维修、报废对象

治理要求：

- 底座层建立统一设备主索引
- `TDM` 与 `MIS` 分别维护专业属性

## 9.3 `Deliverable`、`DataSet`、`KnowledgeAsset`

建议区分：

- `Deliverable`：研发交付结果
- `DataSet`：结构化或半结构化数据结果
- `KnowledgeAsset`：经过抽取、组织和可复用化后的知识对象

治理要求：

- 三者不能混成同一“文档对象”
- 但可以通过 `TraceLink` 形成完整数字主线

---

## 10. 聚合根与服务边界建议

建议优先以以下对象作为聚合根：

| 聚合根 | 建议服务归属 |
|---|---|
| `Requirement` | 需求服务 |
| `Project` | 项目服务 |
| `WorkPackage` | 工作包服务 |
| `Task` | 任务服务 |
| `Deliverable` | 交付物服务 |
| `Test` | 试验服务 |
| `DataSet` | 数据集服务 |
| `Device` | 设备服务 |
| `ResearchProject` | 科研项目服务 |
| `Contract` | 合同服务 |
| `KnowledgeAsset` | 知识服务 |
| `WorkflowInstance` | 流程服务 |
| `TraceLink` | 主线/谱系服务 |

### 边界约束

- 聚合根之间默认通过 `ID + TraceLink` 关联，而不是跨聚合直接嵌套完整对象。
- 共享内核对象不得被任何领域服务私有复制为独立真源。
- 跨域查询可以做聚合视图，但写入必须回到主责服务。

---

## 11. 生命周期、版本与基线

## 11.1 生命周期状态建议

所有核心业务对象至少支持以下状态分层：

- `Draft`
- `Submitted`
- `Approved`
- `Active`
- `Archived`
- `Obsolete`

对象可在此基础上添加领域状态，但不得删除统一治理状态语义。

## 11.2 版本规则

- `Requirement / Deliverable / DataSet / TestSpec / KnowledgeAsset / WorkflowTemplate / ViewTemplate / DashboardView` 必须版本化。
- `Project / WorkPackage / Task` 是否版本化取决于业务复杂度，但至少应支持关键变更快照。

## 11.3 基线规则

建议以下对象纳入基线：

- Requirement
- Project
- WorkPackage
- Deliverable
- TestSpec
- DataSet
- WorkflowTemplate

基线对象通过 `Baseline` 聚合管理，并可挂接里程碑。

---

## 12. 权限、密级与审计模型

## 12.1 权限模型

建议采用：

- `RBAC` 负责角色授权
- `ABAC` 负责组织、项目、密级、对象状态和关系约束

## 12.2 必须受控的对象

以下对象必须强制启用细粒度权限：

- Requirement
- Project / ResearchProject
- Deliverable
- DataSet
- Test
- Contract
- KnowledgeAsset

## 12.3 审计要求

以下行为必须产生 `AuditRecord`：

- 对象创建、修改、删除
- 状态变化
- 版本发布
- 基线冻结
- 权限变更
- 数据导出
- 模板和低代码配置发布

## 12.4 租户、法人与部署隔离模型

建议统一采用四级隔离语义：

- `Tenant`
- `LegalEntity`
- `Project`
- `Object`

补充要求：

- `Tenant` 级隔离决定服务和数据边界
- `LegalEntity` 级隔离决定共享与主数据传播边界
- `Project` 级隔离决定协同与访问范围
- `Object` 级隔离决定实际读写权限

---

## 13. 低/无代码扩展边界

## 13.1 允许扩展的内容

- 核心对象的扩展属性
- 核心对象的视图模型
- 白名单柔性领域对象的派生定义
- 行业模板包中的装配与初始化定义
- 表单布局
- 查询条件、图表配置与工作台展示
- 流程模板和通知规则
- 轻量规则配置
- 轻量集成编排

## 13.2 禁止扩展的内容

- 核心对象主键规则
- 核心对象基础语义
- 核心权限和密级模型
- 核心版本和基线机制
- 核心审计链路
- 深度工程集成和大规模数据处理逻辑

## 13.3 建议白名单对象

建议优先支持低代码扩展的对象：

- `WorkPackage`
- `Task`
- `Deliverable`
- `TestSpec`
- `MeasurementSchema`
- `TODSApplicationModel`
- `ResearchProject`
- `Contract`
- `EquipmentAsset`
- `KnowledgeAsset`
- `WorkflowTemplate`
- `ViewTemplate`
- `DashboardView`

不建议开放低代码重定义的对象：

- `Organization`
- `User`
- `Role`
- `Requirement`
- `ArchitectureDefinition`
- `DataSet`
- `Device`
- `TraceLink`
- `Policy`

## 13.4 UI 模板资产家族

建议把 `Page 04` 涉及的页面、工作台、列表、表单、查询区、图表区、抽屉、弹窗统一纳入 `ViewTemplate` 家族管理。  
这里**不单独引入“投影层对象”**，字段是否出现在 UI、以什么方式出现，直接由 UI 模板中的字段绑定定义。

### 13.4.1 建模边界

- 领域模型只负责业务语义，不负责 UI 布局。
- `ViewTemplate` 负责“长什么样、字段放哪里、动作怎么触发”。
- 字段新增后默认不进入任何模板，必须由设计器显式勾选、拖入或绑定。
- 同一领域对象允许按阶段、角色、任务类型维护多套模板变体。
- 图表、查询区、列表和表单统一视为 `ViewTemplate` 的部件，不再分裂为独立配置体系。
- `DashboardView` 可保留为发布后的运行态看板对象，但建模期应统一由 `ViewTemplate` 编排。

### 13.4.2 对象族

| 对象/概念 | 层级 | 说明 | 核心属性 |
|---|---|---|---|
| `ViewTemplate` | 根对象 | 一套完整 UI 模板，绑定一个主领域对象，可组合多个区域、容器、组件和动作 | `id/code`、`name`、`primaryObjectRef`、`templateKind`、`shellRef`、`schemaVersionRef`、`parentTemplateRef`、`status/version` |
| `ViewShellTemplate` | 可复用骨架 | 页面壳模板，定义工作台的主区域和承载边界 | `shellType`、`regions`、`layoutTokens`、`allowedNodeTypes` |
| `ContainerNode` | 模板子节点 | 结构容器，承载布局与嵌套关系 | `containerType(split/tabs/stack/group/drawer/modal)`、`region`、`children`、`props` |
| `WidgetNode` | 模板子节点 | 业务组件，直接承载字段与数据视图 | `widgetType(tree/table/form/query/detail/chart/timeline)`、`dataSourceRef`、`props` |
| `FieldBinding` | 节点内绑定 | 把领域字段绑定到表格列、表单项、筛选项、图表维度/指标等 UI 位置 | `fieldRef`、`usage`、`label`、`required`、`readonly`、`defaultValue`、`validators`、`sort/filter/aggregate` |
| `TemplateVariant` | 模板变体 | 面向阶段、角色、任务类型的视图变体入口 | `variantKey`、`stageRef`、`roleRef`、`taskTypeRef`、`priority`、`activationCondition` |
| `TemplatePatch` | 结构化差量 | 基于父模板做局部覆盖，不复制整套模板 | `patchOps(add/update/remove/move)`、`targetPath`、`payload`、`reason` |
| `ActionBinding` | 行为绑定 | 把按钮、菜单、行操作、图表点击等交互连接到动作模板/命令 | `triggerRef`、`actionTemplateRef`、`commandRef`、`targetSurface`、`successFlow`、`failureFlow` |
| `ContextFieldMapping` | 动作映射 | 指定动作从当前页、当前行、当前表单或当前上下文读取哪些字段 | `sourceScope`、`fieldRef`、`paramName`、`required`、`transform` |

### 13.4.3 关键关系

- 一个 `ViewTemplate` 必须绑定一个 `primaryObjectRef`，作为主字段面板来源。
- `WidgetNode.dataSourceRef` 可以引用主对象、关联对象路径、聚合结果或图表数据源。
- `FieldBinding` 直接挂在 `WidgetNode` 下，不额外抽象为中间投影层。
- `TemplateVariant` 通过命中条件选择对应模板入口；命中后再叠加 `TemplatePatch`。
- `ActionBinding` 可以引用 `ActionTemplate / RuleSet / WorkflowTemplate`，但动作入口仍归属于视图模板。
- `ContextFieldMapping` 必须引用当前模板已绑定字段或运行时上下文，不允许引用未注册字段。

### 13.4.4 继承与局部覆盖

- 覆盖优先级建议为：`平台基线模板 < 行业模板 < 客户/租户模板 < 项目模板 < 当前变体 patch`。
- 模板继承采用 `parentTemplateRef + TemplatePatch`，不鼓励整套复制后分叉。
- `TemplatePatch` 只允许执行结构化操作：新增节点、更新属性、删除节点、移动节点、增删字段绑定、增删动作绑定。
- 下层模板可以调整字段展现方式，但不能突破领域硬约束，例如核心必填、密级、审计、权限边界。
- 若多个 patch 同时命中同一路径，按显式优先级处理；无优先级或冲突无法消解时阻断发布。

### 13.4.5 模型变更后的处理

- 领域模型新增字段：默认仅进入字段面板候选区，不自动出现在现有模板中。
- 字段删除、改名、类型变化：系统标记受影响的 `FieldBinding / ActionBinding / ContextFieldMapping`。
- `AutomationRegistration` 在 `SchemaVersion` 发布后生成模板兼容性提醒和更新建议，但不直接改写模板。
- 模板维护者可选择接受建议、替换绑定、忽略本次变化或手动修复。
- 模板发布前必须清空阻断级兼容性问题；提醒级问题允许带告警发布。

### 13.4.6 示例

```yaml
viewTemplate:
  id: acceptance-task-workbench
  name: 验收任务工作台
  primaryObjectRef: AcceptanceTask
  templateKind: workbench
  shellRef: WorkbenchShell
  parentTemplateRef: quality-platform-task-base
  variants:
    - variantKey: review.quality.techdoc
      stageRef: review
      roleRef: quality_engineer
      taskTypeRef: technical_document
      priority: 100
      patchRefs:
        - review-panel-patch
        - techdoc-chart-patch
  nodes:
    - id: mainTabs
      nodeType: container
      containerType: tabs
    - id: taskTable
      nodeType: widget
      widgetType: table
      dataSourceRef: primaryObject
      fieldBindings:
        - fieldRef: taskCode
          usage: column
        - fieldRef: reviewStatus
          usage: column
        - fieldRef: riskLevel
          usage: column
    - id: reviewForm
      nodeType: widget
      widgetType: form
      fieldBindings:
        - fieldRef: reviewOpinion
          usage: formField
          required: true
  actionBindings:
    - triggerRef: taskTable.row.signoff
      actionTemplateRef: launch-signoff
      commandRef: launch_signoff
      targetSurface: drawer
      contextMappings:
        - sourceScope: currentRow
          fieldRef: taskId
          paramName: taskId
        - sourceScope: currentRow
          fieldRef: reviewStatus
          paramName: reviewStatus
```

---

## 14. 实施优先级

## 14.1 P0 先定义的对象

- Organization
- User
- Role
- Tenant
- LegalEntity
- Dictionary
- MeasurementQuantity
- PhysicalUnit
- Environment
- Requirement
- ArchitectureDefinition
- FunctionArchitecture
- LogicalArchitecture
- PhysicalArchitecture
- InterfaceSpec
- Project
- WorkPackage
- Task
- Deliverable
- Test
- DataSet
- Device
- ResearchProject
- Contract
- FileAsset
- MetaObjectType
- SchemaVersion
- IndustryTemplatePackage
- InitializationPlan
- WorkflowInstance
- Baseline
- TraceLink

## 14.2 P1 再扩展的对象

- ModelExtensionRequest
- TestSpec
- MeasurementSchema
- TODSApplicationModel
- ExchangePackage
- AnalysisJob
- AnalysisResult
- SimulationTask
- HPCJob
- ComputeQueue
- ResultPackage
- ResultArtifact
- EquipmentAsset
- BorrowRecord
- ScrapRecord
- KnowledgeAsset
- StandardItem
- RuleSet
- PersistenceProfile
- AutomationRegistration
- AIWorkerTask
- PromptVersion
- KnowledgeSourceRef
- ReleasePackage

## 14.3 P2 生态增强对象

- PromptTemplate
- SemanticIndex
- TaxonomyNode
- AdvancedDashboard
- ConnectorTemplate
- PluginPackage

---

## 15. 下一步对服务设计的直接输入

统一对象模型已经为后续服务设计给出明确约束：

1. 服务切分应围绕聚合根，而不是围绕页面菜单。
2. 共享内核必须作为独立平台能力治理。
3. `TraceLink` 应被视为底座一级服务，而不是临时关系表。
4. `Project` 与 `ResearchProject`、`Device` 与 `EquipmentAsset` 必须按“统一索引 + 领域属性分管”处理。
5. `ArchitectureDefinition / FunctionArchitecture / LogicalArchitecture / PhysicalArchitecture / InterfaceSpec` 应形成独立服务组，而不是退化为交付物附件。
6. `MeasurementQuantity / PhysicalUnit / Environment / TODSApplicationModel` 应形成标准语义与交换服务组。
7. 低/无代码只能扩展被授权的对象白名单，不能侵入统一安全和谱系模型。
8. `IndustryTemplatePackage / InitializationPlan / AutomationRegistration / PersistenceProfile` 应作为复制引擎的正式对象进入服务设计。

---

## 16. 证据链

### 核心输入

- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/architecture/industrial-base-architecture-and-governance-draft-2026-03-08.md`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/architecture/industrial-base-unified-capability-map-2026-03-08.md`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/research/industrial-base-asset-business-model-2026-03-08.md`

### 自有资料

- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/docs/reference/3-协同研发/协同研发平台产品白皮书-2024.docx`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/docs/reference/3-协同研发/技术方案.docx`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/docs/reference/1-TDM/数字化试验体系产品2024版.pptx`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/docs/reference/2-管理信息化/科研业务管理系统（一期）系统测试计划V1.1.doc`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/docs/reference/2-管理信息化/主数据管理系统操作手册v1.0.doc`

### 多模态图片证据

- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/research/extracted_images/collab_app_arch.png`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/research/extracted_images/collab_tech_arch.png`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/research/extracted_images/tdm_modeling.png`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/research/extracted_images/tdm_command_center.png`

---

## 17. 当前版本结论

底座真正要统一的，不是所有系统的界面和菜单，而是：

- 一套稳定的共享内核对象
- 一组清晰的领域聚合根
- 一条可追溯的数字主线关系模型
- 一组不被低/无代码破坏的治理规则

这四件事一旦定住，后续服务拆分、集成治理、权限治理和产品装配就会稳定很多。
