# 工业研发底座技术架构蓝图改造建议

状态：`Review Draft v0.1`  
日期：`2026-03-10`  
用途：基于 `docs/reference/INDEX.md` 新增的资产索引，对既有《工业研发底座技术架构蓝图》进行一次增量复盘，提出应进入 `v0.4` 的结构性改造建议。

---

## 1. 本轮再分析的新增价值

相较 `2026-03-08` 的前置研究，本轮最重要的变化不是“发现了更多文档”，而是：

1. `docs/reference/INDEX.md` 已经把公司资产从散点材料升级为可导航的资产图谱。
2. 自有资产边界被更清晰地证明为 `协同研发 + TDM/ODS + 管理信息化/SUN + 竞品映射 + 标准体系` 的组合，而不是单一 `CDM` 平台外扩。
3. `TODS/ASAM ODS`、`主数据管理系统`、`SUN.BPM`、`易立德 MBSE/数字主线` 在蓝图中的映射关系可以更精确，不必继续停留在“方向正确、语义略粗”的状态。

因此，这次建议不是推翻原蓝图，而是把蓝图从：

- `共享能力蓝图`

升级为：

- `工业研发语义底座 + 数字主线引擎 + 产品化复制工厂` 的组合蓝图

---

## 2. 对当前蓝图的总体判断

现有蓝图已经完成了三件正确的事：

1. 明确了 `产品族共用平台内核` 而不是单一大产品。
2. 明确了 `统一对象模型 + TraceLink + 多模存储 + 边缘协同 + 复制引擎` 的主方向。
3. 把 `ITP / SchemaVersion / 自动联动 / 四层扩展治理` 提前纳入了架构骨架。

但结合新的资产索引，当前蓝图仍有四个结构性缺口：

1. **MBSE / 架构对象链缺位**
   - 当前对象模型重点覆盖 `Requirement / Project / WorkPackage / Task / Test / DataSet`，但还没有把 `功能架构 / 逻辑架构 / 物理架构 / 接口架构` 提升为一级对象与一级服务。
2. **TDM 标准语义还未升格为“标准兼容运行时”**
   - 当前蓝图提到了 `MeasurementSchema`，但还没有把 `ODS/TODS 基础模型 + 应用模型 + 标准接口 + 交换格式` 变成底座级兼容能力。
3. **工程工作台与工具流运行时表达偏弱**
   - 现有蓝图更偏 Web 门户和微服务治理，尚未充分体现自有资产中的 `B/S + B/C/S + Workbench + 工具流 + CAD/CAE/仿真` 体系。
4. **管理信息化仍偏“并列域”，尚未完全成为统一主线的治理锚点**
   - 资产索引已说明 `主数据 / 合同 / 科研计划 / 科研项目 / 设备台账` 不是外围系统，而是治理和复制交付的重要入口。

---

## 3. 对蓝图的核心改造结论

建议保留现有 `七层逻辑分层`，但在 `v0.4` 中显式增加三条横切平面：

1. `模型与标准语义平面`
   - 承载 `统一对象模型 + MBSE 架构对象 + TODS/ODS 应用模型 + 主数据标准 + 量纲单位体系`
2. `数字主线平面`
   - 承载 `Requirement -> Architecture -> Design/Simulation -> Test -> Data -> Knowledge -> MIS`
3. `复制交付平面`
   - 承载 `ITP + SchemaVersion + Tenant Bootstrap + 模板装载 + 发布后自动联动`

也就是说，原蓝图不是要改成 `8 层`，而是要从“纵向分层”升级成“纵向七层 + 横向三平面”的架构表达。

---

## 4. 对现有蓝图章节的定向改造建议

## 4.1 修改 `§2 架构驱动因素`

建议补充三条驱动因素：

1. **标准驱动**
   - `TODS/ASAM ODS` 不是参考资料，而是试验域语义底座和交换标准来源。
2. **架构驱动研发**
   - 竞品与自有协同研发方案都表明，工业研发底座不能只停在 `需求 -> 任务 -> 数据`，必须补上 `需求 -> 架构 -> 专业设计/仿真 -> 试验`。
3. **产品化复制驱动**
   - `SUN + 主数据 + 行业模板 + 交付初始化` 说明底座不只是技术平台，还要承担“复制工厂”的角色。

建议把 `§2.1 业务驱动` 中的主链路改写为：

`需求 -> 架构 -> 项目/工作包 -> 设计/仿真/试验 -> 数据 -> 知识 -> 管理闭环`

---

## 4.2 修改 `§3.1 推荐总体形态`

建议把当前推荐形态进一步收敛为：

`1 个平台内核 + 3 个一级业务域 + 1 个复制交付工厂 + N 个工程工具/连接器`

对应解释：

- `1 个平台内核`
  - 身份、主数据、对象模型、TraceLink、文件、流程、任务、规则、搜索、审计、发布治理
- `3 个一级业务域`
  - `研发协同`
  - `试验数据`
  - `管理信息化`
- `1 个复制交付工厂`
  - `ITP + SchemaVersion + Tenant Bootstrap + 模型联动`
- `N 个工程工具/连接器`
  - CAD/CAE/CAM/HPC/OT 设备/外部业务系统/Workbench

这样比“共享平台 + 领域服务”更贴近公司真实资产结构。

---

## 4.3 修改 `§4 逻辑架构蓝图`

建议在 `§4.1 七层逻辑分层` 后新增一节：

### 4.x 三条横切平面

#### A. 模型与标准语义平面

贯穿 `L3/L4/L5/L6`，包括：

- `MetaObjectType / SchemaVersion`
- `ReferenceData`
- `MeasurementQuantity / PhysicalUnit / Environment`
- `TODS Application Model Registry`
- `MBSE Architecture Meta Model`

#### B. 数字主线平面

贯穿 `L2/L3/L4/L5`，包括：

- `TraceLink`
- `Requirement-Architecture`
- `Architecture-Design`
- `Design/Simulation-Test`
- `Data-Knowledge-MIS`

#### C. 复制交付平面

贯穿 `L1/L4/L7`，包括：

- `Industry Template Package`
- `Tenant Bootstrap`
- `模板变量注入`
- `环境初始化`
- `发布后自动联动`

这会显著提升蓝图表达力，避免后续团队只盯“层”，忽略“跨层主线”。

---

## 4.4 修改 `§5 服务蓝图`

当前服务划分总体正确，但建议新增四组关键服务。

### A. 新增 `MBSE / 架构服务组`

建议新增：

- `Architecture Service`
- `Function Architecture Service`
- `Logical Architecture Service`
- `Physical Architecture Service`
- `Interface Definition Service`
- `Architecture Verification Service`

补充原因：

- 资产索引和竞品材料已明确“架构驱动研发”是行业必答题。
- 当前蓝图若没有这一组服务，`TraceLink` 会缺失从需求到设计之间最关键的承上启下层。

### B. 新增 `标准语义与测量模型服务组`

建议新增：

- `Reference Data & Unit Service`
- `Measurement Semantic Service`
- `TODS Model Registry Service`
- `ATF/XML Exchange Service`

补充原因：

- `MeasurementSchema` 解决的是“业务建模”问题；
- `ODS/TODS` 解决的是“标准兼容、语义互通、交换接口”问题；
- 两者不能混为一个对象编辑器。

### C. 强化 `工程工作台与工具流服务组`

建议在 `§5.4 仿真与算力协同服务` 下补充：

- `Engineering Workbench Session Service`
- `Toolflow Orchestration Service`
- `CAD/CAE Integration Runtime`
- `Result Visualization / Lightweight Model Service`

补充原因：

- 自有技术方案明确存在 `B/S + B/C/S + Workbench + 工具流 + 模型轻量化` 能力。
- 这部分如果不写进蓝图，后续实现会偏向通用 Web 平台，而不是工业研发工作平台。

### D. 强化 `管理治理服务组`

建议把 `§5.5 管理信息化领域服务` 从“业务域”进一步提升为“治理锚点”，新增：

- `Research Governance Service`
- `Contract-to-Project Mapping Service`
- `Equipment Master Index Service`
- `Master Data Distribution Service`

补充原因：

- 这些服务决定底座能否把 `计划/合同/设备/组织` 同步反哺到研发与试验链条。

---

## 4.5 修改 `§6 数据架构蓝图`

## 4.5.1 改造 `§6.1 数据分层`

建议把现有五层数据分层细化为：

1. `主数据与标准语义层`
   - 组织、人员、角色、字典、编码、量纲、单位、测量量、环境、分类
2. `核心业务对象层`
   - 需求、架构、项目、工作包、任务、试验、合同、设备
3. `领域应用模型层`
   - `TODS Application Model`
   - `MBSE Architecture Model`
   - 各行业柔性对象模型
4. `数据资产层`
   - 文件、模型、测量数据、结果包、报告、交付物
5. `关系与主线层`
   - `TraceLink`、影响分析、验证关系、BOM/架构/试验关联
6. `分析与检索层`
   - 搜索、知识索引、统计快照、专题视图

核心变化是把“领域应用模型层”单独抽出来，避免 `SchemaVersion` 只被理解成配置元数据。

## 4.5.2 改造 `§6.4 主数据与对象注册`

建议在现有对象注册中心下新增两个注册子中心：

- `Architecture Meta Model Registry`
- `TODS Application Model Registry`

说明：

- 前者管理 `功能/逻辑/物理/接口` 架构对象族；
- 后者管理 `ODS/TODS` 派生应用模型、量纲单位映射、接口元信息。

## 4.5.3 改造 `§6.5 元模型驱动扩展架构`

建议把“元模型扩展”的白名单对象扩大并分组：

- `研发协同柔性对象`
  - `DeliverableType`
  - `ReviewTemplate`
- `试验柔性对象`
  - `TestSpec`
  - `MeasurementSchema`
  - `TODS Application Model`
- `管理柔性对象`
  - `ResearchForm`
  - `ContractSubtype`

同时增加一条红线：

- `MeasurementQuantity / PhysicalUnit / Environment / ArchitectureDefinition` 不应由场景装配层直接改写，只能走模型审批流。

---

## 4.6 修改 `§7 集成架构蓝图`

## 4.6.1 扩展 `§7.1 四类集成面`

建议从“四类集成面”升级为“五类集成面”：

1. `API`
2. `Event`
3. `File`
4. `Agent / Protocol`
5. `Standard Exchange`

其中 `Standard Exchange` 专指：

- `TODS API`
- `ATF/XML`
- 测量语义交换
- 标准化结果包交换

## 4.6.2 强化 `§7.3 集成策略`

建议显式写入：

- `主数据分发` 是平台级发布订阅能力，不是各系统点对点同步脚本。
- `架构对象`、`试验模型`、`设备主索引` 三类对象要有独立连接器策略。
- `CAD/CAE/HPC` 不是普通 IT 连接器，应被归入工程运行时连接器。

## 4.6.3 强化 `§7.4 CAE/SPDM/HPC 集成专题`

当前章节应再补两点：

1. `Engineering Workbench` 的会话、凭据、任务回传、结果回挂机制。
2. `大模型/轻量化模型/结果包` 与 `对象存储 + 搜索 + TraceLink` 的回写机制。

---

## 4.7 修改 `§8/§9 部署与边缘架构`

## 4.7.1 修改 `§8.1 部署分区`

建议增加两个部署角色：

- `Engineering Client Zone`
  - 面向 B/C/S 工作台、工具插件、模型浏览器
- `Standard Exchange Zone`
  - 面向 `TODS API / ATF/XML / 跨网数据交换`

## 4.7.2 修改 `§9 TDM 边缘架构蓝图`

建议把边缘侧组件从“采集代理”扩展为：

- `采集代理`
- `本地语义映射代理`
- `边缘缓存与断点续传代理`
- `标准交换代理`

说明：

- 边缘不只是传文件，还需要承担 `本地测量模型 -> 平台统一语义` 的预处理职责。

---

## 4.8 修改 `§10 安全与治理蓝图`

建议补充三类治理机制：

1. `模型治理委员会`
   - 负责核心对象、领域模型、SchemaVersion、兼容性评审
2. `标准语义委员会`
   - 负责 `主数据 + 量纲单位 + TODS/ODS 兼容策略`
3. `产品复制治理小组`
   - 负责 `ITP`、初始化流程、差量升级和模板资产准入

这三者分别对应：

- 架构治理
- 数据标准治理
- 产品化治理

---

## 5. 对统一对象模型的直接改造建议

建议在《统一对象模型》中补充以下对象族。

## 5.1 新增 MBSE / 架构对象族

- `ArchitectureDefinition`
- `FunctionArchitecture`
- `LogicalArchitecture`
- `PhysicalArchitecture`
- `InterfaceSpec`
- `ArchitectureBaseline`
- `VerificationCase`

建议进入：

- `§6.1 研发协同域对象` 或单独新增 `§6.x 架构与系统工程域对象`

## 5.2 新增 TDM 标准语义对象族

- `MeasurementQuantity`
- `PhysicalUnit`
- `Environment`
- `TestArticle`
- `TODSApplicationModel`
- `ExchangePackage`

建议进入：

- `§5.2 主数据与字典对象`
- `§6.2 试验数据域对象`
- `§7 平台执行与治理对象`

## 5.3 扩展主线关系

建议把当前 `§8 统一对象关系主线` 从六条主线升级为八条：

1. `需求主线`
2. `架构主线`
3. `研发执行主线`
4. `试验数据主线`
5. `设备资源主线`
6. `经营治理主线`
7. `知识反哺主线`
8. `复制交付主线`

其中最关键的新链路是：

`Requirement -> ArchitectureDefinition -> WorkPackage/Task -> Simulation/Test -> DataSet -> KnowledgeAsset`

---

## 6. 对 ADR 清单的增补建议

建议在现有 ADR 基础上新增三条 `P0/P1` 决策。

## ADR-020 MBSE / 架构对象建模策略

推荐结论：

- 将 `ArchitectureDefinition / FunctionArchitecture / LogicalArchitecture / PhysicalArchitecture / InterfaceSpec` 作为一级对象与一级服务；
- 不把架构对象隐含塞入 `Deliverable` 或 `KnowledgeAsset`。

## ADR-021 TDM 标准语义兼容策略

推荐结论：

- 以 `MeasurementSchema` 承载业务侧采集定义；
- 以 `TODS/ODS Application Model Registry` 承载标准侧语义与交换兼容；
- 两者通过映射关系关联，而不是互相替代。

## ADR-022 工程工作台与 B/C/S 通道策略

推荐结论：

- 平台默认同时支持 `Web Portal` 与 `Engineering Workbench` 两类入口；
- CAD/CAE/HPC/工具流相关场景不强制退化为纯浏览器模式；
- 会话、凭据、日志、任务、审计统一纳入平台治理。

---

## 7. 建议的实施优先级调整

建议把原实施节奏从“复制引擎优先”调整为“两条纵切并行”：

### 纵切 A：语义主线 PoC

目标：

- 打通 `需求 -> 架构 -> 试验 -> 数据 -> TraceLink`

最小范围：

- `Requirement`
- `ArchitectureDefinition`
- `TestSpec`
- `MeasurementSchema`
- `DataSet`
- `TraceLink`

### 纵切 B：复制交付 PoC

目标：

- 打通 `ITP + SchemaVersion + Tenant Bootstrap + 模板初始化`

最小范围：

- `主数据初始化`
- `柔性对象模型发布`
- `基础视图/权限/审计自动联动`

理由：

- 只做复制引擎，平台会缺“工业语义”；
- 只做工业语义，平台会缺“复制能力”；
- 两条纵切并行，才能同时证明产品价值与交付价值。

---

## 8. 最终建议

这次基于资产索引的再分析，最核心的结论是：

**当前蓝图方向不需要推翻，但必须从“共享服务架构”进一步升级为“语义驱动的工业研发底座”。**

具体来说，`v0.4` 最少应完成以下升级：

1. 把 `MBSE / 架构对象链` 正式写入蓝图、对象模型和 ADR。
2. 把 `ODS/TODS` 从参考标准升级为 `TDM 标准语义兼容运行时`。
3. 把 `Workbench / Toolflow / B/C/S` 从技术细节升级为正式架构通道。
4. 把 `管理信息化 + 主数据` 从并列域升级为统一治理锚点。
5. 把 `七层分层` 升级为 `七层 + 三平面` 的表达方式。

如果完成以上五项，工业研发底座的蓝图会明显从“正确的架构草案”进入“有行业壁垒的产品蓝图”。

---

## 9. 本轮输入依据

- `docs/reference/INDEX.md`
- `docs/reference/_parsed/3-协同研发/技术方案.md`
- `docs/reference/_parsed/3-协同研发/协同研发平台产品白皮书-2024.md`
- `docs/reference/_parsed/2-管理信息化/SUN(3.0)企业应用统一开发平台-20240417.md`
- `docs/reference/_parsed/1-TDM/ods&tods/国家标准-试验测试开放数据服务-意见.md`
- `docs/reference/_parsed/00-竞争对手/2-数据-华为/工业数据模型驱动引擎DME简介.md`
- `docs/reference/_parsed/00-竞争对手/2-数据-易立德/20230720_全生命周期数字主线平台能力介绍_V1.6_交流.md`
- `_bmad-output/planning-artifacts/architecture/industrial-base-technical-architecture-blueprint-2026-03-08.md`
- `_bmad-output/planning-artifacts/architecture/industrial-base-unified-capability-map-2026-03-08.md`
- `_bmad-output/planning-artifacts/architecture/industrial-base-unified-object-model-2026-03-08.md`
- `_bmad-output/planning-artifacts/architecture/industrial-base-adr-decision-list-2026-03-08.md`

