# FED 与能力包关系及完整层次结构讨论纪要

- 日期: 2026-03-15
- 讨论方式: Party Mode 多角色收敛
- 参与角色: BMad Master, Architect (Winston), PM (John), UX Designer (Sally)
- 前序讨论: Page 04 编辑态交互范式讨论纪要 (2026-03-15)、Page 04/04A 讨论纪要 (2026-03-13)、模型化架构专题讨论纪要 (2026-03-11)
- 讨论主题: FED 与能力包的关系、完整层次结构（P0-P5 × M0-M2）、平台⇄产品⇄项目双向流动

## 一、讨论背景

前序讨论已确定：
- FED 是编辑态操作的自动产出物（2026-03-15 编辑态讨论）
- 模块是定义者（owning），能力包是装配者（including）（2026-03-13 讨论）
- M 轴四级继承 M0→M0.5→M1→M2（2026-03-11 模型化讨论）

本轮讨论需要解决的问题：
1. FED 的分发单元是什么？与能力包如何关联？
2. 跨模块 FED 如何处理？
3. 完整层次结构是否有遗漏？
4. 平台→产品→项目如何逐级支撑？项目→产品→平台如何逐级积累？

## 二、已达成共识

### 1. FED 的五类依赖

FED 的分发方式由依赖图谱决定，不预设固定答案：

| 依赖类型 | 说明 | 示例 |
|---|---|---|
| Schema 依赖 | 目标对象的 SchemaVersion | WorkPackage >= v2.1 |
| 组件依赖 | ComponentRegistry 中的组件 | graph-canvas, PerformanceBadge |
| 视图依赖 | 目标 PageTemplate 的 ExtensionPoint | node-card.badges slot |
| 行为依赖 | ActionTemplate / RuleSet / FlowModel | recalculatePerformance |
| 数据依赖 | 跨对象引用 | Task.actualHours |

### 2. FED 便携性自动分级

| FED 级别 | 依赖范围 | 典型场景 | 分发方式 |
|---|---|---|---|
| 轻量 (independent) | 仅 Schema + 展示组件 | 加几个字段 + 列表列 | 可独立分发 |
| 中量 (module-bound) | Schema + 视图 + 行为 | 绩效扩展（字段+面板+计算+联动） | 需绑定模块 |
| 重量 (pack-bound) | 跨模块 Schema + 行为 + 流程 | 绩效贯穿工作包+项目+报表 | 需绑定能力包 |

便携性由系统自动计算：

```text
if 无 extensionPoints 依赖 && 无 external actions && 无 crossObject 依赖:
   → independent
elif 所有 extensionPoints 属于同一模块 && external actions 属于同一模块:
   → module-bound
else:
   → pack-bound
```

### 3. FED 与模块/能力包的归属关系

```text
定义归属（owning）:
  FED 始终定义在 CapabilityModule 中

装配暴露（including）:
  能力包装配模块时，模块内的 FED 自动随模块带出
  能力包不直接定义或拥有 FED

独立分发（exporting）:
  轻量 FED 可脱离模块独立分发
  但导入时必须有兼容的目标模块接收（FED 寄居在目标模块中）
```

### 4. 跨模块 FED 的处理

不做"一个 FED 跨多个模块"，而是拆成多个协作 FED：

```text
performance-metrics-workpackage    (属于 work-package-center 模块)
  → 定义绩效字段、节点标记、计算逻辑

performance-metrics-project-view   (属于 project-overview 模块)
  → 引用 WorkPackage.performanceScore（crossObject 依赖）
  → 定义项目级聚合面板

两个 FED 通过 dependencies.crossObject 建立关联
打包时必须一起进入同一个能力包
→ portability = pack-bound
```

### 5. 能力包的 FED 控制机制

能力包新增三层 FED 声明：

| 层次 | 说明 | 示例 |
|---|---|---|
| requiredFEDs | 随包必须启用的 FED | 核心绩效字段 |
| optionalFEDs | 客户可选启用的 FED | 绩效报表扩展 |
| fedCoordination | 跨模块 FED 的启用约束 | 工作包+项目必须同时启用 |

### 6. 包级编排模块正式纳入

03-13 讨论中提出的"包级编排模块"正式成为 P1 的标准概念：

- 承载跨模块的总览页、汇总页、装配页
- 归属于能力包，专门承载跨模块协调视图
- 例: 项目绩效总览（汇总多个模块的绩效数据）

### 7. DisplayComponent 注册不需要审批流程

M1/M2 注册的自定义展示组件无需通过审批流程。

## 三、完整层次结构（P0-P5）

```text
P0  产品基线层（ProductBaseline）
    定位: 产品族的顶层锚点，定义出厂形态
    包含: 预置/推荐的 CapabilityPack 集合
    M轴: M0/M0.5

P1  能力包层（CapabilityPack）
    定位: 装配/交付层——面向客户的能力交付单元
    职责: 装配模块、暴露能力、协调跨模块 FED 依赖
    包含: modules[], requiredFEDs[], optionalFEDs[],
          fedCoordination[], orchestrationModule, exposedPages[]
    M轴: M1（领域）或 M2（客户自建）

P2  模块层（CapabilityModule）
    定位: 内部构件层——业务能力的定义单元
    职责: 定义资产（页面、对象、动作、流程、规则、FED）
    关系: 资产的唯一所有者（owning）
    M轴: M0.5/M1/M2

P3  资产层（Module Assets）
    定位: 模块的内容物
    包含: PageTemplate, ActionModel, DomainObject,
          FlowModel, RuleSet, ConnectorGroup, FeatureExtension

P4  构件层（Asset Internals）
    定位: 资产的内部结构，编辑态操作的最细粒度
    包含: ExtensionPoint, FieldBinding, ActionBinding,
          ContextFieldMapping, TemplatePatch, TemplateVariant,
          DesignChangeset

P5  引擎与组件层（ComponentRegistry）
    定位: 平台能力基座——硬编码的交互能力
    包含: Engine（不可替换）, CapabilityComponent（可配置）,
          DisplayComponent（可注册）, FieldType（可注册）
```

### 层次结构 × M 轴 二维视图

```text
              M0 平台内核    M0.5 公共业务    M1 领域 ITP    M2 客户差异
             ─────────────────────────────────────────────────────────
P0 基线       平台出厂基线    ─              领域产品基线    客户派生变体
P1 能力包     平台基础包      公共能力包      领域能力包     客户自建包
P2 模块       核心模块        公共模块        领域模块       客户扩展模块
P3 资产       核心对象/模板   公共对象/模板   领域资产       客户资产+FED
P4 构件       基础字段绑定    公共变体        领域变体/Patch  客户 Patch
P5 组件       引擎+能力组件   ─              展示组件注册    展示组件注册
```

### 各 M 层的变更频率与方式

| M 层 | 控制者 | 变更频率 | 变更方式 |
|---|---|---|---|
| M0 | 平台团队 | 年级 | 代码发布 |
| M0.5 | 平台团队 | 半年级 | 代码 + Schema 发布 |
| M1 | 领域团队 | 季度级 | ITP 包发布 |
| M2 | 客户/实施 | 月级 | 编辑态 → FED → Draft → 发布 |

## 四、双向流动：平台 ⇄ 产品 ⇄ 项目

### 自顶向下：逐级支撑

```text
平台（M0/M0.5）
  提供: 引擎 + 组件 + 扩展点机制 + 编辑态渲染引擎 + 治理管线
  承诺: 在引擎和扩展点上做的任何配置，平台保证可渲染、可审计、可回滚
      ↓
产品（M1 领域 ITP）
  利用: 平台引擎 + 预埋 ExtensionPoint
  产出: 完整业务模块 + 能力包
  承诺: 启用能力包即开箱可用；通过编辑态在预埋扩展点上可安全扩展
      ↓
项目（M2 客户实施）
  利用: 产品能力包 + 编辑态
  操作: 进入设计模式 → 扩展字段/组件/页签 → Draft → 影响分析 → 发布
  产出: FED（自动生成的结构化变更单元）
```

每一层的承诺是下一层安全操作的前提。

### 自底向上：逐级积累

```text
项目（M2）→ 产品（M1）:
  多个项目产出相似 FED → 领域团队提炼共性 → 标准 FED 纳入 ITP
  晋升条件: ≥2 项目验证 + 依赖收敛 + 无客户专有逻辑 + 领域评审

产品（M1）→ 平台（M0/M0.5）:
  多个领域出现相似模式 → 平台团队抽象通用能力 → 组件沉淀到 ComponentRegistry
  晋升条件: ≥2 领域复现 + 通用性抽象 + 平台接管维护 + 性能基线达标
```

**飞轮效应：** 每一个客户项目的 M2 定制，都是平台未来能力的"免费 R&D"。项目做得越多 → FED 积累越丰富 → 领域 ITP 越完整 → 新客户交付越快 → 更多项目 → 更多积累。

### 价值循环图

```text
          ┌──── 平台能力扩大 ←─── 跨领域共性提炼 ────┐
          │                                           │
          ▼                                           │
    ┌──────────┐                               ┌──────────┐
    │  M0/M0.5 │ ─── 引擎+组件+扩展点机制 ──→  │  M1 领域  │
    │  平台    │                               │  ITP     │
    └──────────┘                               └──────────┘
          │                                           │
          │ 平台保证运行质量                    领域保证业务完整性
          │                                           │
          │         ┌──────────┐                      │
          └────→    │  M2 项目  │  ←──── 能力包交付 ──┘
                    │  实施     │
                    └──────────┘
                          │
                    编辑态产出 FED
                          │
                    多项目积累 → M1 提炼标准
                    多领域积累 → M0 抽象通用
```

## 五、工作包 + 绩效场景完整映射

### P5 层（M0 平台内核）

- `graph-canvas` 引擎 → 工作包分解的图形化拖拽
- `form-engine` → 节点信息设置面板
- `io-configurator` → 输入/输出设置
- `relation-browser` → 使能/控制关系面板
- `computed` 字段类型 → 绩效得分计算

### P4 层（M0.5 平台 + M2 客户）

- 5 个 ExtensionPoint 预埋在工作包分解页面中
- M2 编辑态产出: +4 FieldBinding, +1 TemplatePatch, DesignChangeset

### P3 层（M0.5 + M1 领域 + M2 客户）

- M0.5: WorkPackage 基础对象 (v2.0)
- M1: work-package-decomposition PageTemplate + 5 个 ActionModel + FlowModel + RuleSet
- M2: performance-metrics-workpackage FED + performance-metrics-project-view FED

### P2 层（M1 领域 + M2 客户）

- M1: work-package-center 模块 + project-overview 模块
- M2: 两个模块各自新增绩效 FED

### P1 层（M1 领域 + M2 客户）

- M1: project-execution-pack 能力包（含工作包中心 + 任务管理 + 项目总览）
- M2: project-performance-pack 能力包（含绩效 FED 协调 + 包级编排模块）

### P0 层（M0.5 + M2）

- M0.5: OLTran.CDM v1.0 产品基线（预置 project-execution-pack）
- M2: 航天院 CDM 定制版（启用 project-performance-pack）

## 六、下一步

1. ✅ 沉淀讨论纪要（本文档）
2. 更新对象模型，补充 FED 依赖声明、能力包 FED 控制、包级编排模块
3. 更新 ADR，包级编排模块正式入模
4. 设计汇报逻辑，在 PPT 和原型中体现完整层次结构与双向流动
