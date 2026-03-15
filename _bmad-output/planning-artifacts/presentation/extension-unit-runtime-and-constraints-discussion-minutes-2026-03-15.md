# Extension Unit 运行时加载与执行机制讨论纪要

- 日期: 2026-03-15
- 讨论方式: Party Mode 多角色收敛
- 参与角色: Architect (Winston), Developer Agent (Amelia), PM (John)
- 前序讨论: 扩展单元机制与全场景验证讨论纪要 (2026-03-15)
- 讨论主题: 定制代码的前后端加载与执行机制、现实约束适配

## 一、技术栈前提

- 前端：Vue 3 + Vite + Ant Design Vue
- 后端：Java + Spring Boot + Spring Cloud
- 注册/配置中心：Nacos
- 消息中间件：RocketMQ（或东方通 TongRDS）

## 二、现实约束

| # | 约束 | 影响 |
|---|---|---|
| 1 | 无法使用容器（K8s/Docker） | Level 3 用 JVM 进程级隔离替代容器 |
| 2 | 只能离线部署 | 前端 ClientUnit Bundle 使用本地制品仓库，不走 CDN |
| 3 | 分布式事务控制 | ServerUnit 跨对象操作需明确事务边界 |
| 4 | 高性能场景需进程内调用 | 高频 ServerUnit 必须支持 JVM 内直接调用，零网络开销 |
| 5 | 信创兼容（国产硬件/OS/DB/中间件/浏览器） | 技术选型需验证信创 CPU（鲲鹏/飞腾/龙芯）兼容性 |

## 三、后端 ServerUnit 运行时设计

### 四级实现（适配无容器约束）

| 级别 | 方式 | 执行环境 | 隔离方式 | 适用场景 | 信创 |
|---|---|---|---|---|---|
| L1 表达式 | Aviator 表达式 | JVM 内联 | 无需隔离 | 简单公式计算 | ✅ 纯 Java |
| L2 沙箱脚本 | Nashorn/Rhino(JS) + Jython(Python) | JVM 进程内沙箱 | SecurityManager + 白名单 API | 中等复杂度算法 | ✅ 纯 Java |
| L3a 进程内插件 | ServerUnit JAR + ClassLoader | JVM 进程内 | ClassLoader + 独立线程池 + 熔断 | 高频/低延迟 | ✅ |
| L3b 独立子进程 | 可执行 JAR / 脚本 + ProcessBuilder | 独立 JVM 进程 | 进程级隔离 + 本地 Socket | 大资源/长时间/商业软件 | ✅ |

### ServerUnit Runtime 架构组件

```text
extension-unit-runtime-service（Spring Boot 微服务）
  ├── ServerUnitRegistry         — 注册表（从 DB 查找定义，缓存热点）
  ├── ServerUnitDispatcher       — 路由分发（根据 Level 分发到执行器）
  ├── ExpressionExecutor         — L1 Aviator 表达式求值
  ├── ScriptSandboxExecutor      — L2 Nashorn/Rhino/Jython 沙箱
  ├── ServerUnitPluginManager    — L3a ClassLoader 插件管理
  │     · 独立 ClassLoader 加载 JAR
  │     · 独立线程池（每个 Unit 隔离）
  │     · 超时控制 + 熔断
  │     · 热加载（监控插件目录变化）
  ├── SubProcessExecutor         — L3b 独立子进程管理
  │     · ProcessBuilder 启动 JVM
  │     · 本地 Socket / 命名管道通信
  │     · 进程生命周期管理
  └── aspects/
        · AuthorizationAspect    — 权限校验
        · AuditAspect            — 审计记录
        · TimeoutAspect          — 超时控制
        · RateLimitAspect        — 限流限额
```

### L3a 进程内插件的 JAR 规范

```text
perf-score-unit-1.0.jar
  ├── META-INF/server-unit.yaml     — 接口契约声明
  ├── com/customer/perfunit/
  │     └── WeightedPerfScore.java  — 实现 ServerUnitHandler 接口
  └── lib/                          — 私有依赖（不与平台冲突）
```

实现类必须实现平台标准接口：

```java
public interface ServerUnitHandler {
    UnitOutput execute(UnitInput input, PlatformContext ctx);
}
```

### L3a vs L3b 自动选择规则

```text
if resources.requiresIsolation == true
   || resources.memory > 阈值
   || resources.timeout > 阈值
   || resources.externalProcess == true:
   → L3b 独立子进程
else:
   → L3a 进程内插件（零网络开销）
```

## 四、分布式事务控制

### PlatformContext 事务 API

```java
public interface PlatformContext {
    // 单对象操作——平台管理事务
    <T> T createObject(String objectType, Map<String, Object> fields);
    <T> T updateObject(String objectType, String id, Map<String, Object> fields);

    // 声明式事务边界——同服务内多操作
    <T> T withinTransaction(Supplier<T> operations);

    // 跨服务操作——发领域事件，由 Task Orchestration 编排 Saga
    void emitDomainEvent(String eventType, Map<String, Object> payload);

    // 只读查询
    <T> T readObject(String objectType, String id);
    List<?> queryObjects(String objectType, Filter filter);
}
```

### 事务策略自动选择

| 场景 | touchedObjects 特征 | 策略 |
|---|---|---|
| 单聚合根内 | 同一 object 的 create/update | 本地事务 @Transactional |
| 跨聚合根同服务 | 多个 object 但同一 service | 本地事务 + Outbox 事件 |
| 跨服务 | 多个 object 跨不同 service | Saga（Task Orchestration 协调） |

由 ServerUnit 的 `touchedObjects` 声明中的 `service` 字段驱动自动判断。

## 五、前端 ClientUnit 运行时设计

### 加载机制

- Module Federation（Webpack 5 / Vite Plugin）
- 离线模式：从本地制品仓库加载（Nginx 静态托管）
- 运行时从 ClientUnit Registry API 获取本地地址

### Vue 3 集成

- ClientUnit 开发者编写标准 Vue 3 SFC 组件
- 通过 `inject('platform')` 获取平台 API
- 通过 `platform.execute()` 调用后端 ServerUnit
- 共享依赖：vue, ant-design-vue, echarts, pinia

### 隔离机制

- 样式隔离：CSS Modules + 命名空间前缀（默认）；Shadow DOM（可选）
- JS 隔离：Module Federation shared scope
- 错误隔离：Vue Error Boundary 包裹每个 ClientUnit

### 离线部署结构

```text
/opt/platform/artifacts/
  ├── client-units/
  │     ├── perf-radar-dashboard/
  │     │     ├── remoteEntry.js
  │     │     ├── index.js
  │     │     └── styles.css
  │     └── decomposition-wizard/
  │           └── ...
  └── server-units/
        ├── perf-score/
        │     ├── unit.jar
        │     └── unit.yaml
        └── ...
```

ITP 安装时自动将 `behavior/client-units/` 和 `behavior/server-units/` 解压到本地制品仓库并注册到 Registry。

## 六、信创兼容技术选型

| 组件 | 选型 | 信创兼容性 |
|---|---|---|
| JDK | 华为毕昇 JDK / 腾讯 Kona JDK | ✅ 鲲鹏/飞腾/x86 |
| Spring Boot/Cloud | 2.7.x / 3.x | ✅ 纯 Java |
| L1 表达式 | Aviator | ✅ 纯 Java |
| L2 JS 沙箱 | Nashorn(≤JDK14) / Rhino | ✅ 纯 Java |
| L2 Python 沙箱 | Jython | ✅ 纯 Java |
| L3 插件加载 | ClassLoader / ProcessBuilder | ✅ JVM 标准能力 |
| 数据库 | 达梦 DM8 / 人大金仓 / openGauss | ✅ JDBC |
| 中间件 | 东方通 TongWeb / TongRDS | ✅ |
| 消息 | RocketMQ | ✅ 纯 Java |
| 缓存 | Redis（或国产兼容替代） | ✅ |
| 对象存储 | MinIO / 本地文件系统 | ✅ |
| 注册中心 | Nacos | ✅ 纯 Java |
| 前端 | Vue 3 + Vite | ✅ 浏览器端 |
| 信创浏览器 | 360/奇安信/UOS（Chromium 内核） | ✅ |
| Module Federation | Webpack 5 | ✅ Chromium 支持 |

### 信创风险点

- GraalVM CE 在鲲鹏/龙芯上的 Polyglot 性能需 POC 验证
- 如不可用，回退到 Nashorn(JS) + Jython(Python)，功能不受影响
- 商业软件（MATLAB/ANSYS）本身是否支持信创由客户自行负责

## 七、Spring Cloud 微服务拓扑更新

```text
Spring Cloud Gateway
  │
  ├── BFF Services（工作包/试验/合同...）
  │     └── Feign 调用 extension-unit-runtime
  │
  ├── 领域服务 L3（WorkPackage/Test/Contract...）
  │     └── 遇到 ServerUnit 绑定 → Feign 或进程内调用
  │
  ├── 共享服务 L4
  │     ├── extension-unit-runtime（新增）
  │     │     ├── L1 Aviator
  │     │     ├── L2 Nashorn/Jython 沙箱
  │     │     ├── L3a ClassLoader 插件
  │     │     └── L3b 子进程管理
  │     ├── extension-unit-registry（新增）
  │     ├── object-model-registry
  │     ├── trace-service
  │     ├── task-orchestration
  │     ├── workflow-service
  │     └── audit-service
  │
  └── 基础设施
        Nacos / RocketMQ / Redis / 达梦 / MinIO
```

### 进程内调用的优化路径

对于高频 ServerUnit（如绩效计算），可以直接在领域服务 JVM 内加载插件 JAR，避免跨服务网络调用：

```text
WorkPackageService (Spring Boot)
  ├── 业务逻辑
  ├── ServerUnitPluginManager（嵌入式）
  │     └── 加载高频 ServerUnit JAR
  │     └── 进程内直接调用
  └── 低频/高隔离 → Feign 调用 extension-unit-runtime
```

## 八、下一步

1. ✅ 沉淀讨论纪要（本文档）
2. 更新对象模型 §13.5.13 ServerUnit 实现级别
3. 更新蓝图 §4.6 补充运行时约束
4. 新增蓝图附录：信创兼容技术选型清单
