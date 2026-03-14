# 工业研发底座 SchemaVersion 演进策略

状态：`Draft v0.1`  
日期：`2026-03-08`  
用途：定义 `MetaObjectType / SchemaVersion` 的版本继承、差量管理、冲突处理和回滚策略。

---

## 1. 目标

`SchemaVersion` 不是简单的模型版本号，而是底座支持“模板升级 + 客户差量保留 + 柔性对象演化”的关键机制。

这份策略主要解决：

1. 模型如何版本化
2. 模板包升级时客户差量如何保留
3. 冲突何时自动处理，何时人工决策
4. 发布与回滚如何受控

---

## 2. 版本模型

建议 `SchemaVersion` 至少包含以下结构：

```yaml
schemaVersionId: schema-tdm-testspec-1.2.0
metaObjectType: TestSpec
baseVersion: schema-tdm-testspec-1.0.0
parentVersion: itp-aero-tdm-1.1.0
localDelta:
  addedFields: []
  modifiedFields: []
  removedFields: []
  stateBindings: []
  ruleBindings: []
mergeStrategy: auto|manual|locked
status: draft|approved|published|rolled_back
```

---

## 3. 三类版本来源

## 3.1 平台基线版本

- 由平台统一维护
- 代表核心对象与基础柔性对象的官方基线

## 3.2 模板版本

- 由 ITP 提供
- 代表行业模板层的扩展

## 3.3 客户差量版本

- 由租户、项目或客户环境引入
- 代表本地个性化调整

---

## 4. Phase 划分

## Phase 1：必须具备

- 版本继承
- 差量追踪
- 手工冲突检测
- 审批、发布、回滚

## Phase 2：增强能力

- `3-way merge`
- 自动冲突分类
- 冲突修复建议
- 升级预演

结论：

`3-way merge` 是目标能力，但不应卡死当前第一批交付。

---

## 5. 冲突类型

建议先定义五类冲突：

1. 字段同名不同类型
2. 字段状态约束冲突
3. 规则绑定冲突
4. 视图引用冲突
5. 已删除字段被下游继续依赖

处理策略：

- 可自动处理：新增字段、非破坏性属性补充
- 必须人工处理：类型变化、状态机变化、规则冲突
- 禁止自动处理：删除关键字段、修改核心语义字段

---

## 6. 差量模型

建议客户侧不直接覆盖模板版本，而是保留 `localDelta`：

- `addedFields`
- `modifiedFields`
- `deprecatedFields`
- `viewOverrides`
- `ruleOverrides`

原则：

- 删除优先做“弃用标记”，不做物理删除
- 差量必须可审计
- 差量必须可独立回滚

---

## 7. 发布流程

标准流程建议为：

1. 变更申请
2. 生成新 `SchemaVersion`
3. 兼容性校验
4. 冲突检测
5. 审批
6. 发布
7. 自动联动注册
8. 审计落档

其中“自动联动注册”应触发：

- 搜索索引注册
- 权限点注册
- 审计点注册
- 基础视图元数据注册

---

## 8. 回滚策略

支持两类回滚：

### 版本回滚

- 回到上一个已发布 `SchemaVersion`

### 差量回滚

- 只撤销客户本地 `localDelta`

限制：

- 已进入核心业务对象正式提交的数据，不允许无痕回滚
- 回滚必须记录影响范围和恢复动作

---

## 9. 与 ITP 的关系

`ITP` 负责“模板交付包”，`SchemaVersion` 负责“模型演化控制”。

二者关系：

- ITP 装载时创建或引用 `SchemaVersion`
- ITP 升级时触发兼容性校验
- 客户差量配置基于 `SchemaVersion` 叠加

---

## 10. 当前结论

`SchemaVersion` 不应被理解成一个普通版本号字段。  
它是复制引擎能否成立的关键控制器：

- 没有它，模板升级会覆盖客户差量
- 没有它，元模型扩展无法受控
- 没有它，自动联动和自动生成都没有稳定基线

