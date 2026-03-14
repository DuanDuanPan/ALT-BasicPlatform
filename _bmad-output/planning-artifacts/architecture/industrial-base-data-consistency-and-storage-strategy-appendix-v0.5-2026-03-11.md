# 工业研发底座数据一致性与存储策略附录

状态：`Draft v0.5`  
日期：`2026-03-11`  
对应修订项：`M-03`、`M-05`

---

## 1. 一致性总原则

1. 单聚合事务强一致，只允许一个主存事实源。
2. 图数据库、搜索、知识索引、报表快照都是投影，不参与主事务提交。
3. 跨聚合与跨域一致性通过 `Outbox + Event + Projection/Saga` 达成。
4. 任何业务服务都不得同时直接写关系库、图库和搜索引擎。
5. 边缘同步、模板升级和 HPC 长任务采用 Saga，而不是分布式两阶段提交。

---

## 2. 写入方向矩阵

| 对象/对象族 | 主事务写入方 | 主存 | 次级投影 | 一致性模型 | 备注 |
| --- | --- | --- | --- | --- | --- |
| Requirement / Architecture / Project / WorkPackage / Task | 对应 `L3` 领域服务 | 关系库 | 搜索、Trace、审计 | 单聚合强一致，投影最终一致 | 禁止业务代码直接写图谱 |
| Test / TestSpec / MeasurementSchema / DataSet | 试验领域服务 | 关系库或文档库 | 搜索、Trace、时序映射、审计 | 主事务强一致，导入链最终一致 | 时序数据与元数据分离 |
| MeasurementSeries | DataSet 导入链、边缘同步链 | 时序/列式 | 搜索摘要、审计摘要 | 批次级最终一致 | 不回写关系主表 |
| ResearchProject / Contract / EquipmentAsset | MIS 领域服务 | 关系库 | 搜索、Trace、审计 | 单聚合强一致，投影最终一致 | 与研发域通过映射和事件联动 |
| SimulationTask / HPCJob / ResultPackage | 仿真领域服务 | 关系库 | 对象存储、搜索、Trace、审计 | 主事务强一致，长任务 Saga | 结果包通过对象存储引用 |
| TraceLink | Trace Service | 图数据库 + Trace 关系表 | 搜索、影响分析视图 | 幂等最终一致 | 由 `TraceIntent` 驱动 |
| KnowledgeAsset / SemanticIndex | Knowledge Service | 关系库或文档库 | 搜索、向量/语义索引 | 最终一致 | 只引用业务对象真源 |
| IndustryTemplatePackage / InitializationPlan / SchemaVersion | 平台共享服务 | 关系库或文档库 | 搜索、审计、兼容性账本 | Saga + 最终一致 | 与发布治理强绑定 |

---

## 3. `TraceLink` 写入契约

## 3.1 责任分层

- `L3` 领域服务：负责在业务事务提交时生成 `TraceIntent`
- `L4 Trace Service`：负责将 `TraceIntent` 归一化为 `TraceLink`，做幂等 Upsert、关系类型校验和补偿
- `L5` 图数据库：只作为主线和影响分析投影存储，不是业务写入入口

## 3.2 写入时序

1. 领域服务完成聚合主事务
2. 同事务写入 `Outbox`
3. `Trace Service` 消费 `TraceIntentCreated`
4. `Trace Service` 生成或更新 `TraceLink`
5. 图谱、搜索和影响分析视图异步刷新

## 3.3 幂等键

`sourceObjectId + sourceVersion + targetObjectId + targetVersion + relationType`

## 3.4 补偿规则

- 第一次失败：重试队列自动重放
- 连续失败：进入死信队列并发出告警
- 超过阈值：标记主线不完整，阻断基线冻结、正式发布和对外导出

---

## 4. 关系库与文档库分界标准

### 4.1 使用关系库主表 + JSON 扩展字段

满足以下条件时使用该模式：

- 对象有稳定身份和稳定生命周期
- 核心字段稳定且跨对象联查频繁
- 对象需要审批、基线、权限和审计强约束
- 动态字段只占对象属性的小部分
- 需要报表、聚合、主数据映射和一致性校验

典型对象：

- `Requirement`
- `Project`
- `WorkPackage`
- `Task`
- `Test`
- `ResearchProject`
- `Contract`

### 4.2 使用文档库

满足以下条件时使用该模式：

- 同一对象族存在明显多行业、多模板变体
- 属性形态差异大，跨对象 join 要求弱
- 查询以单对象展开和条件检索为主
- 生命周期仍需治理，但结构更偏柔性

典型对象：

- `MeasurementSchema`
- `TODSApplicationModel`
- `TemplateManifest`
- 柔性配置对象

### 4.3 使用时序/列式存储

满足以下条件时进入时序或列式存储：

- 数据按时间序列追加
- 吞吐远高于交易型对象
- 以窗口计算、聚合和批处理为主

典型对象：

- `MeasurementSeries`
- 监测采样批次

---

## 5. 长事务 Saga 目录

| Saga | 起点 | 结束条件 | 补偿动作 |
| --- | --- | --- | --- |
| `TenantBootstrapSaga` | 模板包安装通过 | 租户初始化完成且注册器同步完成 | 回滚初始化记录、撤销注册、清理数据源绑定 |
| `TemplateUpgradeSaga` | 升级计划批准 | 新版本激活且兼容性校验通过 | 回退模板版本、撤销视图/权限/索引注册 |
| `EdgeSyncSaga` | 边缘批次准备上传 | 批次落中心并确认 | 重放批次、锁定冲突对象、人工接管 |
| `HPCRunSaga` | `SimulationTask` 提交 | `ResultPackage` 注册并审计完成 | 取消作业、回收中间结果、保留失败上下文 |
| `ExportApprovalSaga` | 用户发起导出 | 导出包生成、审计完成、回执记录 | 失效审批令牌、删除临时导出包 |

---

## 6. 双写禁令

以下模式一律禁止：

- 领域服务在同一请求里同时写关系库和图数据库
- 页面装配层或脚本直接写 `TraceLink`
- 数据导入程序在未登记主对象的情况下直接写时序库
- 模板安装脚本绕过 `Config & Release Service` 直接改搜索、权限、视图索引

---

## 7. 孤儿数据防护

- 对象存储文件必须由 `FileAsset` 或 `ResultPackage` 元数据引用才能进入正式归档
- 时序批次必须绑定 `DataSetId` 才能进入“已发布”状态
- 图谱关系若任一端对象被删除或归档，由 `Trace Service` 自动标记失效而非静默悬挂
- 模板包卸载和回滚必须自动清理派生视图、权限点、审计类别和无主连接器绑定
