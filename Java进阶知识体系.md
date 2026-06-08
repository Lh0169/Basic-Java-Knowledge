# Java 进阶知识体系

> **定位**：面向已掌握 Java 基础知识（类与对象、集合、IO、多线程基础、泛型等）的开发者，深入 JVM 底层、并发编程、源码原理、性能调优与工程实践。
> **学习路径**：向下挖原理 → 向上看设计 → 横向连生态

---

## 目录

- [第一部分：JVM 底层与内存模型](#第一部分jvm-底层与内存模型)
- [第二部分：并发编程进阶](#第二部分并发编程进阶)
- [第三部分：数据结构与集合源码深化](#第三部分数据结构与集合源码深化)
- [第四部分：IO、NIO 与网络编程](#第四部分ionio-与网络编程)
- [第五部分：反射、动态代理与注解](#第五部分反射动态代理与注解)
- [第六部分：泛型与类型系统进阶](#第六部分泛型与类型系统进阶)
- [第七部分：函数式编程与 Stream 性能](#第七部分函数式编程与-stream-性能)
- [第八部分：设计模式在 Java 源码中的应用](#第八部分设计模式在-java-源码中的应用)
- [第九部分：性能调优与工具链](#第九部分性能调优与工具链)
- [附录](#附录)

---

## 第一部分：JVM 底层与内存模型

### 1.1 JVM 内存区域

#### 1.1.1 运行时数据区（JVM 规范）

| 区域 | 线程共享 | 存储内容 | 常见异常 |
|------|---------|---------|---------|
| **堆（Heap）** | 是 | 对象实例、数组 | `OutOfMemoryError` |
| **方法区（Method Area）** | 是 | 类信息、常量、静态变量、JIT 编译后的代码 | `OutOfMemoryError`（元空间） |
| **虚拟机栈（VM Stack）** | 否 | 栈帧（局部变量表、操作数栈、动态链接、方法返回地址） | `StackOverflowError` / `OutOfMemoryError` |
| **本地方法栈（Native Method Stack）** | 否 | Native 方法信息 | 同上 |
| **程序计数器（PC Register）** | 否 | 当前线程执行的字节码行号指示器 | 无 |

- **堆**：新生代（Eden + Survivor0 + Survivor1）+ 老年代（Old Gen）+ 元空间（Metaspace，JDK 8+，替代永久代）
- **虚拟机栈**：每个方法执行时创建一个栈帧，方法调用对应栈帧的入栈和出栈
- **方法区**：JDK 8 之前为永久代（PermGen），受 JVM 内存限制；JDK 8+ 改为元空间，使用本地内存，默认无上限

#### 1.1.2 深入：对象在堆中的内存布局

一个普通对象在内存中的结构：

```
| 对象头（Header） | 实例数据（Instance Data） | 对齐填充（Padding） |
```

- **对象头（12~16 字节）**：
  - **Mark Word**（8 字节，64 位 JVM）：存储对象的 hashCode、GC 分代年龄、锁状态标志、偏向线程 ID 等
  - **类型指针**（4/8 字节）：指向方法区中该对象的类元数据（开启压缩指针 `-XX:+UseCompressedOops` 后为 4 字节）
  - **数组长度**（仅数组对象有，4 字节）
- **实例数据**：父类字段 + 本类字段，按声明顺序和类型大小排列
- **对齐填充**：HotSpot 要求对象起始地址必须是 8 字节的整数倍，不足则填充

```java
// 使用 JOL (Java Object Layout) 查看内存布局
import org.openjdk.jol.info.ClassLayout;

public class ObjectLayoutDemo {
    public static void main(String[] args) {
        Object obj = new Object();
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());

        // 输出示例（64 位 JVM，开启压缩指针）：
        // java.lang.Object object internals:
        //  OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
        //       0     4        (object header)                           01 00 00 00 (hash=0, age=0, biased=0)
        //       4     4        (object header)                           00 00 00 00
        //       8     4        (object header)                           e5 01 00 20 (klass=0x200001e5)
        //      12     4        (loss due to the next object alignment)
        // Instance size: 16 bytes
    }
}
```

#### 1.1.3 常见问题

- **内存泄漏场景**：
  - 静态集合持有短生命周期对象引用
  - 未关闭的资源（流、连接、监听器）
  - 自定义 ClassLoader 未正确释放
  - ThreadLocal 未调用 `remove()`
- **栈溢出**：递归过深、大量局部变量、栈帧过多

---

### 1.2 垃圾回收机制

#### 1.2.1 判断对象是否存活

**可达性分析算法（主流）**：
- 从一组称为 **GC Roots** 的对象出发，向下搜索，搜索走过的路径称为引用链
- 若对象不在任何引用链上，则判定为可回收

**GC Roots 包括**：
1. 虚拟机栈中局部变量表引用的对象
2. 方法区中静态属性引用的对象
3. 方法区中常量引用的对象
4. 本地方法栈中 JNI（Native 方法）引用的对象
5. JVM 内部的引用（基本数据类型对应的 Class 对象、异常对象、系统类加载器）
6. 所有被同步锁（synchronized 关键字）持有的对象
7. 反映 JVM 内部情况的 JMXBean、JVMTI 中注册的回调、本地代码缓存等

#### 1.2.2 四种引用类型

| 引用类型 | 回收时机 | 使用场景 | 配合类 |
|---------|---------|---------|--------|
| **强引用（Strong）** | 永不回收，即使 OOM | 普通对象引用 | 无 |
| **软引用（Soft）** | 内存不足时回收 | 缓存 | `SoftReference` |
| **弱引用（Weak）** | 下次 GC 时回收 | 缓存、防止内存泄漏 | `WeakReference` |
| **虚引用（Phantom）** | 随时可能被回收，无法通过引用获取对象 | 跟踪对象回收、清理资源 | `PhantomReference` + `ReferenceQueue` |

```java
// 软引用示例：图片缓存
SoftReference<Bitmap> softRef = new SoftReference<>(bitmap);
Bitmap cached = softRef.get();  // 可能返回 null（已被回收）

// 弱引用示例：ThreadLocal 的 Entry 使用弱引用
// 虚引用必须配合 ReferenceQueue 使用
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantom = new PhantomReference<>(obj, queue);
// phantom.get() 永远返回 null，仅用于在对象被回收时收到通知
```

#### 1.2.3 垃圾回收算法

| 算法 | 原理 | 优点 | 缺点 | 适用场景 |
|------|------|------|------|---------|
| **标记-清除** | 标记存活对象，清除未标记对象 | 简单 | 产生内存碎片、效率低 | 老年代（CMS） |
| **复制** | 将存活对象复制到另一块内存，清空原区域 | 无碎片、高效 | 内存利用率仅 50% | 新生代（Serial、ParNew） |
| **标记-整理** | 标记存活对象，向一端移动，清理边界外内存 | 无碎片 | 移动对象开销大 | 老年代（Serial Old、Parallel Old） |
| **分代收集** | 新生代用复制，老年代用标记-清除/整理 | 兼顾效率与空间 | 复杂 | 现代 JVM 默认策略 |

#### 1.2.4 垃圾回收器演进

```
Serial (单线程) → Parallel (多线程) → CMS (并发标记清除) → G1 (区域化) → ZGC / Shenandoah (低延迟)
```

| 回收器 | 算法 | 适用堆大小 | 停顿时间 | 特点 |
|--------|------|-----------|---------|------|
| **Serial** | 复制（新生代）+ 标记-整理（老年代） | < 100MB | ~几十 ms | 单线程，Client 模式默认 |
| **Parallel** | 复制 + 标记-整理 | 数 GB | ~几百 ms | 吞吐量优先，JDK 8 默认 |
| **CMS** | 标记-清除 | 数 GB ~ 20GB | ~几十 ms | 并发回收，内存碎片 |
| **G1** | 标记-整理 + 复制 | 6GB ~ 上百 GB | 目标 < 200ms | 区域化、可预测停顿，JDK 9+ 默认 |
| **ZGC** | 染色指针 + 读屏障 | TB 级 | < 10ms | 并发整理，JDK 15+ 生产可用 |
| **Shenandoah** | 转发指针 + 读屏障 | TB 级 | < 10ms | 与 ZGC 类似，RedHat 开发 |

**G1 核心设计**：
- 将堆划分为多个大小相等的 Region（默认 2048 个）
- 每个 Region 可动态扮演 Eden、Survivor、Old 或 Humongous（存放大对象）
- 优先回收垃圾最多的 Region（Garbage First）
- 使用 Remembered Set 记录跨 Region 引用，避免全堆扫描

```bash
# G1 常用调优参数
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200          # 目标最大停顿时间
-XX:G1HeapRegionSize=16m          # Region 大小
-XX:InitiatingHeapOccupancyPercent=45  # 触发并发标记的堆占用率
```

#### 1.2.5 GC 日志与调优

```bash
# JDK 9+ 统一日志格式
-XX:+PrintGCDetails -Xlog:gc*:file=gc.log:time,uptime,level,tags

# 关键指标
# 1. 吞吐量 = 用户代码时间 / (用户代码时间 + GC 时间)
# 2. 停顿时间：最大停顿、平均停顿、停顿频率
# 3. 内存分配速率：每秒分配多少 MB

# 常用分析工具
jstat -gcutil <pid> 1000 10   # 每秒打印 GC 统计
jcmd <pid> GC.heap_dump       # 生成堆转储
```

---

### 1.3 类加载机制

#### 1.3.1 类加载过程

```
加载（Loading）→ 验证（Verification）→ 准备（Preparation）→ 解析（Resolution）→ 初始化（Initialization）→ 使用 → 卸载
```

| 阶段 | 工作内容 |
|------|---------|
| **加载** | 通过全限定名获取二进制字节流，生成 Class 对象 |
| **验证** | 文件格式、元数据、字节码、符号引用验证 |
| **准备** | 为类变量（static）分配内存并设置零值（`final static` 在此阶段赋初值） |
| **解析** | 将符号引用替换为直接引用（可选，可在初始化后执行） |
| **初始化** | 执行 `<clinit>()`：静态变量赋值 + 静态代码块（按源码顺序） |

**触发初始化的时机（主动引用）**：
1. `new` 实例化对象、访问/设置静态字段（`final` 常量除外）、调用静态方法
2. 反射调用（`Class.forName()`）
3. 子类初始化时，父类先初始化
4. JVM 启动时，包含 `main()` 方法的类
5. 动态语言支持（`MethodHandle` 解析结果为 REF_getStatic 等）

#### 1.3.2 双亲委派模型

```
        自定义类加载器
              ↑
        应用程序类加载器（AppClassLoader）  ← 加载 classpath 下的类
              ↑
        扩展类加载器（ExtClassLoader / PlatformClassLoader JDK 9+）  ← 加载 <JAVA_HOME>/lib/ext
              ↑
        启动类加载器（BootstrapClassLoader）  ← 加载 <JAVA_HOME>/lib（C++ 实现）
```

**工作流程**：类加载器收到加载请求 → 委托父加载器 → 父加载器无法加载 → 子加载器自己加载

**为什么需要双亲委派？**
- **安全性**：防止核心类库被篡改（如自定义 `java.lang.String`）
- **避免重复加载**：保证同一个类只被加载一次

**如何打破双亲委派？**
1. **自定义类加载器**：重写 `loadClass()` 方法（Tomcat、OSGi）
2. **线程上下文类加载器（TCCL）**：`Thread.setContextClassLoader()`，SPI 机制使用（JDBC、JNDI）
3. **OSGi / JPMS**：模块化隔离，每个模块有自己的类加载器

```java
// 自定义类加载器示例：实现热加载
public class HotSwapClassLoader extends ClassLoader {
    private String classPath;

    public HotSwapClassLoader(String classPath) {
        this.classPath = classPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 先从自定义路径加载，打破双亲委派
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        }
        return defineClass(name, classData, 0, classData.length);
    }

    private byte[] loadClassData(String name) {
        // 从文件系统或网络读取 .class 文件
        // ...
        return null;
    }
}
```

#### 1.3.3 Java 模块化系统（JPMS，JDK 9+）

```java
// module-info.java
module com.example.app {
    requires java.base;           // 隐式依赖，可省略
    requires java.sql;            // 显式依赖其他模块
    requires transitive java.xml; // 传递依赖

    exports com.example.api;      // 对外暴露的包
    exports com.example.spi to com.example.plugin;  // 定向暴露

    opens com.example.internal;   // 开放反射访问（运行时）
    opens com.example.entity to org.hibernate.orm.core;

    provides com.example.spi.Logger 
        with com.example.impl.FileLogger;  // 服务提供
    uses com.example.spi.Logger;  // 服务消费
}
```

**模块路径 vs 类路径**：
- **模块路径（`--module-path`）**：强封装，必须显式导出才能访问
- **类路径（`-cp` / `-classpath`）**：所有类放入未命名模块（unnamed module），可读取所有模块但不被强封装保护
- **自动模块（Automatic Module）**：类路径上的 JAR 自动成为模块，模块名从 `MANIFEST.MF` 或文件名推断

**模块化对反射的影响**：
```bash
# 若模块未 opens 包，反射访问私有成员会失败
java --add-opens java.base/java.lang=ALL-UNNAMED MyApp
java --add-exports java.base/sun.nio.ch=ALL-UNNAMED MyApp
```

---

### 1.4 字节码执行引擎

#### 1.4.1 栈帧结构

每个方法执行时创建一个栈帧，包含：

| 组件 | 说明 |
|------|------|
| **局部变量表（Local Variables）** | 存储基本数据类型、对象引用、`returnAddress`。以 Slot（32 位）为单位，`long`/`double` 占 2 个 Slot |
| **操作数栈（Operand Stack）** | 方法执行的工作区，字节码指令从这里取数、压结果 |
| **动态链接（Dynamic Linking）** | 指向运行时常量池中该方法的符号引用，运行时解析为直接引用 |
| **方法返回地址** | 方法执行完毕后回到调用者的位置 |

#### 1.4.2 方法调用与分派

| 分派类型 | 触发时机 | 依据 | 示例 |
|---------|---------|------|------|
| **静态分派** | 编译期 | 静态类型 | 方法重载（Overload） |
| **动态分派** | 运行期 | 实际类型 | 方法重写（Override） |
| **单分派** | - | 一个宗量 | Java 是单分派（方法接收者） |
| **多分派** | - | 多个宗量 | 编译期依据静态类型 + 方法参数（重载），运行期依据实际类型（重写） |

```java
// 静态分派（重载）
public class StaticDispatch {
    static abstract class Human {}
    static class Man extends Human {}
    static class Woman extends Human {}

    public void sayHello(Human guy) { System.out.println("hello, guy"); }
    public void sayHello(Man guy) { System.out.println("hello, gentleman"); }
    public void sayHello(Woman guy) { System.out.println("hello, lady"); }

    public static void main(String[] args) {
        Human man = new Man();      // 静态类型 Human，实际类型 Man
        Human woman = new Woman();  // 静态类型 Human，实际类型 Woman
        StaticDispatch sr = new StaticDispatch();
        sr.sayHello(man);      // 输出：hello, guy（编译期按静态类型分派）
        sr.sayHello(woman);    // 输出：hello, guy
    }
}
```

**`invokedynamic` 指令（JDK 7+）**：
- 运行时动态确定调用目标，支持 JVM 上的动态语言
- Lambda 表达式底层使用 `invokedynamic` + `LambdaMetafactory`，而非生成匿名内部类

---

## 第二部分：并发编程进阶

### 2.1 Java 内存模型（JMM）

#### 2.1.1 主内存与工作内存

JMM 规定所有变量存储在主内存，每个线程有自己的工作内存（本地内存缓存），线程对变量的操作必须在工作内存中进行，不能直接读写主内存。

```
线程 A 工作内存 ←→ 主内存 ←→ 线程 B 工作内存
         ↕               ↕
    load/store      read/write
```

#### 2.1.2 happens-before 规则（8 条）

无需同步即可保证可见性的规则：

1. **程序次序规则**：同一个线程中，前面的操作 happens-before 后面的操作
2. **监视器锁规则**：`unlock` happens-before 后面对同一把锁的 `lock`
3. **volatile 规则**：`volatile` 写 happens-before 后面对该变量的读
4. **线程启动规则**：`Thread.start()` happens-before 线程中的每个动作
5. **线程终止规则**：线程中的所有操作 happens-before 对该线程 `Thread.join()` 的返回
6. **线程中断规则**：`interrupt()` happens-before 检测到中断事件
7. **对象终结规则**：构造函数执行 happens-before `finalize()` 方法
8. **传递性**：A happens-before B，B happens-before C，则 A happens-before C

#### 2.1.3 volatile 内存语义

- **可见性**：写 `volatile` 变量会将工作内存刷新到主内存；读 `volatile` 会从主内存刷新工作内存
- **禁止指令重排序**：通过内存屏障实现
  - 写 `volatile`：前面插入 **StoreStore** 屏障，后面插入 **StoreLoad** 屏障
  - 读 `volatile`：后面插入 **LoadLoad** 屏障和 **LoadStore** 屏障

```java
// volatile 经典用法：双重检查锁定（DCL）单例
public class Singleton {
    private volatile static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {                    // 第一次检查（无锁）
            synchronized (Singleton.class) {
                if (instance == null) {            // 第二次检查（有锁）
                    instance = new Singleton();    // 禁止重排序：1.分配内存 2.初始化 3.赋值引用
                }
            }
        }
        return instance;
    }
}
// 若不用 volatile，步骤 2 和 3 可能重排序，导致其他线程获取到未初始化的对象
```

#### 2.1.4 synchronized 锁升级过程

```
无锁 → 偏向锁 → 轻量级锁 → 重量级锁
```

| 锁状态 | 适用场景 | 原理 | 开销 |
|--------|---------|------|------|
| **无锁** | 没有竞争 | 无 | 无 |
| **偏向锁** | 只有一个线程访问同步块 | Mark Word 记录线程 ID，下次直接进入 | 极低（CAS 替换线程 ID） |
| **轻量级锁** | 有竞争但交替执行 | CAS 自旋尝试获取锁，Mark Word 复制到栈帧的 Lock Record | 低（自旋消耗 CPU） |
| **重量级锁** | 竞争激烈，自旋失败 | 未获取到锁的线程阻塞，依赖操作系统 Mutex | 高（用户态↔内核态切换） |

**锁升级不可逆**：偏向锁 → 轻量级锁（有竞争时撤销偏向）；轻量级锁 → 重量级锁（自旋超过阈值或 CAS 失败）

```bash
# JVM 参数控制锁行为
-XX:+UseBiasedLocking          # 启用偏向锁（JDK 15+ 默认关闭）
-XX:BiasedLockingStartupDelay=0 # 启动后立即启用偏向锁
-XX:+UseHeavyMonitors          # 禁用轻量级锁和偏向锁
```

---

### 2.2 AQS 与并发工具源码

#### 2.2.1 AQS 核心设计

`AbstractQueuedSynchronizer` 是 JUC 并发包的基石：

- **state 变量**：`volatile int`，表示同步状态（ReentrantLock 中：0=未锁定，>0=重入次数）
- **CLH 变体队列**：FIFO 双向队列，存储等待获取锁的线程（Node 节点）
- **模板方法模式**：子类实现 `tryAcquire` / `tryRelease`（独占）或 `tryAcquireShared` / `tryReleaseShared`（共享）

```java
// AQS 核心结构（简化）
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer {
    private volatile int state;                    // 同步状态
    private transient volatile Node head;          // 队列头
    private transient volatile Node tail;          // 队列尾

    // 子类必须实现
    protected boolean tryAcquire(int arg) { throw new UnsupportedOperationException(); }
    protected boolean tryRelease(int arg) { throw new UnsupportedOperationException(); }

    // 获取锁的入口（模板方法）
    public final void acquire(int arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
}
```

#### 2.2.2 ReentrantLock 与 ReentrantReadWriteLock

| 特性 | ReentrantLock | ReentrantReadWriteLock |
|------|--------------|----------------------|
| 锁类型 | 独占锁 | 读锁（共享）+ 写锁（独占） |
| 重入性 | 支持 | 读锁/写锁分别支持重入 |
| 公平性 | 支持公平/非公平 | 支持公平/非公平 |
| 条件变量 | 支持多个 Condition | 写锁支持 Condition |
| 锁降级 | 不支持 | 支持（写锁 → 读锁，防止看到不一致数据） |
| 锁升级 | 不涉及 | 不支持（读锁 → 写锁会死锁） |

```java
// ReentrantReadWriteLock 使用：缓存实现
public class Cache<K, V> {
    private final Map<K, V> map = new HashMap<>();
    private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    private final Lock r = rwl.readLock();
    private final Lock w = rwl.writeLock();

    public V get(K key) {
        r.lock();
        try {
            return map.get(key);
        } finally {
            r.unlock();
        }
    }

    public V put(K key, V value) {
        w.lock();
        try {
            return map.put(key, value);
        } finally {
            w.unlock();
        }
    }
}
```

#### 2.2.3 StampedLock（JDK 8+）

- **乐观读锁**：不加锁直接读，读完后验证戳记（stamp），若未被写锁修改则成功，否则升级为悲观读锁
- **悲观读锁**：与 `ReadWriteLock.readLock()` 类似
- **写锁**：独占

```java
public class Point {
    private double x, y;
    private final StampedLock sl = new StampedLock();

    // 乐观读
    double distanceFromOrigin() {
        long stamp = sl.tryOptimisticRead();
        double currentX = x, currentY = y;
        if (!sl.validate(stamp)) {           // 检查是否有写操作发生
            stamp = sl.readLock();            // 升级为悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                sl.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }

    void move(double deltaX, double deltaY) {
        long stamp = sl.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            sl.unlockWrite(stamp);
        }
    }
}
```

#### 2.2.4 CountDownLatch / CyclicBarrier / Semaphore

| 工具类 | 作用 | 特点 | 适用场景 |
|--------|------|------|---------|
| **CountDownLatch** | 等待 N 个事件完成 | 计数器只能减到 0，不可重置 | 主线程等待子线程完成 |
| **CyclicBarrier** | N 个线程互相等待 | 计数器可重置（循环使用），可设置到达动作 | 分阶段计算（如矩阵分块） |
| **Semaphore** | 控制同时访问的线程数 | 计数信号量，acquire/release | 限流、资源池 |

```java
// CountDownLatch：等待多个线程完成
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        try {
            doWork();
        } finally {
            latch.countDown();  // 计数减 1
        }
    }).start();
}
latch.await();  // 主线程阻塞，直到计数为 0

// Semaphore：限流（同时只允许 10 个请求）
Semaphore semaphore = new Semaphore(10);
semaphore.acquire();    // 获取许可，无可用则阻塞
semaphore.release();    // 释放许可
```

#### 2.2.5 ThreadLocal 内存泄漏分析

```java
// ThreadLocal 结构：Thread → ThreadLocalMap → Entry[] → Entry(key=ThreadLocal弱引用, value=强引用)
```

**内存泄漏原因**：
1. `ThreadLocalMap` 的 `Entry` 使用 `ThreadLocal` 的**弱引用**作为 key
2. 若 `ThreadLocal` 外部强引用被置为 `null`，key 会被 GC 回收，但 value 仍是强引用
3. 若线程是线程池中的线程（长期存活），value 无法被回收，导致内存泄漏

**解决方案**：
```java
try {
    threadLocal.set(value);
    // 使用...
} finally {
    threadLocal.remove();  // 必须手动移除！
}
```

---

### 2.3 并发集合与线程池

#### 2.3.1 ConcurrentHashMap（JDK 8）

- **数据结构**：数组 + 链表 + 红黑树，与 HashMap 类似
- **并发控制**：
  - 数组位置为空：CAS 插入（`Unsafe.compareAndSwapObject`）
  - 数组位置有值：`synchronized` 锁住链表/红黑树的头节点
  - 统计 size：`LongAdder` 思想，分 baseCount + counterCells，减少 CAS 竞争

```java
// put 流程（简化）
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();                          // 初始化数组
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;                                  // CAS 成功，直接插入
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);                 // 协助扩容
        else {
            synchronized (f) {                          // 锁住链表头节点
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {                      // 链表
                        // 遍历链表，key 相同则覆盖，否则尾插
                    }
                    else if (f instanceof TreeBin) {    // 红黑树
                        // 树节点操作
                    }
                }
            }
        }
        if (binCount != 0) {
            if (binCount >= TREEIFY_THRESHOLD)
                treeifyBin(tab, i);                     // 链表转红黑树
            break;
        }
    }
    addCount(1L, binCount);                             // 更新计数
    return null;
}
```

#### 2.3.2 CopyOnWriteArrayList

- **写时复制**：写操作（add/remove/set）先加锁，复制原数组，修改后替换引用
- **读操作**：无锁，直接读取当前数组引用
- **适用场景**：读多写极少（如配置列表、事件监听器列表）
- **缺点**：写操作开销大（复制整个数组），数据一致性弱（读取的是快照）

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);  // volatile 写，保证可见性
        return true;
    } finally {
        lock.unlock();
    }
}
```

#### 2.3.3 ThreadPoolExecutor 核心机制

**核心参数**：

| 参数 | 含义 | 建议 |
|------|------|------|
| `corePoolSize` | 核心线程数，即使空闲也保留 | CPU 密集型：N+1；IO 密集型：2N |
| `maximumPoolSize` | 最大线程数 | 根据系统资源设置 |
| `keepAliveTime` | 非核心线程空闲存活时间 | 通常 60s |
| `workQueue` | 任务等待队列 | `LinkedBlockingQueue`（无界/有界）、`SynchronousQueue`（直接移交） |
| `threadFactory` | 线程工厂 | 自定义命名、设置守护线程、异常处理器 |
| `handler` | 拒绝策略 | 根据业务选择 |

**四种拒绝策略**：

| 策略 | 行为 | 适用场景 |
|------|------|---------|
| `AbortPolicy`（默认） | 抛 `RejectedExecutionException` | 快速失败 |
| `CallerRunsPolicy` | 由调用线程执行任务 | 降低提交速率，自我保护 |
| `DiscardPolicy` | 静默丢弃任务 | 不重要任务 |
| `DiscardOldestPolicy` | 丢弃队列最老任务，重试提交 | 时效性优先 |

**线程池状态（ctl 高 3 位）**：

```
RUNNING（运行）→ SHUTDOWN（关闭，不接收新任务，处理队列中任务）
  → STOP（停止，中断所有线程）→ TIDYING（整理）→ TERMINATED（终止）
```

```java
// 手动创建线程池（推荐，避免 Executors 的坑）
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    4,                                          // 核心线程数
    8,                                          // 最大线程数
    60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100),             // 有界队列！防止 OOM
    new ThreadFactoryBuilder().setNameFormat("pool-%d").build(),
    new ThreadPoolExecutor.CallerRunsPolicy()   // 拒绝策略
);

// 监控线程池
executor.getPoolSize();        // 当前线程数
executor.getActiveCount();     // 活跃线程数
executor.getQueue().size();    // 队列积压数
executor.getCompletedTaskCount(); // 已完成任务数
```

#### 2.3.4 ForkJoinPool 与 CompletableFuture

**Fork/Join 框架**：
- **工作窃取（Work-Stealing）**：每个线程维护自己的双端队列，空闲线程从其他线程队列尾部窃取任务
- `ForkJoinPool.commonPool()`：线程数 = `Runtime.availableProcessors() - 1`

**CompletableFuture 异步编排**：

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchOrder(orderId))           // 异步获取订单
    .thenApply(order -> enrichOrder(order))           // 转换
    .thenCompose(order -> fetchPayment(order))        // 异步组合
    .exceptionally(ex -> handleError(ex));            // 异常处理

// 多任务组合
CompletableFuture.allOf(future1, future2, future3).join();  // 全部完成
CompletableFuture.anyOf(future1, future2, future3).join();  // 任一完成
```

---

### 2.4 虚拟线程（Project Loom，JDK 21+）

#### 2.4.1 虚拟线程 vs 平台线程

| 特性 | 平台线程（Platform Thread） | 虚拟线程（Virtual Thread） |
|------|--------------------------|------------------------|
| 实现 | 操作系统线程（1:1） | JVM 管理的轻量级线程（M:N） |
| 内存占用 | ~1MB 栈空间 | ~几百字节 |
| 创建成本 | 高（系统调用） | 极低（JVM 内部） |
| 阻塞影响 | 占用 OS 线程 | 自动让出载体线程（Carrier Thread），不阻塞 OS 线程 |
| 调度 | OS 调度器 | JVM 调度器（ForkJoinPool） |

#### 2.4.2 使用方式

```java
// 方式 1：直接启动
Thread.startVirtualThread(() -> {
    System.out.println("Running in virtual thread: " + Thread.currentThread());
});

// 方式 2：使用 ExecutorService（自动关闭）
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 100_000).forEach(i -> {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));  // 阻塞操作不占用 OS 线程
            return i;
        });
    });
} // executor.close() 等待所有任务完成

// 方式 3：Thread.ofVirtual()
Thread vt = Thread.ofVirtual().name("my-vt-").unstarted(runnable);
vt.start();
```

#### 2.4.3 注意事项

- **不要池化虚拟线程**：创建成本极低，每个任务一个虚拟线程即可
- **避免同步块/方法**：`synchronized` 会固定（pin）载体线程，降低吞吐量。改用 `ReentrantLock`
- **适用场景**：高并发 IO 密集型（Web 服务、数据库操作、HTTP 调用）
- **不适用场景**：CPU 密集型计算（无法发挥优势）、大量内存操作

---

## 第三部分：数据结构与集合源码深化

### 3.1 HashMap 深度剖析

#### 3.1.1 哈希扰动与索引计算

```java
// 扰动函数：高 16 位与低 16 位异或，减少哈希冲突
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

// 索引计算：(n - 1) & hash 等价于 hash % n，但位运算更高效
// n 为 2 的幂次，n-1 二进制全 1，与 hash 相与得到 [0, n-1]
```

#### 3.1.2 红黑树化条件

- **链表长度 ≥ 8** 且 **数组长度 ≥ 64**：链表转红黑树
- **红黑树节点数 ≤ 6**：退化为链表（避免频繁在链表与树之间转换）
- 若数组长度 < 64，优先扩容而非树化（resize）

#### 3.1.3 JDK 8 扩容优化

```java
// 扩容时，元素在新数组中的位置：要么在原索引，要么在原索引 + 旧容量
// 原理：hash & oldCap == 0 则位置不变，否则位置 = 原位置 + oldCap
// 无需重新计算 hash，只需判断 hash 的第 n 位（oldCap 的最高位）是 0 还是 1

Node<K,V> loHead = null, loTail = null;   // 低位链表（位置不变）
Node<K,V> hiHead = null, hiTail = null;   // 高位链表（位置 + oldCap）
do {
    if ((e.hash & oldCap) == 0) {         // 判断第 n 位
        if (loTail == null) loHead = e;
        else loTail.next = e;
        loTail = e;
    } else {
        if (hiTail == null) hiHead = e;
        else hiTail.next = e;
        hiTail = e;
    }
} while ((e = e.next) != null);

// 直接挂接到新数组对应位置
if (loTail != null) { loTail.next = null; newTab[j] = loHead; }
if (hiTail != null) { hiTail.next = null; newTab[j + oldCap] = hiHead; }
```

#### 3.1.4 多线程问题

- **JDK 7**：并发扩容时链表头插法可能导致环形链表，get 操作死循环
- **JDK 8**：改为尾插法，不会死循环，但会丢数据（两个线程同时 put 可能覆盖）
- **解决方案**：使用 `ConcurrentHashMap` 或 `Collections.synchronizedMap`

### 3.2 LinkedHashMap：LRU 缓存实现

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        // accessOrder = true：按访问顺序排序（最近访问的在尾部）
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当 size > capacity 时，移除最老的元素（头部）
        return size() > capacity;
    }
}

// 使用
LRUCache<String, String> cache = new LRUCache<>(3);
cache.put("a", "1");
cache.put("b", "2");
cache.put("c", "3");
cache.get("a");          // 访问 a，a 移到尾部
cache.put("d", "4");     // 移除最老的 b，加入 d
System.out.println(cache.keySet());  // [c, a, d]
```

### 3.3 TreeMap 与红黑树

- **红黑树规则**：节点红或黑；根和叶子（NIL）为黑；红节点子节点必黑；任意节点到叶子路径黑节点数相同
- **TreeMap**：基于红黑树实现，支持 `Comparable` 自然排序或 `Comparator` 定制排序
- **ConcurrentSkipListMap**：线程安全的排序 Map，基于跳表（Skip List），并发性能优于 `Collections.synchronizedSortedMap`

### 3.4 Queue 实现对比

| 实现类 | 底层结构 | 特性 | 推荐使用场景 |
|--------|---------|------|------------|
| **ArrayDeque** | 循环数组 | 无容量限制、无节点开销、非线程安全 | 栈（push/pop）、队列（offer/poll）首选 |
| **LinkedList** | 双向链表 | 节点对象开销大 | 需要频繁中间插入删除 |
| **PriorityQueue** | 小顶堆 | 按优先级出队、非线程安全 | TopK、任务调度 |
| **ArrayBlockingQueue** | 循环数组 + 锁 | 有界、公平可选 | 生产者-消费者 |
| **LinkedBlockingQueue** | 链表 + 双锁（读写分离） | 可选有界/无界 | 高并发生产者-消费者 |
| **SynchronousQueue** | 无存储空间 | 直接移交、公平/非公平 | 线程池直接移交策略 |

```java
// ArrayDeque 作为栈（比 Stack 快，比 LinkedList 省内存）
Deque<String> stack = new ArrayDeque<>();
stack.push("first");
stack.push("second");
String top = stack.pop();  // "second"

// ArrayDeque 作为队列
Deque<String> queue = new ArrayDeque<>();
queue.offer("task1");
queue.offer("task2");
String task = queue.poll();  // "task1"
```

---

## 第四部分：IO、NIO 与网络编程

### 4.1 BIO / NIO / AIO 对比

| 模型 | 阻塞 | 同步 | 特点 | 适用场景 |
|------|------|------|------|---------|
| **BIO** | 阻塞 | 同步 | 一个连接一个线程 | 连接数少、逻辑简单 |
| **NIO** | 非阻塞 | 同步 | 单线程管理多连接（Selector） | 高并发、连接数多 |
| **AIO** | 非阻塞 | 异步 | 操作系统回调通知完成 | 少量连接、长连接（Windows IOCP） |

**关键概念区分**：
- **阻塞/非阻塞**：调用方是否等待结果返回（关注的是程序状态）
- **同步/异步**：被调用方是否主动通知结果（关注的是消息通信机制）

### 4.2 NIO 三大组件

#### 4.2.1 Buffer

```java
// Buffer 核心属性
// position：下一个读写位置
// limit：最大可读写位置
// capacity：容量
// mark：标记位置，可 reset 回到此处

ByteBuffer buffer = ByteBuffer.allocate(1024);  // 堆内内存
ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);  // 堆外内存（零拷贝）

buffer.put("hello".getBytes());  // 写模式：position 移动
buffer.flip();                    // 切换读模式：limit = position, position = 0
while (buffer.hasRemaining()) {
    System.out.print((char) buffer.get());
}
buffer.clear();                   // 清空：position = 0, limit = capacity
```

#### 4.2.2 Channel

- `FileChannel`：文件读写，支持 `transferTo`/`transferFrom` 零拷贝
- `SocketChannel`：TCP 客户端
- `ServerSocketChannel`：TCP 服务端
- `DatagramChannel`：UDP

#### 4.2.3 Selector

```java
// NIO 服务端示例
Selector selector = Selector.open();
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.bind(new InetSocketAddress(8080));
serverChannel.configureBlocking(false);
serverChannel.register(selector, SelectionKey.OP_ACCEPT);

while (true) {
    selector.select();  // 阻塞等待就绪事件
    Iterator<SelectionKey> keys = selector.selectedKeys().iterator();
    while (keys.hasNext()) {
        SelectionKey key = keys.next();
        keys.remove();

        if (key.isAcceptable()) {
            SocketChannel client = serverChannel.accept();
            client.configureBlocking(false);
            client.register(selector, SelectionKey.OP_READ);
        } else if (key.isReadable()) {
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            client.read(buffer);
            // 处理数据...
        }
    }
}
```

### 4.3 零拷贝技术

| 技术 | 原理 | 数据拷贝次数 | 上下文切换次数 |
|------|------|-----------|------------|
| **传统 IO** | 磁盘 → 内核缓冲区 → 用户缓冲区 → Socket 缓冲区 | 4 | 4 |
| **mmap** | 文件映射到用户空间，直接读写 | 3 | 4 |
| **sendfile** | 内核直接传递文件数据到 Socket | 2（DMA 拷贝） | 2 |
| **sendfile + DMA gather** | 无 CPU 拷贝 | 0（纯 DMA） | 2 |

```java
// FileChannel.transferTo() 使用 sendfile
FileChannel source = new FileInputStream("file.zip").getChannel();
SocketChannel dest = SocketChannel.open(new InetSocketAddress("host", 8080));
source.transferTo(0, source.size(), dest);  // 零拷贝发送文件
```

### 4.4 Netty 核心架构

```
EventLoopGroup（BossGroup + WorkerGroup）
    └── EventLoop（单线程执行器）
            └── Selector + TaskQueue
                  └── ChannelPipeline
                        └── ChannelHandlerContext
                              └── ChannelHandler（入站/出站）
```

- **Reactor 模型**：Netty 使用主从 Reactor 多线程模型
- **ByteBuf**：替代 NIO ByteBuffer，支持池化（减少 GC）、引用计数（自动释放）、复合缓冲区
- **Pipeline**：责任链模式，事件在 Handler 链中传播

---

## 第五部分：反射、动态代理与注解

### 5.1 反射性能优化

| 方式 | 性能 | 特点 |
|------|------|------|
| **原生反射（Method.invoke）** | 慢 | 每次检查访问权限 |
| **setAccessible(true)** | 较快 | 关闭访问检查，但模块化下可能受限 |
| **MethodHandle** | 接近直接调用 | JDK 7+，支持 currying、变长参数 |
| **LambdaMetafactory** | 接近直接调用 | 生成动态调用点 |
| **字节码生成（ASM/Javassist）** | 直接调用速度 | 生成新类，启动成本高 |

```java
// MethodHandle 示例
MethodHandles.Lookup lookup = MethodHandles.lookup();
MethodHandle mh = lookup.findVirtual(String.class, "length", MethodType.methodType(int.class));
int len = (int) mh.invoke("hello");  // 5
```

### 5.2 动态代理

| 特性 | JDK 动态代理 | CGLIB |
|------|------------|-------|
| 实现方式 | 接口 | 继承（生成子类） |
| 依赖 | 内置 | 第三方库 |
| final 方法 | 支持（接口无此限制） | 不能代理 |
| 性能 | 反射调用 | FastClass 机制（方法索引，避免反射） |
| 生成类 | `com.sun.proxy.$ProxyN` | `被代理类$$EnhancerByCGLIB$$xxx` |

```java
// JDK 动态代理
InvocationHandler handler = (proxy, method, args) -> {
    System.out.println("before");
    Object result = method.invoke(target, args);
    System.out.println("after");
    return result;
};
Service proxy = (Service) Proxy.newProxyInstance(
    Service.class.getClassLoader(),
    new Class[]{Service.class},
    handler
);

// CGLIB 动态代理
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(Target.class);
enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
    System.out.println("before");
    Object result = proxy.invokeSuper(obj, args);
    System.out.println("after");
    return result;
});
Target proxy = (Target) enhancer.create();
```

### 5.3 注解处理器（APT）

```java
// 自定义注解
@Retention(RetentionPolicy.SOURCE)  // 编译期处理，不保留到运行时
@Target(ElementType.TYPE)
public @interface AutoGenerate {
    String value();
}

// 注解处理器（编译时生成代码）
@SupportedAnnotationTypes("com.example.AutoGenerate")
@SupportedSourceVersion(SourceVersion.RELEASE_17)
public class AutoGenerateProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        for (Element element : roundEnv.getElementsAnnotatedWith(AutoGenerate.class)) {
            // 使用 JavaPoet 生成代码
            TypeSpec generatedClass = TypeSpec.classBuilder("Generated_" + element.getSimpleName())
                .addModifiers(Modifier.PUBLIC)
                .addMethod(MethodSpec.methodBuilder("hello")
                    .addModifiers(Modifier.PUBLIC)
                    .returns(void.class)
                    .addStatement("$T.out.println($S)", System.class, "Hello")
                    .build())
                .build();

            JavaFile javaFile = JavaFile.builder("com.example.generated", generatedClass).build();
            try {
                javaFile.writeTo(processingEnv.getFiler());
            } catch (IOException e) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, e.getMessage());
            }
        }
        return true;
    }
}
```

---

## 第六部分：泛型与类型系统进阶

### 6.1 类型擦除真相

```java
// 编译前
public class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

// 编译后（擦除为边界类型或 Object）
public class Box {
    private Object value;
    public void set(Object value) { this.value = value; }
    public Object get() { return value; }
}

// 桥方法维持多态
public class IntegerBox extends Box<Integer> {
    public void set(Integer value) { super.set(value); }
    // 编译器生成桥方法：
    public void set(Object value) { set((Integer) value); }
}
```

### 6.2 PECS 原则完整推导

```java
// Producer Extends：只能读取 T 及其子类，安全；不能写入（不知道具体子类型）
List<? extends Number> producer = new ArrayList<Integer>();
Number n = producer.get(0);     // ✓ 安全：取出的至少是 Number
// producer.add(1);             // ✗ 编译错误：可能是 List<Double>

// Consumer Super：只能写入 T 及其子类，安全；读取只能作为 Object
List<? super Integer> consumer = new ArrayList<Number>();
consumer.add(1);                // ✓ 安全：Integer 及其子类都可放入
// Integer i = consumer.get(0); // ✗ 编译错误：可能是 List<Object>
Object obj = consumer.get(0);   // ✓ 只能作为 Object 读取

// 应用：copy 方法
public static <T> void copy(List<? extends T> src, List<? super T> dest) {
    for (T item : src) {
        dest.add(item);
    }
}
```

### 6.3 Java 新类型特性（JDK 14+）

#### 6.3.1 var 类型推断

```java
// 编译时确定类型，不是动态类型
var list = new ArrayList<String>();  // 推断为 ArrayList<String>
var stream = list.stream();           // 推断为 Stream<String>

// 限制：
// 1. 必须初始化
// 2. 不能用于成员变量、方法参数、返回类型
// 3. 不能用于 null 初始化（无法推断类型）
```

#### 6.3.2 记录类（record，JDK 16+）

```java
// 编译器自动生成：构造函数、getter、equals、hashCode、toString
public record Point(int x, int y) {}

// 等价于：
public final class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    public int x() { return x; }  // 注意：不是 getX()
    public int y() { return y; }
    // equals, hashCode, toString...
}

// 可自定义构造函数和方法
public record Range(int start, int end) {
    public Range {
        if (start > end) throw new IllegalArgumentException();
    }
    public int length() { return end - start; }
}
```

#### 6.3.3 密封类（sealed class，JDK 17+）

```java
// 限制继承层次，所有子类必须在同一模块/包中显式声明
public abstract sealed class Shape 
    permits Circle, Rectangle, Square { ... }

public final class Circle extends Shape { ... }      // final：不可再继承
public non-sealed class Rectangle extends Shape { ... } // 开放继承
public sealed class Square extends Shape permits BigSquare { ... }
```

#### 6.3.4 模式匹配

```java
// instanceof 模式匹配（JDK 16+）
if (obj instanceof String s) {  // 自动绑定变量 s
    System.out.println(s.length());  // 无需强制转换
}

// switch 表达式 + 模式匹配（JDK 17+ preview，JDK 21 正式）
String result = switch (obj) {
    case Integer i -> "Integer: " + i;
    case String s when s.length() > 5 -> "Long string: " + s;  // 守卫条件
    case String s -> "Short string: " + s;
    case null -> "null";
    default -> "Unknown";
};
```

---

## 第七部分：函数式编程与 Stream 性能

### 7.1 Lambda 底层原理

- **不是匿名内部类**：不会生成 `外部类$N.class` 文件
- **invokedynamic**：编译时生成 `invokedynamic` 指令，运行时通过 `LambdaMetafactory.metafactory` 动态生成实现类
- **性能**：首次调用有开销（生成类），后续调用接近直接调用

### 7.2 Stream 运行机制

#### 7.2.1 惰性求值与 Sink 链

```java
list.stream()
    .filter(e -> e > 10)      // 中间操作：创建 Sink 节点
    .map(e -> e * 2)          // 中间操作：包装前一个 Sink
    .forEach(System.out::println);  // 终端操作：触发执行

// 执行时形成 Sink 链：
// forEach Sink → map Sink → filter Sink → 数据源
// 数据逐个流过，而非每个操作遍历全量
```

#### 7.2.2 短路操作

```java
// anyMatch：找到第一个匹配即停止
boolean hasBig = list.stream().anyMatch(e -> e > 100);  // 短路

// limit：取前 N 个即停止
list.stream().filter(...).limit(10).collect(...);  // 找到 10 个后停止
```

#### 7.2.3 并行流陷阱

```java
// ❌ 错误：使用共享的线程不安全对象
List<Integer> result = new ArrayList<>();
IntStream.range(0, 1000).parallel().forEach(result::add);  // 可能丢数据

// ✓ 正确：使用 collect 合并结果
List<Integer> result = IntStream.range(0, 1000).parallel()
    .boxed()
    .collect(Collectors.toList());

// ❌ 错误：数据源不可高效拆分（如 Stream.iterate、LinkedList、Stream.of）
// ✓ 正确：使用 IntStream.range、Arrays.stream、IntStream.generate
```

**并行流适用条件**：
1. 数据源可高效拆分（数组、`IntStream.range`、`IntStream.generate`）
2. 无状态、无副作用
3. 计算密集型（否则线程切换开销 > 并行收益）
4. 数据量足够大（通常 > 10,000）

### 7.3 自定义 Collector

```java
// 计算平均值
Collector<Integer, ?, Double> averaging = Collector.of(
    () -> new long[2],                    // supplier：创建累加器 [sum, count]
    (acc, val) -> { acc[0] += val; acc[1]++; },  // accumulator
    (acc1, acc2) -> {                     // combiner（并行时合并）
        acc1[0] += acc2[0];
        acc1[1] += acc2[1];
        return acc1;
    },
    acc -> acc[1] == 0 ? 0.0 : (double) acc[0] / acc[1]  // finisher
);

Double avg = Stream.of(1, 2, 3, 4, 5).collect(averaging);  // 3.0
```

---

## 第八部分：设计模式在 Java 源码中的应用

### 8.1 创建型模式

| 模式 | JDK 源码体现 | 说明 |
|------|------------|------|
| **单例（Singleton）** | `Runtime.getRuntime()`、`Desktop.getDesktop()` | 枚举、静态内部类实现 |
| **工厂（Factory）** | `ThreadFactory`、`Executors.defaultThreadFactory()` | 统一创建线程 |
| **Builder** | `StringBuilder`、`StringBuffer`、`Calendar.Builder` | 链式构建 |

### 8.2 结构型模式

| 模式 | JDK 源码体现 | 说明 |
|------|------------|------|
| **适配器（Adapter）** | `InputStreamReader`（字节流 → 字符流）、`Arrays.asList()` | 接口转换 |
| **装饰器（Decorator）** | `BufferedInputStream` 包装 `FileInputStream`、`Collections.synchronizedXxx()` | 动态添加功能 |
| **代理（Proxy）** | `java.lang.reflect.Proxy`、`RMI` | 控制访问 |

### 8.3 行为型模式

| 模式 | JDK 源码体现 | 说明 |
|------|------------|------|
| **策略（Strategy）** | `Comparator`、`ThreadPoolExecutor` 的拒绝策略 | 算法族可互换 |
| **模板方法（Template）** | `AbstractList`、`AbstractMap`、`AQS` | 骨架算法，子类实现细节 |
| **观察者（Observer）** | `PropertyChangeListener`、`EventListener` | 事件订阅发布 |
| **迭代器（Iterator）** | 所有集合的 `Iterator` 实现 | 统一遍历接口 |
| **责任链（Chain）** | `java.util.logging.Logger` 的父日志器传递 | 请求沿链传递 |

---

## 第九部分：性能调优与工具链

### 9.1 JVM 调优参数

```bash
# 堆内存设置
-Xms4g -Xmx4g                    # 初始/最大堆内存，建议设为相同值避免动态调整
-Xmn1g                           # 新生代大小
-XX:MetaspaceSize=256m           # 元空间初始大小
-XX:MaxMetaspaceSize=512m        # 元空间最大大小
-XX:MaxDirectMemorySize=1g       # 直接内存上限（NIO 使用）

# GC 选择
-XX:+UseG1GC                     # 使用 G1
-XX:MaxGCPauseMillis=200         # 目标最大停顿
-XX:+UseZGC                      # JDK 15+ 使用 ZGC

# 日志与监控
-XX:+PrintGCDetails
-Xlog:gc*:file=gc.log:time,uptime,level,tags
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dump

# JIT 编译优化
-XX:+UseStringDeduplication      # G1 字符串去重（JDK 8u20+）
-XX:+AlwaysPreTouch              # 启动时预分配堆内存，减少运行时停顿
```

### 9.2 常用诊断命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `jps` | 查看 Java 进程 | `jps -lvm` |
| `jstat` | GC 统计 | `jstat -gcutil <pid> 1000 10` |
| `jstack` | 线程堆栈 | `jstack -l <pid> > thread.dump` |
| `jmap` | 内存映射 | `jmap -dump:format=b,file=heap.hprof <pid>` |
| `jcmd` | 综合诊断 | `jcmd <pid> GC.heap_dump` |
| `jfr` | 飞行记录器 | `jcmd <pid> JFR.start` |

### 9.3 Arthas 常用命令

```bash
#  attach 到进程
java -jar arthas-boot.jar

#  常用命令
watch com.example.Service getOrder '{params,returnObj,throwExp}' -x 2  # 观察方法入参和返回值
trace com.example.Service getOrder '#cost>100'                         # 追踪耗时 > 100ms 的调用
thread -b                                                            # 找死锁
thread -n 3                                                          # 查看 CPU 占用最高的 3 个线程
jad com.example.Service                                              # 反编译类
ognl '@com.example.Config@getInstance().getValue()'                  # 执行 OGNL 表达式
```

### 9.4 单元测试与代码质量

```java
// JUnit 5 示例
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    @DisplayName("should create order successfully")
    void createOrder() {
        when(orderRepository.save(any())).thenReturn(new Order(1L, "ITEM-001"));

        Order result = orderService.create("ITEM-001");

        assertEquals("ITEM-001", result.getItemId());
        verify(orderRepository, times(1)).save(any());
    }

    @ParameterizedTest
    @ValueSource(strings = {"", " ", "   "})
    @DisplayName("should reject blank item id")
    void rejectBlankItemId(String itemId) {
        assertThrows(IllegalArgumentException.class, () -> orderService.create(itemId));
    }
}
```

**静态代码分析工具**：
- **SpotBugs**：基于字节码的 Bug 模式检测（FindBugs 继任者）
- **PMD**：基于源码的规则检测（空 catch、未使用变量等）
- **Checkstyle**：代码风格检查
- **SonarQube**：综合代码质量管理平台

---

## 附录

### 附录 A：高频面试题索引

| 题目 | 关联章节 |
|------|---------|
| JVM 内存区域有哪些？对象头包含什么？ | 1.1 |
| G1 和 CMS 的区别？ZGC 的优势？ | 1.2.4 |
| 双亲委派模型是什么？如何打破？ | 1.3.2 |
| volatile 如何保证可见性？能替代 synchronized 吗？ | 2.1.3 |
| synchronized 锁升级过程？ | 2.1.4 |
| AQS 核心设计？ReentrantLock 如何实现可重入？ | 2.2.1 |
| ThreadLocal 内存泄漏原因？ | 2.2.5 |
| ConcurrentHashMap JDK 7 vs JDK 8 实现差异？ | 2.3.1 |
| 线程池核心参数？拒绝策略？ | 2.3.3 |
| 虚拟线程是什么？适用场景？ | 2.4 |
| HashMap 扩容机制？JDK 8 优化？ | 3.1.3 |
| 如何实现 LRU 缓存？ | 3.2 |
| NIO 三大组件？零拷贝？ | 4.2 / 4.3 |
| JDK 动态代理 vs CGLIB？ | 5.2 |
| 泛型类型擦除？PECS 原则？ | 6.1 / 6.2 |
| Lambda 底层实现？ | 7.1 |
| Stream 并行流陷阱？ | 7.2.3 |

### 附录 B：JVM 参数速查表

| 参数 | 说明 |
|------|------|
| `-Xms<size>` | 初始堆大小 |
| `-Xmx<size>` | 最大堆大小 |
| `-Xmn<size>` | 新生代大小 |
| `-Xss<size>` | 线程栈大小 |
| `-XX:NewRatio=N` | 老年代/新生代比例 |
| `-XX:SurvivorRatio=N` | Eden/Survivor 比例 |
| `-XX:+UseG1GC` | 启用 G1 |
| `-XX:+UseZGC` | 启用 ZGC |
| `-XX:MaxGCPauseMillis=N` | 目标最大 GC 停顿 |
| `-XX:+HeapDumpOnOutOfMemoryError` | OOM 时生成堆转储 |

### 附录 C：常用性能分析命令速查

```bash
# CPU 分析
jstack <pid> | grep "java.lang.Thread.State:" | sort | uniq -c  # 线程状态统计
jstack <pid> | grep -A 1 "java.lang.Thread.State: RUNNABLE"    # 查看 RUNNABLE 线程

# 内存分析
jmap -histo <pid> | head -n 30    # 对象数量统计
jmap -dump:format=b,file=... <pid> # 生成堆转储

# GC 分析
jstat -gcutil <pid> 1000          # 每秒打印 GC 统计
jstat -gccause <pid> 1000         # 包含 GC 原因
```

### 附录 D：Java 版本演进图谱

| 版本 | 发布年份 | 关键特性 |
|------|---------|---------|
| **Java 8** | 2014 | Lambda、Stream API、新时间 API、接口默认方法、方法引用 |
| **Java 9** | 2017 | 模块化（JPMS）、JShell、私有接口方法、改进 Stream |
| **Java 11** | 2018 | LTS、HTTP Client、ZGC（实验）、Flight Recorder开源、var（局部变量） |
| **Java 14** | 2020 | switch 表达式（正式）、record（预览）、 helpful NPE |
| **Java 17** | 2021 | LTS、sealed class（正式）、pattern matching for switch（预览）、ZGC 正式 |
| **Java 21** | 2023 | LTS、虚拟线程（正式）、sequenced collections、pattern matching for switch（正式）、record patterns |

---

> **💡 学习建议**：进阶知识体系庞大，建议按「JVM → 并发 → 集合源码 → NIO → 反射代理 → 泛型 → Stream → 设计模式 → 调优工具」的顺序逐步深入，每个主题配合源码阅读和实际项目实践，才能真正内化。面试前重点复习附录 A 的高频题。
