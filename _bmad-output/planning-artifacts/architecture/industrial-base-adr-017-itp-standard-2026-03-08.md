# ADR-017 行业模板包（ITP）格式与加载标准

状态：`Proposed / Draft v0.1`  
日期：`2026-03-08`  
优先级：`P0`

---

## 1. 背景

当前 `v0.2` 底座已经具备：

- 共享内核能力
- 元模型扩展方向
- 租户与部署隔离策略
- 边缘/HPC/AI Worker 的方向性设计

但仍缺少一个可用于“快速复制交付”的标准载体。  
如果没有标准化交付包，以下目标无法成立：

- 新客户 3-4 周上线
- 行业方案复用
- 模板市场
- 租户初始化自动化
- 差量升级和版本化交付

因此需要正式引入 `Industry Template Package (ITP)`。

---

## 2. 决策

采用 `Industry Template Package (ITP)` 作为底座快速复制和行业方案装配的标准载体。

ITP 的定位是：

> 在统一对象模型、统一权限模型和统一发布治理之上，对行业模型、流程、视图、规则、主数据、连接器和权限模板进行标准化封装的版本化交付包。

---

## 3. 适用范围

ITP 适用于：

- 新租户初始化
- 新行业场景装配
- 新客户快速复制
- 同行业模板升级
- 预置连接器和主数据装载

ITP 不适用于：

- 核心内核升级
- 核心对象语义重定义
- 绕开审批和发布链路的本地热改

---

## 4. ITP 包结构标准

建议采用如下目录结构：

```text
Industry Template Package (ITP)
├── manifest.yaml
├── object-models/
├── workflows/
├── views/
├── dashboards/
├── rules/
├── connectors/
├── master-data/
├── permissions/
├── seeds/
├── plugins/
└── docs/
```

说明：

- `object-models/`
  - 白名单对象派生、扩展字段、状态机绑定、SchemaVersion 定义
- `workflows/`
  - 流程模板、任务模板、角色绑定模板
- `views/`
  - 表单、列表、详情、查询模板
- `dashboards/`
  - 工作台、看板、统计模板
- `rules/`
  - 校验规则、策略规则、计算规则
- `connectors/`
  - 预置连接器配置、外部系统映射配置
- `master-data/`
  - 字典、编码规则、参照数据、分类体系
- `permissions/`
  - 角色模板、对象授权模板、字段授权模板
- `seeds/`
  - 演示数据或初始化数据
- `plugins/`
  - 插件声明、挂载点配置、扩展描述
- `docs/`
  - 模板说明、适用范围、依赖约束、实施指引

---

## 5. `manifest.yaml` 标准

## 5.1 必填字段

```yaml
apiVersion: itp/v1
kind: IndustryTemplatePackage
metadata:
  name: aerospace-tdm-basic
  displayName: 航空试验基础模板包
  version: 1.0.0
  owner: platform-architecture
spec:
  targetDomains:
    - TDM
  basePlatformVersion: ">=0.2.0"
  baseSchemaVersion: "schema-tdm-base-1.0"
  dependencies:
    - mdm-base@1.0.0
  loadOrder:
    - master-data
    - object-models
    - permissions
    - workflows
    - views
    - dashboards
    - connectors
  extensionPoints:
    - pluginMounts
    - postLoadHooks
  checksum: sha256:...
```

## 5.2 推荐字段

- `supportedIsolationModes`
- `supportedDatabases`
- `rollbackPolicy`
- `requiredConnectors`
- `requiredFeatures`
- `sampleDataIncluded`
- `postLoadValidation`

## 5.3 强约束

- `manifest.yaml` 必须声明 `basePlatformVersion`
- 必须声明 `baseSchemaVersion`
- 必须声明依赖关系和加载顺序
- 必须声明兼容的租户/隔离模式
- 必须可追溯到发布人、发布时间、审核记录

---

## 6. ITP 加载流程

标准加载流程建议为：

1. 包完整性校验
2. 依赖解析
3. 平台版本与 SchemaVersion 兼容校验
4. 租户或项目空间创建
5. 主数据装载
6. 对象模型注册/派生
7. 权限模板装载
8. 工作流与任务模板装载
9. 页面/看板装载
10. 连接器配置注入
11. 后置校验
12. 发布生效
13. 审计记录生成

关键原则：

- `注册后可预编排，发布后才生效`
- 不采用无闸门的“注册即生效”
- 任一步失败都必须支持回滚

---

## 7. 插件挂载点规范

为了兼容 `Phase 3` 的生态扩展，`manifest.yaml` 必须预留插件挂载点：

```yaml
pluginMounts:
  - name: custom-validator
    mountAt: rule.beforeSubmit
    type: validator
  - name: external-worker
    mountAt: workflow.worker
    type: ai-worker
```

允许的挂载点类型：

- `rule.beforeSubmit`
- `rule.afterApprove`
- `workflow.worker`
- `connector.adapter`
- `view.widget`
- `dashboard.card`

不允许的挂载点：

- 核心权限内核
- 核心审计内核
- `TraceLink` 引擎核心逻辑
- 租户与隔离基础设施核心逻辑

---

## 8. 版本与升级规则

ITP 版本建议遵循：

- 主版本：破坏性变化
- 次版本：兼容性增强
- 修订版本：缺陷修复

升级约束：

- ITP 升级必须与 `SchemaVersion` 联动
- 客户差量配置必须可识别
- 必须支持升级前校验和升级后校验
- 必须支持回滚到上一个已发布模板版本

---

## 9. 与租户初始化的关系

ITP 加载不是单纯“导入模板”，而应纳入标准初始化流程：

1. 选择 ITP
2. 创建租户/项目空间
3. 绑定隔离策略
4. 注入主数据
5. 注册模型与模板
6. 执行连接器配置
7. 验证可访问性

因此 ITP 是交付流程的中心对象，而不是附件。

---

## 10. 验收标准

一版 ITP 规范至少满足：

- 可被机器校验
- 可声明依赖关系
- 可声明扩展点
- 可被标准加载器执行
- 可回滚
- 可审计
- 可适配不同隔离模式

---

## 11. 后续影响

采纳 ADR-017 后，需要同步回写：

- 技术架构蓝图
- 扩展治理体系文档
- `Phase 0` 纵切场景方案
- `ADR-018/019`

---

## 12. 证据链

- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/architecture/industrial-base-party-mode-review-report-2026-03-08.md`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/architecture/industrial-base-adr-decision-list-2026-03-08.md`
- `/Users/enjoyjavapan163.com/Documents/方案雏形/4- 基础底座/_bmad-output/planning-artifacts/architecture/industrial-base-technical-architecture-blueprint-2026-03-08.md`

