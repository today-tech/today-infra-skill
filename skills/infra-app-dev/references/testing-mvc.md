# Web 测试

> 由 `scripts/build-refs.mjs` 从 `infra-docs` 生成，请勿手工编辑；改归类请改 `data/topics.json`。

MockMvc 服务端配置、请求执行与断言、WebTestClient。

覆盖文档：13 页（含示例 8 页）。原文位于仓库 `infra-docs/`，下方每个页面标注了路径。

## 页面索引

- MockMvc —— `testing/mvc-test-framework`
- **异步请求** —— `testing/mvc-test-framework/async-requests`
- **定义期望** —— `testing/mvc-test-framework/server-defining-expectations`
- **Filter 注册** —— `testing/mvc-test-framework/server-filters`
- **执行请求** —— `testing/mvc-test-framework/server-performing-requests`
- 更多示例 —— `testing/mvc-test-framework/server-resources`
- **设置选择** —— `testing/mvc-test-framework/server-setup-options`
- **设置特性** —— `testing/mvc-test-framework/server-setup-steps`
- 静态导入 —— `testing/mvc-test-framework/server-static-imports`
- 概述 —— `testing/mvc-test-framework/server`
- MockMvc 与端到端测试 —— `testing/mvc-test-framework/vs-end-to-end-integration-tests`
- **流式响应** —— `testing/mvc-test-framework/vs-streaming-response`
- **WebTestClient** —— `testing/webtestclient`

## MockMvc

> 原文：[`testing/mvc-test-framework`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework.adoc)

### 概述

Web MVC Test 框架，也被称为 MockMvc，为测试 Infra MVC 应用程序提供支持。它执行完整的 Web MVC 请求处理，但是通过 mock 请求和响应对象，而不是通过运行中的服务器。 可以直接使用 MockMvc 发起请求并验证响应，也可以使用 MockMvcTester 编写 AssertJ 断言。 WebTestClient 也可以把 MockMvc 用作请求处理后端； 如果需要针对运行中的服务器进行端到端 HTTP 测试，可以切换后端并保留同一套 WebTestClient API。

## 异步请求

> 原文：[`testing/mvc-test-framework/async-requests`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/async-requests.adoc)

### 概述

本节展示了如何单独使用 MockMvc 测试异步请求处理。 如果通过 WebTestClient 使用 MockMvc，则无需执行任何特殊操作即可使异步请求工作，因为 WebTestClient 会自动执行本节中描述的操作。 Servlet 异步请求（在 Web MVC 中受支持）的工作原理是退出 Servlet 容器线程并允许应用程序异步计算响应，之后进行异步分派以在 Servlet 容器线程上完成处理。 在 Web MVC Test 中，可以通过首先断言生成的异步值，然后手动执行异步分派，最后验证响应来测试异步请求。 下面是一个针对返回 DeferredResult、Callable 或响应式类型（如 Reactor Mono）的控制器方法的示例测试：

```java
// MockMvcRequestBuilders.* 和 MockMvcResultMatchers.* 的静态导入

@Test
void test() throws Exception {
    MvcResult mvcResult = this.mockMvc.perform(get("/path"))
          .andExpect(status().isOk()) <1>
          .andExpect(request().asyncStarted()) <2>
          .andExpect(request().asyncResult("body")) <3>
          .andReturn();

    this.mockMvc.perform(asyncDispatch(mvcResult)) <4>
          .andExpect(status().isOk()) <5>
          .andExpect(content().string("body"));
}
```

## 定义期望

> 原文：[`testing/mvc-test-framework/server-defining-expectations`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server-defining-expectations.adoc)

### 概述

你可以通过在执行请求后追加一个或多个 andExpect(..) 调用来定义期望，如下例所示。一旦有一个期望失败，就不会断言其他期望。

```java
// MockMvcRequestBuilders.* 和 MockMvcResultMatchers.* 的静态导入
mockMvc.perform(get("/accounts/1")).andExpect(status().isOk());
```

## Filter 注册

> 原文：[`testing/mvc-test-framework/server-filters`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server-filters.adoc)

### 概述

当设置 MockMvc 实例时，你可以注册一个或多个 Filter 实例，如下例所示：

```java
	mockMvc = standaloneSetup(new PersonController()).addFilters(new CharacterEncodingFilter()).build();
```

## 执行请求

> 原文：[`testing/mvc-test-framework/server-performing-requests`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server-performing-requests.adoc)

### 概述

本节展示了如何单独使用 MockMvc 执行请求并验证响应。 如果通过 WebTestClient 使用 MockMvc，请参阅 编写测试 中的相应部分。 要执行使用任何 HTTP 方法的请求，如下例所示：

```java
// MockMvcRequestBuilders.* 的静态导入

mockMvc.perform(post("/hotels/{id}", 42).accept(MediaType.APPLICATION_JSON));
```

## 更多示例

> 原文：[`testing/mvc-test-framework/server-resources`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server-resources.adoc)

### 概述

框架自身的测试包括 许多示例测试，旨在展示如何单独使用 MockMvc 或通过 WebTestClient 使用 MockMvc。浏览这些示例以获取更多思路。

## 设置选择

> 原文：[`testing/mvc-test-framework/server-setup-options`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server-setup-options.adoc)

### 概述

MockMvc 可以通过两种方式之一进行设置。一种是直接指向你要测试的控制器，并以编程方式配置 Web MVC 基础设施。第二种是指向包含 Web MVC 和控制器基础设施的 Infra 配置。 要设置 MockMvc 以测试特定控制器，请使用以下代码：

```java
class MyWebTests {

  MockMvc mockMvc;

  @BeforeEach
  void setup() {
    this.mockMvc = MockMvcBuilders.standaloneSetup(new AccountController()).build();
  }

  // ...

}
```

#### 使用 MockMvcTester 编写 AssertJ 断言

MockMvcTester 使用同一套 MockMvc 请求处理机制，提供 AssertJ 风格的断言。以下独立测试验证控制器的响应状态和文本：

```java
import org.junit.jupiter.api.Test;

import infra.test.web.mock.assertj.MockMvcTester;
import infra.web.annotation.GET;
import infra.web.annotation.RestController;

import static org.assertj.core.api.Assertions.assertThat;

class GreetingTests {

  private final MockMvcTester mvc = MockMvcTester.of(new GreetingController());

  @Test
  void greeting() {
    assertThat(mvc.get().uri("/greet"))
        .hasStatusOk()
        .hasBodyTextEqualTo("Hello");
  }

  @RestController
  static class GreetingController {

    @GET("/greet")
    public String greet() {
      return "Hello";
    }
  }
}
```

## 设置特性

> 原文：[`testing/mvc-test-framework/server-setup-steps`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server-setup-steps.adoc)

### 概述

无论你使用哪种 MockMvc 构建器，所有 MockMvcBuilder 实现都提供了一些通用且非常有用的功能。例如，你可以为所有请求声明一个 Accept 头，并期望所有响应的状态为 200 以及 Content-Type 头，如下所示：

```java
// MockMvcBuilders.standaloneSetup 的静态导入

MockMvc mockMvc = standaloneSetup(new MusicController())
  .defaultRequest(get("/").accept(MediaType.APPLICATION_JSON))
  .alwaysExpect(status().isOk())
  .alwaysExpect(content().contentType("application/json;charset=UTF-8"))
  .build();
```

## 静态导入

> 原文：[`testing/mvc-test-framework/server-static-imports`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server-static-imports.adoc)

### 概述

当直接使用 MockMvc 执行请求时，你需要以下静态导入： - MockMvcBuilders.* - MockMvcRequestBuilders.* - MockMvcResultMatchers.* - MockMvcResultHandlers.* 记住这一点的一个简单方法是搜索 MockMvc*。如果使用 Eclipse，请务必也将上述内容添加为 Eclipse 首选项中的“favorite static members”。 当通过 WebTestClient 使用 MockMvc 时，你不需要静态导入。WebTestClient 提供了一个流畅的 API，无需静态导入。

## 概述

> 原文：[`testing/mvc-test-framework/server`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/server.adoc)

### 概述

你可以通过实例化一个控制器、注入依赖并调用其方法来为 Web MVC 编写普通的单元测试。然而，这样的测试不会验证请求映射、数据绑定、消息转换、类型转换、验证，也不会涉及任何支持性的 @InitBinder、@ModelAttribute 或 @ExceptionHandler 方法。 Web MVC Test 框架（也称为 MockMvc）旨在为 Web MVC 控制器提供更完整的测试，而无需运行服务器。它通过调用 DispatcherHandler 并传递来自 infra-test 模块的 Mock API 的“mock”实现 来实现这一点，从而在没有运行服务器的情况下复制完整的 Web MVC 请求处理。 MockMvc 是一个服务器端测试框架，让你能够使用轻量级和针对性的测试来验证 Web MVC 应用程序的大部分功能。你可以单独使用它来执行请求并验证响应，也可以通过 WebTes

## MockMvc 与端到端测试

> 原文：[`testing/mvc-test-framework/vs-end-to-end-integration-tests`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/vs-end-to-end-integration-tests.adoc)

### 概述

MockMvc 建立在来自 infra-test 模块的 Mock API mock 实现之上，不依赖于运行的容器。因此，与具有实际客户端和实时运行服务器的完整端到端集成测试相比，存在一些差异。 思考这个问题的最简单方法是从一个空白的 MockRequest 开始。你向其中添加的任何内容就是请求变成的样子。可能会让你感到惊讶的事情是，默认情况下没有上下文路径；没有 jsessionid cookie；没有转发、错误或异步分派；因此，没有实际的 JSP 渲染。相反，“forwarded”和“redirected”URL 保存在 MockResponse 中，并可以通过期望进行断言。 这意味着，如果你使用 JSP，你可以验证请求转发到的 JSP 页面，但不会渲染任何 HTML。换句话说，JSP 不会被调用。但是请注意，所有其他不依赖于转发的渲染技术，例如 Thymeleaf 和 Freemar

## 流式响应

> 原文：[`testing/mvc-test-framework/vs-streaming-response`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/mvc-test-framework/vs-streaming-response.adoc)

### 概述

测试流式响应（如 Server-Sent Events）的最佳方法是通过 <<WebTestClient>>，它可以用作测试客户端连接到 MockMvc 实例，以在没有运行服务器的情况下对 Web MVC 控制器执行测试。例如：

```java
WebTestClient client = MockMvcWebTestClient.bindToController(new SseController()).build();

FluxExchangeResult<Person> exchangeResult = client.get()
    .uri("/persons")
    .exchange()
    .expectStatus().isOk()
    .expectHeader().contentType("text/event-stream")
    .returnResult(Person.class);

// 使用 Project Reactor 中的 StepVerifier 测试流式响应

StepVerifier.create(exchangeResult.getResponseBody())
    .expectNext(new Person("N0"), new Person("N1"), new Person("N2"))
    .expectNextCount(4)
    .consumeNextWith(person -> assertThat(person.getName()).endsWith("7"))
    .thenCancel()
    .verify();
```

## WebTestClient

> 原文：[`testing/webtestclient`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/testing/webtestclient.adoc)

### 概述

WebTestClient 是一个设计用于测试服务器应用程序的 HTTP 客户端。它包装了 WebClient 并使用它来执行请求，但暴露了一个测试门面（testing facade）用于验证响应。WebTestClient 可用于执行端到端 HTTP 测试。它还可以通过 mock 服务器请求和响应对象，在没有运行服务器的情况下测试 Web MVC 和 Infra Reactive 应用程序。

##### 绑定到控制器

此设置允许你通过 mock 请求和响应对象测试特定控制器，而无需运行服务器。

```java
WebTestClient client =
    WebTestClient.bindToController(new TestController()).build();
```

##### 绑定到 ApplicationContext

此设置允许你加载带有 Web MVC 或 Infra Reactive 基础设施和控制器声明的 Infra 配置，并使用它通过 mock 请求和响应对象处理请求，而无需运行服务器。 对于 Web MVC，使用以下代码，其中 Infra ApplicationContext 被传递给 MockMvcBuilders.webAppContextSetup 以创建一个 MockMvc 实例来处理请求：

```java
@ExtendWith(InfraExtension.class)
@WebAppConfiguration("classpath:META-INF/web-resources") // <1>
@ContextHierarchy({
  @ContextConfiguration(classes = RootConfig.class),
  @ContextConfiguration(classes = WebConfig.class)
})
class MyTests {

  @Autowired
  WebApplicationContext wac; // <2>

  WebTestClient client;

  @BeforeEach
  void setUp() {
    client = MockMvcWebTestClient.bindToApplicationContext(this.wac).build(); // <3>
  }
}
```

##### 绑定到 Router Function

此设置允许你通过 mock 请求和响应对象测试函数式端点，而无需运行服务器。 对于 Reactive，使用以下代码，它委托给 RouterFunctions.toWebHandler 创建服务器设置以处理请求：

##### 绑定到服务器

此设置连接到运行中的服务器以执行完整的端到端 HTTP 测试：

##### 客户端配置

除了前面描述的服务器设置选项外，你还可以配置客户端选项，包括基础 URL、默认头、客户端过滤器等。这些选项在 bindToServer() 之后立即可用。对于所有其他配置选项，你需要使用 configureClient() 从服务器配置过渡到客户端配置，如下所示：

#### 编写测试

WebTestClient 提供了一个与 WebClient 相同的 API，直到使用 exchange() 执行请求的那一点。有关如何准备包含表单数据、多部分数据等任何内容的请求的示例，请参阅 WebClient 文档。 在调用 exchange() 之后，WebTestClient 与 WebClient 分道扬镳，转而继续执行验证响应的工作流。 要断言响应状态和头，请使用以下代码：

##### 无内容

如果预期响应没有内容，你可以按如下方式断言：

##### JSON 内容

你可以使用不带目标类型的 expectBody() 来对原始内容而不是高级对象执行断言。 要使用 JSONAssert 验证完整的 JSON 内容：

##### 流式响应

要测试潜在的无限流，如 "text/event-stream" 或 "application/x-ndjson"，首先验证响应状态和头，然后获取 FluxExchangeResult：

##### MockMvc 断言

WebTestClient 是一个 HTTP 客户端，因此它只能验证客户端响应中的内容，包括状态、头和体。 当使用 MockMvc 服务器设置测试 Web MVC 应用程序时，你可以额外选择对服务器响应执行进一步断言。为此，请在断言体之后获取 ExchangeResult：

---

> 本文件已达到 60 个示例的上限，其余内容请在 `infra-docs/` 原文查阅。
