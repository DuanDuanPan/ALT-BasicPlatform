# 工业研发底座技术架构蓝图

状态：`Draft v0.5`  
日期：`2026-03-11`  
用途：在 `v0.4` 基础上补齐质量属性场景、运行时契约、数据一致性模型、安全时序、实施依赖矩阵和 ADR 追溯关系，使蓝图具备直接指导实施的粒度。

---

## 1. 蓝图目标

本版蓝图继续回答 `v0.4` 的八个问题，并新增回答五个实施级问题：

1. 四条横切平面在七层逻辑分层中的接入点是什么。
2. `TraceLink` 由谁写入、何时写入、失败如何补偿。
3. 多存储架构下哪些事务强一致，哪些通过事件和 Saga 达成最终一致。
4. 一次请求进入平台后，安全、密级、审计和解密授权按什么顺序执行。
5. 实施团队按照什么优先级、波次和纵切依赖推进，不再出现批次和优先级冲突。

---

## 2. 架构驱动因素

关联 ADR：`ADR-001`、`ADR-003`、`ADR-006`、`ADR-007`、`ADR-009`、`ADR-010`、`ADR-012`、`ADR-014`、`ADR-015`、`ADR-020`、`ADR-021`、`ADR-022`、`ADR-023`

## 2.1 业务驱动

- 同时支撑 `协同研发`、`数字化试验`、`管理信息化`
- 打通 `需求 -> 架构 -> 项目/工作包 -> 设计/仿真/试验 -> 数据 -> 知识 -> 管理`
- 支撑多站点试验接入、跨网传输、离线缓冲和虚实融合
- 支撑多产品线复用和 `ITP` 驱动的快速复制交付
- 支撑 `Web Portal` 和 `Engineering Workbench` 双通道协同

## 2.2 技术驱动

- 共享能力必须平台化，不再按产品线重复造轮子
- `TraceLink`、`TODS/ASAM ODS`、`MBSE` 对象链是底座级能力，不是旁路能力
- 多模存储是前提，不允许退回“一库统管”
- 图能力统一在 `Graph Service` 逻辑层，不要求模型图、实例图和知识图共用同一物理图库
- 边缘和中心必须支持弱网、断网和跨网环境
- 安全与保密必须是运行时控制链，不是制度层口号

## 2.3 约束条件

- 当前仍是研究驱动蓝图，不绑定单一厂商实现
- 不假设旧系统一次性替换完成
- 不允许通过低代码层重定义核心对象、安全模型和主线规则
- 不允许通过双写和人工约定维护多存储一致性

## 2.4 质量属性场景

质量属性场景是本版新增的冻结内容，详细说明见 `industrial-base-runtime-contract-appendix-v0.5-2026-03-11.md` 与 `industrial-base-data-consistency-and-storage-strategy-appendix-v0.5-2026-03-11.md`。

| ID | 场景 | 目标 | 架构策略 |
| --- | --- | --- | --- |
| QA-01 | 单次中心 API 读请求 | 鉴权 + 授权 + 密级判定后的 `P95 < 800ms` | 网关前置鉴权，ACL/密级判定走共享策略服务，业务查询只访问主存和必要投影 |
| QA-02 | 单次中心 API 写请求 | 主事务提交 `P95 < 1.5s`，审计不丢，主线最终收敛 | 单聚合事务强一致，审计与主线使用本地事务 + Outbox |
| QA-03 | `TraceLink` 投影收敛 | 正常情况下 `30s` 内完成图谱和搜索投影 | `L3` 写业务事实，`L4 Trace Service` 负责幂等投影和补偿 |
| QA-04 | 边缘弱网采集 | 边缘断网 `72h` 内可本地持续采集且不丢数 | 边缘本地缓存、分片上传、断点续传、死信回放 |
| QA-05 | 模板包升级回滚 | 单模板包升级失败后 `15min` 内可回滚到上一个稳定版本 | 发布治理链路、兼容性矩阵、升级日志和回滚清单 |
| QA-06 | 高涉密导出 | 未经审批导出必须被阻断并产生审计记录 | 导出前置判定、审批凭据校验、审计强制落地 |
| QA-07 | 多租户隔离 | 逻辑隔离模式下禁止跨租户数据可见性与索引串扰 | 租户隔离标签贯穿主存、搜索、图谱、对象存储和消息 |
| QA-08 | HPC 长任务 | 作业提交、回调、结果回收全链路可追溯，失败可人工接管 | `SimulationTask` 与 `HPCJob` 分离，回调审计，结果包分层存储 |

---

## 3. 总体技术策略

关联 ADR：`ADR-001`、`ADR-002`、`ADR-006`、`ADR-010`、`ADR-013`、`ADR-014`、`ADR-015`、`ADR-016`、`ADR-017`、`ADR-023`

## 3.1 推荐总体形态

建议保持：

- `产品族共用平台内核`
- `1 个平台内核 + 3 个一级业务域 + 1 个复制交付工厂 + N 个工程工具/连接器`
- `七层逻辑分层 + 四条横切平面`
- `受控装配层 + 元模型驱动扩展体系`
- `单实例 / 逻辑隔离 / 物理隔离` 三态部署
- `侧挂增强 + 受控 AI Worker` 双层智能策略

## 3.2 统一实施原则

- 共享能力优先进入 `L4`，业务语义优先进入 `L3`
- 单聚合事务只允许一个系统事实源
- 多存储只允许“主存写入 + 投影更新”，不允许业务层双写
- 图谱统一的是服务语义和治理语义，分治的是底层投影和存储实现
- 横切平面必须定义 injection point、事件、回调和失败责任
- 任何模板包、模型变更和发布动作都必须进入统一发布链路

---

## 4. 逻辑架构蓝图

关联 ADR：`ADR-002`、`ADR-003`、`ADR-006`、`ADR-010`、`ADR-012`、`ADR-020`、`ADR-021`、`ADR-022`、`ADR-023`

## 4.1 七层逻辑分层

沿用 `v0.4` 的 `L1-L7` 分层定义，不再重复展开。

## 4.2 四条横切平面

### A. 模型与标准语义平面

- 接入层：`L2-L6`
- 运行锚点：`Object Model Registry`、`Meta-model Extension Service`、`TODS Application Model Registry`
- 运行职责：模型解析、模式校验、标准语义绑定、持久化策略分派

### B. 数字主线平面

- 接入层：`L2-L5`
- 运行锚点：`L3` 领域服务、`L4 Trace Service`、`L4 Graph Service`、`L5` 图谱/搜索投影
- 运行职责：业务事实触发、关系归一化、幂等写入、统一图查询、影响分析和回溯查询

### C. 复制交付平面

- 接入层：`L1`、`L4`、`L7`
- 运行锚点：`Industry Template Package Service`、`Tenant Bootstrap Service`、`Config & Release Service`
- 运行职责：模板包校验、环境注入、初始化、升级、回滚和差量保留

### D. 安全与保密治理平面

- 接入层：`L1-L7`
- 运行锚点：`Identity`、`ACL`、`Classification`、`Crypto`、`Audit`
- 运行职责：认证、授权、密级判定、解密授权、审计留痕、三员互斥和高危变更审批

## 4.3 横切平面运行时契约摘要

完整矩阵见 `industrial-base-runtime-contract-appendix-v0.5-2026-03-11.md`。

| 平面 | 业务写入入口 | 平台写入入口 | 持久化入口 | 失败责任 |
| --- | --- | --- | --- | --- |
| A 模型与标准语义 | `L3` 创建/变更对象时调用模式校验 | `L4` 注册模型、发布自动联动 | `L5` 按 `PersistenceProfile` 落主存 | 模型发布链路负责回滚与版本冻结 |
| B 数字主线 | `L3` 提交业务事实和 `TraceIntent` | `L4 Trace Service` 归一化写入，`L4 Graph Service` 统一查询与遍历接口 | `L5` 图谱、搜索、影响分析投影 | `Trace Service` 负责重试、去重和补偿，`Graph Service` 负责统一查询语义 |
| C 复制交付 | `L1/L2` 触发场景装载和初始化 | `L4` 模板校验、租户初始化、自动注册 | `L5` 记录版本账本、安装清单、差量状态 | 发布治理链路负责阻断、回滚和恢复 |
| D 安全与保密 | 请求进入 `L1/L2/L6` 即触发 | `L4` 统一判定和审计 | `L5` 执行加密、标签、留存和 WORM | 安全策略服务负责拒绝、告警和审计 |

---

## 5. 服务蓝图

关联 ADR：`ADR-002`、`ADR-006`、`ADR-008`、`ADR-011`、`ADR-017`、`ADR-018`、`ADR-020`、`ADR-021`

## 5.1 服务优先级与波次重排

`v0.5` 不再使用区分度过低的 `P0/P1` 作为唯一排序方式，改为 `MVP Core / Wave 1 / Wave 2 / Deferred`。

| 层次 | 服务 | 说明 |
| --- | --- | --- |
| MVP Core | Identity、Master Data、Data Source Management、Object Model Registry、File/Object、Workflow、Task Orchestration、ACL Policy、Classification & Secrecy Policy、Crypto Policy、Meta-model Extension、Rule、Trace、Audit、Search、Integration Gateway、Config & Release、Industry Template Package、Tenant Bootstrap | 第一批必须具备的共享控制面和复制引擎最小闭环 |
| Wave 1 | Requirement、Architecture、Function/Logical/Physical Architecture、Interface Definition、Project、WorkPackage、Task Collaboration、Review & Change、Test Planning、Test Execution、DataSet、Analysis、Site & Device、Research Project、Contract、Approval、Model Automation | 第一轮业务闭环和纵切 A/B 主体 |
| Wave 2 | Simulation Task、HPC Job Proxy、Result Package、Workbench Session、Toolflow Orchestration、CAD/CAE Runtime、Storage Tiering、TODS Model Registry、ATF/XML Exchange、Knowledge、Semantic Retrieval、Worker Executor | 工业特性深化与 HPC/知识增强 |
| Deferred | Plugin Lifecycle、AI Assistant 高级能力、Advanced Dashboard、生态开放组件 | 不阻塞主线闭环 |

## 5.2 领域边界摘要

详细聚合根、事务边界和领域事件见 `industrial-base-domain-model-and-events-appendix-v0.5-2026-03-11.md`。

| 域 | 核心聚合根 | 写入边界 | 跨域关联方式 |
| --- | --- | --- | --- |
| 研发协同与系统工程 | Requirement、ArchitectureDefinition、Project、WorkPackage、Task、Deliverable | 单聚合强一致，跨聚合通过事件 | `ID + TraceLink`，禁止跨聚合嵌套 |
| 试验数据 | Test、TestSpec、MeasurementSchema、DataSet、AnalysisJob、Device、Site | `Test` 与 `DataSet` 独立聚合 | 通过 `TraceLink`、`DataSetRef`、`DeviceRef` 关联 |
| 仿真与算力 | SimulationTask、HPCJob、ResultPackage | `SimulationTask` 管业务语义，`HPCJob` 管调度状态 | 结果通过 `ResultPackage` 回流，不直接回写业务聚合 |
| 管理信息化 | AnnualPlan、ResearchProject、Contract、EquipmentAsset、ApprovalCase | MIS 聚合独立提交 | 通过映射索引和 `TraceLink` 连接研发域 |
| 知识与智能 | KnowledgeAsset、StandardItem、SemanticIndex、AIWorkerTask | 知识沉淀与检索独立聚合 | 引用业务对象 ID，不私有复制真源数据 |

---

## 6. 数据架构蓝图

关联 ADR：`ADR-006`、`ADR-007`、`ADR-017`、`ADR-018`、`ADR-019`、`ADR-021`、`ADR-023`

## 6.1 一致性模型

完整策略见 `industrial-base-data-consistency-and-storage-strategy-appendix-v0.5-2026-03-11.md`。

- 单聚合内：强一致，只允许一个主存事实源
- 跨聚合但同域：`本地事务 + Outbox + 领域事件`
- 跨域与跨存储：最终一致，由投影器和 Saga 收敛
- 长事务：`Tenant Bootstrap`、`ITP Upgrade`、`Edge Sync`、`HPCRun` 使用 Saga
- 禁止模式：业务服务对关系库、图库、搜索和对象存储进行无编排双写

## 6.2 存储分工

| 存储 | 主责对象 | 写入方式 |
| --- | --- | --- |
| 关系型数据库 | 核心业务聚合、审批、权限、版本账本 | 主事务写入 |
| 文档数据库 | 柔性对象、模板清单、模型扩展、复杂配置 | 主事务或模型事务写入 |
| 对象存储 | 文件、原始包、结果包、归档对象 | 对象服务持有元数据引用 |
| 时序/列式存储 | `MeasurementSeries`、高频采样数据 | 由数据集导入与边缘同步链路写入 |
| 图数据库 | `TraceLink`、知识图谱、影响分析 | 只接受 `Trace Service` 和知识投影器写入 |
| 搜索引擎 | 条件检索、全文、标签、语义前置召回 | 只接受投影更新，不作为主事务真源 |

## 6.3 `TraceLink` 写入责任

- `L3` 领域服务负责生成业务事实和 `TraceIntent`
- `L4 Trace Service` 负责校验关系类型、去重、幂等 upsert 和补偿
- `L5` 图数据库和搜索引擎只保存投影，不承载业务事务真相

## 6.4 统一 `Graph Service` 与底层分治

- `Graph Service` 统一暴露模型依赖分析、正向追踪、反向溯源、影响分析和路径查询，不直接承载业务写入责任。
- 模型图域由 `Meta-model Extension / Model Automation / ITP Loader` 负责投影生成；实例图域由 `Trace Service` 写入；知识图域由知识投影器写入。
- `Graph Service` 统一执行租户、权限、密级和审计接入规则，避免模型图和实例图形成两套查询口径。
- 底层允许按图域、负载、隔离模式和生命周期分治部署，不强制模型图、实例图和知识图共用同一物理图库。
- 页面装配层、脚本、插件和 `L3` 领域服务都不允许绕过 `Trace Service` / 图投影器直接写图数据库。

---

## 7. 集成架构蓝图

关联 ADR：`ADR-009`、`ADR-011`、`ADR-021`、`ADR-022`

## 7.1 五类集成面

继续采用 `API / Event / File / Agent-Protocol / Standard Exchange` 五类集成面。

## 7.2 工具链与超算运行边界

详细生命周期见 `industrial-base-hpc-and-workbench-lifecycle-appendix-v0.5-2026-03-11.md`。

- `Task Orchestration` 负责业务待办、责任分派和状态编排
- `Toolflow Orchestration` 负责工程工具步骤编排、运行依赖和回调收敛
- `SimulationTask` 承载仿真业务语义，`HPCJob` 承载调度语义
- `ResultPackage` 作为结果回流的唯一对象入口，不允许求解器直接改写领域聚合

## 7.3 标准交换与数据源绑定

- `Data Source Management Service` 是数据源注册与读写范围控制的正式名称
- `TODSApplicationModel` 和 `ATF/XML Exchange` 通过标准交换面进入平台
- 标准交换包的校验结果必须进入审计和主线记录

---

## 8. 部署架构蓝图

关联 ADR：`ADR-011`、`ADR-015`

详细拓扑与容量基线见 `industrial-base-deployment-topology-and-capacity-baseline-appendix-v0.5-2026-03-11.md`。

## 8.1 推荐部署区

建议保留 `接入区 / 应用区 / 共享服务区 / 数据区 / 管理区 / 集成区 / 工程客户端区 / 边缘区` 八个部署区。

## 8.2 网络与流量基线

- `接入区 -> 应用区`：只允许南北向 API 流量，经网关与 WAF
- `应用区 -> 共享服务区`：只允许服务调用和事件流量，不允许直连数据区
- `共享服务区 -> 数据区`：按最小权限访问主存和投影存储
- `集成区 <-> 边缘区`：经标准交换或受控文件通道，不允许隐式直连
- `管理区` 与业务区隔离，三员分立账户不能与普通业务会话复用

## 8.3 容量与扩展原则

- 网关、事件总线、搜索、图谱、对象存储优先水平扩展
- 关系库优先主从高可用和按租户/域拆分
- 时序和对象存储按容量增长规划，避免与交易库共盘

---

## 9. 边缘架构蓝图

关联 ADR：`ADR-009`、`ADR-011`、`ADR-012`

详细离线和故障语义见 `industrial-base-edge-offline-and-failure-semantics-appendix-v0.5-2026-03-11.md`。

## 9.1 正常链路

`设备 -> 协议适配器 -> 边缘代理 -> 本地缓存/预处理 -> 同步代理 -> 中心接入`

## 9.2 失败路径

- 中心与边缘同时修改同一业务主对象时：中心主对象优先，边缘侧进入冲突队列等待人工处理
- 采集数据和结果包属于 append-only 或版本化对象时：边缘版本保留，中心按版本向量或导入批次收敛
- 上传连续失败达到阈值后进入死信区，由运维或站点管理员接管
- 边缘脱网时，本地仅允许缓存、采集、审批凭据缓存和最小 ACL 降级判定，不允许越权导出

---

## 10. 安全与治理蓝图

关联 ADR：`ADR-003`、`ADR-012`、`ADR-015`、`ADR-017`

## 10.1 统一安全判定顺序

完整时序、错误码和审计策略见 `industrial-base-security-runtime-sequence-and-error-code-appendix-v0.5-2026-03-11.md`。

统一顺序固定为：

1. 接入区认证与网络来源校验
2. 身份鉴别与会话校验
3. 租户/法人/项目隔离校验
4. 角色、对象、范围授权
5. 密级与保密规则判定
6. 解密或导出审批凭据校验
7. 业务执行
8. 审计写入与告警

## 10.2 治理组织最小运行规则

详细治理运营机制见 `industrial-base-governance-operating-model-appendix-v0.5-2026-03-11.md`。

- 模型治理委员会：模型、SchemaVersion、对象扩展审批，普通申请 `5` 个工作日内答复
- 产品复制治理小组：模板包、升级、回滚、兼容性审批，普通申请 `3` 个工作日内答复
- 安全与保密治理小组：涉密策略、跨网和导出审批，普通申请 `2` 个工作日内答复
- 架构与发布治理小组：跨域重大变更、例外发布、回滚裁决，普通申请 `2` 个工作日内答复

紧急例外通道：

- 由架构与发布治理小组牵头
- 至少需要业务负责人 + 安全负责人 + 运维负责人三方确认
- 例外发布后 `24h` 内必须补齐审计和复盘记录

---

## 11. 运维与可观测性蓝图

关联 ADR：`ADR-011`、`ADR-012`、`ADR-014`

详细非功能基线见 `industrial-base-nonfunctional-architecture-baseline-appendix-v0.5-2026-03-11.md`。

- 必须观测：登录失败、授权拒绝、导出拒绝、模型发布、模板安装、边缘同步失败、HPC 作业失败、Trace 投影积压
- 必须告警：跨租户访问尝试、审计写入失败、密钥调用失败、死信堆积、版本兼容校验失败
- 必须保全：审计日志、发布记录、升级回滚记录、边缘同步日志

---

## 12. 参考实现策略

关联 ADR：`ADR-014`、`ADR-017`、`ADR-018`、`ADR-023`

## 12.1 技术资产兼容方向

保持 `v0.4` 的原则：能力统一，不在蓝图层绑定单一厂商。

复制交付兼容和回滚规则见 `industrial-base-replication-delivery-version-compatibility-and-rollback-appendix-v0.5-2026-03-11.md`。

图能力兼容方向补充如下：

- 统一 `Graph Service` 逻辑接口，不在蓝图层强绑单一图库产品。
- 优先冻结图服务语义、写入边界和治理规则，再根据容量和隔离需求决定模型图、实例图、知识图是否分库。
- 任何图后端切换不得改变 `Trace Service` 的写入权威，也不得改变上层查询与影响分析口径。

## 12.2 服务拆分波次

详细依赖矩阵见 `industrial-base-implementation-slices-and-dependency-matrix-appendix-v0.5-2026-03-11.md`。

1. `MVP Core`：先冻结共享控制面和复制引擎最小闭环
2. `Wave 1`：再完成 `Requirement + Architecture + Test/DataSet + Project/Contract` 两条业务主线
3. `Wave 2`：补强 `HPC / Workbench / Knowledge / AI Worker`
4. `Deferred`：生态开放和高级智能

---

## 13. 对实施的直接指导

关联 ADR：`ADR-006`、`ADR-010`、`ADR-014`、`ADR-017`、`ADR-020`、`ADR-021`、`ADR-023`

## 13.1 纵切 A：工业语义主线最小闭环

对象范围：

- `Requirement`
- `ArchitectureDefinition`
- `TestSpec`
- `MeasurementSchema`
- `DataSet`
- `TraceLink`

最小依赖服务：

- Identity
- ACL Policy
- Classification & Secrecy Policy
- Object Model Registry
- File/Object
- Workflow
- Rule
- Trace
- Audit
- Search
- Config & Release

验收标准：

- 能完成 `需求 -> 架构 -> 试验规范 -> 数据集 -> 主线关系` 闭环
- 新对象和关系能被搜索、审计和权限识别
- `TraceLink` 投影可在目标时限内收敛
- 可通过统一 `Graph Service` 查询主线正向追踪、反向溯源和基础影响分析

## 13.2 纵切 B：复制交付与柔性建模最小闭环

对象范围：

- `IndustryTemplatePackage`
- `SchemaVersion`
- `InitializationPlan`
- `TODSApplicationModel`
- `MeasurementSchema`
- `DataSource`

最小依赖服务：

- Identity
- Data Source Management
- Object Model Registry
- Meta-model Extension
- Industry Template Package
- Tenant Bootstrap
- Config & Release
- Audit
- Search

验收标准：

- 一个租户可装载模板包并完成初始化
- `TODSApplicationModel` 与 `MeasurementSchema` 映射可发布
- 升级失败时可按模板包粒度回滚
- 模型依赖图谱可通过统一 `Graph Service` 对外提供依赖解释和裁剪影响分析

## 13.3 纵切集成点

- `纵切 A` 的 `TestSpec / MeasurementSchema` 引用 `纵切 B` 发布的 `SchemaVersion` 和标准语义
- `纵切 B` 的模板包安装必须自动注册 `纵切 A` 所需的权限点、审计类别和基础视图

---

## 14. ADR 映射与证据链

## 14.1 章节级 ADR 映射

| 章节 | ADR |
| --- | --- |
| `§2` | `ADR-001`、`ADR-003`、`ADR-006`、`ADR-007`、`ADR-009`、`ADR-010`、`ADR-012`、`ADR-014`、`ADR-015`、`ADR-020`、`ADR-021`、`ADR-022`、`ADR-023` |
| `§3` | `ADR-001`、`ADR-002`、`ADR-006`、`ADR-010`、`ADR-013`、`ADR-014`、`ADR-015`、`ADR-016`、`ADR-017`、`ADR-023` |
| `§4` | `ADR-002`、`ADR-003`、`ADR-006`、`ADR-010`、`ADR-012`、`ADR-020`、`ADR-021`、`ADR-022`、`ADR-023` |
| `§5` | `ADR-002`、`ADR-006`、`ADR-008`、`ADR-011`、`ADR-017`、`ADR-018`、`ADR-020`、`ADR-021` |
| `§6` | `ADR-006`、`ADR-007`、`ADR-017`、`ADR-018`、`ADR-019`、`ADR-021`、`ADR-023` |
| `§7` | `ADR-009`、`ADR-011`、`ADR-021`、`ADR-022` |
| `§8` | `ADR-011`、`ADR-015` |
| `§9` | `ADR-009`、`ADR-011`、`ADR-012` |
| `§10` | `ADR-003`、`ADR-012`、`ADR-015`、`ADR-017` |
| `§12-§13` | `ADR-006`、`ADR-010`、`ADR-014`、`ADR-017`、`ADR-020`、`ADR-021`、`ADR-023` |

## 14.2 证据链

详细规范化清单见 `industrial-base-evidence-chain-normalization-appendix-v0.5-2026-03-11.md`。

| 状态 | 来源 | 版本 |
| --- | --- | --- |
| 已采纳 | `_bmad-output/planning-artifacts/architecture/industrial-base-adr-decision-list-2026-03-10.md` | `2026-03-10` |
| 已采纳 | `_bmad-output/planning-artifacts/architecture/industrial-base-unified-object-model-2026-03-10.md` | `2026-03-10` |
| 已采纳 | `_bmad-output/planning-artifacts/architecture/industrial-base-technical-architecture-blueprint-2026-03-10.md` | `2026-03-10` |
| 已采纳 | `_bmad-output/planning-artifacts/architecture/industrial-base-phase0-vertical-slice-plan-2026-03-08.md` | `2026-03-08` |
| 已采纳 | `_bmad-output/planning-artifacts/architecture/industrial-base-technical-architecture-blueprint-refactor-recommendations-2026-03-10.md` | `2026-03-10` |
| 已阅读未完全采纳 | `docs/reference/INDEX.md` 及其索引资料 | 以索引更新时间为准 |

## 14.3 当前 ADR 缺口

以下章节已有设计内容，但 ADR 支撑仍偏弱，后续建议补充专门 ADR：

- `§8.2 高可用基线`：缺少高可用、容灾、SLO/SLA 类 ADR
- `§10.2 治理组织最小运行规则`：缺少治理组织、决策权限和例外流程 ADR
- `§11 运维与可观测性蓝图`：缺少可观测性责任模型和运维治理 ADR
- `§12.1 技术资产兼容方向`：`ADR-023` 已补齐图服务逻辑统一/物理分治方向，但更广泛的技术选型 ADR 仍待补充

---

## 15. 当前版本结论

`v0.5` 与 `v0.4` 的差异不在于增加更多服务，而在于把原先“方向正确”的部分收敛成了可以直接用于实施的规则：

- 质量属性从目标口号变成了带指标的场景
- 横切平面从概念描述变成了运行时契约
- `TraceLink`、多存储和安全控制有了明确写入与执行责任
- 图能力统一到 `Graph Service` 逻辑层，避免模型图与实例图形成两套查询和治理口径
- 优先级、批次、纵切和治理规则形成了同一口径
- 非功能视图和证据链从缺席状态提升到最小可复核基线

仍待后续版本继续深化的重点是更精细的容量数字、正式合规逐条映射和生态开放细则，但这些不再阻塞 `v0.5` 作为实施蓝图使用。
