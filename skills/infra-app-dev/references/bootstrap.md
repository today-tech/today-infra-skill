# 工程搭建与启动

> 由 `scripts/build-refs.mjs` 从 `infra-docs` 生成，请勿手工编辑；改归类请改 `data/topics.json`。

从零建工程、Gradle 插件、@InfraApplication 启动模型、打包运行。

覆盖文档：38 页（含示例 20 页）。原文位于仓库 `infra-docs/`，下方每个页面标注了路径。

## 页面索引

- **开发您的第一个 Web 应用程序** —— `tutorial:first-application/index`
- 教程 —— `tutorial:index`
- **SQL 数据库** —— `application:data/sql`
- **面向切面编程** —— `application:features/aop`
- **Application** —— `application:features/application`
- 开发期服务 —— `application:features/dev-services`
- **创建自己的自动配置** —— `application:features/developing-auto-configuration`
- **外部化配置** —— `application:features/external-config`
- 国际化 —— `application:features/internationalization`
- **JSON** —— `application:features/json`
- **日志** —— `application:features/logging`
- **Profile** —— `application:features/profiles`
- SSL Bundle —— `application:features/ssl`
- **任务执行和调度** —— `application:features/task-execution-and-scheduling`
- 应用开发 —— `application:index`
- 缓存 —— `application:io/caching`
- 发送邮件 —— `application:io/email`
- **调用 REST 服务** —— `application:io/rest-client`
- **校验** —— `application:io/validation`
- 应用测试支持 —— `application:testing/index`
- Testcontainers —— `application:testing/testcontainers`
- **自动配置** —— `application:using/auto-configuration`
- **Bean 与依赖注入** —— `application:using/beans-and-dependency-injection`
- **配置类** —— `application:using/configuration-classes`
- **开发普通应用（非 Web）** —— `application:using/non-web-applications`
- **运行应用** —— `application:using/running-your-application`
- **组织代码** —— `application:using/structuring-your-code`
- **使用 @InfraApplication** —— `application:using/using-the-infraapplication-annotation`
- 提前处理（AOT） —— `gradle-plugin:aot`
- 构建信息 —— `gradle-plugin:build-info`
- 快速入门 —— `gradle-plugin:getting-started`
- Gradle 插件 —— `gradle-plugin:index`
- 简介 —— `gradle-plugin:introduction`
- 依赖管理 —— `gradle-plugin:managing-dependencies`
- 打包可执行归档 —— `gradle-plugin:packaging`
- 发布应用 —— `gradle-plugin:publishing`
- 与其他插件协作 —— `gradle-plugin:reacting`
- **使用 Gradle 运行应用** —— `gradle-plugin:running`

## 开发您的第一个 Web 应用程序

> 原文：[`tutorial:first-application/index`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/tutorial/pages/first-application/index.adoc)

### 概述

本节介绍如何开发一个小型的 "Hello World!" Web 应用程序，该应用程序重点介绍了该框架的一些关键功能。 您可以选择 Gradle 或 Maven 作为构建系统。

#### 准备

在开始之前，打开终端并运行以下命令以确保您安装了有效版本的 Java：

```shell
$ java -version
openjdk version "17.0.9" 2023-10-17 LTS
OpenJDK Runtime Environment Zulu17.46+19-CA (build 17.0.9+8-LTS)
OpenJDK 64-Bit Server VM Zulu17.46+19-CA (build 17.0.9+8-LTS, mixed mode, sharing)
```

##### Gradle

如果您想使用 Gradle，请确保已安装 Gradle：

```shell
$ gradle --version

------------------------------------------------------------
Gradle 8.7
------------------------------------------------------------

Build time:   2024-03-22 15:52:46 UTC
Revision:     650af14d7653aa949fce5e886e685efc9cf97c10

Kotlin:       1.9.22
Groovy:       3.0.17
Ant:          Apache Ant(TM) version 1.10.13 compiled on January 4 2023
JVM:          17.0.9 (Azul Systems, Inc. 17.0.9+8-LTS)
OS:           Mac OS X 14.4.1 aarch64
```

#### 使用 Gradle 构建

我们需要首先创建一个 Gradle build.gradle 文件。build.gradle 是用于构建项目的构建脚本。

#### 添加 Hello World 代码

和 Spring Boot 一样我们也只需要一个文件就能完成这个简单的 Web 应用。 在 build.gradle 文件所在的同级目录下创建 src/main/java/demo/DemoApplication.java 文件。

#### 使用 Gradle 运行

可以直接使用 gradle infraRun 命令启动。 打开浏览器输入 http://localhost:8080, 你会看到：

#### 使用 Gradle 打包一个可执行 Jar

使用 gradle infraJar 命令构建:

## 教程

> 原文：[`tutorial:index`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/tutorial/pages/index.adoc)

### 概述

本节提供教程，帮助您开始使用这个框架。

## SQL 数据库

> 原文：[`application:data/sql`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/data/sql.adoc)

### 概述

TODAY INFRA 提供 JDBC 访问，包括 infra.jdbc.core.JdbcTemplate、infra.jdbc.core.simple.JdbcClient 和事务支持。 应用可以添加 cn.taketoday:infra-starter-jdbc，它引入 JDBC 集成及 HikariCP。 数据库驱动仍需按实际数据库添加运行时依赖。 基础 API 用法见 JDBC。

##### 连接池

目标优先选择 HikariCP，其次 DBCP2、Oracle UCP，Builder 还具有 C3P0 类型映射。 可以用 datasource.type 指定实现类，也可以用 infra.jdbc.config.DataSourceBuilder 手动创建。 目标没有上游 Tomcat JDBC 连接池的专门自动配置分支，不应照搬 spring.datasource.tomcat.*。 专用属性前缀为 datasource.hikari.、datasource.dbcp2. 和 datasource.oracleucp.*。 例如 HikariCP 最大池大小：

```properties
datasource.hikari.maximum-pool-size=16
```

#### JdbcTemplate 和 JdbcClient

存在单一候选 DataSource 时，可以自动配置 JdbcTemplate。 NamedParameterJdbcTemplate 复用同一个底层模板；多个模板且没有主候选时，不自动选择。 存在单一候选 NamedParameterJdbcTemplate 且没有用户的 JdbcClient 时，可以自动配置 JdbcClient。

```properties
jdbc.template.max-rows=500
jdbc.template.query-timeout=10s
```

## 面向切面编程

> 原文：[`application:features/aop`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/aop.adoc)

### 概述

TODAY INFRA 为 AOP 提供自动配置。关于 AOP 的详细说明，参见 AOP和 AOP API。 默认情况下，自动配置使用基于类的 CGLIB 代理。 如果希望改用 JDK 动态代理，可以设置：

```properties
infra.aop.proxy-target-class=false
```

## Application

> 原文：[`application:features/application`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/application.adoc)

### 概述

infra.app.Application 提供从 main 方法启动应用的入口。 简单应用可以直接调用静态 Application.run(MyApplication.class, args)，完整示例见 。

#### 定制启动过程

需要改变默认设置时，先创建实例，再调用 run：

```java
Application application = new Application(MyApplication.class);
application.setBannerMode(Banner.Mode.OFF);
application.setAdditionalProfiles("local");
application.run(args);
```

## 开发期服务

> 原文：[`application:features/dev-services`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/dev-services.adoc)

### 概述

开发期服务为本地开发提供数据库等外部依赖，不应成为部署环境默认启动容器的要求。 TODAY INFRA 提供 Docker Compose 与 Testcontainers 集成。

## 创建自己的自动配置

> 原文：[`application:features/developing-auto-configuration`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/developing-auto-configuration.adoc)

### 概述

共享库可以把自动配置放在独立 JAR 中，让使用者添加依赖后自动获得默认 Bean。 还可以提供 Starter，集中声明自动配置模块和该技术通常需要的依赖。

#### 配置顺序与替换

通过 @AutoConfiguration 的 before、beforeName、after 和 afterName 控制自动配置相对顺序，也可以使用 AutoConfigureBefore、AutoConfigureAfter。 互不直接依赖的配置可以用 AutoConfigureOrder 给出排序值。 这些设置只决定 Bean 定义的注册顺序；Bean 实例创建顺序由依赖关系和 @DependsOn 等约束决定。 移动或替换自动配置类时，还需兼容旧名称在排序和排除列表中的引用。 创建 META-INF/config/infra.context.annotation.config.AutoConfiguration.replacements：

```properties
com.mycorp.libx.autoconfigure.LibXAutoConfiguration=com.mycorp.libx.autoconfigure.core.LibXAutoConfiguration
```

## 外部化配置

> 原文：[`application:features/external-config`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/external-config.adoc)

### 概述

Adapted from the Spring Boot external configuration documentation and examples. Copyright 2012-present the original author or authors. Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at Unless required by applicable law or agreed to in writing, software distributed under the Licens

#### 属性源与覆盖顺序

标准 Application 启动流程中，常见属性源按以下顺序排列，后面的值覆盖前面的值： 测试中的内联属性、@DynamicPropertySource 和 @TestPropertySource 还可以增加高优先级属性源。 其最终顺序取决于使用的测试引导器和上下文定制器；不能把 Boot 的测试切片和 Devtools 属性源顺序直接套用到 infra。 同样，infra 的标准 Web 环境不会自动增加 Servlet 初始化参数或 java:comp/env JNDI 属性源。 需要其他来源时，应显式配置属性源或环境后处理器。 配置数据内部按以下层次覆盖：打包在 JAR 内的普通配置、JAR 内的 Profile 专用配置、JAR 外的普通配置、JAR 外的 Profile 专用配置。 显式配置位置与位置组可以改变搜索安排，详见下文。 例如，一个组件可以通过 @Value 使用 n

```java
import infra.beans.factory.annotation.Value;
import infra.stereotype.Component;

@Component
public class MyBean {

  @Value("${name}")
  private String name;
}
```

#### 命令行参数

Application 默认把 -- 开头的选项转换为环境属性，例如 --server.port=9000。 如果不希望这样处理，启动前调用 setAddCommandLineProperties(false)。

```shell
java -jar myapp.jar --my.service.name=production
```

#### JSON 属性

可以用 infra.application.json 系统属性或 INFRAAPPLICATIONJSON 环境变量提供一段 JSON：

#### 更改名称和位置

app.config.name 更改配置文件基本名称，不包含扩展名：

#### 导入配置数据

使用 app.config.import 从其他位置导入配置数据：

##### 链式相对导入

假设应用位于 /demo，其 /demo/application.properties 包含：

##### 导入声明的顺序与扩展点

下列两个文档的效果相同：

##### 无扩展名文件

可以使用扩展名提示加载没有扩展名的文件：

##### 从环境变量导入配置

env: 将一个环境变量的完整内容作为配置文档加载，而不是将环境变量名作为单个属性名。 默认按 Java Properties 格式解析：

##### 配置树

配置树把目录和文件名组成属性名，把文件内容作为属性值，适合读取挂载到文件系统的配置或 Secret：

#### 占位符与多文档配置

配置值使用时可通过环境解析 $\{name} 占位符，也可以指定默认值 ${name:default}。 推荐使用小写连字符形式的规范属性名，例如 ${demo.item-price}。 在标准应用环境中，这种写法可以同时匹配配置文件中的 demo.item-price、demo.itemPrice 和环境变量 DEMO_ITEMPRICE。 如果改用 ${demo.itemPrice}，则不会反向匹配配置文件中的 demo.item-price。 占位符可以出现在值的任意位置，也可以引用系统属性或环境变量。 还可以定义 server.port=${port:8080}，以 --port=9000 作为现有属性的短命令行参数。

##### 映射为扁平属性

YAML 适合描述层次化配置。例如：

#### 随机值配置

infra.app.env.RandomValuePropertySource 可以为配置占位符生成随机整数、长整数、UUID 或字符串：

##### 注册与扫描

对于需要按条件启用的配置属性，或者自动配置中使用的类型，可以在配置类上显式注册：

##### JavaBean 绑定

JavaBean 绑定通常使用无参构造器、getter 和 setter。 预先初始化为可变实现的集合、Map，以及预先初始化的嵌套对象，可以省略 setter。 目标还支持公开字段绑定；静态成员不应作为配置属性。 嵌套属性名由成员名决定，而不是由嵌套类型名决定。 下面的类型将 my.service 下的属性绑定为服务配置：

##### 嵌套构造器与默认值

下面的示例同时绑定顶层属性与嵌套安全设置。嵌套对象也通过构造器创建：

##### 使用和第三方对象

配置属性 Bean 可以像其他 Bean 一样注入。 通常应只承载环境配置，不在构造器中注入其他业务 Bean。 也可以在创建第三方对象的 @Bean 方法上使用 @ConfigurationProperties，为返回的对象绑定指定前缀下的属性。 例如，将前面的 MyProperties 注册后，可以使用以下 YAML 配置它：

##### 第三方对象绑定示例

无法修改第三方类型时，可以把注解放到创建该对象的公开 @Bean 方法上。 下面用一个具有 name JavaBean 属性的组件演示这种方式；实际使用时将其替换为第三方库中的类型：

##### 宽松绑定

绑定时支持连字符、驼峰和下划线等常见命名形式，例如 first-name、firstName、first_name。 注解的前缀必须使用小写连字符形式。推荐配置文件也使用这一形式。 例如下面的配置类型：

##### Map 的特殊键

Map 的特殊键可以用方括号保留，例如 YAML 中的 "[/key1]"。 没有方括号时，键中的部分非字母数字字符会被删除。 对于 Map<String, String>，点可以保留为键的一部分；对于 Map<String, Object>，不加括号的点通常表示嵌套结构。 宽松绑定默认缓存不可变属性源；需要控制缓存时使用 infra.context.properties.source.ConfigurationPropertyCaching。 例如，以下 YAML 绑定到 Map<String, String>：

##### 环境变量中的 Map

##### 集合合并规则

多个来源中的列表按最高优先级来源整体替换，不追加元素，也不合并同一索引对象的字段。 Map 则可以从多个来源合并不同键，同一属性由更高优先级的值覆盖。 这些规则适用于全部属性源，不限于 Profile 配置文件。 下面的类型同时展示可变列表与 Map 的可绑定入口；两个集合也可以分别放到独立的配置类型中：

##### Duration 示例

下面的不可变配置属性类型通过构造器绑定超时值。 使用前需要通过 @EnableConfigurationProperties 或配置属性扫描注册类型，并保留编译参数 -parameters：

##### DataSize 示例

容量属性同样支持构造器绑定与默认单位：

##### Base64 数据

配置属性为 infra.core.io.Resource 时，可以使用 base64: 前缀提供编码后的二进制内容：

##### 校验

存在 Jakarta Bean Validation 实现时，可以在配置属性类型上声明 infra.validation.annotation.Validated，并为成员添加约束。 嵌套对象的级联校验使用 jakarta.validation.Valid。 也可以提供名为 configurationPropertiesValidator 的 infra.validation.Validator Bean；它的工厂方法应是静态的，避免过早创建配置类。 下面的配置类型要求提供远端地址，并级联校验嵌套的用户名：

## 国际化

> 原文：[`application:features/internationalization`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/internationalization.adoc)

### 概述

TODAY INFRA 支持本地化消息，使应用能够服务于使用不同语言的用户。 默认情况下，自动配置在类路径根目录查找名为 messages 的资源包。 NOTE: 只有配置的资源包存在默认属性文件（默认是 messages.properties）时，消息源自动配置才会生效。如果只有 messageszhCN.properties 等语言专用文件，仍需要添加默认文件。如果全部配置的基本名称都找不到默认属性文件，就不会自动配置 MessageSource。 资源包的基本名称和其他选项通过 messages 命名空间配置： messages: basename 支持以逗号分隔的多个基本名称，可以包含包限定名；没有包限定名时从类路径根目录解析。 common-messages 支持多个属性文件资源，用于提供与 Locale 无关的通用消息。

## JSON

> 原文：[`application:features/json`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/json.adoc)

### 概述

TODAY INFRA 提供 Jackson 3、Gson 和 JSON-B 的自动配置。 各自动配置必须随对应模块进入类路径；仅添加某个底层 JSON 库，并不代表其自动配置模块已经存在。

#### Gson

使用 cn.taketoday:infra-starter-gson 引入相应集成。 自动配置在不存在用户提供的匹配 Bean 时创建 com.google.gson.GsonBuilder 和 Gson。 属性由 infra.gson.config.GsonProperties 定义，使用 gson.* 前缀。 可以通过 infra.gson.config.GsonBuilderCustomizer Bean 定制 Builder。

```properties
gson.pretty-printing=true
gson.serialize-nulls=true
```

## 日志

> 原文：[`application:features/logging`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/logging.adoc)

### 概述

TODAY INFRA 内部使用 infra.logging 日志抽象，底层日志实现可替换。 应用日志初始化支持 Logback 和 Java Util Logging。 添加 cn.taketoday:infra-starter-logback 可以获得 Logback 集成；具体桥接依赖应以所选 Starter 的依赖集合为准。 目标没有与上游同等的 Log4j2 应用日志初始化模块，因此不能直接采用上游 Log4j2 自动配置和扩展说明。

#### Logback 扩展

目标扩展标签名为 infra-profile 和 infra-property，不是 springProfile、springProperty。 在 logback-infra.xml 或由 logging.config 指定并交给框架加载的配置中使用：

```xml
<infra-profile name="dev | staging">
  <!-- 开发和预发布环境的日志设置 -->
</infra-profile>

<infra-property scope="context" name="remoteHost"
    source="myapp.logging.host" defaultValue="localhost"/>
```

## Profile

> 原文：[`application:features/profiles`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/profiles.adoc)

### 概述

Profile 用于把应用配置分成不同组，仅在指定环境中启用。 可以在组件、配置类或参与配置属性扫描的配置属性类上使用 infra.context.annotation.Profile 限制加载条件。 例如，下面的配置类只有在 production Profile 激活时才会加载：

```java
import infra.context.annotation.Configuration;
import infra.context.annotation.Profile;

@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfig {
}
```

#### 编程方式设置 Profile

可以在启动前调用 infra.app.Application.setAdditionalProfiles(...) 增加 Profile。 它增加额外的 Profile，不替换 infra.profiles.active 指定的列表：

```java
import infra.app.Application;
import infra.context.annotation.Configuration;
import infra.context.annotation.config.EnableAutoConfiguration;

@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
public class MyApplication {

  public static void main(String[] args) {
    Application application = new Application(MyApplication.class);
    application.setAdditionalProfiles("common", "local");
    application.run(args);
  }
}
```

## SSL Bundle

> 原文：[`application:features/ssl`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/ssl.adoc)

### 概述

SSL Bundle 为可复用的密钥和信任材料提供命名配置。 目标使用 ssl.bundle.* 前缀，每个 Bundle 名称由应用自行指定。

## 任务执行和调度

> 原文：[`application:features/task-execution-and-scheduling`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/features/task-execution-and-scheduling.adoc)

### 概述

没有用户定义的 java.util.concurrent.Executor Bean 时，自动配置提供名为 applicationTaskExecutor 的异步任务执行器。 默认使用 infra.scheduling.concurrent.ThreadPoolTaskExecutor。 Java 21 及以上且 infra.threads.virtual.enabled=true 时，改用支持虚拟线程的 infra.core.task.SimpleAsyncTaskExecutor。

#### 异步执行

使用 infra.scheduling.annotation.EnableAsync 启用异步方法处理。 自动配置在不存在用户 AsyncConfigurer 时提供一个配置器，使异步任务使用 applicationTaskExecutor。 用户提供自己的 infra.scheduling.annotation.AsyncConfigurer 时，可以明确控制异步方法的执行器。 默认情况下，用户提供 Executor Bean 会让默认执行器配置退让。 需要保留用户执行器并同时创建默认执行器时，可以设置：

```properties
task.execution.mode=force
```

## 应用开发

> 原文：[`application:index`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/index.adoc)

### 概述

本节介绍使用 TODAY INFRA 开发应用时的代码组织、依赖注入和自动配置，以及配置加载、日志、SSL、JDBC、REST 客户端和应用测试等集成功能。 开发命令行工具、定时任务或后台工作进程，可以从 开发普通应用（非 Web）开始。 开发 Web 服务，可以从 第一个 Web 应用程序开始。 这两个入门示例都介绍了应用启动和可执行 JAR 的构建方式。

## 缓存

> 原文：[`application:io/caching`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/io/caching.adoc)

### 概述

TODAY INFRA 的缓存抽象将缓存逻辑应用于方法调用。 命中缓存时直接返回已有结果，未命中时执行方法并缓存结果，调用者无需自己操作存储。 详细注解和缓存键规则见 缓存抽象。 使用 infra.cache.annotation.EnableCaching 启用注解缓存，使用 Cacheable、CachePut 和 CacheEvict 分别缓存、更新和清理结果。 推荐把启用注解放在独立配置类中，避免所有测试都强制启用缓存。 代理模式下只有经过代理的方法调用才受缓存拦截，同一对象内部的直接调用不会经过缓存代理。

## 发送邮件

> 原文：[`application:io/email`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/io/email.adoc)

### 概述

TODAY INFRA 使用 infra.mail.javamail.JavaMailSender 抽象发送邮件，并提供相应自动配置。 具体邮件 API 用法见 邮件集成。 应用可以添加 cn.taketoday:infra-starter-mail，引入邮件模块及 Jakarta Mail 实现。 当 mail.host 已配置，且类路径中存在邮件所需的类型时，自动配置在没有用户提供的 infra.mail.MailSender Bean 的情况下创建默认邮件发送器。 mail.jndi-name 也可以触发自动配置。 邮件相关选项由 infra.mail.config.MailProperties 定义，使用 mail 命名空间，而不是 spring.mail 或 infra.mail。 mail: TIP: 某些邮件协议超时默认值为无限。显式设置连接、读取和写入超时，可以避免无响应的

## 调用 REST 服务

> 原文：[`application:io/rest-client`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/io/rest-client.adoc)

### 概述

TODAY INFRA 提供命令式 infra.restclient.RestClient 和响应式 infra.web.reactive.client.WebClient。 普通阻塞应用通常选择 RestClient；使用 Reactor 的非阻塞应用可以选择 WebClient。 底层 API 的详细说明见 REST 客户端。

#### 限制 HTTP 客户端可访问的 IP 地址

如果应用需要请求由用户提供的 URL，可以使用 InetAddressFilter 限制目标地址，降低服务端请求伪造（SSRF）的风险。 infra.http.client.InetAddressFilter 支持精确 IP 地址、CIDR 网段以及 and、or、andNot 和 negate 组合操作。 例如，InetAddressFilter.externalAddresses() 可排除私有地址、组播地址以及 RFC 6890 的特殊用途地址。 为单个客户端配置过滤器：

```java
HttpClientSettings settings = HttpClientSettings.defaults()
        .withInetAddressFilter(InetAddressFilter.externalAddresses());
RestClient client = RestClient.builder()
        .requestFactory(ClientHttpRequestFactoryBuilder.jdk().build(settings))
        .baseUrl("https://example.org")
        .build();
```

## 校验

> 原文：[`application:io/validation`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/io/validation.adoc)

### 概述

当类路径中存在 Jakarta Bean Validation 的方法校验 API 和可发现的实现（例如 Hibernate Validator）时，自动配置可以启用方法校验。 可以在 Bean 方法的参数和返回值上添加 jakarta.validation 约束。 需要在目标类上声明 infra.validation.annotation.Validated，以启用对该 Bean 的方法约束检测。 例如，以下服务限制第一个参数的长度为 8 到 10（包含边界）：

```java
import infra.stereotype.Service;
import infra.validation.annotation.Validated;
import jakarta.validation.constraints.Size;

@Service
@Validated
public class MyBean {

  public Archive findByCodeAndAuthor(@Size(min = 8, max = 10) String code, Author author) {
    return null;
  }

}
```

## 应用测试支持

> 原文：[`application:testing/index`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/testing/index.adoc)

### 概述

cn.taketoday:infra-starter-test 提供应用测试常用依赖，应放在测试范围。 Gradle 使用 testImplementation，Maven 使用 test scope。 它包括 JUnit Jupiter、infra 测试设施、AssertJ、Mockito、JSONassert、JsonPath、Awaitility 和 XMLUnit 等。 Starter 当前也包含 Hamcrest，但本仓库测试约定使用 AssertJ 和 Mockito，示例遵循该约定。 基础上下文测试见 TestContext Framework。 应用级入口是 infra.test.app.context.InfraTest，不是 SpringBootTest。

## Testcontainers

> 原文：[`application:testing/testcontainers`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/testing/testcontainers.adoc)

### 概述

Testcontainers 可以为集成测试管理 Docker 容器，让测试连接真实后端服务。 容器库本身提供 JUnit Jupiter 扩展，也可以与 TODAY INFRA 的应用测试和容器生命周期支持组合使用。

## 自动配置

> 原文：[`application:using/auto-configuration`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/using/auto-configuration.adoc)

### 概述

自动配置根据应用类路径中的依赖、已有 Bean 和配置属性，尝试提供所需的默认配置。 例如，存在 Netty 服务器相关依赖且应用没有定义自己的服务器工厂时，对应自动配置可以提供服务器工厂。 在主配置类上使用 infra.context.annotation.config.EnableAutoConfiguration 或 infra.app.InfraApplication 启用自动配置。 通常只在主配置类上声明其中一个注解，不要在多个配置类上重复声明。

#### 禁用特定自动配置

使用 @InfraApplication 的 exclude 属性指定要排除的自动配置类。 如果不能直接引用该类，可以使用 excludeName 指定完全限定类名。 @EnableAutoConfiguration 同样提供这两个属性。 也可以通过 infra.auto-configuration.exclude 配置排除列表：

```properties
infra.auto-configuration.exclude=infra.context.config.MessageSourceAutoConfiguration
```

## Bean 与依赖注入

> 原文：[`application:using/beans-and-dependency-injection`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/using/beans-and-dependency-injection.adoc)

### 概述

可以使用容器支持的标准方式定义 Bean 及其依赖。 通常推荐构造器注入，并使用 @ComponentScan 发现组件。 按照 推荐布局把主类放在根包后，可以使用不带参数的 @ComponentScan，或使用已经包含组件扫描的 @InfraApplication。 扫描范围内的 infra.stereotype.Component、Service、Repository 和 Controller 等组件注解类会注册为 Bean。 下面的服务通过构造器获得所需的 RiskAssessor：

```java
import infra.stereotype.Service;

@Service
public class MyAccountService implements AccountService {

  private final RiskAssessor riskAssessor;

  public MyAccountService(RiskAssessor riskAssessor) {
    this.riskAssessor = riskAssessor;
  }

  // ...
}
```

## 配置类

> 原文：[`application:using/configuration-classes`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/using/configuration-classes.adoc)

### 概述

TODAY INFRA 推荐基于 Java 的配置。 虽然 infra.app.Application 可以加载 XML 配置源，通常仍建议以一个 infra.context.annotation.Configuration 配置类作为主配置源。 包含 main 方法的类通常适合作为主配置类；@InfraApplication 已包含配置类语义。 TIP: 网上许多配置示例使用 XML。条件允许时，优先寻找等价的 Java 配置；Enable* 注解通常是一个好的起点。

#### 导入其他配置类

不必把全部配置放在同一个类中。 使用 infra.context.annotation.Import 导入其他配置类，也可以通过 infra.context.annotation.ComponentScan 自动发现组件，包括 @Configuration 类。

```java
package com.example.myapplication;

import infra.context.annotation.Configuration;
import infra.context.annotation.Import;

@Configuration(proxyBeanMethods = false)
@Import(DatabaseConfiguration.class)
public class ApplicationConfiguration {
}
```

#### 导入 XML 配置

如果必须使用 XML，仍建议从 @Configuration 类启动，再使用 infra.context.annotation.ImportResource 加载 XML 文件。

```java
@Configuration(proxyBeanMethods = false)
@ImportResource("classpath:application-context.xml")
public class XmlConfiguration {
}
```

## 开发普通应用（非 Web）

> 原文：[`application:using/non-web-applications`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/using/non-web-applications.adoc)

### 概述

TODAY INFRA 可以用于开发命令行工具、数据处理程序、定时任务和后台工作进程。 这些应用通过 main 方法启动，同样可以使用依赖注入、自动配置、外部化配置和日志等能力。 本节从一个运行后退出的命令行应用开始，再介绍如何改为持续运行的后台应用。 完整命令行示例位于仓库的 infra-samples/hello-cli 目录。 在 infra-samples 中执行 ./gradlew :hello-cli:infraRun 即可运行；参数传递和打包命令见该示例的 README.md。

#### 创建项目

准备 JDK 17 或更高版本，在项目根目录创建以下文件： 在 settings.gradle 中指定项目名称：

```groovy
rootProject.name = 'hello-cli'
```

#### 编写应用入口

创建 src/main/java/com/example/cli/CliApplication.java：

```java
package com.example.cli;

import infra.app.Application;
import infra.app.ApplicationRunner;
import infra.app.InfraApplication;
import infra.context.annotation.Bean;

@InfraApplication
public class CliApplication {

  public static void main(String[] args) {
    try (var context = Application.forNormalApplication(CliApplication.class).run(args)) {
      // 启动任务已经执行完成，离开作用域时关闭应用上下文。
    }
  }

  @Bean
  ApplicationRunner greet(GreetingService greetingService) {
    return args -> {
      if (args.getNonOptionArgs().isEmpty()) {
        greetingService.greet("World");
      }
      else {
        for (String name : args.getNonOptionArgs()) {
          greetingService.greet(name);
        }
      }
    };
  }
}
```

##### 添加业务组件

创建 src/main/java/com/example/cli/GreetingService.java：

#### 读取配置和命令行参数

在 src/main/resources/application.properties 中添加默认配置：

##### 返回进程退出码

需要把执行结果交给 Shell 或调度平台时，可以用以下方式替换 main 方法： 该片段需要额外导入 infra.context.ConfigurableApplicationContext。

#### 开发常驻后台应用

定时任务、消息消费等应用需要在启动后持续运行。 此时应保留应用上下文，并根据需要启用保活，而不是在 main 方法返回前关闭上下文：

##### 添加定时任务

为 CliApplication 添加 infra.scheduling.annotation.EnableScheduling 注解：

#### 打包与部署

使用应用插件生成包含运行依赖的可执行 JAR：

## 运行应用

> 原文：[`application:using/running-your-application`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/using/running-your-application.adoc)

### 概述

TODAY INFRA 应用可以像普通 Java 应用一样运行和调试，无需专用 IDE 插件。 本节介绍 JAR 应用的本地开发运行方式。 命令行工具、定时任务和后台工作进程的完整示例见 。

#### 运行打包后的应用

使用 TODAY INFRA 的构建插件生成可执行 JAR 后，可以通过 java -jar 运行：

```shell
java -jar build/libs/myapplication-0.0.1-SNAPSHOT.jar
```

#### 使用 Gradle 插件

应用 infra.application 和 java 插件后，可以使用 infraRun 任务从未打包的类目录运行应用：

```shell
./gradlew infraRun
```

## 组织代码

> 原文：[`application:using/structuring-your-code`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/using/structuring-your-code.adoc)

### 概述

TODAY INFRA 不要求特定的代码目录布局，但遵循以下实践有助于限制扫描范围并保持结构清晰。

#### 放置主应用类

建议把主应用类放在其他应用类所在包的上层根包中。 主类通常使用 @InfraApplication；其中的组件扫描默认从主类所在包开始。 根包布局能使组件扫描集中在自己的项目中。 自动配置包由 @EnableAutoConfiguration 注册，供需要默认扫描包的自动配置使用；它与组件扫描包是不同的概念。 如果不使用 @InfraApplication，可以单独声明 @EnableAutoConfiguration 和 @ComponentScan。 MyApplication.java 声明 main 方法和应用注解：

```java
package com.example.myapplication;

import infra.app.Application;
import infra.app.InfraApplication;

@InfraApplication
public class MyApplication {

  public static void main(String[] args) {
    Application.run(MyApplication.class, args);
  }

}
```

## 使用 @InfraApplication

> 原文：[`application:using/using-the-infraapplication-annotation`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/application/pages/using/using-the-infraapplication-annotation.adoc)

### 概述

许多应用希望同时启用自动配置、组件扫描，并在主应用类中定义额外 Bean。 infra.app.InfraApplication 将这三个功能组合在一个注解中：

```java
package com.example.myapplication;

import infra.app.Application;
import infra.app.InfraApplication;

@InfraApplication
public class MyApplication {

  public static void main(String[] args) {
    Application.run(MyApplication.class, args);
  }

}
```

#### 分别声明所需功能

这些功能不是强制的，可以使用各自的注解组合代替 @InfraApplication。 例如，下面的应用启用自动配置，但不进行组件扫描，而是显式导入用户配置：

```java
package com.example.myapplication;

import infra.app.Application;
import infra.app.InfraConfiguration;
import infra.context.annotation.Import;
import infra.context.annotation.config.EnableAutoConfiguration;

@InfraConfiguration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import(UserConfiguration.class)
public class MyApplication {

  public static void main(String[] args) {
    Application.run(MyApplication.class, args);
  }

}
```

## 提前处理（AOT）

> 原文：[`gradle-plugin:aot`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/aot.adoc)

### 概述

Infra AOT 在构建时分析代码，并生成经过优化的代码。 它最常用于辅助生成 GraalVM 原生镜像。 Infra Gradle 插件内置对应用代码和测试代码执行 AOT 处理的支持，但默认不启用。 无需应用独立的 AOT 插件 ID；要在 JVM 应用中使用 AOT，请在应用 cn.taketoday.application 后显式启用： infraApplication { } enableAheadOfTimeProcessing() 在项目配置阶段接入 AOT 源集和处理任务，重复调用不会重复注册。 启用后，AOT 输出会加入主源集和测试源集的运行时类路径，消费这些输出的运行、测试或打包任务可能触发 AOT 处理。 此配置不要求应用 GraalVM Native Image 插件；AOT 处理与运行时使用 AOT 初始化是不同的步骤。 应用 {nbt-gradle-plugi

## 构建信息

> 原文：[`gradle-plugin:build-info`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/build-info.adoc)

### 概述

## 快速入门

> 原文：[`gradle-plugin:getting-started`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/getting-started.adoc)

### 概述

要开始使用此插件，需要先将其应用到项目中。 相应版本在 Gradle 插件门户发布并通过审核后，即可使用 plugins 块应用此插件： 插件门户中的 ID 为 cn.taketoday.application，包含可按需启用的 AOT 支持，不单独发布 AOT 插件 ID。 Gradle 插件门户仅接收非快照版本，不提供 cn.taketoday.application 的快照版本。 通过 buildscript 或 buildSrc 加载 cn.taketoday:infra-gradle-plugin 时， 仍可使用兼容 ID infra.application；独立的 infra.application.aot ID 已移除。 请通过 infraApplication.enableAheadOfTimeProcessing() 启用 AOT，详见 AOT 配置入口。 如需使用快照版

## Gradle 插件

> 原文：[`gradle-plugin:index`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/index.adoc)

### 概述

TODAY Infrastructure Gradle 插件用于构建、运行和发布应用，并提供依赖管理和 AOT 处理支持。 本手册由 Andy Wilkinson、Scott Frederick 和 Harry Yang 编写，现统一纳入框架参考文档。

## 简介

> 原文：[`gradle-plugin:introduction`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/introduction.adoc)

### 概述

Infra Gradle 插件为 Gradle 提供 Infra 支持。 它允许你打包可执行 jar 或 war 归档、运行 Infra 应用，以及使用 infra-dependencies 提供的依赖管理。 目前尚未声明支持配置缓存（Configuration Cache）。 除本用户指南外，还提供了 API 文档[API 文档]。

## 依赖管理

> 原文：[`gradle-plugin:managing-dependencies`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/managing-dependencies.adoc)

### 概述

Infra 插件默认自动应用 io.spring.dependency-management[io.spring.dependency-management] 插件并导入 Infra BOM，无需额外声明该插件的依赖、应用它或重复导入 BOM。 也可以关闭插件自动应用后，使用 Gradle 原生的 BOM 支持。 前者的主要优势在于可以通过属性自定义受管理的版本，而后者通常能带来更快的构建速度。

## 打包可执行归档

> 原文：[`gradle-plugin:packaging`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/packaging.adoc)

### 概述

此插件可以创建包含应用所有依赖的可执行归档（jar 文件和 war 文件），并通过 java -jar 运行。

## 发布应用

> 原文：[`gradle-plugin:publishing`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/publishing.adoc)

### 概述

## 与其他插件协作

> 原文：[`gradle-plugin:reacting`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/reacting.adoc)

### 概述

应用其他插件时，Infra 插件会相应地调整项目配置。 默认情况下，Infra 插件会自动应用 java 和 io.spring.dependency-management 插件。 如果项目中存在 skipAutoApplyPlugins 属性，则不自动应用它们，而是在这些插件被显式应用时执行相应配置。 war、application 和 GraalVM Native Image 插件始终需要显式应用。 本节介绍这些配置变化。

## 使用 Gradle 运行应用

> 原文：[`gradle-plugin:running`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/gradle-plugin/pages/running.adoc)

### 概述

要在不预先构建归档的情况下运行应用，请使用 infraRun 任务：

```bash
./gradlew infraRun
```

#### 向应用传递参数

与所有 JavaExec 任务一样，在 Gradle 4.9 及更高版本中，可以在命令行中使用 --args='<arguments>' 向 infraRun 传递参数。 例如，可以使用以下命令运行应用并激活名为 dev 的 profile（环境配置）：

```bash
./gradlew infraRun --args='--infra.profiles.active=dev'
```

#### 向应用传递系统属性

由于 infraRun 是标准的 JavaExec 任务，因此可以在构建脚本中指定系统属性，将其传递给应用的 JVM。 要使系统属性的值可配置，可以通过Gradle DSL/org.gradle.api.Project.html#N14FE1[项目属性]设置它。 如果项目属性是可选的，请使用 findProperty 获取其值。 这样还可以通过 ?: Elvis 运算符提供默认值，如下例所示： 上例将 com.example.property 系统属性设置为 example 项目属性的值。 如果未设置 example 项目属性，则系统属性的值为 default。 Gradle 支持多种设置项目属性的方式，包括在命令行中使用 -P 选项，如下例所示：

---

> 本文件已达到 60 个示例的上限，其余内容请在 `infra-docs/` 原文查阅。
