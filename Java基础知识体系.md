# Java 基础知识体系

## 1. 类与对象

### 1.1 基本概念
- **类（Class）**：对象的模板，定义了对象的属性和行为。使用 `class` 关键字声明。
- **对象（Object）**：类的实例，是内存中真实存在的数据。通过 `new` 关键字调用构造方法创建，在堆内存中分配空间，返回对象引用。
- **构造方法（Constructor）**：用于创建对象时初始化成员变量。方法名与类名相同，无返回类型。

### 1.2 深入理解
- **内存模型**：对象存储在堆（Heap）中，引用变量存储在栈（Stack）中。
- **this 关键字**：指向当前对象的引用，用于区分成员变量与局部变量。`this()` 可调用本类其他构造方法，但必须放在第一行。
- **构造方法重载**：一个类可以有多个构造方法，参数列表不同。如果没有显式定义，编译器会提供默认无参构造方法；一旦定义了有参构造，默认构造消失，建议手动补充。
- **对象初始化顺序**：静态变量/静态块（类加载时）→ 实例变量/非静态块 → 构造方法。注意不要在构造方法中调用可被重写的方法，此时子类可能还未初始化完毕，会导致不可预期错误。
- **PS**：如果 getter 返回可变对象（如 `Date`），应返回其防御性副本，防止外部修改破坏封装。

### 1.3 代码示例
```java
public class Person {
    // 成员变量（属性）
    private String name;
    private int age;

    // 无参构造
    public Person() {}

    // 有参构造（重载）
    public Person(String name, int age) {
        this.name = name;  // this 区分成员变量
        this.age = age;
    }

    // 成员方法（行为）
    public void sayHello() {
        System.out.println("Hello");
    }

    public void introduce() {
        System.out.println("我叫" + this.name + "，今年" + this.age + "岁");
    }
}

// 创建对象
Person p = new Person("张三", 20);   // p 存储在栈中，指向堆中的对象
p.sayHello();
p.introduce();
```

---

## 2. 三大特性

### 2.1 封装

#### 2.1.1 基本概念
将对象的属性和实现细节隐藏，仅对外提供公共的访问方式。通过 `private` 修饰符隐藏属性，通过 getter/setter 方法暴露访问接口。

#### 2.1.2 深入理解
- **封装的意义**：
  - 保护数据安全性（防止外部直接修改）
  - 控制数据的合法性（在 setter 中做校验）
  - 降低耦合度，提高代码可维护性
- **JavaBean 规范**：私有属性 + 无参构造 + getter/setter 方法
- **不可变对象**：只提供 getter，不提供 setter（如 String、Integer）
- **PS**：如果 getter 返回可变对象，应返回其防御性副本，防止外部修改破坏封装。

#### 2.1.3 代码示例
```java
public class Student {
    private String name;
    private int age;        // 私有属性，外部不可直接访问

    // getter：读取属性
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // setter：设置属性（可加入校验逻辑）
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("年龄不合法");
        }
        this.age = age;
    }
}
```

### 2.2 继承

#### 2.2.1 基本概念
子类（SubClass）继承父类（SuperClass）的属性和方法。使用 `extends` 关键字实现。Java 只支持单继承（一个子类只能有一个直接父类）。

#### 2.2.2 深入理解
- **继承的本质**：代码复用 + 建立类之间的层次关系
- **方法重写（Override）**：子类重新实现父类的方法，"两同两小一大"：方法名和参数列表相同；返回值类型为子类型或相同；访问权限不能更小；抛出的异常不能更大。
- **super 关键字**：
  - `super()`：调用父类构造方法（必须在子类构造方法第一行）
  - `super.method()`：调用父类的方法
- **继承中的构造方法**：子类构造方法默认先调用父类无参构造 `super()`
- **继承的弊端**：耦合度高，滥用继承会导致类层次过深。遵循"组合优于继承"原则，父类变化容易殃及子类。
- **PS**：构造方法链，new 子类时必先调用父类构造。

#### 2.2.3 代码示例
```java
// 父类
public class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void speak() {
        System.out.println("动物发出声音");
    }
}

// 子类
public class Dog extends Animal {
    public Dog(String name) {
        super(name);  // 调用父类构造方法
    }

    @Override
    public void speak() {  // 方法重写
        System.out.println(name + "说：汪汪汪");
    }

    public void watchDoor() {  // 子类特有方法
        System.out.println(name + "在看门");
    }
}
```

### 2.3 多态

#### 2.3.1 基本概念
同一个行为具有多个不同表现形式或形态的能力。三要素：继承/实现、方法重写、父类引用指向子类对象。

#### 2.3.2 深入理解
- **编译时类型 vs 运行时类型**：
  - 编译时类型：声明时的类型（如 `Animal a`）
  - 运行时类型：实际创建的对象类型（如 `new Dog()`）
- **动态绑定**：方法调用在运行时根据实际对象类型确定执行哪个方法（仅对实例方法有效）
- **向上转型（Upcasting）**：`Animal a = new Dog();` — 自动转换，安全
- **向下转型（Downcasting）**：`Dog d = (Dog) a;` — 强制转换，可能抛出 `ClassCastException`
- **instanceof 关键字**：用于判断对象是否是某个类的实例，避免转型异常
- **PS**：
  - 静态方法、字段不参与多态（字段看引用类型，称"字段隐藏"）
  - 向上转型：子转父，自动安全；向下转型需用 `instanceof` 判断

#### 2.3.3 代码示例
```java
Animal a1 = new Dog("旺财");    // 向上转型
Animal a2 = new Cat("咪咪");    // 向上转型

a1.speak();  // 输出：旺财说：汪汪汪（运行时调用 Dog 的 speak）
a2.speak();  // 输出：咪咪说：喵喵喵（运行时调用 Cat 的 speak）

// 向下转型（需先判断）
if (a1 instanceof Dog) {
    Dog dog = (Dog) a1;
    dog.watchDoor();  // 调用 Dog 特有方法
}
```

#### 2.3.4 多态的应用场景
- 方法参数：`feed(Animal animal)` 可以传入 Dog、Cat 等任意 Animal 子类
- 集合容器：`List<Animal>` 可以存放各种 Animal 子类对象
- 设计模式：工厂模式、策略模式等都依赖多态

---

## 3. 抽象与接口

### 3.1 抽象类（Abstract Class）

#### 3.1.1 基本概念
用 `abstract` 修饰的类，不能被实例化。可以包含抽象方法（没有方法体）和具体方法。子类必须实现所有抽象方法，否则子类也必须声明为抽象类。

#### 3.1.2 深入理解
- **抽象类的意义**：为子类提供通用的模板，提取共性
- **抽象方法**：只有声明没有实现，强制子类重写
- **与普通类的区别**：
  - 抽象类可以有构造方法（用于子类调用）
  - 抽象类可以有成员变量、普通方法
  - 抽象类可以没有抽象方法（但通常有）
- **适用场景**：有共同属性和部分共同实现的场景
- **PS：模板方法模式**：抽象类中定义算法骨架，将某些步骤延迟到子类实现。

#### 3.1.3 代码示例
```java
public abstract class Shape {
    protected String color;

    public Shape(String color) {
        this.color = color;
    }

    // 抽象方法：子类必须实现
    public abstract double getArea();

    // 普通方法
    public void printColor() {
        System.out.println("颜色：" + color);
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
}
```

### 3.2 接口（Interface）

#### 3.2.1 基本概念
用 `interface` 定义，是完全抽象的规范。
- JDK 8 之前：只能有抽象方法和常量
- JDK 8+：可以有默认方法（`default`）和静态方法
- JDK 9+：可以有私有方法

#### 3.2.2 深入理解
- **接口的本质**：定义一组行为规范，与实现分离
- **多实现**：一个类可以实现多个接口，弥补了 Java 单继承的不足
- **接口与抽象类的区别**：

| 特性 | 抽象类 | 接口 |
|------|--------|------|
| 关键字 | `abstract class` | `interface` |
| 继承/实现 | extends（单继承） | implements（多实现） |
| 构造方法 | 有 | 无 |
| 成员变量 | 可以有各种类型 | 只能是 `public static final` 常量 |
| 方法 | 可以有具体方法 | JDK 8 前只能有抽象方法 |
| 设计目的 | "是什么"（is-a） | "能做什么"（can-do） |

- **函数式接口**：只有一个抽象方法的接口，可用 Lambda 表达式实现（JDK 8+），可用 `@FunctionalInterface` 校验
- **抽象类 vs 接口**：抽象类偏"is-a"关系，复用代码；接口偏"has-a/can-do"能力，解耦。优先使用接口。

#### 3.2.3 代码示例
```java
public interface Flyable {
    // 常量（默认 public static final）
    int MAX_HEIGHT = 10000;

    // 抽象方法
    void fly();

    // 默认方法（JDK 8+）
    default void land() {
        System.out.println("安全着陆");
    }

    // 静态方法（JDK 8+）
    static void checkWeather() {
        System.out.println("检查天气状况");
    }
}

public class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("鸟儿展翅飞翔");
    }
}
```

---

## 4. 常用关键字

### 4.1 static
- **静态变量**：属于类，所有对象共享一份
- **静态方法**：属于类，可以直接通过类名调用，不能访问非静态成员，不能使用 `this/super`
- **静态代码块**：类加载时执行，只执行一次，用于初始化静态变量
- **静态内部类**：不依赖外部类实例，是独立的

```java
public class StaticDemo {
    static int count = 0;           // 静态变量

    static {                        // 静态代码块
        System.out.println("类加载时执行");
    }

    public static void show() {     // 静态方法
        System.out.println("count=" + count);
    }
}
```

### 4.2 final
- **final 变量**：常量，赋值后不可修改。基本类型值不可变，引用类型引用地址不可变（对象内容可变）
- **final 方法**：不能被子类重写
- **final 类**：不能被继承（如 String、System）
- **final 引用**：引用不可变，但对象内容可变
- **PS**：final 修饰域的内存语义（对象安全发布）

### 4.3 this
- 指代当前对象
- `this()`：调用本类其他构造方法（必须第一行）
- `this.field`：区分成员变量与局部变量

### 4.4 super
- 指代父类对象
- `super()`：调用父类构造方法（必须第一行）
- `super.method()`：调用父类方法
- **PS**：`this()` 和 `super()` 调用构造方法都必须在第一行，故不能同时显式出现

---

## 5. 包与访问权限

### 5.1 包（Package）
- 用于组织和管理类，避免命名冲突
- 命名规范：公司域名倒写 + 项目名 + 模块名（如 `com.example.project.utils`）
- `import` 导入其他包的类；`java.lang` 包下的类自动导入
- 静态导入可导入静态成员，慎用

### 5.2 四种访问权限

| 修饰符 | 同类 | 同包 | 子类 | 任何地方 |
|--------|------|------|------|----------|
| public | ✓ | ✓ | ✓ | ✓ |
| protected | ✓ | ✓ | ✓ | ✗ |
| default（无修饰符） | ✓ | ✓ | ✗ | ✗ |
| private | ✓ | ✗ | ✗ | ✗ |

- **PS**：`protected` 是继承访问权限，并非单纯子类可见。子类通过父类引用访问 `protected` 成员仅限同包；不同包子类只能通过子类引用访问继承来的 `protected` 成员。

---

## 6. 内部类

### 6.1 成员内部类
- 依赖外部类实例，可以访问外部类的所有成员
- 隐式持有外部类引用 `Outer.this`
- 创建方式：`Outer.Inner inner = new Outer().new Inner();`
- 编译为 `Outer$Inner.class`
- **PS**：持有外部引用容易造成内存泄漏（如内部类在异步回调中存活更长）

### 6.2 静态内部类
- 用 `static` 修饰，不依赖外部类实例，不持有外部引用
- 只能访问外部类的静态成员
- 常用场景：Builder 模式
- 最常用的工具类嵌套方式

### 6.3 局部内部类
- 定义在方法内部，作用域仅限于该方法

### 6.4 匿名内部类 
- 没有名字的内部类，常用于实现接口或继承抽象类
- 是 Lambda 表达式的前身
- 编译后生成 `外部类$数字.class` 文件
- **PS**：必须访问 `final` 或 effectively final 的局部变量，因为匿名类会将变量拷贝一份副本，为保证副本与原变量值一致，编译器强制不可变

```java
// 匿名内部类：实现接口
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("匿名内部类实现 Runnable");
    }
};

// JDK 8+ Lambda 简化
Runnable runnable2 = () -> System.out.println("Lambda 表达式");

// 匿名内部类：继承抽象类
Thread t = new Thread() {
    @Override
    public void run() {
        System.out.println("自定义线程");
    }
};
```

---

## 7. 字符串处理

### 7.1 三个核心类对比

| 特性 | String | StringBuilder | StringBuffer |
|------|--------|---------------|--------------|
| 可变性 | 不可变 | 可变 | 可变 |
| 线程安全 | 安全（不可变即安全） | 不安全 | 安全（synchronized） |
| 性能 | 拼接时产生大量对象 | 最高 | 较高 |
| 使用场景 | 字符串常量 | 单线程字符串拼接 | 多线程字符串拼接 |

### 7.2 String 深入理解
- **不可变性**：String 的值存储在 `byte[]` 中（JDK 9+，之前为 `char[]`），用 `final` 修饰，修改会创建新对象
- **字符串常量池**：字面量创建的字符串存储在常量池中，相同字符串共享引用
- `==` vs `equals()`：`==` 比较地址，`equals()` 比较内容
- `intern()` 方法：将字符串放入常量池并返回引用。JDK 7+ 常量池移至堆中，`intern()` 不再复制对象，仅在池中记录引用，若池中已有则返回已有引用
- **PS**：
  - `String s = "abc";` 会在池中创建；`new String("abc")` 可能创建两个对象（池中一个 + 堆中一个）
  - 循环拼接应避免用 `+`，因为它会被编译为 `new StringBuilder().append()`，但仍可能生成大量临时对象

```java
String s1 = "hello";           // 常量池
String s2 = new String("hello"); // 堆中创建新对象
String s3 = "hello";           // 常量池已有，直接引用

System.out.println(s1 == s3);  // true（同一常量池引用）
System.out.println(s1 == s2);  // false（不同对象）
System.out.println(s1.equals(s2)); // true（内容相同）
```

### 7.3 StringBuilder/StringBuffer 深入
- 底层都是可扩容的 `char[]` 数组（JDK 9+ 为 `byte[]`）
- 默认初始容量 16，扩容时翻倍 + 2
- 频繁字符串拼接时，优先使用 StringBuilder

```java
StringBuilder sb = new StringBuilder();
sb.append("Hello").append(" ").append("World");
String result = sb.toString();  // "Hello World"
```

---

## 8. 集合框架

### 8.1 整体架构
```
Collection（单列集合）
├── List（有序，可重复）
│   ├── ArrayList（数组实现，查询快）
│   ├── LinkedList（链表实现，增删快）
│   └── Vector（线程安全的 ArrayList）
└── Set（无序，不重复）
    ├── HashSet（HashMap 实现，无序）
    ├── LinkedHashSet（维护插入顺序）
    └── TreeSet（红黑树实现，可排序）

Map（双列集合，键值对）
├── HashMap（数组+链表+红黑树，JDK 8+）
├── LinkedHashMap（维护插入/访问顺序）
├── TreeMap（红黑树实现，按键排序）
└── Hashtable（线程安全，已不推荐使用）
```

### 8.2 ArrayList 
- **底层结构**：`Object[]` 数组
- **扩容机制**：默认容量 10，扩容为原来的 1.5 倍（`oldCapacity + (oldCapacity >> 1)`）
- **时间复杂度**：
  - `get(index)`：O(1)
  - `add(E)`：均摊 O(1)，扩容时 O(n)
  - `add(index, E)`：O(n)（需移动元素）
  - `remove(index)`：O(n)
- 与数组的区别：自动扩容、只能存储对象（基本类型自动装箱）
- 线程不安全，会抛 `ConcurrentModificationException`（fail-fast）

```java
ArrayList<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add(1, "C");  // 在索引 1 处插入
list.remove("A");
list.get(0);        // "C"
```

### 8.3 LinkedList 
- **底层结构**：双向链表（Node 包含 prev、item、next）
- **时间复杂度**：
  - `get(index)`：O(n)
  - `add(E)`：O(1)
  - `add(index, E)`：O(n)（需先遍历到位置）
  - `remove(index)`：O(n)
- 同时实现了 `Deque` 接口：可用作栈（push/pop）或队列（offer/poll）

### 8.4 HashMap 
- **底层结构**：
  - JDK 7：数组 + 链表
  - JDK 8：数组 + 链表 + 红黑树（链表长度 ≥ 8 且数组长度 ≥ 64 时转为红黑树）
- **核心参数**：
  - 默认初始容量：16
  - 默认负载因子：0.75
  - 扩容阈值：容量 × 负载因子 = 12
- **哈希算法**：`(h = key.hashCode()) ^ (h >>> 16)` — 高 16 位与低 16 位异或，减少哈希冲突
- **put 流程**：
  1. 计算 key 的哈希值
  2. `(n - 1) & hash` 计算数组索引（等价于取模，但效率更高）
  3. 索引位置无元素：直接插入
  4. 有元素：遍历链表/红黑树，key 相同则覆盖，不同则追加
  5. 超过阈值则扩容（容量翻倍，重新哈希）
- **线程安全问题**：
  - JDK 7：并发扩容可能导致死循环
  - JDK 8：不会死循环，但会丢数据
  - 解决方案：`ConcurrentHashMap`、`Collections.synchronizedMap`
- **切记**：重写 `equals` 必须重写 `hashCode`，否则散列表无法正确判重

```java
HashMap<String, Integer> map = new HashMap<>();
map.put("Alice", 25);
map.put("Bob", 30);

// 遍历方式
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + "=" + entry.getValue());
}

// JDK 8+ forEach + Lambda
map.forEach((k, v) -> System.out.println(k + "=" + v));
```

### 8.5 HashSet 
- 底层就是 `HashMap`，元素作为 key，value 是一个固定的 Object 对象（`PRESENT`）
- 去重原理：先比较 `hashCode`，再比较 `equals`
- 自定义对象去重需重写 `hashCode()` 和 `equals()`

### 8.6 TreeSet / TreeMap
- 基于红黑树实现，元素自然排序或传入 `Comparator`
- 要求元素可比较（实现 `Comparable` 或传入 `Comparator`）

### 8.7 LinkedHashMap
- 在 `HashMap` 上增加了双向链表维护插入或访问顺序
- 可实现 LRU（最近最少使用）缓存

### 8.8 迭代器（Iterator）
- 统一遍历集合的方式，不暴露集合内部结构
- `hasNext()`、`next()`、`remove()`
- **快速失败（fail-fast）**：遍历过程中修改集合（非迭代器自身修改）会抛出 `ConcurrentModificationException`
- **安全失败（fail-safe）**：`CopyOnWriteArrayList` 等并发集合采用

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.equals("remove")) {
        it.remove();  // 安全的删除方式
    }
}
```

---

## 9. 异常处理

### 9.1 异常体系
```
Throwable
├── Error（严重错误，不可恢复）
│   └── OutOfMemoryError、StackOverflowError 等
└── Exception（可处理的异常）
    ├── RuntimeException（运行时异常，非受检异常）
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── ClassCastException
    │   └── IllegalArgumentException 等
    └── 其他 Exception（受检异常，必须处理）
        ├── IOException
        ├── SQLException
        └── ClassNotFoundException 等
```

### 9.2 深入理解
- **受检异常（Checked Exception）**：编译器强制要求处理，用 try-catch 或 throws 声明
- **非受检异常（Unchecked Exception）**：`RuntimeException` 及其子类，编译器不强制处理
- **try-catch-finally**：
  - `try`：可能抛出异常的代码
  - `catch`：捕获并处理异常（可以有多个，子类异常在前）
  - `finally`：无论是否异常都会执行（常用于释放资源）。除调用 `System.exit()` 外，总会执行。若 finally 中也有 return，会覆盖 try 的返回结果
- **try-with-resources（JDK 7+）**：自动关闭实现了 `AutoCloseable` 接口的资源。推荐使用
- **异常链**：在 catch 中抛出新异常时保留原始异常信息，`new Exception("msg", cause)` 保留根本原因，便于排查
- **自定义异常**：继承 `Exception` 或 `RuntimeException`，提供构造方法

```java
// try-with-resources 自动关闭资源
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}

// 自定义异常
public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
    public BusinessException(String message, Throwable cause) {
        super(message, cause);  // 异常链
    }
}
```

---

## 10. IO 流

### 10.1 流分类

| 分类维度 | 类型 | 说明 |
|----------|------|------|
| 数据单位 | 字节流 | InputStream / OutputStream（处理二进制数据） |
| | 字符流 | Reader / Writer（处理文本数据，自动编解码） |
| 功能 | 节点流 | 直接连接数据源（FileInputStream 等） |
| | 处理流 | 包装节点流，提供额外功能（BufferedInputStream 等） |
| 方向 | 输入流 | 从外部读入程序 |
| | 输出流 | 从程序写出外部 |

### 10.2 常用流
- **文件流**：`FileInputStream` / `FileOutputStream` / `FileReader` / `FileWriter`
- **缓冲流**：`BufferedInputStream` / `BufferedOutputStream`（内置 8KB 缓冲区，减少系统调用）
- **转换流**：`InputStreamReader` / `OutputStreamWriter`（字节流 ↔ 字符流转换，可指定编码）
- **对象流**：`ObjectInputStream` / `ObjectOutputStream`（序列化/反序列化）
- **打印流**：`PrintStream` / `PrintWriter`（`System.out` 就是 `PrintStream`）

### 10.3 序列化与反序列化
- `Serializable` 接口：标记接口，无需实现方法
- `serialVersionUID`：序列化版本号，必须显式指定，用于兼容性校验。否则修改类结构会导致反序列化失败（`InvalidClassException`）
- `transient` 关键字：标记不需要序列化的字段
- `Externalizable` 接口：自定义序列化逻辑
- 静态字段不被序列化

```java
// 序列化
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("obj.dat"));
oos.writeObject(new Person("张三", 20));
oos.close();

// 反序列化
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("obj.dat"));
Person p = (Person) ois.readObject();
ois.close();
```

### 10.4 NIO（New IO）简介
- JDK 1.4 引入，JDK 7 升级为 NIO.2
- 核心组件：`Buffer`（缓冲区/容器）、`Channel`（双向通道，如 `FileChannel`）、`Selector`（选择器，多路复用，用于网络编程）
- 特点：非阻塞 IO、面向缓冲区、支持内存映射文件
- 与 IO 区别：IO 面向流、阻塞；NIO 面向缓冲、非阻塞。这是网络编程的重要基础，但基础阶段了解概念即可

---

## 11. 泛型

### 11.1 基本概念
参数化类型，将类型作为参数传递。编译时类型检查，避免强制类型转换。

### 11.2 核心机制：类型擦除
编译后泛型信息被擦除，替换为边界类型或 `Object`。故不能 `new T()`，也不能用 `instanceof` 检查泛型参数。桥方法用于维持多态。

### 11.3 泛型类
```java
public class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

Box<String> box = new Box<>();
box.set("hello");
String s = box.get();  // 无需强制转换
```

### 11.4 泛型方法
```java
public <T> void printArray(T[] array) {
    for (T item : array) {
        System.out.println(item);
    }
}
```

### 11.5 通配符
- `?`：任意类型
- `? extends T`：上界通配符，可读取 T 类型数据，不可写入（Producer）
- `? super T`：下界通配符，可写入 T 类型数据，读取为 Object（Consumer）
- **PECS 原则**：Producer Extends, Consumer Super

```java
// 上界：可以安全读取 Number
List<? extends Number> list1 = new ArrayList<Integer>();
Number n = list1.get(0);  // ✓
// list1.add(1);  // ✗ 编译错误

// 下界：可以安全写入 Integer
List<? super Integer> list2 = new ArrayList<Number>();
list2.add(1);  // ✓
// Integer i = list2.get(0);  // ✗ 编译错误，只能作为 Object 读取
```

---

## 12. 多线程基础

### 12.1 基本概念
- **进程**：操作系统资源分配的基本单位
- **线程**：CPU 调度的基本单位，一个进程包含多个线程
- **主线程**：`main()` 方法所在的线程
- **线程状态**：NEW → RUNNABLE → BLOCKED / WAITING / TIMED_WAITING → TERMINATED
  - **BLOCKED**：等待获取监视器锁（如 `synchronized`），线程被阻塞在锁上
  - **WAITING**：等待其他线程唤醒（如 `wait()`、`LockSupport.park()`），无限期等待
  - **TIMED_WAITING**：限时等待（如 `sleep(time)`、`wait(time)`），到时间自动恢复

### 12.2 创建线程的三种方式

#### 1. 继承 Thread 类
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("线程执行");
    }
}
MyThread t = new MyThread();
t.start();
```

#### 2. 实现 Runnable 接口（推荐）
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("线程执行");
    }
}
Thread t = new Thread(new MyRunnable());
t.start();
```
- 更好，解耦任务与线程，并可共享任务实例

#### 3. 实现 Callable 接口（JDK 5+，可返回结果）
```java
Callable<String> callable = () -> "返回结果";
FutureTask<String> futureTask = new FutureTask<>(callable);
new Thread(futureTask).start();
String result = futureTask.get();  // 阻塞获取结果
```

### 12.3 深入理解

#### 线程同步
- **synchronized 关键字**：
  - 修饰非静态方法：锁对象是当前实例（`this`）
  - 修饰静态方法：锁是类的 `Class` 对象
  - 修饰代码块：可指定锁对象
  - **可重入性**：同一个线程可多次获取同一把锁
  - **底层原理**：每个对象都有监视器锁（monitor），JDK6 后引入锁升级：偏向锁 -> 轻量级锁 -> 重量级锁，减少无竞争下的性能开销
- **volatile 关键字**：
  - 保证可见性（一个线程修改，其他线程立即可见）
  - 不保证原子性（`i++` 仍需同步）
  - 禁止指令重排序
- **wait/notify**：
  - 必须在同步代码块中使用
  - `wait()` 释放锁并等待，进入 WAITING；`sleep()` 持有锁不放，进入 TIMED_WAITING
  - `notify()` 唤醒等待线程；`notifyAll()` 唤醒全部
  - 必须在循环中检查条件，防止虚假唤醒
- **死锁**：多线程互相持有对方所需资源，需避免嵌套锁

#### 线程控制方法
- **join()**：等待该线程执行完毕。如 `t.join()` 使当前线程阻塞直到 t 结束
- **yield()**：提示调度器当前线程愿意让出 CPU，但调度器可以忽略
- **setDaemon(true)**：设置为守护线程。当所有非守护线程结束时，JVM 会退出，守护线程自动终止。常用于后台服务（如垃圾回收线程）

#### 线程池（JDK 5+）
- 使用 `Executors` 工厂方法或 `ThreadPoolExecutor` 手动创建（推荐）
- **核心参数**：
  - `corePoolSize`：核心线程数
  - `maximumPoolSize`：最大线程数
  - `keepAliveTime`：非核心线程空闲存活时间
  - `workQueue`：任务等待队列
  - `handler`：拒绝策略

```java
// 手动创建线程池（推荐）
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                      // 核心线程数
    10,                     // 最大线程数
    60L,                    // 空闲线程存活时间
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100),  // 任务队列
    new ThreadPoolExecutor.CallerRunsPolicy()  // 拒绝策略
);

executor.execute(() -> System.out.println("任务执行"));
executor.shutdown();
```

#### 线程安全集合
- `Vector`、`Hashtable`：方法级同步，性能差
- `Collections.synchronizedXxx()`：包装器，方法级同步
- `CopyOnWriteArrayList` / `CopyOnWriteArraySet`：写时复制，读多写少场景
- `ConcurrentHashMap`：分段锁（JDK 7）/ CAS + `synchronized`（JDK 8），高并发首选

---

## 13. Object 类的核心方法

### 13.1 地位
一切类的父类，默认继承自 `Object`。理解其方法相当于理解 Java 对象的基本契约。

### 13.2 核心方法
- `equals(Object obj)`：用于判断对象内容是否相等，与 `==`（比较引用地址）区分
- `hashCode()`：返回对象的哈希值，用于散列表（`HashMap`、`HashSet` 等）定位存储桶
- `toString()`：返回对象的字符串表示，默认 `类名@十六进制哈希`，建议重写
- `clone()`：创建并返回对象的拷贝，需实现 `Cloneable` 接口，否则抛 `CloneNotSupportedException`
- `getClass()`：返回运行时的 `Class` 对象
- `finalize()`：对象被回收前由 GC 调用，已废弃，不建议使用
- `wait / notify / notifyAll`：多线程通信基础，必须在同步块中调用

### 13.3 深入重点
- **equals 与 hashCode 的契约**：若两对象 `equals` 相等，则 `hashCode` 必须相等；反之不强制，但散列表性能依赖。重写 `equals` 必须同步重写 `hashCode`
- **浅拷贝 vs 深拷贝**：`clone` 默认是浅拷贝（基本类型字段复制值，引用字段复制引用）。可通过递归拷贝引用对象实现深拷贝，或使用序列化工具
- **wait/notify 机制**：`wait()` 释放锁进入等待队列；`notify()` 随机唤醒一个等待线程；`notifyAll()` 唤醒全部。必须在循环中检查条件，防止虚假唤醒

---

## 14. 基本类型、包装类与自动装箱拆箱

### 14.1 八种基本类型
- `byte`(1字节)、`short`(2)、`int`(4)、`long`(8)、`float`(4)、`double`(8)、`char`(2, Unicode)、`boolean`(JVM 相关)
- **成员变量（全局）有默认值**：`int`→0，`boolean`→false，`char`→'\u0000'（空字符），引用类型→null。局部变量必须显式初始化，否则编译报错
- **数组元素也有默认值**：与成员变量一致

### 14.2 包装类
`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean`，均不可变，提供 `parseXxx`、`valueOf` 等工具方法。

### 14.3 自动装箱 / 拆箱
编译器自动在基本类型和包装类之间转换，如 `Integer i = 10;` 相当于 `Integer.valueOf(10)`。

### 14.4 深入陷阱
- **缓存机制**：`Integer` 缓存默认 [-128, 127] 的对象（可调上限），`valueOf` 在此范围内复用对象。因此 `Integer a = 127; Integer b = 127; a == b` 为 `true`，而 128 则为 `false`。`Long`、`Short` 等也有类似缓存
- **性能问题**：循环内大量自动装箱会产生大量临时对象，增加 GC 压力
- **NullPointerException**：参与运算或三目表达式时，自动拆箱的包装类若为 `null`，会抛空指针

---

## 15. 枚举 (enum)

### 15.1 基础定义
`enum` 是特殊的类，实例数量固定，每个枚举常量就是 `public static final` 的实例。

```java
public enum Season { SPRING, SUMMER, AUTUMN, WINTER }
```

### 15.2 常用方法
- `values()`：返回所有常量数组
- `ordinal()`：返回声明顺序（从 0 开始）
- `name()`：返回枚举名称字符串
- `valueOf(String)`：根据名称获取实例

### 15.3 深入能力
- 可定义字段、构造方法（必须私有）、方法，甚至实现接口
- **枚举实现单例**：《Effective Java》推荐，无偿提供序列化安全（枚举序列化机制特殊，不会生成新实例）、反射安全、线程安全，写法极简

```java
public enum Singleton { 
    INSTANCE; 
    public void doSomething() {} 
}
```
- `EnumSet` 和 `EnumMap` 是专门的高效集合实现

---

## 16. 注解 (Annotation)

### 16.1 作用
为代码添加元数据，本身不直接影响逻辑，由编译工具或框架通过反射读取处理。

### 16.2 内置注解
- `@Override`：编译器校验方法重写
- `@Deprecated`：标记过时
- `@SuppressWarnings("unchecked")`：抑制编译器警告
- `@FunctionalInterface`：校验是否为函数式接口

### 16.3 元注解（注解的注解）
- `@Target`：指定可修饰的位置（`ElementType.TYPE, FIELD, METHOD` 等）
- `@Retention`：保留策略，`RetentionPolicy.SOURCE`（编译时丢弃）、`CLASS`（字节码保留但运行时不可见）、`RUNTIME`（运行时可通过反射读取，多数框架用）
- `@Documented`、`@Inherited`

### 16.4 自定义注解
使用 `@interface` 定义，元素类似无参方法，可设 `default` 值。

---

## 17. 反射 (Reflection)

### 17.1 核心能力
在运行时动态获取类的信息、创建对象、调用方法、操作字段，即使它们是私有的。

### 17.2 核心类
- `Class`：类的入口，获取方式 `Class.forName("全限定名")`、`类名.class`、`对象.getClass()`
- `Constructor`、`Method`、`Field`

### 17.3 常用操作
- `getDeclaredConstructor(参数类型)` 获取构造方法（含私有），调用 `newInstance()` 创建对象
- `getDeclaredMethod(方法名, 参数类型)` 获取方法，调用 `invoke(对象, 实参)` 执行
- `getDeclaredField(字段名)` 获取字段，`setAccessible(true)` 突破私有权限，使用 `set/get` 读写值

### 17.4 深入关注
- `setAccessible(true)` 会关闭访问检查，但在 JDK 9+ 模块化系统下，若目标模块未通过 `module-info` 开放包，或命令行未添加 `--add-opens`，仍可能抛出 `InaccessibleObjectException`
- 反射有性能损耗，频繁调用应缓存 Method 对象或使用高性能替代方案（如 `MethodHandle`）
- 反射是 Spring、MyBatis 等框架的基石，但过度使用会破坏封装性

---

## 18. Lambda 表达式、方法引用与函数式接口

### 18.1 Lambda 表达式
以 `(参数) -> { 代码 }` 形式实现函数式接口的实例，本质是利用 `invokedynamic` 指令，并非简单的匿名内部类。

```java
Runnable r = () -> System.out.println("Hello Lambda");
```

### 18.2 函数式接口
只有一个抽象方法的接口，可用 `@FunctionalInterface` 标注。

**四大核心接口**：
- `Predicate<T>`：`boolean test(T t)`，用于过滤
- `Consumer<T>`：`void accept(T t)`，消费数据
- `Function<T, R>`：`R apply(T t)`，转换
- `Supplier<T>`：`T get()`，提供数据

扩展：`BiConsumer<T,U>`、`BiFunction<T,U,R>`、`UnaryOperator<T>`（一元操作符，继承 Function）、`BinaryOperator<T>`（二元）等。

### 18.3 方法引用
简化 Lambda 写法，格式 `类/对象::方法`：
- 静态方法引用：`ClassName::staticMethod`
- 特定对象实例方法引用：`object::instanceMethod`
- 任意对象的实例方法引用：`ClassName::instanceMethod`（如 `String::length`）
- 构造方法引用：`ClassName::new`

---

## 19. Stream API

### 19.1 设计思想
声明式处理集合数据，配合 Lambda 实现链式操作，内部迭代、函数式编程。

### 19.2 创建流
`Collection.stream()` / `parallelStream()`、`Arrays.stream()`、`Stream.of()`、`IntStream.range()` 等。

### 19.3 中间操作（惰性）
- `filter`：根据 Predicate 过滤
- `map`：元素一对一转换
- `flatMap`：将每个元素转换为流，再合并成一个流
- `distinct`：去重（依赖 `equals/hashCode`）
- `sorted`：排序（可传 Comparator）
- `limit / skip`：截断 / 跳过

### 19.4 终端操作（触发计算）
- `collect(Collectors.toList/toSet/toMap)`：归集
- `forEach`：遍历
- `reduce`：归约（求和、求积等）
- `count`、`anyMatch / allMatch / noneMatch`：计数、匹配

### 19.5 深入
- 惰性求值意味着没有终端操作就不会执行
- `parallelStream` 默认使用 `ForkJoinPool.commonPool()`（线程数 = CPU 核心数 - 1），若池被其他任务占满，并行流可能被阻塞。数据量小、有状态操作时不建议并行；需要自定义线程池时，可通过 `ForkJoinPool` 包装执行

---

## 20. Java 时间 API (java.time)

### 20.1 替代旧 API
线程安全、不可变、设计清晰，完全取代 `Date`、`Calendar`、`SimpleDateFormat`。

### 20.2 核心类
- `LocalDate`：日期（年-月-日）
- `LocalTime`：时间（时-分-秒-纳秒）
- `LocalDateTime`：日期时间组合
- `Instant`：时间戳（与 `Date` 可互转）
- `ZonedDateTime`：带时区的日期时间
- `Duration`：基于时间的间隔（时、分、秒、纳秒）
- `Period`：基于日期的间隔（年、月、日）

### 20.3 格式化
`DateTimeFormatter`，线程安全，可自定义模式或使用预置常量如 `DateTimeFormatter.ISO_LOCAL_DATE`。与 `LocalDate.parse()` 配合使用。

### 20.4 操作
使用 `plusDays/minusMonths` 等方法创建新实例，实现时间计算。`TemporalAdjusters` 提供常用调节器（如当月第一天）。

### 20.5 新旧 API 互转
- `Date ↔ Instant`：`date.toInstant()`、`Date.from(instant)`
- `Calendar ↔ ZonedDateTime`：`calendar.toInstant().atZone(ZoneId.systemDefault())`
- **SimpleDateFormat 线程不安全**：多个线程共享同一个 `SimpleDateFormat` 实例会导致数据混乱，需使用 `ThreadLocal` 或改用线程安全的 `DateTimeFormatter`

---

## 21. 其他重要零碎基础

### 21.1 main 方法
`public static void main(String[] args)` 是 Java 程序的标准入口：
- `public`：JVM 需要从外部调用，必须公开
- `static`：JVM 调用时无需创建对象实例
- `void`：程序入口不需要返回值给 JVM
- `String[] args`：接收命令行传入的参数，如 `java MyApp arg1 arg2`，args 即为 `["arg1", "arg2"]`

```java
public class MainDemo {
    public static void main(String[] args) {
        for (String arg : args) {
            System.out.println("参数：" + arg);
        }
    }
}
```

### 21.2 代码块
- **静态代码块**：`static {}`，类加载时执行一次，用于初始化静态资源
- **实例代码块**：`{}`，每次创建对象时执行，在构造方法之前执行，常用于提取多个构造方法共有的初始化逻辑

```java
public class BlockDemo {
    static {
        System.out.println("静态代码块：类加载时执行");
    }
    {
        System.out.println("实例代码块：构造方法前执行");
    }
    public BlockDemo() {
        System.out.println("构造方法执行");
    }
}
```

### 21.3 Comparable / Comparator
- **Comparable<T>**：内比较器，类实现 `compareTo(T o)` 方法定义自然排序。如 `String`、`Integer` 已内置实现
- **Comparator<T>**：外比较器，独立实现比较逻辑，可灵活定制排序规则，不修改原类

```java
// Comparable：类自身实现
public class Student implements Comparable<Student> {
    private int score;
    @Override
    public int compareTo(Student o) {
        return this.score - o.score;  // 升序
    }
}

// Comparator：外部定义
Comparator<Student> byName = Comparator.comparing(Student::getName);
Collections.sort(list, byName);  // 或 list.sort(byName)
```

### 21.4 Queue / Deque 实现类
- **ArrayDeque**：基于循环数组实现的双端队列，比 `LinkedList` 做栈/队列更高效（无节点对象开销），是日常首选
- **PriorityQueue**：基于堆结构实现，元素按优先级出队，默认自然排序或传入 `Comparator`

```java
Deque<String> stack = new ArrayDeque<>();  // 栈：push/pop
Deque<String> queue = new ArrayDeque<>();  // 队列：offer/poll

PriorityQueue<Integer> pq = new PriorityQueue<>();  // 小顶堆
pq.offer(5); pq.offer(1); pq.offer(3);
System.out.println(pq.poll());  // 1（最小优先）
```

### 21.5 数组与 Arrays 工具类
- 数组初始化：`int[] arr = {1,2};` 或 `new int[5]`。多维数组实质为数组的数组
- `Arrays.sort(arr)`：排序；`Arrays.binarySearch(arr, key)`：二分查找（需先排序）
- `Arrays.asList(T... a)`：注意返回的 `List` 是固定大小的，**不支持 `add/remove`**，但可修改元素。若要可变，可 `new ArrayList<>(Arrays.asList(...))`
- `Arrays.copyOf(arr, newLength)`：复制数组并指定新长度
- `System.arraycopy(src, srcPos, dest, destPos, length)`：native 方法，高效数组复制，底层可能使用内存拷贝指令
- `Arrays.fill(arr, value)`：将数组所有元素填充为指定值

### 21.6 可变参数 (varargs)
- 语法：`void method(String... args) {}`，内部当作数组，调用时可传任意数量实参
- 规则：一个方法只能有一个可变参数，且放在参数列表最后。重载时注意优先匹配固定参数方法

### 21.7 try-with-resources
- 资源自动关闭：实现了 `AutoCloseable` 接口的对象（如流、连接）可在 `try(资源声明)` 结束后自动调用 `close()`，即使发生异常
- **异常屏蔽**：若 finally 和 catch 都抛异常，try-with-resources 会优先暴露原始异常，关闭时的异常会被压制（suppressed），可通过 `Throwable.getSuppressed()` 获取
