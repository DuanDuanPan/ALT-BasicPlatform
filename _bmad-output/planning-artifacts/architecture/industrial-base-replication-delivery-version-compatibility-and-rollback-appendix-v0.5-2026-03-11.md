# 工业研发底座复制交付版本兼容与回滚附录

状态：`Draft v0.5`  
日期：`2026-03-11`  
对应修订项：`S-02`

---

## 1. 目标

本附录定义 `ITP`、平台内核、`SchemaVersion` 和客户差量之间的兼容关系、升级粒度和回滚契约。

---

## 2. 版本对象

| 对象 | 含义 |
| --- | --- |
| `PlatformCoreVersion` | 平台内核版本 |
| `ITPVersion` | 模板包版本 |
| `SchemaVersion` | 模型版本 |
| `LocalDeltaVersion` | 客户差量版本 |
| `ReleasePackageVersion` | 发布包版本 |

---

## 3. 兼容性矩阵

| 组合 | 默认策略 | 说明 |
| --- | --- | --- |
| `ITP minor` on same `PlatformCore major` | 兼容 | 允许直接升级，需跑兼容性校验 |
| `ITP major` on same `PlatformCore major` | 条件兼容 | 需人工审批和预演 |
| `ITP major` on lower `PlatformCore major` | 不兼容 | 必须先升级内核 |
| `SchemaVersion minor` with same `MetaObjectType` | 条件兼容 | 非破坏性字段可自动处理 |
| `SchemaVersion major` with local deltas | 需人工合并 | 必须跑冲突检测 |
| `LocalDeltaVersion` on retired `SchemaVersion` | 不兼容 | 必须先迁移或回退 |

---

## 4. 升级粒度

支持三类升级：

1. 模板包升级  
   - 作用对象：单 `ITP`
2. 模型版本升级  
   - 作用对象：单对象族或单 `SchemaVersion`
3. 租户级升级  
   - 作用对象：租户环境中的模板、模型、视图和连接器组合

不支持：

- 无计划的跨租户整体回滚
- 绕过发布治理的局部热修复

---

## 5. 回滚粒度

| 粒度 | 允许 | 回滚内容 |
| --- | --- | --- |
| 单模板包 | 是 | 模板清单、视图、动作、数据源绑定、注册项 |
| 单 `SchemaVersion` 差量 | 是 | `localDelta`、派生字段、视图覆写、规则覆写 |
| 单租户发布包 | 是 | 租户初始化项、模板集合、配置快照 |
| 平台内核 | 受限 | 仅在发布窗口内，需全量影响评估 |

---

## 6. 升级流程

1. 生成升级计划
2. 校验 `PlatformCoreVersion / ITPVersion / SchemaVersion`
3. 比对 `LocalDeltaVersion`
4. 生成兼容性报告
5. 执行预演
6. 批准后正式发布
7. 自动注册和数据迁移
8. 结果审计与回执

---

## 7. 差量升级技术方案

`v0.5` 采用三段式方案：

- 模型差量：`SchemaVersion + localDelta`
- 配置差量：结构化 patch 清单
- 数据迁移：受控 migration script

约束：

- 不使用无版本信息的裸 `JSON Patch`
- 不使用不可回放的临时脚本
- 不使用直接修改生产库的人工 SQL 作为标准方案

---

## 8. 回滚后自动清理规则

回滚完成后必须自动处理：

- 搜索索引注销或重建
- 权限点注销
- 审计类别回滚登记
- 视图和动作目录恢复
- 无主连接器绑定清理

图谱与审计要求：

- `TraceLink` 不做物理删除，标记为失效版本
- 审计记录永不回滚删除，只新增“回滚动作”记录

---

## 9. 迁移与兼容性检查清单

发布前至少检查：

- 核心字段是否破坏兼容
- 规则绑定是否冲突
- 视图引用是否失效
- 已删除字段是否被连接器、报表或动作依赖
- `localDelta` 是否涉及核心语义字段

---

## 10. 成功与失败判定

升级成功：

- 新版本激活
- 自动联动注册完成
- 搜索/权限/审计对账通过
- 没有遗留死信或未确认冲突

升级失败：

- 兼容性校验未通过
- 自动联动注册失败
- 迁移脚本失败且无法自动补偿
- 回滚后仍存在失效引用或未清理注册项
