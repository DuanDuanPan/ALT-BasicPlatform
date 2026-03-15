# 扩展点设计方法论讨论纪要

- 日期: 2026-03-15
- 讨论方式: Party Mode 多角色收敛
- 参与角色: Architect (Winston), PM (John), UX Designer (Sally)
- 前序讨论: Extension Unit 运行时加载与执行机制讨论纪要 (2026-03-15)
- 讨论主题: 扩展点的系统性预留策略、"P5 自动提供 → M1 选择性暴露 → M2 使用"三层方法论、原型页面合并

## 一、核心方法论

### Convention over Configuration（约定优于配置）

扩展点不靠 M1 开发者逐个手动预埋，而是由 P5 层引擎和组件**自动提供标准扩展点模式**，M1 开发者**决定哪些暴露给 M2**。

```text
P5 引擎/组件自动提供     → "标准插座模式"（潜在扩展点）
  ↓ 白名单暴露
M1 领域模块选择性暴露     → "哪些插座对外开放"（暴露策略）
  ↓ 受控使用
M2 客户在编辑态使用       → "往开放的插座上插电器"
```

### 三层职责

| 层级 | 职责 | 操作 |
|---|---|---|
| M0 平台团队 | 定义扩展点**模式** | 在引擎和组件中内置标准扩展点 |
| M1 领域开发者 | 决定扩展点**暴露** | 声明 extensionExposure，白名单制，默认关闭 |
| M2 实施顾问 | 使用扩展点**扩展** | 在编辑态看到 ⚙️ 的地方操作 |

## 二、P5 引擎/组件的标准扩展点模式

### UI 层——每种引擎/组件自动提供

```text
table-engine:
  columns / columns.{col}.render / filters /
  actions.toolbar / actions.row / actions.batch /
  pagination / empty-state

form-engine:
  fields / fields.{field}.render / sections /
  actions.submit / validation / layout

graph-canvas:
  node.content / node.badges / node.context-menu /
  edge.label / toolbar / canvas.background / minimap

tab-panel:
  tabs / tabs.{tab}.content

dynamic-form:
  fields / actions

io-configurator:
  input-types / output-types / validation

relation-browser:
  relation-types / display / actions
```

### 后端层——每种资产类型自动提供

```text
ActionModel 自动提供：
  pre / validate / compute / post / error

FlowModel 自动提供：
  event.{name}.handler / gateway.{name}.decide /
  transition.{name}.guard

DomainObject 自动提供：
  onCreate.pre/post / onUpdate.pre/post /
  onStatusChange.pre/post / onDelete.pre/post
```

## 三、M1 暴露控制声明规范

### 声明方式

```yaml
capabilityModule:
  name: work-package-center

  extensionExposure:
    defaultPolicy: closed        # 默认不暴露（白名单制）

    exposed:
      - pattern: "table-engine.*"
        expose: [columns, filters, actions.toolbar, actions.row]
        constraints:
          columns: { maxCount: 20, allowedTypes: [string, number, date, computed] }
          actions.toolbar: { maxCount: 5 }

      - pattern: "graph-canvas.*"
        expose: [node.content, node.badges, node.context-menu, toolbar]
        constraints:
          node.badges: { maxCount: 3 }

      - pattern: "tab-panel.*"
        expose: [tabs]
        constraints:
          tabs: { maxCount: 8 }

      - pattern: "action.*.validate"
        expose: all

      - pattern: "action.*.compute"
        expose: all

      - pattern: "action.*.post"
        expose: all

      - pattern: "domain.WorkPackage.onStatusChange"
        expose: [pre, post]

      - pattern: "event.*.handler"
        expose: all
```

### 约束类型

| 约束 | 说明 | 示例 |
|---|---|---|
| maxCount | 最多扩展数量 | columns 最多 20 列 |
| allowedTypes | 允许的字段/组件类型 | 只允许 string/number/date |
| exclusivity | shared / exclusive | 是否允许多个 FED 同时挂载 |
| requiresApproval | 是否需要审批 | 高风险扩展点需管理员批准 |

### 编辑态的 UX 影响

- M1 暴露的扩展点 → 编辑态出现 ⚙️ 和 ➕
- M1 未暴露的扩展点 → 编辑态无任何编辑入口
- 用户不需要知道哪些被关闭，只看到开放的

## 四、原型页面合并——5 个合并为 3 个

| 合并后 | 合并前 | 面向角色 |
|---|---|---|
| **A 扩展设计工作台** | ① 扩展点设计台 + ③ FED 编排台 | M1 领域开发者 |
| **B Extension Unit 管理台** | ② Unit 开发台 + ④ 运行监控台 | 开发者 + 运维 |
| **C 晋升管理台** | ⑤ 不变 | 领域管理员 |

### A 扩展设计工作台的设计修正

之前设计的是"手动预埋扩展点"，修正为"管理扩展点暴露策略"：

- 中间（Dominant）：模块扩展蓝图——自动显示所有潜在扩展点（P5 提供），M1 通过勾选决定暴露
- 底部：orchestration 链路编排画布
- 左栏：模块资产导航 + FED/Unit 浏览器
- 右栏：选中项的暴露策略配置 / FED 组成 / Unit 接口

### B Extension Unit 管理台

通过顶部 Tab 切换开发视图和运行视图：
- 开发视图（Dominant）：接口契约 + Drawer 展开测试/实现
- 运行视图（Dominant）：仪表盘（调用量/耗时/错误率）

### C 晋升管理台

不变。中间（Dominant）：晋升评审面板（条件检查+多项目对比）。

## 五、关键澄清

之前展示的 `node-card.body.fields` 不是"为绩效定制的扩展点"，而是：

```text
graph-canvas 引擎自动提供 → node.content 扩展点
  → M1 work-package-center 模块暴露了 node.content
  → M2 实施顾问在设计模式下看到 ⚙️
  → 实施顾问加了绩效字段
```

绩效扩展只是**使用了已有的扩展点**，不是创建了新的扩展点。

## 六、下一步

1. ✅ 沉淀讨论纪要（本文档）
2. 更新对象模型，补充 extensionExposure 声明规范
3. 更新蓝图，补充扩展点方法论
4. 在原型中实现 A/B/C 三个页面
