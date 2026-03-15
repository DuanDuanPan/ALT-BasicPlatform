# 扩展单元机制与全场景验证讨论纪要

- 日期: 2026-03-15
- 讨论方式: Party Mode 多角色收敛
- 参与角色: Architect (Winston), PM (John), UX Designer (Sally), Developer Agent (Amelia)
- 前序讨论: FED 与能力包关系讨论 (2026-03-15)、编辑态交互范式讨论 (2026-03-15)
- 讨论主题: 代码级扩展机制设计、全场景压力测试、FED 生成模式与防错机制

## 一、讨论背景

前序讨论确定了 FED 的零代码扩展机制（schemaDelta / viewBindings / actionBindings），但以下场景超出零代码能力边界：
- 自定义算法（加权聚合、趋势预测）
- 自定义展示组件（雷达图+热力图混合仪表盘）
- 多对象复合操作（创建工作包+生成子任务+分配资源+写 TraceLink）
- 事件编排（绩效低于阈值→分析→整改→通知）
- 流程触发（子工作包全完成→自动发起父级评审）
- 外部系统集成（MATLAB/LabVIEW/ANSYS 等商业软件调用）
- 前后端配套实现

本轮讨论需要设计代码级扩展机制，并确保其纳入 FED 的复用、积累、沉淀体系。

## 二、已达成共识

### 1. 扩展机制统一模型——四个概念各司其职

| 概念 | 职责 | 层级 | 一句话 |
|---|---|---|---|
| **ExtensionPoint** | WHERE — 哪里可以扩展 | P4 构件层 | 插座 |
| **FED** | WHAT — 扩展了什么 | P3 资产层 | 扩展记录（零代码 + 代码级引用） |
| **Extension Unit** | HOW — 用什么实现 | P5 组件层 | 电器（ServerUnit 后端 + ClientUnit 前端） |
| **ConnectorGroup** | INTEGRATE — 跟外部怎么连 | P3 资产层 | 集成声明 |

### 2. plugins 概念废弃，统一为 Extension Unit

| | 旧概念 plugins | 新概念 Extension Unit |
|---|---|---|
| 定义 | 模糊——"客户特有适配扩展" | 明确——有接口契约、依赖声明、沙箱约束 |
| 管理 | 独立存在，不被 FED 追踪 | 必须通过 FED 引用，纳入统一管理 |
| 复用 | 无标准化机制 | 注册到 Registry，可搜索、引用、晋升 |
| 安全 | 无沙箱要求 | 沙箱执行 + 接口校验 |
| 分发 | 手动部署 | 随 FED/ITP 自动分发 |

ITP 包结构更新：`behavior/plugins/` 废弃，改为 `behavior/server-units/` + `behavior/client-units/`。

### 3. ExtensionPoint 范围扩展至后端层

前序讨论只在 UI 层定义了扩展点（fields/badges/tabs/actions）。本轮扩展到后端层：

**UI 层扩展点（已有）：**
- `node-card.body.fields` → 挂字段
- `node-card.badges` → 挂标记
- `side-panel.tabs` → 挂页签
- `toolbar.actions` → 挂按钮

**后端层扩展点（新增）：**
- `action.{name}.compute` → 挂自定义计算
- `action.{name}.validate` → 挂自定义校验
- `action.{name}.pre` → 挂前置处理
- `action.{name}.post` → 挂后置处理
- `event.{name}.handler` → 挂事件处理链
- `flow.{name}.gateway` → 挂流程网关决策
- `connector.{name}.transform` → 挂数据转换

### 4. ServerUnit 设计

后端扩展单元，覆盖算法、多对象操作、事件处理、流程触发、外部集成等所有后端逻辑。

**三级复杂度：**

| 级别 | 方式 | 谁写 | 运行环境 |
|---|---|---|---|
| Level 1 表达式 | 编辑态配公式 | 实施顾问 | 前端/后端内联计算 |
| Level 2 脚本 | 简单函数 | 高级实施/开发 | 平台沙箱（QuickJS / V8 Isolate） |
| Level 3 服务 | 完整服务/容器 | 开发团队 | 容器化部署 |

**接口与实现分离：** 接口声明（inputs/output）统一，实现可从 Level 1 升级到 Level 3 而不影响调用方。

**接口契约核心字段：**
- `trigger`: action | event | schedule | api
- `inputs` / `outputs`: 参数类型声明
- `errors`: 错误码与可恢复性
- `touchedObjects`: 涉及的领域对象及操作类型（create/update/delete）
- `externalDependencies`: 依赖的外部服务
- `crossTenantAccess`: 跨租户数据访问声明（需管理员授权）
- `resources`: CPU/Memory/GPU/License/超时/并发限制

### 5. ClientUnit 设计

前端扩展单元，覆盖自定义展示组件、交互行为、编辑器。

**接口契约核心字段：**
- `props`: 属性声明
- `slots`: 内容插槽
- `events`: 事件声明
- `actions`: 可调用的后端能力（引用 ServerUnit）

**前后端配对机制：** ClientUnit 通过 `actions` 字段声明式引用 ServerUnit，运行时通过 `platform.execute()` 统一入口调用。平台负责权限、审计、路由、沙箱。

### 6. FED 的完整结构（含代码级扩展）

```yaml
featureExtension:
  meta:
    name: performance-metrics-workpackage
    version: 1.0.0
    portability: module-bound

  # 零代码部分
  schemaDelta: [...]
  viewBindings: [...]
  actionBindings: [...]

  # 代码级扩展引用
  extensionUnits:
    serverUnits:
      - ref: weighted-performance-score@1.0
        binding: action.computeScore.compute
      - ref: perf-threshold-alert-chain@1.0
        binding: event.scoreChanged.handler
    clientUnits:
      - ref: performance-radar-dashboard@1.0
        binding: side-panel.tabs.perf-dashboard

  # 链式编排（链式扩展场景）
  orchestration:
    - chain: data-to-report
      steps:
        - unit: binary-to-measurement
          trigger: event.dataAcquired
          next: vibration-spectrum-analysis
        - unit: vibration-spectrum-analysis
          parallel: [matlab-fatigue-analysis]
          next: extract-analysis-result
        - approval: result-review          # 人机交替
          type: workflow
          next: generate-report
        - unit: generate-report
          next: null
      errorStrategy: stop-on-first
      loopGuard:                           # 防循环
        source: analysis-chain
        ignoreIf: "source == 'reverse-sync'"
```

### 7. 声明式能力 vs 代码级扩展的边界

| 场景特征 | 用声明式（P3 资产） | 用代码级（Extension Unit） |
|---|---|---|
| 标准条件→动作 | ✅ RuleSet | |
| 标准审批流程 | ✅ FlowModel + Workflow | |
| 单对象操作 | ✅ ActionModel | |
| 事件→简单联动 | ✅ FlowModel 事件链 | |
| 非标算法 | | ✅ ServerUnit |
| 跨聚合根事务 | | ✅ ServerUnit |
| 包含业务决策的编排 | | ✅ ServerUnit |
| 商业软件调用 | | ✅ ServerUnit Level 3 |
| 非标准 UI 组件 | | ✅ ClientUnit |

**代码级扩展是声明式能力的"逃逸舱"。当一个 ServerUnit 被足够多领域使用后，平台可以将其吸收为声明式能力的新标准模式。**

## 三、全场景压力测试结果

### TDM 试验域

| 场景 | 结果 | 机制 |
|---|---|---|
| 多源数据采集（GPIB/CAN/1553B） | ✅ | ConnectorGroup |
| 数据格式转换（二进制→结构化） | ✅ | ServerUnit (data-transform) |
| Python 自研算法 | ✅ | ServerUnit Level 3 容器 |
| 商业软件调用（MATLAB/ANSYS） | ✅ | ServerUnit Level 3 + License 管理 |
| 链式编排（采集→转换→分析→报告） | ✅ | FED.orchestration → Task Orchestration |
| 定制看板 | ✅ | ClientUnit |

### MIS 数字化运营域

| 场景 | 结果 | 机制 |
|---|---|---|
| 合同履约风险动态评估 | ✅ | ServerUnit + 定时触发 |
| 预算多层审批 | ✅ | 声明式即可（RuleSet + Workflow），不需要 Extension Unit |
| 多法人主数据冲突仲裁 | ✅ | ServerUnit + crossTenantAccess 声明 |

### CDM 协同研发域

| 场景 | 结果 | 机制 |
|---|---|---|
| MBSE 模型双向同步 | ✅ | ServerUnit + ConnectorGroup + loopGuard 防循环 |
| 多学科协同仿真（人机交替） | ✅ | orchestration 区分 unit（自动）和 approval（人工） |
| 研发知识自动抽取与推荐 | ✅ | ServerUnit (NLP) + ClientUnit (推荐侧栏)，符合 AD-13 |

### 压力测试发现的三个边界补充

| # | 发现 | 来源场景 | 补充方案 |
|---|---|---|---|
| 1 | 跨租户数据访问 | MIS 主数据冲突仲裁 | ServerUnit 接口增加 `crossTenantAccess` 声明 |
| 2 | 双向同步防循环 | CDM MBSE 双向同步 | orchestration 增加 `loopGuard` |
| 3 | 人机交替编排 | CDM 多学科仿真 | orchestration step 区分 `unit`（自动）和 `approval`（人工） |

## 四、FED 生成模式

| 角色 | 方式 | 场景 |
|---|---|---|
| M2 实施顾问 | **100% 自动生成** | 编辑态操作，零 YAML |
| M1 领域开发者 | **自动生成 + 手动增强** | 零代码部分自动生成，代码级引用和 orchestration 手动补充 |
| M1 领域开发者（推荐） | **FED 可视化编辑工作台** | 拖拽引用 + 可视化编排 + 实时校验 |
| 紧急/高级场景 | **直接编辑 YAML** | 四道防线保障 |

### FED 防错四道防线

| 防线 | 时机 | 机制 |
|---|---|---|
| 第一道 | 编写时 | IDE 实时校验（JSON Schema 自动补全 + 标红） |
| 第二道 | 提交时 | 契约校验（引用完整性、接口匹配、链路完整性） |
| 第三道 | 注册时 | 平台校验（兼容性、冲突检测、依赖图谱、资源） |
| 第四道 | 发布时 | 影响分析（DesignChangeset 治理流程） |

## 五、ITP 包结构更新

```text
{domain}-itp-v{version}/
  ├── schema/
  ├── templates/
  ├── behavior/
  │     ├── views/                    ← PageTemplate (UI Schema)
  │     │     └── extensions/         ← FED 声明
  │     ├── server-units/             ← ServerUnit（替代原 plugins/）
  │     │     ├── weighted-perf-score.yaml
  │     │     └── weighted-perf-score.js
  │     └── client-units/             ← ClientUnit（替代原 plugins/）
  │           ├── perf-radar-dashboard.yaml
  │           └── perf-radar-dashboard/
  ├── runtime/
  └── pack-coordination/
```

## 六、晋升路径统一

Extension Unit 与 FED 走同一条晋升路径：

```text
M2 项目实施
  → FED + ServerUnit + ClientUnit 作为整体管理
  → 便携性自动计算时，ServerUnit 的 touchedObjects 和
    externalDependencies 纳入依赖图谱

M1 领域标准化
  → 接口锁定（inputs/output/props 稳定化）
  → 实现可替换（脚本→服务），接口不变
  → 纳入 ITP

M0 平台沉淀
  → 通用算法沉淀为平台内置 ComputeEngine
  → 通用组件从 display-components 晋升到 capability-components
  → 通用操作模式吸收为声明式能力的新标准模式
```

## 七、下一步

1. ✅ 沉淀讨论纪要（本文档）
2. 更新统一对象模型，补充 Extension Unit 相关概念
3. 更新 ADR，废弃 plugins 概念，正式引入 Extension Unit
