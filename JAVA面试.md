# 服务端面试速刷指南

## 1️⃣ Spring / Spring Boot

### 核心概念

* `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
* 自动配置（Auto-Configuration）：根据 classpath 条件和 Bean 判断自动加载配置
* Starter 依赖示例：`spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-actuator`
* 嵌入式 Tomcat/Jetty：快速启动、无需外部容器
* `@RestController` vs `@Controller`：前者默认返回 JSON
* Profiles（dev/prod）、Actuator 监控端点

### 常见面试题

1. Spring Boot 自动配置如何工作？
* Spring Boot 的自动配置通过 @EnableAutoConfiguration 启动，它会扫描类路径下的 META-INF/spring.factories 文件，找到所有自动配置类。
* 每个配置类通过条件注解（如 @ConditionalOnClass、@ConditionalOnMissingBean）判断是否创建 Bean。
* 用户自定义 Bean 会覆盖自动配置，保证灵活性。
* 举例来说，添加 spring-boot-starter-web 时，Spring Boot 会自动配置嵌入式 Tomcat 和 DispatcherServlet，无需手动配置。
2. @Component, @Service, @Repository 区别？
* @Component 是最通用的组件注解，标记普通 Bean 交给 Spring 容器管理；
* @Service 专用于业务逻辑层，语义上更明确；
* @Repository 专用于 DAO/持久层，除了交给容器管理外，还会启用 Spring 的异常转换机制，把数据库异常转换为 Spring 的 DataAccessException。
3. Spring Boot 中 @Autowired 的原理
* @Autowired 是 Spring 的依赖注入（DI）注解，用于自动注入 Bean。
* Spring 容器会根据类型（默认）或名称（通过 @Qualifier）匹配合适的 Bean 并注入到字段、构造方法或 setter 方法。
* Spring 在应用启动时扫描并管理所有 Bean，@Autowired 通过反射将匹配的 Bean 自动注入到目标类。
4. Spring Boot 异常处理机制（@ControllerAdvice）
* @ControllerAdvice 是 Spring 提供的全局异常处理注解，用于集中处理所有控制器的异常。
* 它可以与 @ExceptionHandler 一起使用来定义处理特定异常的方法。
* 通过 @ControllerAdvice，我们可以统一返回错误响应，避免每个控制器都写重复的异常处理代码。
* 例如，@ExceptionHandler 可以捕获特定异常并返回自定义的错误信息或状态码，增强了代码的可维护性。

## 2️⃣ MyBatis

### 核心概念

* #{ } vs ${ }：#{ } 自动参数绑定防 SQL 注入；${ } 直接拼接
* 动态 SQL 标签：<if>, <where>, <choose>, <foreach>
* Mapper 接口 + XML 映射文件
* 一级缓存（Session 内）、二级缓存（Namespace 内）
* 与 Spring 集成：@MapperScan、事务管理、数据源配置

### 常见面试题

1. MyBatis 与 Hibernate 优势比较
* Hibernate 是一个全自动的 ORM 框架，自动生成 SQL 并将 Java 对象与数据库表映射。它适合 简单的 CRUD 操作 和 数据模型复杂 的场景，但性能可能较差，尤其在复杂查询和优化方面。

* MyBatis 是一个 半自动化的持久层框架，允许开发者手动编写 SQL，适合 复杂查询 和 性能优化。它提供更高的灵活性，可以精确控制 SQL，尤其适合需要精细控制数据库操作的项目，但需要开发
者自己编写和维护 SQL。

* 选择：Hibernate 更适合快速开发和简单业务，而 MyBatis 更适合需要复杂查询和性能优化的系统

2. MyBatis 缓存机制及作用
* 一级缓存：是 SqlSession 级别的缓存，默认启用，只在当前会话中有效。对于同一 SqlSession 内的重复查询，MyBatis 会直接从缓存中获取结果，避免重复查询数据库。

* 二级缓存：是 Mapper 级别的缓存，需要手动配置启用，缓存可以跨 SqlSession 使用。适合跨会话的重复查询，可以显著减少数据库访问。

* 作用：缓存机制提高查询性能，减少数据库压力和响应时间，但需要注意数据一致性问题，尤其在执行写操作时要清除相关缓存。

3. 动态 SQL 使用场景
* 复杂查询：根据不同条件动态生成 SQL，例如按多个条件查询。

* 分页查询：动态生成分页 SQL，适应不同页码和每页大小。

* 多条件搜索：根据用户输入动态拼接查询条件。

* 批量操作：批量插入或更新时，动态生成 SQL。

* 动态表名/列名：在多租户或动态数据源场景下使用。

* 条件变化：根据不同业务需求调整查询逻辑。

4. MyBatis + Spring Boot 的配置示例
* 添加依赖：包括 Spring Boot、MyBatis 和数据库连接池等。

* 配置数据库和 MyBatis：在 application.properties 中配置数据源和 MyBatis 设置。

* 创建 Mapper 文件和接口：通过 XML 或注解方式编写 SQL，并将其与 Java 接口绑定。

* Service 和 Controller 层调用：使用 @Service 调用 Mapper 层，实现业务逻辑和控制层交互。

## 3️⃣ Java EE / 后端基础

### 核心概念

* JVM 内存模型、堆栈、GC 类型
* 多线程 & 并发：Thread, Runnable, Callable, ThreadPoolExecutor, synchronized, volatile
* RESTful API 设计：状态码、幂等性、安全处理
* 数据库事务隔离级别及锁机制

### 面试题示例

1. synchronized 与 ReentrantLock 和redis分布式锁 区别
- synchronized
  - 作用范围：用于单机应用的线程同步，基于 Java 内部实现。
  - 实现方式：通过 JVM 实现的隐式锁（加锁和解锁由 Java 自动管理）。
  - 特点：
    - 简单易用，不需要显式管理锁。
    - 只能用于单个 JVM 进程内的线程同步。
    - 不支持超时设置，且无法中断线程。
    - 由于是内存锁，不能跨进程或分布式环境使用。

- ReentrantLock
  - 作用范围：用于多线程同步，提供比 synchronized 更加丰富的控制，主要用于单机应用。
  - 实现方式：基于 java.util.concurrent.locks 包的类，需要手动管理锁（lock() 和 unlock()）。
  - 特点：
    - 支持可中断锁（lockInterruptibly()）和定时锁（tryLock()）。
    - 支持公平锁（ReentrantLock(true)）来控制线程获取锁的顺序。
    - 适用于复杂的同步控制，可以通过显式解锁避免死锁。
    - 同样只能用于单个 JVM 内，不能跨进程使用。

- Redis 分布式锁
	- 作用范围：用于分布式环境中的资源同步，跨多个 JVM 或服务实例。
	- 实现方式：使用 Redis 提供的 SETNX 命令或者其他分布式锁框架（如 Redisson）来控制访问。
	- 特点：
		- 支持跨进程和跨服务器的分布式锁。
		- 需要显式设置锁过期时间，防止死锁。
		- 适用于分布式系统中多个节点对共享资源的访问控制。
		- 相比 ReentrantLock 和 synchronized 更加灵活，可以设置超时、重试等功能，适合分布式环境。
- 总结：
	- synchronized 适用于单机应用，简单易用，但不能跨进程。
	- ReentrantLock 提供更高的灵活性和功能（如可中断、定时锁等），适用于复杂的单机同步需求。
	- Redis 分布式锁 适用于分布式系统，能跨进程和跨服务实例保证资源访问的互斥性。

2. volatile 的作用、
volatile 保证了变量的可见性和禁止指令重排，但不能保证原子性，适用于多线程环境中对单一变量的共享和同步

3. Java 内存泄漏原因及排查方法
- 内存泄漏原因：
	- 对象引用未释放，静态集合持有大量对象。
	- 事件监听器未移除，资源（如数据库连接）未关闭。
	- 线程池中的线程未回收，堆外内存未释放。
- 排查方法：
	- 使用 JVisualVM、MAT 等工具分析堆转储。
	- 查看 GC 日志 和 内存监控，识别未回收的对象。
	- 审查代码，确保资源和对象及时释放

4. Spring 事务传播行为（PROPAGATION_REQUIRED, REQUIRES_NEW）
- PROPAGATION_REQUIRED
	- 定义：如果当前存在事务，则加入该事务；如果没有事务，则创建一个新的事务。
	- 适用场景：最常用的事务传播行为。它允许方法参与现有的事务，并在没有事务的情况下自动创建一个新的事务。
	- 行为：
		- 如果调用的事务方法已经在事务中执行（即父事务存在），则该方法会参与父事务。
		- 如果没有事务在执行，则该方法会创建一个新的事务。
	- 默认行为：这是 Spring 的默认传播行为。
- PROPAGATION_REQUIRES_NEW
	- 定义：总是创建一个新的事务。如果当前有事务存在，挂起当前事务并创建一个新的事务，执行完后恢复原事务。
	- 适用场景：当需要确保某个方法独立执行，并且不受外部事务影响时使用。它适用于需要保证某些操作具有独立性（如日志记录或独立的子事务）的场景。
	- 行为：
		- 不管当前是否有事务存在，它总是会新建一个事务。
		- 如果当前已经有一个事务，原事务会被挂起，直到新事务执行完成后再恢复。
	- 使用场景：例如，某些操作需要独立的事务，不希望受到外部事务的提交或回滚影响。

## 4️⃣ 系统设计 + 项目经验

### 海外业务关注点

* 多语言、多时区处理（UTC 与当地时间转换）
* 高并发消息队列（异步任务处理）
* 缓存策略（Redis、本地缓存、过期策略）
* 分布式事务和微服务架构

### 项目经验亮点示例

* 使用 Java + Redis + 异步执行优化统计任务
* 短信/消息调度策略平台化设计
* 监控告警、异常日志排查

### 面试题示例

1. 描述你负责的后端项目架构与关键技术
2. 如何保证高并发下的数据一致性
3. 业务跨时区如何处理时间相关逻辑

## 6️⃣ 参考资料链接

* [Spring Boot 面试题](https://www.interviewbit.com/spring-boot-interview-questions/?utm_source=chatgpt.com)
* [MyBatis 面试题](https://climbtheladder.com/mybatis-interview-questions/?utm_source=chatgpt.com)
* [Spring MVC 面试题](https://www.baeldung.com/spring-mvc-interview-questions?utm_source=chatgpt.com)
* [Java 后端面试题](https://medium.com/@poojaauma/how-to-ace-a-java-backend-developer-interview-in-2025-6aea12b4b804?utm_source=chatgpt.com)
