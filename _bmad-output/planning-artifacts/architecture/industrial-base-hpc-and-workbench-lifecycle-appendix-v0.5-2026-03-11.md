# 工业研发底座 HPC 与 Workbench 生命周期附录

状态：`Draft v0.5`  
日期：`2026-03-11`  
对应修订项：`S-03`

---

## 1. 目标

本附录定义 `Engineering Workbench`、工具流、`SimulationTask`、`HPCJob` 和 `ResultPackage` 的生命周期边界与并发规则。

---

## 2. 角色边界

| 服务 | 职责 | 不负责 |
| --- | --- | --- |
| `Task Orchestration` | 业务待办、责任分派、状态提醒 | HPC 调度、工具步骤编排 |
| `Toolflow Orchestration` | 工具步骤、输入输出依赖、回调收敛 | 业务审批和责任分派 |
| `Workbench Session Service` | 会话、租约、终端绑定、协同上下文 | 结果持久化和超算调度 |
| `Simulation Task Service` | 仿真业务语义、输入上下文、执行意图 | 调度器细节 |
| `HPC Job Proxy Service` | 队列、资源申请、调度回调、重试 | 业务语义审批 |
| `Result Package Service` | 结果登记、版本绑定、存储分层 | 业务审批状态 |

---

## 3. Workbench 会话模型

## 3.1 会话类型

- `SingleUserSession`：同一用户单终端独占
- `MultiTerminalSession`：同一用户多终端续接
- `CollaborativeReviewSession`：多人查看、单人编辑

## 3.2 锁策略

| 场景 | 锁策略 |
| --- | --- |
| 同一用户多终端编辑同一模型 | 逻辑续租，同一主体可接管旧终端 |
| 不同用户同时编辑同一模型 | 默认单写锁，多人可只读协同 |
| 评审场景 | 主持人写锁，参与者只读批注 |

## 3.3 锁超时

- 默认租约：`30min`
- 心跳续租：`5min`
- 超时释放后进入可接管状态，但保留未提交草稿快照

---

## 4. 工具版本绑定与结果可复现性

每次 `SimulationTask` 必须绑定：

- 工具名称
- 工具版本
- 求解器版本
- 网格版本或输入模型版本
- 参数快照
- 运行环境标识

结果包必须可追溯到以上信息，否则不得进入“正式发布”状态。

---

## 5. 生命周期

## 5.1 `SimulationTask`

`Draft -> Prepared -> Submitted -> Running -> Completed/Failed -> Archived`

## 5.2 `HPCJob`

`Created -> Queued -> Running -> CallbackReceived -> Succeeded/Failed -> Closed`

## 5.3 `ResultPackage`

`Registered -> Verified -> Published -> Tiered -> Archived`

---

## 6. 并发与冲突处理

- 同一 `SimulationTask` 默认只允许一个活动中的 `HPCJob`
- 需要重复试算时，生成新的 `SimulationTask` 或新版本，不覆盖原任务
- 同一 `ResultPackage` 不允许被多个回调并发写入；按 `resultPackageId + artifactId` 幂等
- 工具流步骤失败不得直接把 `SimulationTask` 标记完成，必须经 `Toolflow Orchestration` 汇总后回写

---

## 7. 结果回流规则

1. `HPCJob` 成功后先登记结果摘要
2. `ResultPackage Service` 校验结果完整性和版本绑定
3. 大文件进入对象存储，元数据进入主存
4. 通过 `TraceLink` 关联到 `SimulationTask`、`Deliverable`、`KnowledgeAsset`
5. 完成后才允许触发知识沉淀和搜索投影

---

## 8. 失败与人工接管

以下情况必须人工接管：

- 回调缺失超过 SLA
- 结果校验失败但作业已结束
- 工具版本缺失或不可追溯
- 同一模型发生多方并发编辑冲突

接管角色：

- 工具链负责人：工具和求解器问题
- 平台运维：调度和回调问题
- 领域负责人：业务结果是否可接受
