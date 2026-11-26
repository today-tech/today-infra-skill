# 持久化与 ORM

> 由 `scripts/build-refs.mjs` 从 `infra-docs` 生成，请勿手工编辑；改归类请改 `data/topics.json`。

实体管理、映射、分页、审计、批量与原生 SQL。

覆盖文档：15 页（含示例 13 页）。原文位于仓库 `infra-docs/`，下方每个页面标注了路径。

## 页面索引

- 轻量级持久化框架 (Infra Persistence) —— `data-access/persistence`
- **实体审计** —— `data-access/persistence/auditing`
- **批量持久化** —— `data-access/persistence/batch`
- **自动配置与依赖** —— `data-access/persistence/configuration`
- **EntityManager** —— `data-access/persistence/entity-manager`
- **生命周期事件** —— `data-access/persistence/events`
- **实体映射** —— `data-access/persistence/mapping`
- **分页** —— `data-access/persistence/paging`
- **查询条件与示例查询 (QueryCondition)** —— `data-access/persistence/query-by-example`
- **滚动查询** —— `data-access/persistence/scrolling`
- **SQL 构建与数据库平台** —— `data-access/persistence/sql`
- **属性更新策略** —— `data-access/persistence/update-strategy`
- **乐观锁** —— `data-access/persistence/version`
- 对象关系映射 (ORM) 数据访问 —— `data-access/orm`
- **一般 ORM 集成注意事项** —— `data-access/orm/general`

## 轻量级持久化框架 (Infra Persistence)

> 原文：[`data-access/persistence`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence.adoc)

### 概述

infra-persistence 在 JDBC 抽象之上提供了一个轻量级的实体持久化层。 它不依赖 JPA/Hibernate，也不提供 JPA 的实体上下文、一级缓存或脏检查；框架通过注解与运行时元数据 把 Java 对象映射到关系表，并由 EntityManager 负责 SQL 写入、条件查询、分页、主键生成、乐观锁与生命周期事件。 它更适合希望直接控制 SQL 形状、事务边界和实体映射，同时又不想重复编写基础 JDBC CRUD 代码的应用。 复杂查询可以直接实现 QueryStatement / QueryCondition，而不是被限制在示例查询的表达能力内。 本部分按层次拆分介绍持久化框架的各个方面：

## 实体审计

> 原文：[`data-access/persistence/auditing`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/auditing.adoc)

### 概述

infra.persistence.auditing 包提供审计 API、监听器和四个审计注解，自动填充创建时间、创建人、最后修改时间和最后修改人。 审计支持单条插入、批量插入、按主键更新、按主键批量更新、按条件更新，以及 saveOrUpdate 对应的插入或更新操作。

#### 启用审计

使用 EntityManager 自动配置时，审计默认关闭，通过以下配置启用：

```properties
persistence.auditing-enabled=true
```

#### 声明审计属性

```java
@Table("orders")
public class Order {
  public Long id;

  @CreatedDate
  public Instant createdAt;

  @CreatedBy
  public Long createdBy;

  @LastModifiedDate
  public Instant updatedAt;

  @LastModifiedBy
  public Long updatedBy;
}
```

#### 同时记录操作者 ID 和姓名

AuditorAware 可以返回完整用户对象，不同字段通过注解参数选择不同的属性：

## 批量持久化

> 原文：[`data-access/persistence/batch`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/batch.adoc)

### 概述

批量写入通过 persistence.max-batch-records 控制：每个具有相同实体类型和有序写入列的 JDBC PreparedStatement 批次达到该值时，会隐式执行一次 executeBatch()。它不是整个操作的总缓存上限， 也不是事务提交阈值；为 0 时不做中途刷新，输入消费结束后仍会执行所有剩余批次。 BatchExecutionListener 会在每次批次执行前后收到通知。BatchExecution 的 getOperation() 区分插入和更新， getStatement()、getEntityMetadata()、getProperties()、isAutoGenerateId() 和 getEntities() 提供本次执行的上下文。 getAffectedRows() 是同一个 PreparedStatement 已累计的影响行数，不只是本次

#### 主键生成与属性选择

批量插入为每个实体调用 onPrePersist，随后生成尚未赋值的应用主键，再绑定参数并加入 JDBC 批次。 需要生成主键的实体会分别调用自定义 IdGenerator，不会自动合并成一次远程 ID 服务请求。 UUID 或自定义生成型主键始终包含在 INSERT 中；数据库自增主键在批次执行后逐实体回填， 返回主键不足时操作失败并回滚，不发送实体插入成功回调。 每个实体完成前置回调与主键生成后，再依据当前属性值和更新策略选择 INSERT 列。 实体类型和有序属性列表一致的记录复用 PreparedStatement； 仅在首次创建对应批次时生成 SQL。 不同列集合分别进入对应批次，不会因首个实体的空值状态而遗漏后续实体的属性。 批量与单条路径使用相同的注解优先规则：声明 @GeneratedId 时，数据库自增策略请求 JDBC 主键， UUID 和自定义生成器不请求；未声明生成

```java
class LoggingBatchListener implements BatchExecutionListener {

  @Override
  public void preProcessing(BatchExecution execution, boolean implicitExecution) {
    // execution.getStatement(), execution.getEntities()
  }

  @Override
  public void postProcessing(BatchExecution execution, boolean implicitExecution, Throwable ex) {
    if (ex != null) {
      // 处理批次失败
    }
  }
}
```

#### 按主键批量更新

updateById(Iterable<?>) 为每个实体写入不同值，返回累计受影响行数：

```java
int count = entityManager.updateById(entities);
int selectedCount = entityManager.updateById(entities, strategy);
```

## 自动配置与依赖

> 原文：[`data-access/persistence/configuration`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/configuration.adoc)

### 概述

在项目中引入 infra-persistence：

```groovy
dependencies {
  implementation("cn.taketoday:infra-persistence")
}
```

#### 配置属性

PersistenceProperties 绑定到 persistence 前缀： 例如在 application.yml 中设置默认每页 20 条：

```yaml
persistence:
  page-size: 20
```

#### 自定义

可以通过实现以下接口来定制自动配置的组件，它们会被自动收集并应用： 还可以直接注册以下扩展点：

#### 手动创建

不使用容器自动配置时，最小配置是一个 RepositoryManager：

## EntityManager

> 原文：[`data-access/persistence/entity-manager`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/entity-manager.adoc)

### 概述

EntityManager 是持久化的中心 API。数据访问失败通常转换为 DataAccessException， 实体元数据、主键或参数不符合要求时可能抛出 IllegalEntityException、 InvalidDataAccessApiUsageException 或 IllegalArgumentException。应用回调抛出的运行时异常通常保留原类型。 单条操作不会自动开启或提交事务；批量写入通过 RepositoryManager 请求事务，详见 <<persistence-transaction>>。

#### 持久化

```java
// 插入：支持单个实体、集合或 Stream；可指定属性写入策略
int rows = entityManager.persist(user);
entityManager.persist(users);
entityManager.persist(user, PropertyUpdateStrategy.noneNull());
entityManager.persist(userStream);
```

#### 更新

update(Object) 依据实体自身状态更新，可指定更新策略。条件选择顺序为：优先使用实体主键作为 WHERE；当实体没有可用主键时， 则使用标注了 @UpdateBy 的属性值构成 WHERE 条件（此时若没有任何 @UpdateBy 属性，会抛出 InvalidDataAccessApiUsageException）。

```java
// 按实体状态更新（主键作为条件，或 @UpdateBy 属性作为条件），可指定更新策略
int rows = entityManager.update(user);
entityManager.update(user, PropertyUpdateStrategy.noneNull());

// 按主键更新
entityManager.updateById(user);
entityManager.updateById(user, 10L);
entityManager.updateById(user, PropertyUpdateStrategy.noneNull());
entityManager.updateById(user, 10L, PropertyUpdateStrategy.noneNull());

// 按新实体判定选择插入或更新，不查询数据库
entityManager.saveOrUpdate(user);
entityManager.saveOrUpdate(user, PropertyUpdateStrategy.noneNull());
```

#### 删除与清空

#### 查询

find、findFirst、findUnique 和 iterate 可直接接收 QueryStatement；count、page、slice 和 scroll 可直接接收 QueryCondition。两组方法也可通过各自的 Object example 重载转换示例对象，详见 查询条件与示例查询。

## 生命周期事件

> 原文：[`data-access/persistence/events`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/events.adoc)

### 概述

监听器通过泛型参数声明所关注的实体类型（无法解析泛型时监听所有实体），由 EntityManager 在操作期间 也可手动注册到 EntityEventRegistry。 持久化 / 更新的前置回调接收并返回 PropertyUpdateStrategy，决定哪些属性参与写入。 后置回调接收 SQL 参数顺序下的不可变实际属性列表；失败回调也接收该列表， 但前置回调、主键生成、版本递增或属性选择过程中失败时传入 null，不会暴露选择到一半的列表。 选择已完成但没有写入属性时，失败回调可以收到空列表。 失败回调接收 EntityFailureContext，可读取 getMetadata()、getPhase()、getProperties()、getId() 和 getException()。 阶段区分前置处理、SQL 准备、执行和后置处理，不代表事务结果。失败监听器抛出的异常会作为 s

#### 主键生成与回调顺序

UUID 和自定义生成器在 onPrePersist 之后 执行，因此前置回调可以填充生成器需要的实体属性， 也可以自行设置主键。回调结束后已有非空主键时，应用生成器不会执行。 数据库自增主键在 INSERT 执行后读取；前置回调不能依赖它已经就绪。 onPostPersist 在 INSERT 及主键回填处理之后调用，仍处于当前操作的事务上下文中， 不等同于提交成功通知。需要提交后执行的逻辑应使用事务同步机制。 更多说明见 主键与插入流程。

```java
@Component
class UserAuditListener implements UpdateEventListener<User> {

  @Override
  public PropertyUpdateStrategy onPreUpdate(User entity, EntityMetadata metadata, PropertyUpdateStrategy strategy) {
    // 校验或审计；strategy 决定哪些属性会被写回
    return strategy;
  }

  @Override
  public void onPostUpdate(User entity, EntityMetadata metadata, List<EntityProperty> properties, int affectedRows) {
    // 更新后处理
  }
}
```

## 实体映射

> 原文：[`data-access/persistence/mapping`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/mapping.adoc)

### 概述

通过 infra.persistence.annotation 包中的注解描述实体与表之间的映射：

#### 主键生成策略

主键标识与主键生成是两个不同的职责：@Id 只标识主键，@GeneratedId 同时声明主键的生成方式。 GenerationType 与持久化生成器接口 IdGenerator 均位于 infra.persistence 包。

```java
@Table("t_user")
class User {

  @GeneratedId
  Long id;

  String name;
}

@Table("t_document")
class Document {

  @GeneratedUuid
  String id;

  String title;
}
```

##### 自定义生成器与依赖注入

实现 infra.persistence.IdGenerator，返回一个非空且可赋给主键属性的值。 框架负责将返回值设置到实体上，生成器无需直接修改主键属性：

```java
import infra.persistence.EntityMetadata;
import infra.persistence.EntityProperty;
import infra.persistence.IdGenerator;

class OrderIdGenerator implements IdGenerator {

  private final OrderSequenceService sequenceService;

  OrderIdGenerator(OrderSequenceService sequenceService) {
    this.sequenceService = sequenceService;
  }

  @Override
  public Object generateId(Object entity, EntityMetadata metadata, EntityProperty idProperty) {
    return sequenceService.nextId();
  }
}

@Table("t_order")
class Order {

  @GeneratedId(generator = OrderIdGenerator.class)
  Long id;
}
```

##### 分布式 ID

IdGenerator 可以接入本地雪花算法、UUID v7、ULID、数据库号段、Redis 递增或远程 ID 服务。 当前内置策略为 IDENTITY 与 UUID（默认 v7），其他算法通过自定义生成器接入。 多节点唯一性、节点编号、时钟回拨处理和号段分配由生成器及其依赖服务负责。 对雪花或号段等有状态生成器，建议注册为线程安全的单例 Bean，通过名称引用：

##### Meta 注解与属性别名

@GeneratedId 可以作为 Meta 注解使用。框架通过合并注解解析声明，支持 @AliasFor 覆盖属性：

#### 引用实体 (@EntityRef)

标注 @EntityRef 的类型没有自己的表，而是复用被引用实体的主表与主键，常用于"只读查询视图"或"部分字段更新模型"：

#### 元数据

EntityMetadataFactory#getEntityMetadata(Class) 返回并缓存 EntityMetadata，其中包含表名、主键属性 / 列名、 版本属性、属性与列名的对应关系以及主键生成声明。元数据被视为只读，由工厂创建后缓存复用。 getGeneratedId() 返回合成后的 @GeneratedId（包含组合注解覆盖），无声明时返回 null； isAutoGeneratedId() 仅表示数据库自增，不包含 UUID 或自定义应用生成器。生成器实例不保存在共享元数据中。 可以替换其中任意策略：

## 分页

> 原文：[`data-access/persistence/paging`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/paging.adoc)

### 概述

Pageable 描述基于页码和 offset 的分页参数，Page 承载包含总数的分页结果：

```java
Pageable pageable = Pageable.of(2, 10);   // 页码从 1 开始
int offset = pageable.offset();

Page<User> page = entityManager.page(User.class, pageable);
List<User> rows = page.getRows();
Number totalRows = page.getTotalRows();
int totalPages = page.getTotalPages();
boolean hasNext = page.hasNextPage();
```

#### Slice：不统计总数

如果只需要知道是否有下一页，可以使用 slice。框架会读取 pageSize + 1 行，丢弃多读的一行， 以计算 hasNext，因此不会额外执行 count 查询：

```java
Slice<User> slice = entityManager.slice(User.class, userQuery, Pageable.of(1, 20));
for (User user : slice.rows()) {
  process(user);
}
if (slice.hasNext()) {
  Pageable next = Pageable.of(slice.pageNumber() + 1, slice.pageSize());
}
```

## 查询条件与示例查询 (QueryCondition)

> 原文：[`data-access/persistence/query-by-example`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/query-by-example.adoc)

### 概述

面向用户的通用查询抽象是 QueryCondition：它描述“要查什么条件”（collectRestrictions 收集条件、 setParameter 绑定参数、resolveOrderByClause 解析排序）。具体条件如何表达由实现决定， count、page、slice 和 scroll 可直接使用 QueryCondition；find 等查询方法使用 QueryStatement。 默认示例查询实现同时支持两种接口。 示例查询（Query by Example，QBE）是 QueryCondition 的一种：以对象作为“探针（probe）”， 把它的非空属性转换为 WHERE 条件，并可通过注解在属性上精细描述。 空值通常不生成条件（@WhereIsNull 除外），默认等值策略也忽略空白字符串。需要表达“未设置”时尽量使用包装类型： 基本类型默认值（如 0、false

#### 条件注解

例如，定义一个引用 User 主表的查询对象：

```java
@EntityRef(User.class)
class UserQuery {

  @Like
  String name;              // name LIKE ?

  @PrefixLike
  String mobilePhone;       // mobile_phone LIKE ?

  @Where(operator = ">")
  Integer age;              // age > ?

  @OR
  @Where(operator = "<=")
  Integer score;            // ... OR score <= ?

  @OrderBy(Order.DESC)
  @Column(name = "created_at")
  LocalDateTime createdAt;
}

UserQuery query = new UserQuery();
query.name = "Jo";
query.age = 18;
List<User> users = entityManager.find(User.class, query);
```

#### 多对多子查询 (@Subquery)

@Subquery 用于通过连接表查询被引用实体。应将它用在 @EntityRef 示例类型的字段上；没有引用元数据时， 子查询策略不会生成条件，后续策略仍可能生成其他谓词。默认生成：

```sql
targetColumn IN (SELECT select FROM junctionTable WHERE fieldColumn = ?)
```

#### ORDER BY 解析优先级

示例查询的 ORDER BY 按以下优先级解析，跳过空规格（OrderSpec.empty()）；OrderSpecSource 不能返回 null： 主键（@Id）属性与普通属性一样会被扫描，因此同样可以用 @OrderBy 参与排序。例如：

#### 自定义查询工厂

EntityQueryFactories 会按以下顺序选择查询实现：应用显式注册的 EntityQueryFactory、通过策略机制发现的工厂、内置的排序 / Map 工厂，最后才是默认示例查询工厂。 工厂必须实现 createQuery 和 createCondition 两个方法；不支持的示例或用法返回 null，让下一个工厂处理。

## 滚动查询

> 原文：[`data-access/persistence/scrolling`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/scrolling.adoc)

### 概述

scroll 使用 keyset（也称 seek）分页返回一个个窗口，不依赖越来越大的 OFFSET。 它适合导出、后台任务和需要遍历大量记录的接口。每个窗口只多读取一行来判断是否还有下一窗口， 不会执行总数查询。

#### 基本用法

滚动查询必须能够解析出结构化排序键。下面通过 OrderSpecSource 提供排序，通过 ScrollPageable 单独传递游标。 假设 User 映射了非空的 createdAt 和 id，其中数据库列 created_at 对应属性 createdAt：

```java
@EntityRef(User.class)
class UserQuery implements OrderSpecSource {

  @Like
  String name;

  @Override
  public OrderSpec orderSpec() {
    return OrderSpec.desc("created_at"); // 未列出的主键 id 自动追加为 DESC
  }
}

UserQuery query = new UserQuery();
query.name = "Jo";
ScrollPageable request = ScrollPageable.of(100);

Scroll<User> window = entityManager.scroll(User.class, query, request);
while (true) {
  ScrollPosition next = window.nextPosition();
  for (User user : window) {
    process(user);
  }
  if (next == null) {
    break;
  }
  window = entityManager.scroll(User.class, query, request.withPosition(next));
}
```

#### 在 HTTP 请求间继续查询

将 ScrollPosition 留在持久化层与应用之间。下面的 POST /users/scroll 用 name 精确筛选， 由服务端固定按 created_at DESC, id DESC 排序。客户端调用时只需要遵守以下约定： 例如，收到 {"items":["9007199254740994","9007199254740993"],"nextCursor":{"name":"Jo","createdAt":"2026-01-01T00:00:00.123456789Z","id":"9007199254740993"}} 后，续查请求为：

```json
{
  "name": "Jo",
  "limit": 2,
  "cursor": {"name":"Jo","createdAt":"2026-01-01T00:00:00.123456789Z","id":"9007199254740993"}
}
```

#### 由示例对象提供游标

另一种写法是让示例对象实现 ScrollPositionSource，用类型化属性构建位置。 以下按 age ASC, id ASC 查询，两个游标字段要么同时为空，要么同时提供：

## SQL 构建与数据库平台

> 原文：[`data-access/persistence/sql`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/sql.adoc)

### 概述

infra.persistence.sql 包提供了一组可复用的 SQL 片段构建器（移植自 Hibernate 的 SQL 支持）， 包括 Select / SimpleSelect / QuerySelect、Insert / InsertSelect、Update、Delete、 JoinFragment / ANSIJoinFragment、OrderSpec（结构化排序键或原始子句）、CaseFragment 等。

#### Restriction 与 Condition 分层

SQL 谓词被拆成两层，便于渲染与参数绑定： Restrictions 是条件工厂，并提供 WHERE 关键字与连接符的渲染支持：

```java
// 以调用方提供的操作符与右值构造比较
Restriction r = Restrictions.forOperator("age", " >= ", "?");

// 组合：OR 会自动加括号，保持优先级
Restriction where = Restrictions.and(
        Restrictions.like("name"),
        Restrictions.or(Restrictions.isNull("age"), Restrictions.in("status", 3)));

// 渲染到已有缓冲区
StringBuilder buf = new StringBuilder();
Restrictions.append(platform, List.of(where), buf);   // " WHERE ..."
```

#### OrderSpec

OrderSpec 是不可变的排序规格：一组有序的排序键与/或原始 SQL 片段，按顺序渲染。结构化键的列名 以 Identifier 表示，渲染时交给 Platform 决定引号；原始片段原样输出。两者可自由混排。

```java
OrderSpec asc = OrderSpec.asc("name");                        // name ASC
OrderSpec desc = OrderSpec.desc("age");                       // age DESC
OrderSpec of = OrderSpec.of(Pair.of("name", Order.ASC),       // name ASC, age DESC
        Pair.of("age", Order.DESC));
OrderSpec raw = OrderSpec.plain("LENGTH(name) DESC");         // 原始片段，原样渲染

// 增量构建，并可混排结构化键与原始片段
OrderSpec spec = OrderSpec.builder()
        .raw("LENGTH(name) DESC")
        .asc("created_at")
        .build();                                             // LENGTH(name) DESC, created_at ASC

// 从已有规格转出 Builder，继续追加（排序键与原始片段都保留）
OrderSpec extended = spec.mutate().desc("age").build();

// 空规格：不产生 ORDER BY，避免返回 null
OrderSpec empty = OrderSpec.empty();

CharSequence clause = spec.toClause(platform);                // 交由 Platform 方言渲染
boolean noOrdering = spec.isEmpty();
```

#### 数据库平台

infra.persistence.platform.Platform 抽象了数据库方言差异，默认实现为 GenericPlatform （ANSI SQL），MySQL / MariaDB 使用 MySQLPlatform。可按需覆盖：

## 属性更新策略

> 原文：[`data-access/persistence/update-strategy`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/update-strategy.adoc)

### 概述

PropertyUpdateStrategy 决定更新 / 持久化时哪些属性参与写入。它支持用 and / or 组合， 并内置常用策略：

```java
PropertyUpdateStrategy.noneNull();        // 仅非 null 属性（默认）
PropertyUpdateStrategy.always();          // 全部属性
PropertyUpdateStrategy.never();           // 全部忽略
PropertyUpdateStrategy.notId();           // 排除主键
PropertyUpdateStrategy.isId();            // 仅主键
PropertyUpdateStrategy.isEqual(value);    // 值等于给定值
PropertyUpdateStrategy.isNotEqual(value); // 值不等于给定值

// 组合：非 null 且排除主键
PropertyUpdateStrategy strategy = PropertyUpdateStrategy.noneNull()
        .and(PropertyUpdateStrategy.notId());
entityManager.update(user, strategy);
```

## 乐观锁

> 原文：[`data-access/persistence/version`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/persistence/version.adoc)

### 概述

在版本属性上标注 @Version 即可启用乐观锁。更新时框架将新版本写入 SET，并将旧版本加入 WHERE， 影响行数不等于 1 时抛出 OptimisticLockingFailureException。插入不会自动初始化版本；调用方应为新实体设置初始值， 更新时实体必须携带非空的旧版本。 每个实体只允许一个版本属性，否则抛出 IllegalEntityException。 开箱支持的类型：int/Integer、long/Long、short/Short，以及 Instant、LocalDateTime、ZonedDateTime、OffsetDateTime。数值类型递增，时间类型取当前时间。

```java
@Table(name = "t_user")
class User {

  @GeneratedId
  Integer id;

  @Version
  Integer version = 0;
}
```

## 对象关系映射 (ORM) 数据访问

> 原文：[`data-access/orm`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/orm.adoc)

### 概述

本节涵盖了使用对象关系映射 (ORM) 时的数据访问。

## 一般 ORM 集成注意事项

> 原文：[`data-access/orm/general`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/data-access/orm/general.adoc)

### 概述

本节重点介绍适用于所有 ORM 技术的注意事项。 Infra ORM 集成的主要目标是清晰的应用程序分层（使用任何数据访问和事务技术）以及应用程序对象的松散耦合——业务服务不再依赖于数据访问或事务策略，不再有硬编码的资源查找，不再有难以替换的单例，不再有自定义服务注册表。目标是拥有一种简单且一致的方法来连接应用程序对象，使它们尽可能可重用且不受容器依赖项的影响。所有单独的数据访问功能都可以单独使用，但也可以很好地与 Infra 应用程序上下文概念集成，提供基于 XML 的配置和不需要感知 Infra 的普通 JavaBean 实例的交叉引用。在典型的 Infra 应用程序中，许多重要对象都是 JavaBean：数据访问模板、数据访问对象、事务管理器、使用数据访问对象和事务管理器的业务服务、Web 视图解析器、使用业务服务的 Web 控制器等等。

#### 异常转换

当您在 DAO 中使用 Hibernate 或 JPA 时，您必须决定如何处理持久性技术的原生异常类。DAO 会抛出 HibernateException 或 PersistenceException 的子类，具体取决于技术。这些异常都是运行时异常，无需声明或捕获。您可能还需要处理 IllegalArgumentException 和 IllegalStateException。这意味着调用者只能将异常视为通常是致命的，除非他们想依赖于持久性技术本身的异常结构。如果不将调用者绑定到实现策略，就不可能捕获特定原因（例如乐观锁定失败）。 对于强烈基于 ORM 或不需要任何特殊异常处理（或两者兼而有之）的应用程序，这种权衡可能是可以接受的。但是，Infra 允许通过 @Repository 注解透明地应用异常转换。以下示例（一个用于 Java 配置，一个用于 XML 配置）显示了如何执行此操作：

```java
@Repository
public class ProductDaoImpl implements ProductDao {

  // 类体...

}
```

---

> 本文件已达到 60 个示例的上限，其余内容请在 `infra-docs/` 原文查阅。
