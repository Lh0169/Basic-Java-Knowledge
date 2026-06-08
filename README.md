# Java 知识体系

> 一套从入门到精通、从理论到实战的 Java 系统化学习文档，分为**基础篇**、**进阶篇**、**高并发篇**、**实战篇**与**源码篇**五册，覆盖 Java 语法、核心 API、JVM 原理、并发编程、性能调优、源码分析与工程实践。

---

## 📚 文档清单

| 文档 | 定位 | 核心内容 | 篇幅 |
|------|------|---------|------|
| **[Java 基础知识体系](Java基础知识体系.md)** | 基础篇 | 类与对象、三大特性、抽象与接口、常用关键字、包与权限、内部类、字符串、集合框架、异常处理、IO 流、泛型、多线程基础、Object 方法、基本类型与包装类、枚举、注解、反射、Lambda、Stream API、时间 API | 约 26K 字符 |
| **[Java 进阶知识体系](Java进阶知识体系.md)** | 进阶篇 | JVM 内存与对象模型、垃圾回收机制（G1/ZGC）、类加载与模块化（JPMS）、字节码执行引擎、JMM 内存模型、AQS 与并发工具源码、线程池与 CompletableFuture、虚拟线程（JDK 21）、集合源码深度剖析、NIO 与 Netty、零拷贝、反射与动态代理、泛型类型系统、函数式编程与 Stream 性能、设计模式在 JDK 中的体现、性能调优与工具链（Arthas/JFR）、Java 版本演进图谱 | 约 41K 字符 |
| **[高并发极致优化手册](高并发极致优化手册.md)** | 高并发篇 | 无锁编程与 CAS 深度、LongAdder/Striped64 原理、伪共享与 @Contended 注解、Disruptor 框架剖析、高并发锁策略选择、内存模型与指令重排验证 | 约 18K 字符 |
| **[性能调优实战手册](性能调优实战手册.md)** | 实战篇 | CPU 飙高排查实战、内存泄漏与 OOM 分析、死锁与锁竞争优化、GC 长暂停诊断、IO 瓶颈与网络排查、Arthas 深度用法 | 约 22K 字符 |
| **[中间件源码剖析手册](中间件源码剖析手册.md)** | 源码篇 | Spring IoC/AOP/事务/MVC 源码、MyBatis 初始化与 SQL 执行链路、Tomcat 架构与类加载隔离、Netty Reactor 模型与内存管理、Dubbo/RocketMQ/Redis/ZooKeeper/Spring Boot 自动装配 | 约 35K 字符 |

---

## 🎯 学习路径建议

```
第一阶段：基础夯实（1-2 个月）
├── 类与对象 → 封装/继承/多态
├── 常用 API → 字符串、集合、异常、IO
├── 核心机制 → 泛型、反射、注解、Lambda
└── 多线程入门 → 线程创建、同步、线程池基础

第二阶段：进阶深入（2-3 个月）
├── JVM 原理 → 内存区域、GC 算法、类加载、模块化
├── 并发进阶 → JMM、AQS、锁优化、CompletableFuture、虚拟线程
├── 源码剖析 → HashMap/ConcurrentHashMap/ThreadPoolExecutor 源码
├── IO 网络 → NIO、Netty、零拷贝、RPC 原理
└── 性能工具 → Arthas、JFR、GC 调优、版本演进

第三阶段：高并发与极致优化（1-2 个月）
├── 无锁编程 → CAS 原理、VarHandle、无锁栈/队列
├── 分段累加 → LongAdder/Striped64 设计思想
├── 缓存行优化 → 伪共享、@Contended、Disruptor 填充
├── 锁策略选择 → synchronized/ReentrantLock/StampedLock 场景化选型
└── 并发验证 → jcstress 测试、内存屏障、DCL 单例剖析

第四阶段：性能调优实战（持续）
├── 线上排查 → CPU 飙高、内存泄漏、死锁、GC 长暂停
├── 工具链 → jstack/jmap/jstat、Arthas 深度用法、MAT 分析
├── 系统优化 → IO/网络瓶颈、零拷贝、线程数模型
└── 监控体系 → GC 日志解读、JVM 参数调优、火焰图分析

第五阶段：中间件源码与架构（持续）
├── Spring 核心 → IoC 容器、AOP 代理、事务传播、MVC 请求链
├── 数据访问 → MyBatis 动态代理、缓存、插件机制
├── Web 容器 → Tomcat 架构、类加载隔离、Session 集群
├── 网络框架 → Netty Reactor、Pipeline、ByteBuf 内存池
└── 生态组件 → Dubbo SPI、RocketMQ 存储、Redis 客户端、ZK 协调
```

---

## 📖 文档特色

### 统一的编写风格

每章按 **"基本概念 → 深入原理 → 代码示例 → 易错陷阱"** 四层结构编写：

- **基本概念**：清晰定义术语，建立知识框架
- **深入原理**：底层机制、源码关键片段、设计思想
- **代码示例**：可直接运行的 Java 代码，配合注释说明
- **易错陷阱（PS）**：标注面试高频点、常见误区与最佳实践

### 面试导向

- 每章末尾 **PS** 标注高频面试题与易错点
- 进阶篇 **附录 A** 汇总 20+ 高频面试题索引，关联具体章节
- 进阶篇 **附录 B/C/D** 提供 JVM 参数速查、性能分析命令速查、Java 版本演进图谱
- 高并发篇与实战篇每个案例均附 **面试点睛**，提炼回答要点

### 前沿特性覆盖

- **JDK 21 虚拟线程**（Project Loom）与结构化并发
- **JDK 17+ 新类型特性**：`record`、`sealed class`、模式匹配
- **现代 GC**：ZGC、Shenandoah 亚毫秒级停顿
- **模块化系统**：JPMS、`module-info.java` 实战
- **无锁编程**：VarHandle、Disruptor、缓存行填充

### 实战案例驱动

- 性能调优篇提供 **CPU 飙高、内存泄漏、死锁、GC 长暂停** 等真实线上案例
- 每个案例包含：问题表现 → 排查步骤 → 根因定位 → 解决方案 → 预防措施
- Arthas 命令速查表，支持离线环境部署

---

## 🔍 快速导航

### 基础知识体系核心章节

| 章节 | 关键词 | 面试频率 |
|------|--------|---------|
| [类与对象](Java基础知识体系.md#1-类与对象) | 内存模型、this、构造方法链 | ⭐⭐⭐ |
| [三大特性](Java基础知识体系.md#2-三大特性) | 封装、继承、多态、instanceof | ⭐⭐⭐⭐ |
| [抽象与接口](Java基础知识体系.md#3-抽象与接口) | 抽象类 vs 接口、函数式接口 | ⭐⭐⭐⭐ |
| [集合框架](Java基础知识体系.md#8-集合框架) | ArrayList/HashMap 原理、迭代器 | ⭐⭐⭐⭐⭐ |
| [多线程基础](Java基础知识体系.md#12-多线程基础) | synchronized、volatile、线程池 | ⭐⭐⭐⭐⭐ |
| [Object 方法](Java基础知识体系.md#13-object-类的核心方法) | equals/hashCode/clone 契约 | ⭐⭐⭐⭐ |
| [反射](Java基础知识体系.md#17-反射) | Class/Method/Field、setAccessible | ⭐⭐⭐ |
| [Stream API](Java基础知识体系.md#19-stream-api) | 惰性求值、并行流、Collector | ⭐⭐⭐⭐ |

### 进阶知识体系核心章节

| 章节 | 关键词 | 面试频率 |
|------|--------|---------|
| [JVM 内存与对象模型](Java进阶知识体系.md#1-jvm-内存区域与对象模型) | Mark Word、CompressedOops、对齐填充 | ⭐⭐⭐⭐⭐ |
| [垃圾回收机制](Java进阶知识体系.md#2-垃圾回收机制) | G1/ZGC、可达性分析、引用类型 | ⭐⭐⭐⭐⭐ |
| [类加载与模块化](Java进阶知识体系.md#3-类加载机制与模块系统jpms) | 双亲委派、自定义类加载器、JPMS | ⭐⭐⭐⭐⭐ |
| [JMM 与并发](Java进阶知识体系.md#1-java-内存模型jmm) | happens-before、volatile、synchronized 锁升级 | ⭐⭐⭐⭐⭐ |
| [AQS 与并发工具](Java进阶知识体系.md#2-aqs-与并发工具源码) | state/CLH 队列、ReentrantLock、StampedLock | ⭐⭐⭐⭐⭐ |
| [线程池进阶](Java进阶知识体系.md#34-threadpoolexecutor) | 核心参数、拒绝策略、ctl 状态位 | ⭐⭐⭐⭐⭐ |
| [虚拟线程](Java进阶知识体系.md#4-虚拟线程project-loomjdk-21) | 载体线程、Pinning、结构化并发 | ⭐⭐⭐⭐ |
| [HashMap 源码](Java进阶知识体系.md#12-hashmap-深度剖析) | 哈希扰动、红黑树化、扩容优化 | ⭐⭐⭐⭐⭐ |
| [NIO 与 Netty](Java进阶知识体系.md#第四部分ionio-与网络编程) | Selector、零拷贝、Reactor、ByteBuf | ⭐⭐⭐⭐ |
| [动态代理](Java进阶知识体系.md#2-动态代理) | JDK 代理、CGLIB、字节码生成 | ⭐⭐⭐⭐ |
| [性能调优工具](Java进阶知识体系.md#第九部分性能调优与工具链) | Arthas、JFR、JVM 参数、GC 日志 | ⭐⭐⭐⭐ |

### 高并发极致优化核心章节

| 章节 | 关键词 | 面试频率 |
|------|--------|---------|
| [CAS 深度](高并发极致优化手册.md#1-无锁编程与-cas-深度) | LOCK CMPXCHG、ABA 问题、VarHandle | ⭐⭐⭐⭐⭐ |
| [LongAdder 原理](高并发极致优化手册.md#2-longadder-与-striped64-原理) | base + Cell 数组、分段累加、伪共享 | ⭐⭐⭐⭐⭐ |
| [伪共享优化](高并发极致优化手册.md#3-伪共享与-contended-注解) | 缓存行填充、@Contended、Disruptor Sequence | ⭐⭐⭐⭐ |
| [Disruptor 框架](高并发极致优化手册.md#4-disruptor-框架剖析) | RingBuffer、Sequencer、无锁生产消费 | ⭐⭐⭐⭐ |
| [锁策略选择](高并发极致优化手册.md#5-高并发场景的锁策略选择) | StampedLock 乐观读、ThreadLocalRandom | ⭐⭐⭐⭐⭐ |
| [内存模型验证](高并发极致优化手册.md#6-内存模型与指令重排验证) | StoreLoad 屏障、jcstress、DCL 单例 | ⭐⭐⭐⭐⭐ |

### 性能调优实战核心章节

| 章节 | 关键词 | 面试频率 |
|------|--------|---------|
| [CPU 飙高排查](性能调优实战手册.md#1-cpu-飙高排查实战) | top -Hp、jstack、Arthas thread -n | ⭐⭐⭐⭐⭐ |
| [内存泄漏排查](性能调优实战手册.md#2-内存泄漏排查与-oom-分析) | jmap dump、MAT、ThreadLocal 泄漏 | ⭐⭐⭐⭐⭐ |
| [死锁与锁竞争](性能调优实战手册.md#3-死锁与锁竞争优化) | jstack -l、thread -b、tryLock | ⭐⭐⭐⭐⭐ |
| [GC 长暂停诊断](性能调优实战手册.md#4-gc-长暂停问题诊断) | GC 日志、晋升失败、ZGC/G1 调优 | ⭐⭐⭐⭐⭐ |
| [IO 与网络排查](性能调优实战手册.md#5-io-瓶颈与网络排查) | iostat、TIME_WAIT、零拷贝 | ⭐⭐⭐⭐ |
| [Arthas 深度用法](性能调优实战手册.md#6-arthas-深度用法) | watch、trace、vmtool、monitor | ⭐⭐⭐⭐⭐ |

### 中间件源码剖析核心章节

| 章节 | 关键词 | 面试频率 |
|------|--------|---------|
| [Spring IoC](中间件源码剖析手册.md#1-spring-framework-核心源码) | 三级缓存、循环依赖、Bean 生命周期 | ⭐⭐⭐⭐⭐ |
| [Spring AOP](中间件源码剖析手册.md#1-spring-framework-核心源码) | JDK/CGLIB 代理、拦截器链、事务失效 | ⭐⭐⭐⭐⭐ |
| [Spring 事务](中间件源码剖析手册.md#1-spring-framework-核心源码) | 传播行为、TransactionInterceptor、保存点 | ⭐⭐⭐⭐⭐ |
| [Spring MVC](中间件源码剖析手册.md#1-spring-framework-核心源码) | DispatcherServlet、HandlerMapping、拦截器 | ⭐⭐⭐⭐⭐ |
| [MyBatis 执行链路](中间件源码剖析手册.md#2-mybatis-源码剖析) | MapperProxy、Executor、缓存机制 | ⭐⭐⭐⭐⭐ |
| [Tomcat 架构](中间件源码剖析手册.md#3-tomcat-源码剖析) | Connector、Container、类加载隔离 | ⭐⭐⭐⭐ |
| [Netty 核心](中间件源码剖析手册.md#4-netty-源码剖析) | Reactor、Pipeline、ByteBuf、内存池 | ⭐⭐⭐⭐⭐ |
| [Dubbo SPI](中间件源码剖析手册.md#5-其他中间件选读) | 自适应扩展、服务暴露与引用 | ⭐⭐⭐⭐ |
| [Spring Boot 自动装配](中间件源码剖析手册.md#5-其他中间件选读) | @EnableAutoConfiguration、spring.factories | ⭐⭐⭐⭐⭐ |

---

## 🛠️ 环境要求

- **JDK 版本**：基础篇兼容 JDK 8+；进阶篇涉及 JDK 9+（模块化）、JDK 17+（record/sealed）、JDK 21+（虚拟线程）特性
- **阅读工具**：任意 Markdown 编辑器（VS Code、Typora、Obsidian）或 IDE 内置预览
- **实践环境**：建议安装 OpenJDK 17 或 21，配合 Maven/Gradle 构建工具
- **诊断工具**：Arthas（生产环境诊断）、MAT（内存分析）、JMH（基准测试）、jcstress（并发测试）

---

## 📌 使用建议

1. **基础篇**：适合 Java 初学者系统学习，或面试前快速复习语法与 API
2. **进阶篇**：适合 1-3 年经验的开发者深入原理，或准备高级/架构师面试
3. **高并发篇**：适合需要深入理解无锁编程、低延迟架构的开发者，配合 JMH/jcstress 动手实验
4. **实战篇**：适合线上运维和性能优化工程师，建议结合实际生产案例对照阅读
5. **源码篇**：适合准备架构师面试或需要深入理解框架设计的开发者，建议配合 Debug 跟踪
6. **交叉阅读**：
   - 基础篇"集合框架" + 进阶篇"集合源码深化" → 从 API 使用到源码实现
   - 进阶篇"JVM 调优" + 实战篇"GC 长暂停诊断" → 从理论参数到线上排障
   - 高并发篇"锁策略" + 实战篇"死锁优化" → 从选型原理到问题根治
   - 进阶篇"NIO/Netty" + 源码篇"Netty 源码" → 从使用到设计精髓

---

## 📝 文档维护

- 编写日期：2026-06-08
- 适配 Java 版本：Java 8 ~ Java 24
- 编写风格：Markdown，GitHub Flavored Markdown 兼容
- 建议反馈：如发现内容错误、过时或遗漏，欢迎提交 Issue 或 PR

---

> **💡 学习理念**：基础决定下限，原理决定上限，实战检验真知。不要停留在"会用 API"，要追问"为什么这样设计"、"底层如何实现"、"生产环境如何调优"。这套知识体系从"用"到"懂"到"精"再到"排"，陪你走完 Java 工程师从初级到架构师的完整成长之路。
