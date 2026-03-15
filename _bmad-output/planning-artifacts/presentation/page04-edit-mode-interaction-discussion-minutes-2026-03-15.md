# Page 04 编辑态交互范式讨论纪要

- 日期: 2026-03-15
- 讨论方式: Party Mode 多角色收敛
- 参与角色: BMad Master, Architect (Winston), UX Designer (Sally), PM (John), Developer Agent (Amelia)
- 相关原型: `presentation.pen` → Page 04
- 讨论主题: 编辑态交互范式、引擎+Schema 混合架构、Feature Extension Descriptor、NocoBase 式原地编辑

## 一、讨论背景

围绕 Page 04（模块页面与动作建模页），讨论聚焦在以下问题：

1. 界面搭建应支持**基于领域产品功能扩展**和**从0设计**两种模式，二者如何统一。
2. 面对工作包中心（工作包分解、输入/输出、使能、控制等复杂功能）这类复杂模块，如何支持横切特性（如"绩效"）的定制化扩展。
3. 如何实现**所见即所得**的编辑形态来扩展功能。
4. 扩展功能如何进一步复用。

## 二、已达成共识

### 1. 引擎 + Schema 混合架构（核心决策）

平台 UI 架构采用三层分离：

| 层级 | 实现方式 | 负责方 | 示例 |
|---|---|---|---|
| **引擎层** | 硬编码 | 平台开发团队 | graph-canvas（图形画布）、form-engine（表单）、table-engine（表格） |
| **能力组件层** | 硬编码 + 配置接口 | 平台开发团队 | io-configurator（输入输出配置器）、relation-browser（关系浏览器）、dynamic-form（动态表单） |
| **页面组合层** | 声明式 UI Schema | M1 领域团队 / 编辑态可调 | 哪些组件组合成页面、字段显隐、布局、扩展点 |

**关键原则：**

- 引擎保证复杂交互品质（拖拽、连线、布局算法等），不走 Schema
- Schema 保证灵活扩展（字段增删、组件挂载、菜单配置等），编辑态可原地修改
- 两种模式（领域扩展 / 从0设计）的产出物必须**同构**——进入 Object Model Registry 的 PageTemplate 遵循同一元模型

**类比：** 如同 Excel——电子表格渲染引擎是硬编码的，但"有哪些列、什么格式、什么校验规则"是配置驱动的。

### 2. NocoBase 式原地编辑（交互范式）

**否决 Feature Wizard 向导式设计，改为 NocoBase 式"运行即配置"范式：**

- 用户看到的就是最终运行界面
- 通过"设计模式"开关切换编辑态
- 编辑态下每个可配置区域出现视觉装饰器（虚线边框、⚙️ 齿轮、➕ 添加按钮）
- 关闭开关即回到运行态，修改即时生效（在 Draft 暂存区中）

**与 NocoBase 的关键差异：**

- NocoBase 处理的是表单+列表+看板等规整数据展示
- 工业研发底座包含 graph-canvas（图形化分解树）、io-configurator（输入输出）、relation-browser（使能/控制关系）等复杂交互
- 因此采用**分层编辑策略**：

| 编辑层级 | 交互方式 | 覆盖范围 |
|---|---|---|
| **属性层** | NocoBase 式直接编辑 | 字段增删、显隐、校验规则、展示格式 |
| **组件层** | 区块级拖拽配置 | 面板增删、布局调整、组件替换 |
| **行为层** | 可视化连线/规则面板 | 数据传递、联动触发、计算规则 |
| **结构层** | 模块级编排 | 新增页面组、调整导航结构 |

前两层完全原地编辑，后两层使用辅助面板但不离开当前工作区。

### 3. Feature Extension Descriptor 角色转变

**FED 从"输入"变为"产出"：**

```text
之前的思路：
  用户先写/生成 FED → 系统根据 FED 修改 UI Schema

现在的思路：
  用户在编辑态操作 → UI Schema 变更 → 系统自动 diff →
  生成 FED（作为变更记录和复用单元） → 注册到 Registry
```

- 用户全程不需要写 YAML，所有操作都是可视化的
- FED 是后端架构资产，不是用户操作的界面
- FED 仍然是复用和分发的载体（可从 M2 晋升到 M1）

### 4. PageTemplate 演进为完整 UI Schema

PageTemplate 的结构从"模板ID + 参数引用"演进为**完整的声明式 UI Schema**：

```yaml
PageTemplate:
  meta:
    name: work-package-decomposition
    module: work-package-center
    schemaVersion: 2.1

  uiSchema:
    root:
      type: page
      children:
        - key: main-canvas
          type: graph-canvas          # 引用平台预置引擎
          engine: tree-decomposition
          config:
            layout: top-down
            allowDragDrop: true
          nodeTemplate:
            type: card
            slots:
              header: { fields: [name, code] }
              body: { fields: [status, assignee, startDate, endDate] }
              badges: { items: [statusBadge] }
          contextMenus:
            node:
              - { action: addChild, label: "添加子工作包" }
              - { action: configureIO, label: "输入/输出设置" }
        - key: side-panel
          type: tab-panel
          tabs:
            - { key: node-settings, type: dynamic-form }
            - { key: io-settings, type: io-configurator }
            - { key: enablement, type: relation-browser }

  extensionPoints:
    - slot: node-card.body.fields
      type: field-list
      extensible: true
    - slot: side-panel.tabs
      type: tab-list
      extensible: true
    - slot: toolbar.actions
      type: action-list
      extensible: true
```

**运行态和编辑态共享同一份 UI Schema**，区别仅在于渲染引擎是否激活"配置装饰器"。

### 5. ExtensionPoints 受控边界

不是所有地方都能随意编辑，模块设计者通过 `extensionPoints` 预埋可扩展点：

- M1 层定义哪些 slot 可扩展
- M2 层客户管理员只能在预埋的 extensionPoints 中操作
- 用户在编辑态看到 ➕ 按钮的地方，就是可扩展的地方
- 不可扩展的区域没有编辑入口，用户不会困惑

**这是相对于 NocoBase 的差异化竞争力：** 不是"无限灵活"，而是"在安全边界内的灵活"。

### 6. 变更治理：Draft → 影响分析 → 发布

编辑态的所有变更不直接生效于运行态：

```text
编辑态操作 → Draft 暂存区 → 影响分析预览 → 发布确认 → 正式生效
```

类似 Git 的 staging 概念。解决工业场景下"不能随便改生产系统"的治理要求。

### 7. 编辑态权限分级

| 角色 | 编辑权限 | 场景 |
|---|---|---|
| 平台开发者（M0/M0.5） | 全量编辑 | 平台迭代 |
| 领域实施顾问（M1） | 模块级编辑 | ITP 构建 |
| 客户管理员（M2） | 受控扩展（extensionPoints 约束） | 字段增删、显隐、规则调整 |
| 业务用户 | 个人视图定制 | 列显隐、排序偏好 |

### 8. 双模渲染引擎

前端需构建双模渲染引擎：

```text
UISchemaRenderer
  ├── mode: "runtime"  → 标准渲染，无配置装饰器
  ├── mode: "design"   → 渲染 + 配置装饰器
  │     ├── FieldDecorator    → 字段级：齿轮/拖拽/删除
  │     ├── BlockDecorator    → 区块级：拖拽/替换/配置
  │     ├── SlotDecorator     → 扩展点：[+] 按钮 + 候选列表
  │     └── CanvasDecorator   → 画布级：节点模板编辑
  └── mode: "preview"  → 渲染 + 只读差异高亮
```

### 9. 复用体系

FED 支持四级复用粒度（与已有复用体系对应）：

| 复用级别 | FED 复用形态 | 示例 |
|---|---|---|
| Level 1 | 单个 FED | 绩效扩展 |
| Level 2 | FED 组合 | 绩效 + 风险 + 资源（项目管理扩展包） |
| Level 3 | 模块 + FED 集合 | 工作包中心（含内置 FED） |
| Level 4 | ITP（含多模块 + FED） | CDM 协同研发全套 |

FED 可从 M2（客户级）晋升到 M1（领域 ITP），符合 AD-3"标准 ITP 从项目中提炼"策略。

### 10. ComponentRegistry 组件注册表

```text
ComponentRegistry
  ├── engines/                    # 引擎级（硬编码，不可替换）
  │     ├── graph-canvas          # 图形画布
  │     ├── form-engine           # 表单引擎
  │     ├── table-engine          # 表格引擎
  │     └── kanban-engine         # 看板引擎
  ├── capability-components/      # 能力组件（硬编码，可配置）
  │     ├── io-configurator       # 输入输出配置器
  │     ├── relation-browser      # 关系浏览器
  │     ├── dynamic-form          # 动态表单
  │     ├── tree-navigator        # 树导航
  │     └── rule-editor           # 规则编辑器
  ├── display-components/         # 展示组件（Schema 可选用）
  │     ├── card / badge / tag
  │     ├── chart-bar / chart-pie
  │     └── progress-ring
  └── field-types/                # 字段类型（Schema 可选用）
        ├── string / number / date / file
        ├── domain-object-ref     # 领域对象引用
        └── computed              # 计算字段
```

引擎和能力组件通过代码发布更新，展示组件和字段类型可通过 M1/M2 层注册自定义实现。

### 11. "领域扩展"与"从0设计"的统一

```text
基于领域产品扩展：
  已有 PageTemplate (含引擎 + 完整 Schema)
  → 进入编辑态 → 修改 Schema 层
  → 引擎不变，配置改变

从0设计：
  空白 PageTemplate
  → 从 ComponentRegistry 中选择引擎和能力组件
  → 组合成页面 Schema
  → 本质上是"组装"而不是"编程"
```

产品策略："扩展优先、创建兜底"——进入 Page 04 时默认展示 ITP 已有模块，引导在此基础上扩展。

## 三、对 Page 04 原型的直接调整方案

在军检验收场景上增加编辑态交互展示。

### 1. 顶部状态条

- 左侧增加 🔧 **设计模式** Toggle 开关（大而醒目）
- 保留层级路径面包屑

### 2. 中间画布区域（重大调整）

画布主体展示编辑态下的实际交互：

- **表格区域：** 列头出现 ⚙️ 图标，选中列高亮，最右侧出现 ➕ 列
- **Tab 区域：** 每个 Tab 标签旁有 ⚙️，最右侧有 ➕ 新建 Tab
- **按钮区域：** 每个按钮旁有 ⚙️，旁边有 ➕ 新建动作
- **面板区域：** 虚线边框标记可配置区域，角落有 ⚙️

从"标注式"静态标签改为"交互式"视觉装饰器：
```text
当前:  [Panel 可配置]          ← 静态标签
改为:  ┌──── Panel ──⚙️──┐    ← 虚线边框 + 齿轮图标
       │  (业务内容)      │
       └────────── ➕ ───┘    ← 底部添加按钮
```

画布底部增加 Draft 暂存条：
`📝 3 项变更待发布 │ [查看变更清单] │ [影响分析] │ [发布]`

### 3. 右栏（重大调整）

从"说明文档"变为"上下文属性面板"：

**上半：当前选中元素的属性配置**
- 标题：当前选中 → `Table: 证据清单`
- 字段列表（可拖拽排序）
- 字段属性（类型、校验、显隐条件）
- ➕ 新建字段

**下半：变更与影响**
- 本次 Draft 变更清单
- 影响分析摘要
- 发布前置校验状态

### 4. 左栏（保持 + 微调）

- 模块资产树方向正确
- 在树底部增加 `➕ 新建特性扩展` 入口
- 每个资产节点旁增加状态标记（已修改 🟠 / 新增 🟢 / 未变更 ⚪）

## 四、建议新增架构决策

### AD-24：平台 UI 架构采用"引擎 + Schema"混合模式

- 引擎层和能力组件硬编码保证交互品质
- 通过声明式 UI Schema 暴露配置面和扩展点
- Schema 编辑态（设计模式）仅操作 Schema 层，不触及引擎层代码
- 运行态和编辑态共享同一份 UI Schema，由双模渲染引擎驱动

## 五、对已有文档的影响

| 文档 | 影响 |
|---|---|
| 统一对象模型 v0.4 → v0.5 | 需补充 `UISchema`、`ExtensionPoint`、`FeatureExtension`、`DesignChangeset`、`ComponentType` 概念 |
| 蓝图 v0.5 → v0.6 | 需补充"引擎 + Schema"混合架构章节，正式收录 AD-24 |
| ITP 包结构 | `behavior/views/` 下的内容从"视图模板引用"演进为"完整 UI Schema" |
| Page 04 原型 | 按本纪要第三节调整 |
| 12 页原型内容文档 | Page 04 章节需同步更新 |

## 六、暂未最终定项

1. Computed Fields 的计算落点——主存储写入时计算还是前端实时计算
2. MVP 阶段是否同时支持高级用户直接编辑 YAML
3. FED 从 M2 晋升到 M1 的治理流程优先级
4. 引擎库的扩展优先级排序（第二批、第三批引擎何时启动）
5. 工作包分解的 graph-canvas 引擎的技术选型（待后续技术选型阶段决定）

## 七、下一步

1. ✅ 沉淀讨论纪要（本文档）
2. 更新统一对象模型，补充 UI Schema 相关概念
3. 更新 Page 04 原型，体现编辑态交互范式
4. 正式提出 AD-24，纳入蓝图 v0.6
