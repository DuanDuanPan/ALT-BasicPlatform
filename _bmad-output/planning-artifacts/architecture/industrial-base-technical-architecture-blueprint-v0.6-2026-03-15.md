# 工业研发底座技术架构蓝图

状态：`Draft v0.6`
日期：`2026-03-15`
基线来源：`v0.5`（2026-03-11 冻结评审基线）

---

## 版本范围说明

本文档是 `v0.6 Draft`，在 `v0.5` 冻结基线的基础上演进。两个版本的范围边界如下：

**v0.5 已冻结内容（运行时视角，对应修订任务 M-01 至 M-09）：**
- 七层逻辑分层（L1-L7）与四条横切平面的运行时契约
- 质量属性场景（QA-01 至 QA-08）
- TraceLink 写入责任与多存储一致性原则
- 核心聚合根边界与跨域引用规则
- 安全判定时序、错误码与审计策略
- MVP Core / Wave 1 / Wave 2 / Deferred 统一优先级
- 纵切 A/B 最小依赖与验收标准
- 边缘、模板升级、HPC、部署与治理的原则级规则
- ADR-001 至 ADR-023

**v0.6 新增内容（产品与扩展视角）：**
- 产品架构层次（P0-P5）与运行时逻辑分层（L1-L7）正交体系（ADR-024）
- 引擎 + Schema 混合 UI 架构（AD-24）
- Extension Unit（ServerUnit/ClientUnit）替代原 plugins 概念
- FED 与编辑态范式
- 平台⇄产品⇄项目双向流动模型
- 扩展点（ExtensionPoint）从 UI 层扩展至后端层
- 概念精简：CapabilityPack 并入 ITP、ViewTemplate 统一为 PageTemplate、FlowModel 改名 OrchestrationModel、新增 SolutionInstance（方案包）

**v0.6 新增内容的配套状态：**
- ADR-024 已起草，状态 `Proposed`，待正式评审
- Extension Unit 运行时技术选型待冻结（当前为多方案并存，需 POC 验证后收敛）
- P0-P5 对象定义已纳入统一对象模型 §13.5，但尚未完成独立验收标准和责任人分配
- FED 可视化编辑工作台 UX 细化待后续版本

---

## 1. 蓝图目标

本版蓝图在 `v0.5` 已冻结的实施级问题基础上，新增回答六个产品架构级问题：

1. 运行时逻辑分层（L1-L7）与产品交付层次（P0-P5）的关系是什么——两者正交，不冲突。
2. 平台 UI 架构采用什么模式——引擎+Schema 混合架构（AD-24），支持 NocoBase 式编辑态。
3. 代码级扩展如何纳入统一治理——Extension Unit（ServerUnit/ClientUnit）替代原 plugins 概念。
4. 零代码扩展和代码级扩展如何统一管理——FED 同时管理声明式扩展和 Extension Unit 引用。
5. 扩展能力如何从项目积累为产品——FED 晋升路径 M2→M1→M0，飞轮效应。
6. 扩展的扩展点不仅限于 UI 层——ExtensionPoint 扩展至后端层（compute/validate/pre/post/handler/gateway）。

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

- 当前是技术中立蓝图，不绑定单一厂商实现，但已达到可指导实施的深度
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

关联 ADR：`ADR-001`、`ADR-002`、`ADR-006`、`ADR-010`、`ADR-013`、`ADR-014`、`ADR-015`、`ADR-016`、`ADR-017`、`ADR-023`、`ADR-024`

## 3.1 推荐总体形态

建议保持：

- `产品族共用平台内核`
- `1 个平台内核 + 3 个一级业务域 + 1 个复制交付工厂 + N 个工程工具/连接器`
- `七层逻辑分层（L1-L7）+ 四条横切平面`——运行时视角
- `六层产品架构（P0-P5）`——交付与扩展视角，与 L1-L7 正交；SolutionInstance（方案包）作为 P0 的项目级实例化
- `引擎 + Schema 混合 UI 架构（AD-24）`——编辑态与运行态共享 UI Schema
- `受控装配层 + 元模型驱动扩展体系 + Extension Unit 代码级扩展`
- `单实例 / 逻辑隔离 / 物理隔离` 三态部署
- `侧挂增强 + 受控 AI Worker` 双层智能策略

## 3.2 统一实施原则

- 共享能力优先进入 `L4`，业务语义优先进入 `L3`
- 单聚合事务只允许一个系统事实源
- 多存储只允许“主存写入 + 投影更新”，不允许业务层双写
- 图谱统一的是服务语义和治理语义，分治的是底层投影和存储实现
- 横切平面必须定义 injection point、事件、回调和失败责任
- 任何模板包、模型变更和发布动作都必须进入统一发布链路
- UI 层引擎（graph-canvas 等）硬编码保证交互品质，Schema 层保证灵活扩展，两者不混淆
- 代码级扩展（Extension Unit）必须通过 FED 引用和管理，不允许未经注册和治理的脱管扩展
- 编辑态变更必须经 DesignChangeset 的 Draft→影响分析→发布治理管线
- 扩展点（ExtensionPoint）覆盖 UI 层和后端层，统一"插座"机制

---

## 4. 逻辑架构蓝图

关联 ADR：`ADR-002`、`ADR-003`、`ADR-006`、`ADR-010`、`ADR-012`、`ADR-020`、`ADR-021`、`ADR-022`、`ADR-023`

## 4.1 七层逻辑分层

| 层级 | 名称 | 说明 | 主要能力 |
| --- | --- | --- | --- |
| L1 | 体验与装配层 | 统一入口和场景装配 | 门户、角色主页、工作台、表单、视图、看板、导航、页面区块、动作编排 |
| L2 | 场景应用层 | 面向用户的业务应用 | CDM 应用、TDM 应用、MIS 应用、知识应用 |
| L3 | 领域服务层 | 承载领域语义和核心聚合根 | 需求、项目、工作包、任务、试验、数据集、合同、设备、知识 |
| L4 | 平台共享服务层 | 承载共享内核能力 | 身份鉴别与访问控制、主数据、数据源管理、对象模型注册、对象与文件服务、轻量业务流程、任务编排、ACL 策略、密级与保密策略、加解密控制接口、元模型扩展、规则服务、搜索服务、数字主线服务（Trace Service）、图服务（Graph Service）、审计服务、集成控制面、配置发布、模型自动联动 |
| L5 | 数据与智能层 | 承载多模数据与检索知识底座 | 关系数据服务、文档数据服务、对象存储服务、图谱服务、测量/时序数据服务、搜索索引服务、知识库服务 |
| L6 | 集成与边缘层 | 承载外部系统、工具和设备接入 | API 网关、事件总线、文件交换、连接器、协议适配、HPC 代理、边缘代理 |
| L7 | 治理与运行层 | 承载运行治理与非功能保障 | 配置、发布、观测、安全运营、等保/分保合规、三员治理、日志保全与留存、密钥治理、租户隔离、国产化、资产目录与复用运营、应用包发布与升级治理 |

L1-L7 回答"请求怎么流转"，与 P0-P5（§4.4）正交——后者回答"资产谁定义、谁装配、谁扩展"。

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

## 4.4 产品架构层次（P0-P5）

关联 ADR：`ADR-024`、`ADR-010`、`ADR-017`

运行时逻辑分层（L1-L7）回答"请求怎么流转"，产品架构层次（P0-P5）回答"资产谁定义、谁装配、谁扩展"。两者正交、互补。

| 层级 | 名称 | 定位 | 核心对象 |
|---|---|---|---|
| P0 | 产品基线层 | 产品族顶层锚点 | ProductBaseline, ProductVariant |
| — | 方案包（项目级） | P0 的项目实例化 | SolutionInstance（= 基线 + ITP + M2 FED + 连接器） |
| P1 | ITP 层 | 装配/交付层 | ITP (IndustryTemplatePackage), OrchestrationModule |
| P2 | 模块层 | 内部构件层 | CapabilityModule |
| P3 | 资产层 | 模块的内容物 | PageTemplate, ActionModel, MetaObjectType/SchemaVersion, OrchestrationModel, RuleSet, ConnectorGroup, FeatureExtension |
| P4 | 构件层 | 资产的内部结构 | ExtensionPoint, FieldBinding, ActionBinding, TemplatePatch, DesignChangeset |
| P5 | 引擎与组件层 | 平台能力基座 | Engine, CapabilityComponent, DisplayComponent, FieldType, ServerUnit, ClientUnit |

**P 轴与 M 轴交叉视图：**

- M0 平台内核（年级变更）：P5 引擎、核心对象定义
- M0.5 公共业务（半年级）：P3 WorkPackage/Task 等公共对象
- M1 领域 ITP（季度级）：P2 模块 + P1 ITP + P3 领域资产
- M2 客户差异（月级）：P3 FED + P4 编辑态扩展

**关键约束：** ITP 是 P1-P3 的打包产物。M2 编辑态操作 P3-P4 层，由 P5 引擎兜底。用户在一个页面上操作时，P0-P5 多层同时生效，不是逐层深入。

## 4.5 引擎 + Schema 混合 UI 架构（AD-24）

关联 ADR：`ADR-024`

平台 UI 采用三层分离：

| 层级 | 实现方式 | 说明 |
|---|---|---|
| 引擎层 | 硬编码 | 复杂交互引擎（graph-canvas、form-engine、table-engine 等） |
| 能力组件层 | 硬编码 + 配置接口 | 通用业务组件（io-configurator、relation-browser 等） |
| 页面组合层 | 声明式 UI Schema | 由引擎和组件组合而成的页面定义，编辑态可原地修改 |

运行态和编辑态共享同一份 UI Schema，由双模渲染引擎切换。编辑态通过 ExtensionPoint 控制可编辑边界，变更通过 DesignChangeset 暂存并经影响分析后发布。

扩展点（ExtensionPoint）采用 **Convention over Configuration** 方法论：P5 层引擎和组件自动提供标准扩展点模式（table-engine 自动有 columns/filters/actions，ActionModel 自动有 pre/validate/compute/post 钩子），M1 领域开发者通过 `extensionExposure` 白名单声明决定哪些暴露给 M2 客户。编辑态中 M2 只能看到已暴露的扩展点（⚙️ 和 ➕），未暴露的无编辑入口。

## 4.6 Extension Unit 与扩展机制统一

关联 ADR：`ADR-024`、`ADR-010`

原 ITP `behavior/plugins/` 概念废弃，代码级扩展统一为 Extension Unit：

- **ServerUnit**：后端扩展单元——覆盖自定义算法、多对象复合操作、事件编排、流程触发、外部系统集成。四级实现（L1 表达式 → L2 沙箱脚本 → L3a 进程内插件 → L3b 独立子进程），接口与实现分离。受现实约束（无容器、离线部署、信创兼容），不使用 K8s/Docker，改为 JVM ClassLoader 隔离和进程级隔离。高频场景支持进程内直接调用，零网络开销。
- **ClientUnit**：前端扩展单元——覆盖自定义展示组件、交互行为、编辑器。通过 `platform.execute()` 声明式调用 ServerUnit。

**扩展机制统一模型：**

| 概念 | 职责 | 一句话 |
|---|---|---|
| ExtensionPoint | WHERE — 哪里可以扩展 | 插座（UI 层 + 后端层） |
| FED | WHAT — 扩展了什么 | 扩展记录（零代码 + 代码级引用 + orchestration 编排） |
| Extension Unit | HOW — 用什么实现 | 电器（ServerUnit + ClientUnit） |
| ConnectorGroup | INTEGRATE — 跟外部怎么连 | 集成声明 |

**运行时约束（v0.6 新增）：**
- 无容器：Level 3 使用 JVM ClassLoader 隔离（L3a 进程内插件）和独立子进程（L3b）替代 K8s 容器
- 离线部署：ClientUnit Bundle 从本地制品仓库加载，不依赖外网 CDN
- 进程内调用：高频 ServerUnit 支持嵌入领域服务 JVM 内直接调用
- 分布式事务：同服务内用本地事务，跨服务用 Saga（Task Orchestration 协调），由 touchedObjects 声明自动选择
- 信创兼容：基于毕昇/Kona JDK + Spring Boot + Vue 3，全栈纯 Java + 标准 Web 技术，支持鲲鹏/飞腾/龙芯 CPU 和国产 DB/中间件/浏览器

**沙箱脚本引擎选型（待冻结）：**

L2 沙箱脚本层的引擎选型当前为多方案并存状态，需通过 POC 验证后收敛为单一推荐方案：

| 候选方案 | 优势 | 风险 | 信创兼容 |
|---|---|---|---|
| Nashorn（JDK 内置，JDK 15 后移除） | 零外部依赖、JVM 原生 | JDK 15+ 需额外引入、ECMAScript 5 限制 | 毕昇/Kona JDK 可用 |
| GraalVM CE JavaScript | 高性能、ECMAScript 最新标准 | 信创 CPU（鲲鹏/飞腾/龙芯）支持需验证 | **待 POC 验证** |
| Jython | Python 生态、工程师熟悉 | 性能偏低、Python 3 支持不完整 | 纯 JVM，信创可用 |

收敛时间节点：Phase 0 标杆域 POC 阶段完成选型冻结。

FED 支持 `orchestration` 声明，描述多个 Extension Unit 的链式编排关系，由 Task Orchestration Service 解释执行。支持 `unit`（自动步骤）和 `approval`（人工审批步骤）交替、`parallel` 并行、`loopGuard` 防循环。

**编排职责边界澄清：**

| 编排层 | 职责范围 | 不负责什么 |
|---|---|---|
| Task Orchestration | 业务活动编排、FED orchestration 声明的链式步骤调度 | 不执行工具运行时，遇到工具步骤时委派 Toolflow Orchestration |
| Toolflow Orchestration | 工程工具步骤编排、运行依赖管理、工具版本绑定、回调收敛 | 不管业务活动状态和审批流程 |
| Workflow Service | 人工审批子流程 | 不管业务活动编排和工具执行 |

当 FED orchestration 中的某个 `unit` 步骤实质是工具调用（如 CAE 求解、数据采集脚本）时，Task Orchestration 负责发起委派，Toolflow Orchestration 负责执行和回调，Task Orchestration 根据回调结果推进后续步骤。两者通过明确的委派-回调接口协作，不存在职责交叉。

## 4.7 平台⇄产品⇄项目双向流动

自顶向下（逐级支撑）：
- 平台提供引擎 + 组件 + 扩展点机制 → 产品团队构建标准 ITP → 客户在编辑态做差异化扩展
- 每一层的承诺是下一层安全操作的前提

自底向上（逐级积累）：
- 客户编辑态产出 FED → 多项目相似 FED 被领域团队提炼为标准 ITP → 多领域相似模式被平台团队抽象为通用引擎/组件
- 晋升条件：M2→M1 需 ≥2 项目验证 + 领域评审；M1→M0 需 ≥2 领域复现 + 平台接管维护
- 代码级扩展（Extension Unit）与零代码扩展走同一条晋升路径

**飞轮效应：** 每一个客户项目的 M2 定制，都在为平台未来能力做增量积累。项目越多 → FED 积累越丰富 → ITP 越完整 → 新客户交付越快 → 更多项目 → 更多积累。

详细对象定义见统一对象模型 §13.5。

---

## 5. 服务蓝图

关联 ADR：`ADR-002`、`ADR-006`、`ADR-008`、`ADR-011`、`ADR-017`、`ADR-018`、`ADR-020`、`ADR-021`

## 5.1 服务优先级与波次重排

`v0.5` 不再使用区分度过低的 `P0/P1` 作为唯一排序方式，改为 `MVP Core / Wave 1 / Wave 2 / Deferred`。

| 层次 | 服务 | 说明 |
| --- | --- | --- |
| MVP Core | Identity、Master Data、Data Source Management、Object Model Registry、File/Object、Workflow、Task Orchestration、ACL Policy、Classification & Secrecy Policy、Crypto Policy、Meta-model Extension、Rule、Trace、Graph Service、Audit、Search、Integration Gateway、Config & Release、Industry Template Package、Tenant Bootstrap | 第一批必须具备的共享控制面和复制引擎最小闭环。Graph Service 在 MVP 阶段提供统一查询接口（模型依赖分析、正向追踪、反向溯源），由 Trace Service 负责写入，Graph Service 负责读取与遍历 |
| Wave 1 | Requirement、Architecture、Function/Logical/Physical Architecture、Interface Definition、Project、WorkPackage、Task Collaboration、Review & Change、Test Planning、Test Execution、DataSet、Analysis、Site & Device、Research Project、Contract、Approval、Model Automation | 第一轮业务闭环和纵切 A/B 主体 |
| Wave 2 | Simulation Task、HPC Job Proxy、Result Package、Workbench Session、Toolflow Orchestration、CAD/CAE Runtime、Storage Tiering、TODS Model Registry、ATF/XML Exchange、Knowledge、Semantic Retrieval、Worker Executor | 工业特性深化与 HPC/知识增强 |
| Wave 2+ | Extension Unit Lifecycle（ServerUnit 注册/沙箱/进程管理、ClientUnit 注册/加载）、FED 可视化编辑工作台 | 编辑态与代码级扩展的完整运行时基础设施。**前期过渡方案：** 在 Wave 2+ 交付前，扩展治理通过以下最小门禁实现——(1) FED 注册校验由 Config & Release Service 承载；(2) Extension Unit 以白名单方式手工注册，受 Meta-model Extension Service 管控；(3) DesignChangeset 的 Draft→影响分析→发布流程由 Config & Release Service 的最小发布管线支撑。Wave 2+ 交付后切换为完整的自动化运行时基础设施 |
| Deferred | AI Assistant 高级能力、Advanced Dashboard、生态开放组件 | 不阻塞主线闭环 |

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

1. `MVP Core`：先冻结共享控制面（含 Graph Service）和复制引擎最小闭环
2. `Wave 1`：再完成 `Requirement + Architecture + Test/DataSet + Project/Contract` 两条业务主线
3. `Wave 2`：补强 `HPC / Workbench / Knowledge / AI Worker`
4. `Wave 2+`：交付 Extension Unit Lifecycle 和 FED 可视化编辑工作台的完整运行时基础设施（前期通过 Config & Release 最小门禁过渡，详见 §5.1）
5. `Deferred`：生态开放和高级智能

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
- Graph Service（提供主线正向追踪、反向溯源和基础影响分析查询，由 Trace Service 写入）
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
- `SolutionInstance`
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
- Graph Service（提供模型依赖图谱查询和裁剪影响分析，由 Meta-model Extension / ITP Loader 负责投影生成）

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
| `§4.4-§4.7` | `ADR-010`、`ADR-017`、`ADR-024` |
| `§12-§13` | `ADR-006`、`ADR-010`、`ADR-014`、`ADR-017`、`ADR-020`、`ADR-021`、`ADR-023`、`ADR-024` |

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
- `§4.6 Extension Unit 运行时`：已有运行时约束设计，后续需补充信创环境的详细 POC 验证报告

---

## 15. 当前版本结论

本文档为 `v0.6 Draft`，在 `v0.5` 冻结基线上从”运行时架构”扩展到了”产品架构”，形成双维架构体系。两个版本的范围关系详见文档开头”版本范围说明”。

**v0.5 已冻结的（运行时视角，对应 M-01 至 M-09）：**
- 质量属性从目标口号变成了带指标的场景
- 横切平面从概念描述变成了运行时契约
- `TraceLink`、多存储和安全控制有了明确写入与执行责任
- 图能力统一到 `Graph Service` 逻辑层，Graph Service 正式纳入 MVP Core 波次
- 优先级、批次、纵切和治理规则形成了同一口径
- 非功能视图和证据链从缺席状态提升到最小可复核基线

**v0.6 新增的（产品与扩展视角，ADR-024 + 待补充 ADR）：**
- 产品架构层次（P0-P5）与运行时逻辑分层（L1-L7）正交互补，解决了”资产谁定义、谁装配、谁扩展”
- 引擎+Schema 混合 UI 架构（AD-24）支持编辑态，实现”运行即配置”的所见即所得体验
- Extension Unit（ServerUnit/ClientUnit）替代原 plugins 概念，代码级扩展纳入 FED 统一管理体系
- ExtensionPoint 从 UI 层扩展到后端层，统一”插座”机制覆盖前后端
- FED 同时管理零代码扩展和代码级引用，支持 orchestration 链式编排
- 平台⇄产品⇄项目双向流动模型和飞轮效应，保证项目积累反哺产品和平台
- FED 防错四道防线（编写→提交→注册→发布）和可视化编辑工作台

**v0.6 新增内容的待完成项：**
- 沙箱脚本引擎选型需通过 Phase 0 POC 收敛为单一推荐方案
- P0-P5 各层对象需补充独立验收标准和责任人
- FED 可视化编辑工作台的 UX 细化
- 扩展治理在 Wave 2+ 前的过渡门禁方案需在 Phase 0 实践中验证
- 更精细的容量数字和正式合规逐条映射
