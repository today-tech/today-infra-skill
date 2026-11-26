# Web MVC

> 由 `scripts/build-refs.mjs` 从 `infra-docs` 生成，请勿手工编辑；改归类请改 `data/topics.json`。

控制器、请求映射、方法参数与返回值、异常处理、MVC 配置、Netty 配置。

覆盖文档：61 页（含示例 44 页）。原文位于仓库 `infra-docs/`，下方每个页面标注了路径。

## 页面索引

- **异步请求** —— `web/webmvc/mvc-ann-async`
- **错误响应** —— `web/webmvc/mvc-ann-rest-exceptions`
- **HTTP 缓存** —— `web/webmvc/mvc-caching`
- MVC 配置 —— `web/webmvc/mvc-config`
- **Java 配置** —— `web/webmvc/mvc-config/advanced-java`
- **Body 类型 (Content Types)** —— `web/webmvc/mvc-config/content-negotiation`
- **类型转换** —— `web/webmvc/mvc-config/conversion`
- **MVC 配置 API** —— `web/webmvc/mvc-config/customize`
- **启用 MVC 配置** —— `web/webmvc/mvc-config/enable`
- **参数解析与返回值扩展** —— `web/webmvc/mvc-config/handler-methods`
- **拦截器** —— `web/webmvc/mvc-config/interceptors`
- **消息转换器** —— `web/webmvc/mvc-config/message-converters`
- **路径匹配** —— `web/webmvc/mvc-config/path-matching`
- **静态资源** —— `web/webmvc/mvc-config/static-resources`
- **验证** —— `web/webmvc/mvc-config/validation`
- **视图控制器** —— `web/webmvc/mvc-config/view-controller`
- **视图解析器** —— `web/webmvc/mvc-config/view-resolvers`
- **注解控制器** —— `web/webmvc/mvc-controller`
- **Controller Advice** —— `web/webmvc/mvc-controller/ann-advice`
- **异常处理** —— `web/webmvc/mvc-controller/ann-exceptionhandler`
- **@InitBinder** —— `web/webmvc/mvc-controller/ann-initbinder`
- 注解处理器 —— `web/webmvc/mvc-controller/ann-methods`
- 方法参数 —— `web/webmvc/mvc-controller/ann-methods/arguments`
- **@CookieValue** —— `web/webmvc/mvc-controller/ann-methods/cookievalue`
- **HttpEntity** —— `web/webmvc/mvc-controller/ann-methods/httpentity`
- **Jackson JSON** —— `web/webmvc/mvc-controller/ann-methods/jackson`
- **矩阵变量** —— `web/webmvc/mvc-controller/ann-methods/matrix-variables`
- **@ModelAttribute** —— `web/webmvc/mvc-controller/ann-methods/modelattrib-method-args`
- **Multipart** —— `web/webmvc/mvc-controller/ann-methods/multipart-forms`
- **重定向 Model** —— `web/webmvc/mvc-controller/ann-methods/redirect-model`
- **重定向属性** —— `web/webmvc/mvc-controller/ann-methods/redirecting-passing-data`
- **@RequestAttribute** —— `web/webmvc/mvc-controller/ann-methods/requestattrib`
- **@RequestBody** —— `web/webmvc/mvc-controller/ann-methods/requestbody`
- **@RequestHeader** —— `web/webmvc/mvc-controller/ann-methods/requestheader`
- **@RequestParam** —— `web/webmvc/mvc-controller/ann-methods/requestparam`
- **@ResponseBody** —— `web/webmvc/mvc-controller/ann-methods/responsebody`
- **ResponseEntity** —— `web/webmvc/mvc-controller/ann-methods/responseentity`
- 返回值 —— `web/webmvc/mvc-controller/ann-methods/return-types`
- **@SessionAttribute** —— `web/webmvc/mvc-controller/ann-methods/sessionattribute`
- 类型转换 —— `web/webmvc/mvc-controller/ann-methods/typeconversion`
- **Model** —— `web/webmvc/mvc-controller/ann-modelattrib-methods`
- **请求映射** —— `web/webmvc/mvc-controller/ann-requestmapping`
- 验证 —— `web/webmvc/mvc-controller/ann-validation`
- **声明** —— `web/webmvc/mvc-controller/ann`
- DispatcherHandler —— `web/webmvc/mvc-core`
- Web MVC 配置 —— `web/webmvc/mvc-core/config`
- **异常** —— `web/webmvc/mvc-core/exceptionhandlers`
- **过滤器** —— `web/webmvc/mvc-core/filters`
- 拦截 —— `web/webmvc/mvc-core/handlermapping-interceptor`
- 路径匹配 —— `web/webmvc/mvc-core/handlermapping-path`
- **Locale** —— `web/webmvc/mvc-core/localeresolver`
- 日志 —— `web/webmvc/mvc-core/logging`
- Multipart 解析器 —— `web/webmvc/mvc-core/multipart`
- **Netty 配置** —— `web/webmvc/mvc-core/netty-config`
- 处理流程 —— `web/webmvc/mvc-core/sequence`
- 特殊 Bean 类型 —— `web/webmvc/mvc-core/special-bean-types`
- 视图解析 —— `web/webmvc/mvc-core/viewresolver`
- 数据绑定 —— `web/webmvc/mvc-data-binding`
- HTTP/2 —— `web/webmvc/mvc-http2`
- **响应 Trailer** —— `web/webmvc/mvc-trailers`
- **URI 链接** —— `web/webmvc/mvc-uri-building`

## 异步请求

> 原文：[`web/webmvc/mvc-ann-async`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-ann-async.adoc)

### 概述

Web MVC 异步请求集成，如下： 和 Callable 返回值提供了对单个异步返回值的基本支持。 包括 SSE 和 原始数据。

#### DeferredResult

只要 HTTP 处理器返回了 DeferredResult 对象就会进入异步处理状态。例如：

```java
@GET("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
  DeferredResult<String> deferredResult = new DeferredResult<>();
  // 其他地方引用 deferredResult 变量.
  return deferredResult;
}

// 或者其他方式的处理器
public class MyHttpRequestHandler implements HttpRequestHandler {

  @Nullable
  @Override
  public DeferredResult<String> handleRequest(HttpContext request) throws Throwable {
    DeferredResult<String> deferredResult = new DeferredResult<>();
    // 其他地方引用 deferredResult 变量.
    return deferredResult;
  }

}

// 从其他线程设置返回结果...
deferredResult.setResult(result);
```

#### Callable

HTTP 处理器也可以使用 java.util.concurrent.Callable 包装任何支持的返回值，如下例所示：

```java
@POST
public Callable<String> processUpload(@RequestParam("file") Part file) {
  return () -> "someView";
}

// 或者其他方式的处理器
public class MyHttpRequestHandler implements HttpRequestHandler {

  @Nullable
  @Override
  public Callable<String> handleRequest(HttpContext request) throws Throwable {
    Part file = request.asMultipartRequest().getPart("file");
    // ...
    return () -> "someView";
  }

}
```

##### 流对象

你可以使用 ResponseBodyEmitter 产生流式响应，每个对象都会被一个 HttpMessageConverter 序列化，然后 写入 HTTP 连接。示例如下：

##### SSE

SseEmitter 是 ResponseBodyEmitter 的子类，提供了 Server-Sent Events 支持。服务器发出来的数据格式遵循 W3C SSE 标准。 要通过 HTTP 处理器 产生一个 SSE 流，您可以返回一个 SseEmitter，如下例所示：

##### Raw Data

有些场景需要直接写入 OutputStream (文件下载) 你就可以使用 StreamingResponseBody。如下例所示：

##### Web MVC

使用 WebMvcConfigurer 的 configureAsyncSupport 回调设置 MVC 的默认异步超时：

## 错误响应

> 原文：[`web/webmvc/mvc-ann-rest-exceptions`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-ann-rest-exceptions.adoc)

### 概述

REST 服务通常需要在错误响应体中提供结构化的错误信息。Web MVC 支持 RFC 9457：HTTP API 问题详情。 以下是这方面的主要抽象： - ProblemDetail -- RFC 9457 问题详情的表示，包含标准字段和应用自定义的扩展字段。 - ErrorResponse -- 描述异常如何映射为 HTTP 错误响应的契约，包括状态码、响应头和问题详情正文。 - ErrorResponseException -- 基本的 ErrorResponse 实现，其他人可以将其用作方便的基类。 - ResponseEntityExceptionHandler -- 一个方便的基类，用于处理所有 Web MVC 异常和任何 ErrorResponseException 的 @ControllerAdvice ，并渲染一个带正文的错误响应。

#### 未处理异常的兜底响应

除了声明 ResponseEntityExceptionHandler，还可以通过 configureProblemDetails， 将未被控制器或 @ControllerAdvice 中的 @ExceptionHandler 处理的异常转换为 RFC 9457 问题详情：

```java
import infra.context.annotation.Configuration;
import infra.web.config.annotation.ProblemDetailsConfigurer;
import infra.web.config.annotation.WebMvcConfigurer;

@Configuration
public class ProblemDetailsConfiguration implements WebMvcConfigurer {

  @Override
  public void configureProblemDetails(ProblemDetailsConfigurer configurer) {
    configurer.renderUnhandledExceptions(true);
  }
}
```

#### HTML 错误视图

对于同一端点，可以向 API 客户端写出问题详情，向偏好 HTML 的浏览器返回错误页面。 通过 ProblemDetailsConfigurer.viewResolver(...) 注册 infra.web.ErrorResponseViewResolver：

```java
@Override
public void configureProblemDetails(ProblemDetailsConfigurer configurer) {
  configurer.renderUnhandledExceptions(true)
    .viewResolver((request, problem, errorResponse) ->
      new ModelAndView("errors/problem").addObject("problem", problem));
}
```

#### 错误响应拦截器

通过 WebMvcConfigurer.addErrorResponseInterceptors 为问题详情添加统一的非标准字段：

## HTTP 缓存

> 原文：[`web/webmvc/mvc-caching`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-caching.adoc)

### 概述

HTTP 缓存可以显著提高 Web 应用程序的性能。HTTP 缓存围绕 Cache-Control 响应头， 随后是条件请求头（例如 Last-Modified 和 ETag）。Cache-Control 建议私有 （例如，浏览器）和公共（例如，代理）缓存如何缓存和重用响应。ETag 头用于进行条件请求， 如果内容未更改，可能会产生 304 (NOT_MODIFIED) 响应，而无需正文。ETag 可以被视为 Last-Modified 头的更高级的继承者。 本节描述了在 Web MVC 中可用的与 HTTP 缓存相关的选项。

#### CacheControl

CacheControl 提供了对配置与 Cache-Control 头相关的设置的支持，并在多个地方被接受为参数： 虽然 RFC 7234 描述了 Cache-Control 响应头的所有可能指令， CacheControl 类型采用了面向用例的方法，专注于常见的场景：

```java
// Cache for an hour - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// Prevent caching - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// Cache for ten days in public and private caches,
// public caches should not transform the response
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS).noTransform().cachePublic();
```

#### HTTP 处理器

HTTP 处理器可以添加对 HTTP 缓存的显式支持。我们建议这样做，因为资源的 lastModified 或 ETag 值需要在与条件请求头进行比较之前计算出来。控制器可以向 ResponseEntity 添加 ETag 头和 Cache-Control 设置，如下例所示：

```java
@GET("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

  Book book = findBook(id);
  String version = book.getVersion();

  return ResponseEntity
          .ok()
          .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
          .eTag(version) // lastModified is also available
          .body(book);
}

// 或者其他方式的处理器
public class MyHttpRequestHandler implements HttpRequestHandler {

  @Nullable
  @Override
  public ResponseEntity<Book> handleRequest(HttpContext request) throws Throwable {
    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
            .ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
            .eTag(version) // lastModified is also available
            .body(book);
  }

}
```

## MVC 配置

> 原文：[`web/webmvc/mvc-config`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config.adoc)

### 概述

Web MVC Java 配置提供了适合大多数应用程序的默认配置以及一个用于自定义的配置 API。 对于更多的高级配置，请参见 高级 Java 配置。 您无需了解由 Web MVC Java 配置和 MVC 命名空间创建的底层 Bean。如果您想了解更多， 请参阅 特殊 Bean 类型 和 Web MVC 配置。

## Java 配置

> 原文：[`web/webmvc/mvc-config/advanced-java`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/advanced-java.adoc)

### 概述

@EnableWebMvc 导入了 DelegatingWebMvcConfiguration，它： 对于高级模式，您可以移除 @EnableWebMvc 并直接扩展 DelegatingWebMvcConfiguration 而不是实现 WebMvcConfigurer，如下例所示：

```java
@Configuration
public class WebConfiguration extends DelegatingWebMvcConfiguration {

  public WebConfiguration(List<WebMvcConfigurer> configurers) {
    super(configurers);
  }

  // ...
}
```

## Body 类型 (Content Types)

> 原文：[`web/webmvc/mvc-config/content-negotiation`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/content-negotiation.adoc)

### 概述

可以配置 Web MVC 如何从请求中确定请求的媒体类型（例如，Accept 请求头、URL 路径扩展名、查询参数等）。 默认情况下，仅检查 Accept 头。 如果您必须使用基于 URL 的内容类型解析，请考虑使用查询参数策略而不是路径扩展。 您可以根据以下示例自定义请求内容类型解析：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.mediaType("json", MediaType.APPLICATION_JSON);
    configurer.mediaType("xml", MediaType.APPLICATION_XML);
  }
}
```

## 类型转换

> 原文：[`web/webmvc/mvc-config/conversion`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/conversion.adoc)

### 概述

默认情况下，框架配置了各种数字和日期类型的格式化器，并支持通过字段上的 @NumberFormat 和 @DateTimeFormat 进行自定义。 要注册自定义格式化器和转换器，请使用以下方法：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void addFormatters(FormatterRegistry registry) {
    // ...
  }
}
```

## MVC 配置 API

> 原文：[`web/webmvc/mvc-config/customize`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/customize.adoc)

### 概述

在 Java 配置中，您可以实现 WebMvcConfigurer 接口，如下例所示：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  // 实现配置方法...
}
```

## 启用 MVC 配置

> 原文：[`web/webmvc/mvc-config/enable`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/enable.adoc)

### 概述

您可以使用 @EnableWebMvc 注解以编程方式启用 MVC 配置，如下例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfiguration {
}
```

## 参数解析与返回值扩展

> 原文：[`web/webmvc/mvc-config/handler-methods`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/handler-methods.adoc)

### 概述

Web MVC 使用 ParameterResolvingStrategy 将请求转换为控制器方法参数， 使用 ReturnValueHandler 处理方法结果。两者都可以通过 WebMvcConfigurer 扩展，无需替换整个请求映射适配器。

#### 自定义参数解析

以下示例将 X-Tenant-Id 请求头封装为专用参数类型；业务场景仍需自行验证租户身份和访问权限，不能将请求头视为可信身份。

```java
import infra.context.annotation.Configuration;
import infra.http.HttpStatus;
import infra.web.HttpContext;
import infra.web.config.annotation.WebMvcConfigurer;
import infra.web.handler.method.HandlerParameter;
import infra.web.handler.method.ParameterResolvingStrategies;
import infra.web.handler.method.ParameterResolvingStrategy;
import infra.web.server.ResponseStatusException;

@Configuration
public class TenantWebConfiguration implements WebMvcConfigurer {

  public record TenantId(String value) {}

  @Override
  public void configureParameterResolving(ParameterResolvingStrategies strategies) {
    strategies.add(new ParameterResolvingStrategy() {
      @Override
      public boolean supportsParameter(HandlerParameter parameter) {
        return parameter.getParameterType() == TenantId.class;
      }

      @Override
      public Object resolveArgument(HttpContext context, HandlerParameter parameter) {
        String value = context.requestHeaders().getFirst("X-Tenant-Id");
        if (value == null || value.isBlank()) {
          throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Missing tenant id");
        }
        return new TenantId(value);
      }
    });
  }
}
```

#### 自定义返回值处理

modifyReturnValueHandlerManager(ReturnValueHandlerManager) 在默认返回值处理器注册后执行。 通过 manager.getHandlers() 可检查和调整有序列表，manager.addHandlers(...) 则追加到末尾。 当自定义返回类型可能被默认兜底处理器接收时，应将专用处理器放到它之前，而不是简单追加。 实现 infra.web.ReturnValueHandler 时，需提供 supportsHandler(Object) 和 handleReturnValue(HttpContext, Object, Object)；也可以重写 supportsReturnValue(Object) 按运行时结果匹配。 普通处理器的处理器匹配与返回值匹配是“或”关系，因此不要用无条件返回 true 的匹配方法， 否则会截获视图、异步

```java
import java.nio.charset.StandardCharsets;

import org.jspecify.annotations.Nullable;

import infra.context.annotation.Configuration;
import infra.web.HttpContext;
import infra.web.ReturnValueHandler;
import infra.web.config.annotation.WebMvcConfigurer;
import infra.web.handler.ReturnValueHandlerManager;

@Configuration
public class TextResultConfiguration implements WebMvcConfigurer {

  public record TextResult(String text) {}

  @Override
  public void modifyReturnValueHandlerManager(ReturnValueHandlerManager manager) {
    manager.getHandlers().add(0, new ReturnValueHandler() {
      @Override
      public boolean supportsHandler(Object handler) {
        return false;
      }

      @Override
      public boolean supportsReturnValue(@Nullable Object returnValue) {
        return returnValue instanceof TextResult;
      }

      @Override
      public void handleReturnValue(HttpContext context, @Nullable Object handler,
          @Nullable Object returnValue) throws Exception {
        TextResult result = (TextResult) returnValue;
        context.responseHeaders().set("Content-Type", "text/plain;charset=UTF-8");
        context.getOutputStream().write(result.text().getBytes(StandardCharsets.UTF_8));
      }
    });
  }
}
```

## 拦截器

> 原文：[`web/webmvc/mvc-config/interceptors`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/interceptors.adoc)

### 概述

您可以注册拦截器以应用于传入的请求，如下例所示：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LocaleChangeInterceptor())
      .addPathPatterns("/**").excludePathPatterns("/admin/**");
  }
}
```

## 消息转换器

> 原文：[`web/webmvc/mvc-config/message-converters`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/message-converters.adoc)

### 概述

您可以通过重写 configureMessageConverters(List)， 在Java配置中设置要使用的 HttpMessageConverter 实例，替换默认使用的实例。您也可以通过重写 extendMessageConverters(List)， 在最后定制配置的消息转换器列表。 TIP: 在 Infra App 应用程序中，WebMvcAutoConfiguration 除了默认转换器外，还会添加它检测到的任何 HttpMessageConverter Bean。或者，也可以使用 extendMessageConverters 在最后修改消息转换器。 FormHttpMessageConverter 支持读取和写入 application/x-www-form-urlencoded 表单， 既可以使用 MultiValueMap<String, String> 保留同一名称

```java
import infra.http.converter.json.JacksonJsonHttpMessageConverter;
import infra.http.converter.xml.JacksonXmlHttpMessageConverter;
import tools.jackson.databind.SerializationFeature;
import tools.jackson.databind.json.JsonMapper;
import tools.jackson.dataformat.xml.XmlMapper;

@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.add(new JacksonJsonHttpMessageConverter(
      JsonMapper.builder().enable(SerializationFeature.INDENT_OUTPUT)));
    converters.add(new JacksonXmlHttpMessageConverter(
      XmlMapper.builder(JacksonXmlHttpMessageConverter.defensiveXmlFactory())
        .enable(SerializationFeature.INDENT_OUTPUT)));
  }
}
```

#### 保留默认转换器的构建器配置

如果只需替换某一种格式的转换器，同时保留字节数组、字符串、表单等默认支持， 可以重写 configureMessageConverters(HttpMessageConverters.ServerBuilder)：

```java
@Override
public void configureMessageConverters(HttpMessageConverters.ServerBuilder builder) {
  builder.withJsonConverter(new JacksonJsonHttpMessageConverter(
    JsonMapper.builder().enable(SerializationFeature.INDENT_OUTPUT)));
}
```

## 路径匹配

> 原文：[`web/webmvc/mvc-config/path-matching`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/path-matching.adoc)

### 概述

可以定制与路径匹配和URL处理相关的选项。 有关各个选项的详细信息，请参阅 PathMatchConfigurer javadoc。 以下示例展示了如何定制路径匹配：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void configurePathMatch(PathMatchConfigurer configurer) {
    configurer.addPathPrefix("/api", HandlerTypePredicate.forAnnotation(RestController.class));
  }

}
```

## 静态资源

> 原文：[`web/webmvc/mvc-config/static-resources`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/static-resources.adoc)

### 概述

此选项提供了一种方便的方式，可以从基于 Resource 的位置列表中提供静态资源。 在下一个示例中，对于以 /resources 开头的请求，相对路径用于查找并提供相对于 Web 应用程序根目录下的 /public 或类路径下的 /static 的静态资源。资源被提供具有一年未来的到期时间，以确保最大限度地利用浏览器缓存并减少浏览器发出的 HTTP 请求。Last-Modified 信息从 Resource#lastModified 推断出来，以便通过 Last-Modified 头支持 HTTP 条件请求。 以下列表展示了如何进行：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCacheControl(CacheControl.maxAge(Duration.ofDays(365)));
  }
}
```

## 验证

> 原文：[`web/webmvc/mvc-config/validation`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/validation.adoc)

### 概述

默认情况下，如果类路径上存在 Bean 验证 （例如 Hibernate Validator），则 LocalValidatorFactoryBean 被注册为全局 Validator， 用于与控制器方法参数上的 @Valid 和 @Validated 一起使用。 您可以像以下示例所示自定义全局 Validator 实例：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public Validator getValidator() {
    Validator validator = new OptionalValidatorFactoryBean();
    // ...
    return validator;
  }
}
```

## 视图控制器

> 原文：[`web/webmvc/mvc-config/view-controller`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/view-controller.adoc)

### 概述

这是一个定义 ParameterizableViewController 的快捷方式，当被调用时，它会立即转发到一个视图。 当没有 Java 控制器逻辑需要在生成响应之前运行时，您可以在静态情况下使用它。 以下示例将对 / 的请求转发到名为 home 的视图：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/").setViewName("home");
  }
}
```

## 视图解析器

> 原文：[`web/webmvc/mvc-config/view-resolvers`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-config/view-resolvers.adoc)

### 概述

MVC 配置简化了视图解析器的注册。 以下示例通过使用 JSP 和 Jackson 配置内容协商视图解析，将默认的 View 设置为 JSON 渲染：

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

  @Override
  public void configureViewResolvers(ViewResolverRegistry registry) {
    registry.enableContentNegotiation(new MappingJackson2JsonView());
  }
}
```

## 注解控制器

> 原文：[`web/webmvc/mvc-controller`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller.adoc)

### 概述

Web 模块 提供了基于注解的编程模型，其中 @Controller 和 @RestController 组件使用注解来表达请求映射、请求输入、 异常处理等等。 带注解的控制器支持非常灵活的方法签名（参数和返回值）和不必继承基类，也不必实现特定的接口。 以下示例显示了通过注解定义的控制器：

```java
@RestController
public class HelloController {

  @GET("/hello")
  public String handle() {
    return "Hello World!";
  }
}
```

## Controller Advice

> 原文：[`web/webmvc/mvc-controller/ann-advice`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-advice.adoc)

### 概述

@ExceptionHandler、@InitBinder 和 @ModelAttribute 方法仅适用于在其中声明的 @Controller 类或类层次结构。相反，如果它们在 @ControllerAdvice 或 @RestControllerAdvice 类中声明，则它们可以应用于多个控制器。未指定选择器时默认全局适用； 不限定控制器类型的 @ControllerAdvice 中的 @ExceptionHandler 也可用于处理其他处理器的异常。 @ControllerAdvice 被 @Component 元注释，因此可以通过 组件扫描 注册为 Infra bean。@RestControllerAdvice 被 @ControllerAdvice 和 @ResponseBody 元注释， 这意味着 @ExceptionHandler 方法的返回值将通过响应体消息转换进行渲染，

```java
// 针对所有带有 @RestController 注解的控制器
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// 针对特定包中的所有控制器
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// 针对可分配给特定类的任何控制器
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

## 异常处理

> 原文：[`web/webmvc/mvc-controller/ann-exceptionhandler`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-exceptionhandler.adoc)

### 概述

@Controller 和 @ControllerAdvice 类可以有 @ExceptionHandler 方法来处理控制器方法中的异常，如下例所示：

```java
@Controller
public class SimpleController {

  // ...

  @ExceptionHandler
  public ResponseEntity<String> handle(IOException ex) {
    // ...
  }
}
```

## @InitBinder

> 原文：[`web/webmvc/mvc-controller/ann-initbinder`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-initbinder.adoc)

### 概述

@Controller 或 @ControllerAdvice 类可以有 @InitBinder 方法来初始化 WebDataBinder 实例，这些实例可以： 在 @Controller 中，DataBinder 自定义应用于控制器内的局部，或者甚至通过注解引用名称来应用于特定的模型属性。 在 @ControllerAdvice 中，自定义可以应用于所有或控制器的一个子集。 您可以在 DataBinder 中注册 PropertyEditor、Converter 和 Formatter 组件进行类型转换。 或者，您可以使用 MVC 配置 在全局共享的 FormattingConversionService 中注册 Converter 和 Formatter 组件。 @InitBinder 方法可以具有许多与 @RequestMapping 方法相同的参数，显著的例外是 @ModelAtt

```java
@Controller
public class FormController {

  @InitBinder // <1>
  public void initBinder(WebDataBinder binder) {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    dateFormat.setLenient(false);
    binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
  }

  // ...
}
```

## 注解处理器

> 原文：[`web/webmvc/mvc-controller/ann-methods`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods.adoc)

### 概述

@RequestMapping 处理器方法具有灵活的签名，并可以从支持的控制器方法参数和返回值中进行选择。

## 方法参数

> 原文：[`web/webmvc/mvc-controller/ann-methods/arguments`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/arguments.adoc)

### 概述

下表列出 Web MVC 内置解析器支持的控制器方法参数。Session 和 @SessionAttribute 需要启用 Web 会话支持，例如在配置类上添加 infra.session.config.EnableSession（@EnableSession）。 默认的参数解析器不支持以响应式类型接收请求输入。 对 @RequestParam、@RequestHeader 等按名称绑定的参数，可以将参数声明为 @Nullable，或在支持 required 的绑定注解上设置 required=false，以允许值缺失。 例如，对于 @RequestParam @Nullable Integer page，未提供 page 时参数值为 null，传入 page=abc 时仍会触发类型转换错误。 如果需要为 java.util.Optional<T> 参数定义绑定规则，可以注册自定义 Par

## @CookieValue

> 原文：[`web/webmvc/mvc-controller/ann-methods/cookievalue`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/cookievalue.adoc)

### 概述

您可以使用 @CookieValue 注解将 HTTP cookie 的值绑定到控制器中的方法参数。 考虑一个带有以下 cookie 的请求： JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84 以下示例展示了如何获取 cookie 值：

```java
@GET("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { <1>
  //...
}
```

## HttpEntity

> 原文：[`web/webmvc/mvc-controller/ann-methods/httpentity`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/httpentity.adoc)

### 概述

HttpEntity 与使用 @RequestBody 或多或少相同，但它是基于一个公开请求头和正文的容器对象。以下清单显示了一个示例：

```java
@POST("/accounts")
public void handle(HttpEntity<Account> entity) {
  // ...
}
```

## Jackson JSON

> 原文：[`web/webmvc/mvc-controller/ann-methods/jackson`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/jackson.adoc)

### 概述

Infra 提供了对 Jackson JSON 库的支持。

#### JSON 视图

Web MVC 为 Jackson 的序列化视图 提供了内置支持， 这允许仅渲染 Object 中所有字段的一个子集。要将其与 @ResponseBody 或 ResponseEntity 控制器方法一起使用， 您可以使用 Jackson 的 @JsonView 注解来激活序列化视图类，如下例所示：

```java
@RestController
public class UserController {

  @GET("/user")
  @JsonView(User.WithoutPasswordView.class)
  public User getUser() {
    return new User("eric", "7!jd#h23");
  }
}

public class User {

  public interface WithoutPasswordView {}
  public interface WithPasswordView extends WithoutPasswordView {}

  private String username;
  private String password;

  public User() {
  }

  public User(String username, String password) {
    this.username = username;
    this.password = password;
  }

  @JsonView(WithoutPasswordView.class)
  public String getUsername() {
    return this.username;
  }

  @JsonView(WithPasswordView.class)
  public String getPassword() {
    return this.password;
  }
}
```

## 矩阵变量

> 原文：[`web/webmvc/mvc-controller/ann-methods/matrix-variables`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/matrix-variables.adoc)

### 概述

RFC 3986 讨论了路径段中的名称-值对。在 Web MVC 中， 我们根据 Tim Berners-Lee 的一篇 "旧文章" 将它们称为“矩阵变量”，但它们也可以称为 URI 路径参数。 矩阵变量可以出现在任何路径段中，每个变量之间用分号分隔，多个值之间用逗号分隔 （例如，/cars;color=red,green;year=2012）。也可以通过重复变量名来指定多个值 （例如，color=red;color=green;color=blue）。 如果预期 URL 包含矩阵变量，则控制器方法的请求映射必须使用 URI 变量来掩盖该变量内容， 并确保请求可以成功匹配，而不管矩阵变量的顺序和存在与否。 以下示例使用了矩阵变量：

```java
	// GET /pets/42;q=11;r=22

	@GET("/pets/{petId}")
	public void findPet(@PathVariable String petId, @MatrixVariable int q) {

		// petId == 42
		// q == 11
	}
```

## @ModelAttribute

> 原文：[`web/webmvc/mvc-controller/ann-methods/modelattrib-method-args`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/modelattrib-method-args.adoc)

### 概述

@ModelAttribute 注解使用在方法参数上，会将请求参数绑定到模型对象上。例如：

```java
@POST("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { // <1>
  // 业务逻辑。。。
}
```

## Multipart

> 原文：[`web/webmvc/mvc-controller/ann-methods/multipart-forms`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/multipart-forms.adoc)

### 概述

在 配置 MultipartParser 之后， 带有 multipart/form-data 的 POST 请求的内容将被解析并作为常规请求参数访问。 以下示例访问一个常规表单字段和一个上传的文件：

```java
import java.io.IOException;

import infra.web.multipart.Part;

@Controller
public class FileUploadController {

  @POST("/form")
  public String handleFormUpload(@RequestParam("name") String name,
      @RequestParam("file") Part file) throws IOException {

    if (!file.isEmpty()) {
      byte[] bytes = file.getContentAsByteArray();
      // store the bytes somewhere
      return "redirect:uploadSuccess";
    }
    return "redirect:uploadFailure";
  }
}
```

## 重定向 Model

> 原文：[`web/webmvc/mvc-controller/ann-methods/redirect-model`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/redirect-model.adoc)

### 概述

重定向 Model（infra.web.RedirectModel）用于将提示消息等临时数据从当前请求传给重定向后的请求，典型场景是 Post-Redirect-Get。 与 URL 中的重定向参数 不同， 重定向 Model 中的属性默认不会写入重定向 URL。 在发起重定向的控制器方法中声明 infra.web.RedirectModel 参数，并用 addAttribute 添加输出属性。 RedirectView 会将非空的输出模型交给已配置的 RedirectModelManager 保存。 使用 SessionRedirectModelManager 时，属性临时保存在 HTTP 会话中；如果没有要保存的属性，就不会为此创建会话。 目标请求匹配后，管理器取出并移除已保存的模型。对于带注解的控制器，框架在调用目标方法前将这些输入属性合并到普通 Model 中。 例如，提交订单后重

```java
import infra.web.RedirectModel;
import infra.web.annotation.GET;
import infra.web.annotation.POST;
import infra.web.view.RedirectView;

// 以下方法位于同一个 @Controller 中。
@POST("/orders/{id}")
public RedirectView save(RedirectModel redirectModel) {
  // 保存订单，成功后再添加提示。
  redirectModel.addAttribute("notice", "订单已保存");
  RedirectView view = new RedirectView("/orders/{id}");
  view.setHttp10Compatible(false);
  return view;
}

@GET("/orders/{id}")
public String show() {
  return "orders/detail";
}
```

## 重定向属性

> 原文：[`web/webmvc/mvc-controller/ann-methods/redirecting-passing-data`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/redirecting-passing-data.adoc)

### 概述

重定向时，先区分数据应该出现在目标 URL 中，还是只供目标请求临时使用： 声明 RedirectModel 参数不会阻止普通模型属性出现在 URL 中。 如果不希望把视图模型中的属性自动附加为查询参数，可以显式返回 RedirectView 并调用 setExposeModelAttributes(false)。这不会关闭 URI 模板展开，也不会移除目标 URL 中显式指定的查询参数。 不要将不应公开的数据用于构造重定向 URL。 以下示例使用当前请求的 +{id}+ 展开目标路径，并在目标 URL 中显式携带 tab 查询参数：

```java
RedirectView view = new RedirectView("/orders/{id}?tab=summary");
view.setExposeModelAttributes(false);
return view;
```

## @RequestAttribute

> 原文：[`web/webmvc/mvc-controller/ann-methods/requestattrib`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/requestattrib.adoc)

### 概述

类似于 @SessionAttribute，你可以使用 @RequestAttribute 注解来访问之前创建的请求属性（这种策略主要用来传递参数） （例如，HandlerInterceptor 创建的）：

```java
@GET("/")
public String handle(@RequestAttribute Client client) { // <1>
  // ...
}
```

## @RequestBody

> 原文：[`web/webmvc/mvc-controller/ann-methods/requestbody`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/requestbody.adoc)

### 概述

您可以使用 @RequestBody 注解让请求体通过 HttpMessageConverter 读取并反序列化为 Object。以下示例使用了一个 @RequestBody 参数：

```java
@POST("/accounts")
public void handle(@RequestBody Account account) {
  // ...
}
```

## @RequestHeader

> 原文：[`web/webmvc/mvc-controller/ann-methods/requestheader`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/requestheader.adoc)

### 概述

您可以使用 @RequestHeader 注解将请求头绑定到控制器中的方法参数。 考虑以下带有请求头的请求： Host localhost:8080 Accept text/html,application/xhtml+xml,application/xml;q=0.9 Accept-Language fr,en-gb;q=0.7,en;q=0.3 Accept-Encoding gzip,deflate Accept-Charset ISO-8859-1,utf-8;q=0.7,*;q=0.7 Keep-Alive 300 以下示例获取 Accept-Encoding 和 Keep-Alive 头的值：

```java
@GET("/demo")
public void handle(
    @RequestHeader("Accept-Encoding") String encoding, // <1>
    @RequestHeader("Keep-Alive") long keepAlive) { // <2>
  //...
}
```

## @RequestParam

> 原文：[`web/webmvc/mvc-controller/ann-methods/requestparam`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/requestparam.adoc)

### 概述

您可以使用 @RequestParam 注解将 HTTP 请求参数（即查询参数或表单数据）绑定到控制器中的方法参数。 以下示例显示了如何执行此操作：

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {

  // ...

  @GET
  public String setupForm(@RequestParam("petId") int petId, Model model) { <1>
    Pet pet = this.clinic.loadPet(petId);
    model.addAttribute("pet", pet);
    return "petForm";
  }

  // ...

}
```

## @ResponseBody

> 原文：[`web/webmvc/mvc-controller/ann-methods/responsebody`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/responsebody.adoc)

### 概述

您可以在方法上使用 @ResponseBody 注解，使返回内容通过 HttpMessageConverter 序列化到响应体。 以下清单显示了一个示例：

```java
@GET("/accounts/{id}")
@ResponseBody
public Account handle() {
  // ...
}
```

## ResponseEntity

> 原文：[`web/webmvc/mvc-controller/ann-methods/responseentity`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/responseentity.adoc)

### 概述

ResponseEntity 类似于 @ResponseBody，但带有状态和头。例如：

```java
@GET("/something")
public ResponseEntity<String> handle() {
  String body = ... ;
  String etag = ... ;
  return ResponseEntity.ok().eTag(etag).body(body);
}
```

## 返回值

> 原文：[`web/webmvc/mvc-controller/ann-methods/return-types`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/return-types.adoc)

### 概述

下表描述了支持的控制器方法返回值。所有返回值都支持响应式类型。

## @SessionAttribute

> 原文：[`web/webmvc/mvc-controller/ann-methods/sessionattribute`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/sessionattribute.adoc)

### 概述

如果您需要访问全局管理的（即在控制器之外——例如，由过滤器管理）且可能存在也可能不存在的预先存在的会话属性， 您可以在方法参数上使用 @SessionAttribute 注解，如下例所示：

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) { <1>
  // ...
}
```

## 类型转换

> 原文：[`web/webmvc/mvc-controller/ann-methods/typeconversion`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-methods/typeconversion.adoc)

### 概述

一些表示基于 String 的请求输入的注解控制器方法参数（例如 @RequestParam、@RequestHeader、@PathVariable、 @MatrixVariable 和 @CookieValue）如果声明为除 String 之外的其他类型，则可能需要类型转换。 对于这些情况，将根据配置的转换器自动应用类型转换。默认情况下，支持简单类型（如 int、long、Date 等）。 您可以通过 WebDataBinder（见 DataBinder）或通过向 FormattingConversionService 注册 Formatters 来自定义类型转换。见 Infra 字段格式化。 类型转换中的一个实际问题是空字符串源值的处理。如果类型转换后结果为 null，则这样的值被视为缺失。这可能适用于 Long、UUID 及其他目标类型。如果您希望允许注入 null，则要么在参数注

## Model

> 原文：[`web/webmvc/mvc-controller/ann-modelattrib-methods`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-modelattrib-methods.adoc)

### 概述

您可以使用 @ModelAttribute 注解： 用于创建或访问模型中的 Object 并通过 WebDataBinder 将其绑定到请求。 本节讨论 @ModelAttribute 方法——上述列表中的第二项。控制器可以有任意数量的 @ModelAttribute 方法。 所有这些方法在同一个控制器中的 @RequestMapping 方法之前被调用。@ModelAttribute 方法也可以通过 @ControllerAdvice 在控制器之间共享。更多详细信息请参阅 控制器建议 部分。 @ModelAttribute 方法具有灵活的方法签名。它们支持许多与 @RequestMapping 方法相同的参数，除了 @ModelAttribute 本身或与请求体相关的任何内容。 以下示例展示了一个 @ModelAttribute 方法：

```java
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
  model.addAttribute(accountRepository.findAccount(number));
  // 添加更多 ...
}
```

## 请求映射

> 原文：[`web/webmvc/mvc-controller/ann-requestmapping`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-requestmapping.adoc)

### 概述

本节讨论了注解控制器的请求映射。

#### @RequestMapping

使用 @RequestMapping 将请求映射到控制器方法。映射可以按 URL 路径、HTTP 方法、请求参数、请求头、媒体类型和 API 版本匹配。 可以在类级别声明共享条件，再在方法级别声明具体端点。value 和 path 互为别名，都可以指定一个或多个路径模式。 以下 组合注解 是指定 HTTP 方法的快捷方式： 建议为控制器端点显式指定 HTTP 方法。不设置 method 时，该条件通常不限制 HTTP 方法； 普通 OPTIONS 请求由框架 默认处理。 NOTE: @RequestMapping 不能与在同一元素（类、接口或方法）上声明的其他 @RequestMapping 注解一起使用。如果在同一个元素上检测到多个 @RequestMapping 注解，将记录一条警告，并且只使用第一个映射。 这也适用于诸如 @GET、@POST 等组合的 @RequestMapping

```java
@RestController
@RequestMapping("/persons")
class PersonController {

  @GET("/{id}")
  public Person getPerson(@PathVariable Long id) {
    // ...
  }

  @POST
  @ResponseStatus(HttpStatus.CREATED)
  public void add(@RequestBody Person person) {
    // ...
  }
}
```

#### URI 匹配

Web MVC 使用预先解析的 PathPattern 匹配请求路径。请求路径表示为 PathContainer，可以逐段处理 URL 编码和路径参数。 有关匹配选项，请参阅 MVC 配置。 ++ 和 +{name}+ 都匹配零个或多个路径段，后者还会将匹配到的路径捕获为变量。 每个模式只能包含一个这样的多段匹配元素，且只能放在模式开头或末尾。 完整语法参见 PathPattern。 路径模式示例： 捕获的 URI 变量可以使用 @PathVariable 访问。例如：

```java
@GET("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
  // ...
}
```

#### 可处理的媒体类型

您可以根据请求的 Content-Type 来缩小请求映射的范围，如下例所示：

#### 返回的媒体类型

您可以根据 Accept 请求头和控制器方法产生的内容类型列表来缩小请求映射的范围，如下例所示：

#### 请求参数和请求头

可以根据请求参数条件缩小映射范围。支持检查参数是否存在（myParam）、是否不存在（!myParam）、 是否等于指定值（myParam=myValue）或是否不等于指定值（myParam!=myValue）。声明多个条件时，必须同时满足。 以下示例检查参数值：

#### 显式注册

可以以编程方式注册处理器方法，这可以用于动态注册或高级用例，例如在不同 URL 下使用相同的处理器的不同实例。 以下示例展示了如何注册一个处理器方法：

#### @HttpExchange

虽然 @HttpExchange 的主要目的是通过生成的代理抽象 HTTP 客户端代码，但这样的注解所放置的 HTTP 接口 是一个与客户端与服务器使用无关的契约。 除了简化客户端代码外，还有一些情况下，HTTP 接口可能是服务器方便地暴露其 API 供客户端访问的方式。 这导致客户端和服务器之间的耦合增加，通常不是一个好的选择，特别是对于公共 API，但可能正是内部 API 的目标。 这种方法也是为什么 @HttpExchange 被支持作为 @RequestMapping 的替代品，用于控制器类中的服务器端处理。 例如：

## 验证

> 原文：[`web/webmvc/mvc-controller/ann-validation`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann-validation.adoc)

### 概述

Web MVC 内置了对 @RequestMapping 方法的 验证 支持， 包括使用 Java Bean 验证 的选项。验证支持在两个层面上工作。 首先，对于 @ModelAttribute、 @RequestBody 和 @RequestPart 方法参数的解析器， 如果参数具有 Jakarta 的 @Valid 或框架的 @Validated 注解，则执行验证，并在必要时引发 MethodArgumentNotValidException。 或者，你可以通过在验证参数之后立即添加一个 Errors 或 BindingResult 方法参数来在控制器方法中处理错误。 你可以通过 Web MVC 配置 全局配置一个 Validator，或者通过 @Controller 或 @ControllerAdvice 中的 @InitBinder 方法局部配置。你也可以使用多个验证器。 错误响应

## 声明

> 原文：[`web/webmvc/mvc-controller/ann`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-controller/ann.adoc)

### 概述

你可以通过在应用的 ApplicationContext 中使用标准的 bean 定义来定义控制器 bean。 @Controller 允许自动检测，与框架通用支持在类路径中检测 @Component 类并自动注册 bean 定义一致。它还作为被注释类的刻板印象，指示其作为 Web 组件的角色。 要启用这些 @Controller bean 的自动检测，你可以将组件扫描添加到你的 Java 配置中，如下例所示：

```java
@Configuration
@ComponentScan("org.example.web")
public class WebConfiguration {

  // ...
}
```

## DispatcherHandler

> 原文：[`web/webmvc/mvc-core`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core.adoc)

### 概述

Web MVC 提供共享算法的模式用于请求处理，而实际工作由可配置的委托组件执行。该模型非常灵活，扩展性极强，支持多种工作流程。 反过来理解，DispatcherHandler 使用 IoC 配置来发现 HandlerMapping 、ViewResolver、异常所需的委托组件处理， and more.

## Web MVC 配置

> 原文：[`web/webmvc/mvc-core/config`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/config.adoc)

### 概述

应用程序可以声明在 特殊 Bean 类型 中列出的处理请求所需的基础设施 Bean。DispatcherHandler 检查 ApplicationContext 中的每个特殊 Bean。 在大多数情况下，MVC 配置 是最佳起点。它以 Java 的方式声明所需的 Bean， 并提供了一个更高级别的配置回调 API 来自定义它。 NOTE: Infra App 依赖于 MVC Java 配置来配置 Web MVC，并提供了许多额外的便捷选项。

## 异常

> 原文：[`web/webmvc/mvc-core/exceptionhandlers`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/exceptionhandlers.adoc)

### 概述

如果在请求映射期间发生异常或从请求处理程序（例如 @Controller）抛出异常，DispatcherHandler 会委托给 HandlerExceptionHandler bean 链来解决异常并提供替代处理，通常是错误响应。 下表列出了可用的 HandlerExceptionHandler 实现：

#### @ResponseStatus 与错误说明

@ResponseStatus 可以声明在处理器方法、控制器类或异常类上。它的 code （也可以通过别名 value 指定）用于设置 HTTP 响应状态。控制器类上的注解会应用于该控制器的请求映射方法；异常类上的注解则由 ResponseStatusExceptionHandler 在异常处理链中解析。 仅设置状态码时，处理器方法的返回值仍会按照正常的返回值处理流程写入响应：

```java
@ResponseStatus(HttpStatus.CREATED)
public User create(@RequestBody User user) {
  return userService.create(user);
}
```

## 过滤器

> 原文：[`web/webmvc/mvc-core/filters`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/filters.adoc)

### 概述

infra.web.Filter 用于在请求进入处理器之前执行检查，或围绕后续处理执行逻辑。 调用 FilterChain.doFilter(HttpContext) 将请求交给后续过滤器和处理器； 不调用过滤链并直接返回，则短路后续处理。 过滤器从 5.0 开始提供，运行在 DispatcherHandler 层，在处理器查找和执行之前介入。 它适合认证检查、限流、维护模式、请求日志和通用请求头处理等不依赖具体控制器的逻辑。 这里的 Filter 是框架的 infra.web.Filter，不是 Servlet 的过滤器接口。

#### 注册与排序

将 Filter 注册为应用上下文中的 Bean 即可。DispatcherHandler 初始化时会自动发现当前上下文 及祖先上下文中的过滤器，并根据 infra.core.annotation.Order 或 infra.core.Ordered 排序。 数值越小的过滤器越先进入；有顺序要求时应显式指定不同的顺序值，不要依赖相同顺序值之间的排列。 可以像下文的认证过滤器一样使用 @Component，也可以通过 @Bean 注册。 Filter 是函数式接口，因此简单逻辑可以使用 Lambda 表达式：

```java
import infra.context.annotation.Bean;
import infra.context.annotation.Configuration;
import infra.core.annotation.Order;
import infra.web.Filter;

@Configuration
public class FilterConfiguration {

  @Bean
  @Order(0)
  Filter responseHeaderFilter() {
    return (http, chain) -> {
      http.setHeader("X-Application", "example");
      chain.doFilter(http);
    };
  }
}
```

#### 执行顺序与前后置处理

过滤器通过 doFilter(HttpContext, FilterChain) 处理请求。通常只调用一次 chain.doFilter(http)；重复调用会再次执行下游链，而不是从上次结束的位置继续。 链的终点进入处理器查找、执行及返回值处理。 调用 HttpContext.forward(path) 则直接重新查找目标处理器，不会重新执行过滤器链。 参见 内部转发。 例如，两个过滤器的顺序值分别为 0 和 10，正常同步执行时顺序如下： 如果前面的过滤器短路，后续过滤器和处理器不会执行。 使用 try/finally 可以在下游调用返回或抛出异常时释放过滤器自己持有的资源：

```java
@Override
public void doFilter(HttpContext http, FilterChain chain) throws Exception {
  long start = System.nanoTime();
  try {
    chain.doFilter(http);
  }
  finally {
    long elapsedNanos = System.nanoTime() - start;
    // 将本次过滤链调用耗时写入指标或日志。
  }
}
```

#### 在过滤器中发送响应

从 5.0 开始，可以通过 HttpContext.send(Object) 发送响应结果，无需注入额外的发送器， 也不需要先匹配控制器方法。该方法复用已配置的返回值处理器和消息转换器。 下面的过滤器在缺少认证信息时发送 JSON 错误响应，否则继续过滤链。 示例仅演示短路响应，实际应用仍需验证凭据的有效性。

#### 结果处理规则

send 将结果原样传给返回值处理器，不会自动包装为 HttpEntity， 也不会强制将所有结果解释为响应体。处理器选择时没有控制器方法元数据， 因此不能依赖控制器方法上的 @ResponseBody 等注解来决定处理方式。 例如，发送纯文本响应体：

## 拦截

> 原文：[`web/webmvc/mvc-core/handlermapping-interceptor`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/handlermapping-interceptor.adoc)

### 概述

所有 HandlerMapping 实现都支持处理程序拦截器，当您想要将特定功能应用于某些请求（例如，检查主体）时，这些拦截器非常有用。 拦截器必须从 infra.web 包中实现 HandlerInterceptor，该接口提供以下回调， 应提供足够的灵活性来执行各种预处理和后处理： preProcessing(..) 方法返回一个布尔值。您可以使用此方法中断或继续执行链的处理。 当此方法返回 true 时，处理程序执行链继续。 返回 false 时跳过剩余链和当前拦截器的 postProcessing，拦截器需自行处理响应。 默认实现仅在剩余链正常返回时调用 postProcessing，它不是请求完成回调。 有关如何配置拦截器的示例，请参阅 MVC 配置部分中的 拦截器。 您也可以通过在各个 HandlerMapping 实现上使用 setter 直接注册它们。 对于通常返回的 @R

## 路径匹配

> 原文：[`web/webmvc/mvc-core/handlermapping-path`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/handlermapping-path.adoc)

### 概述

使用 PathPatternParser 解析的 PathPattern 与称为 RequestPath 的路径的解析表示形式匹配， 一次一个路径段。这允许单独解码和清理路径段值，而没有改变路径结构的风险。有关模式语法详细信息和比较， 请参阅 模式比较。

## Locale

> 原文：[`web/webmvc/mvc-core/localeresolver`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/localeresolver.adoc)

### 概述

Infra 架构的大部分都支持国际化，Infra Web MVC 框架也是如此。DispatcherHandler 让您可以使用客户端的区域设置自动解析消息。 这是通过 LocaleResolver 对象完成的。 请求语言偏好和应用最终使用的区域设置是两个不同的概念： HttpContext.getPreferredLocale() 按质量权重降序选择具体语言，忽略通配符范围和 q=0 的范围。 请求头缺失、空白或不含可接受的具体语言时返回 null；非法请求头仍会抛出解析异常。 该方法不读取 Cookie、Session 或 LocaleResolver，也不回退到 JVM 默认语言。 它不保证应用支持返回的语言；需要所有语言候选和权重时，使用 request.requestHeaders().getAcceptLanguage()。 应用代码通常应使用 WebUtils.resolve

#### Header 解析器

此区域设置解析器检查客户端（例如，Web 浏览器）发送的请求中的 accept-language 头。 通常，此头字段包含客户端操作系统的区域设置。请注意，此解析器不支持时区信息。 默认情况下，AcceptHeaderLocaleResolver 没有限制支持的语言。 可通过 setSupportedLocales(...) 配置候选语言，并通过 setDefaultLocale(...) 配置没有可用请求语言或匹配失败时的默认值：

```java
@Component(name = LocaleResolver.BEAN_NAME)
public LocaleResolver mvcLocaleResolver() {
  AcceptHeaderLocaleResolver resolver = new AcceptHeaderLocaleResolver();
  resolver.setSupportedLocales(List.of(Locale.CHINESE, Locale.ENGLISH));
  resolver.setDefaultLocale(Locale.CHINESE);
  return resolver;
}
```

#### Cookie 解析器

此区域设置解析器检查客户端上可能存在的 Cookie，以查看是否指定了 Locale 或 TimeZone。 如果是，它将使用指定的详细信息。通过使用此区域设置解析器的属性，您可以指定 cookie 的名称以及最大寿命。 以下示例定义了一个 CookieLocaleResolver：

```xml
<bean id="mvcLocaleResolver" class="infra.web.i18n.CookieLocaleResolver">

  <property name="cookieName" value="clientlanguage"/>

  <!-- in seconds. If set to -1, the cookie is not persisted (deleted when browser shuts down) -->
  <property name="cookieMaxAge" value="100000"/>

</bean>
```

#### Locale 拦截器

您可以通过将 LocaleChangeInterceptor 添加到 HandlerMapping 定义之一来启用区域设置更改。 它检测请求中的参数并相应地更改区域设置，在调度程序的应用程序上下文中的 LocaleResolver 上调用 setLocale 方法。 也可以通过拦截器的 setLocaleResolver(...) 显式指定解析器。 Cookie 和 Session 策略支持保存语言选择，而 AcceptHeaderLocaleResolver 不支持动态修改，调用其 setLocale 会抛出 UnsupportedOperationException。拦截器更改的是策略中的选择，不是 HttpContext.getPreferredLocale() 表示的请求头偏好。 下一个示例显示，对包含名为 siteLanguage 参数的所有 *.view 资源的调用现在会更改区

## 日志

> 原文：[`web/webmvc/mvc-core/logging`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/logging.adoc)

### 概述

Web MVC 中的 DEBUG 级别日志旨在紧凑、最小且人性化。它专注于一遍又一遍有用的高价值信息，而不是仅在调试特定问题时有用的其他信息。 TRACE 级别日志通常遵循与 DEBUG 相同的原则（例如，也不应该是大量的），但可用于调试任何问题。 此外，一些日志消息可能会在 TRACE 与 DEBUG 级别显示不同的详细程度。 良好的日志来自使用日志的经验。如果您发现任何不符合既定目标的内容，请告诉我们。

## Multipart 解析器

> 原文：[`web/webmvc/mvc-core/multipart`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/multipart.adoc)

### 概述

infra.web.multipart 包中的 MultipartParser 是用于解析 multipart 请求（包括文件上传）的策略。 有一个默认的 infra.web.multipart.parsing.DefaultMultipartParser 实现。底层来源于 （Apache Commons FileUpload） 在使用 Netty 自动配置的 Infra App 应用中，框架会在没有自定义 MultipartParser bean 时注册默认实现， 并将其注入 NettyRequestConfig；自定义解析器按类型提供，无需使用特定 bean 名称。 手动配置 Netty 时，通过 NettyRequestConfig.Builder.multipartParser(...) 指定解析器。 通过 HttpContext.asMultipartRequest() 访问解析

## Netty 配置

> 原文：[`web/webmvc/mvc-core/netty-config`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/netty-config.adoc)

### 概述

infra-web-netty-server 模块提供了基于 Netty 的嵌入式 Web 服务器实现。 它可以通过 infra-starter-netty-server 启动器自动配置，也可以通过直接使用 NettyWebServerFactory 进行编程式配置。

#### 依赖引入

要使用 Netty 作为嵌入式容器，您需要添加 infra-starter-netty-server 依赖：

```groovy
dependencies {
  implementation 'cn.taketoday:infra-starter-netty-server'
}
```

#### 编程式配置

如果您需要更细粒度的控制，可以注册一个 WebServerFactoryCustomizer bean 来定制 NettyWebServerFactory：

```java
@Bean
public WebServerFactoryCustomizer<NettyWebServerFactory> nettyCustomizer() {
  return factory -> {
    factory.setPort(8088);
    factory.setWorkerThreadCount(10);
  };
}

@Bean
public ServerBootstrapCustomizer serverBootstrapCustomizer(){
  return bootstrap -> {
    bootstrap.option(ChannelOption.AUTO_READ, false);
  };
}
```

#### 请求配置定制

您还可以通过 NettyRequestConfigCustomizer 定制请求配置：

## 处理流程

> 原文：[`web/webmvc/mvc-core/sequence`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/sequence.adoc)

### 概述

DispatcherHandler 按以下顺序处理请求： HandlerExceptionHandler 用于将处理请求时的异常转换为错误响应或视图。响应提交后不能再生成替代响应。 参见 异常，以及 异步异常处理。 对于 HTTP 缓存支持，处理程序可以使用 HttpContext 的 checkNotModified 方法， 以及 控制器的 HTTP 缓存 中描述的带注解控制器的更多选项。

## 特殊 Bean 类型

> 原文：[`web/webmvc/mvc-core/special-bean-types`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/special-bean-types.adoc)

### 概述

MVC 将请求映射、处理器调用和响应生成委托给不同的基础设施 Bean。 有些 Bean 由 DispatcherHandler 直接查找，有些由 MVC 配置装配后交给映射器、适配器或返回值处理器使用。

## 视图解析

> 原文：[`web/webmvc/mvc-core/viewresolver`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-core/viewresolver.adoc)

### 概述

Web MVC 定义了 ViewResolver 和 View 接口，让您可以在浏览器中渲染模型，而无需将您绑定到特定的视图技术。 ViewResolver 提供了视图名称和实际视图之间的映射。View 解决了在移交给特定视图技术之前的数据准备问题。 下表提供了有关 ViewResolver 层次结构的更多详细信息：

## 数据绑定

> 原文：[`web/webmvc/mvc-data-binding`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-data-binding.adoc)

### 概述

数据绑定将字符串参数绑定到对象图，并在此过程中进行类型转换。 这是框架用于应用配置的核心机制之一。在 Web 应用中，它让应用能够通过具有丰富类型的对象 访问查询参数和表单数据，而不必直接操作字符串值的映射。 有关构造函数绑定、setter 绑定、属性名称语法和类型转换等机制的详细介绍， 请参阅核心技术部分的 数据绑定。 对于注解控制器，数据绑定应用于 @ModelAttribute 方法参数。 对于函数式端点，使用 ServerRequest 的 bind 方法。 TIP: 对于采用注解控制器的浏览器应用，可以使用 @ModelAttribute 方法 初始化供视图渲染使用的其他模型属性。 框架按绑定目标创建 WebDataBinder，不会在不同请求之间共享实例。对于注解控制器，可以通过控制器内的 @InitBinder 方法自定义该实例， 也可以通过 Controller Advic

## HTTP/2

> 原文：[`web/webmvc/mvc-http2`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-http2.adoc)

### 概述

通过配置 server.http2.enabled=true 开启 server:

## 响应 Trailer

> 原文：[`web/webmvc/mvc-trailers`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-trailers.adoc)

### 概述

当响应元数据只有在正文生产结束后才能确定时，可以通过 HttpContext.setTrailerFields(Trailer...) 注册尾部字段，例如流式计算的摘要或统计信息。Trailer 来自 infra.web，将字段名与延迟求值的 Supplier<String> 绑定。

```java
@GET("/report")
public void report(HttpContext context) throws IOException {
  byte[] body = "report\n".getBytes(StandardCharsets.UTF_8);
  context.setTrailerFields(Trailer.field("X-Report-Bytes", () -> Integer.toString(body.length)));
  context.getOutputStream().write(body);
}
```

## URI 链接

> 原文：[`web/webmvc/mvc-uri-building`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/web/webmvc/mvc-uri-building.adoc)

### 概述

本节描述了 TODAY Framework 中用于处理 URI 的各种选项。

#### 相对 Web 请求

您可以使用 HttpContextUriComponentsBuilder 创建相对于当前请求的 URI，如下例所示：

```java
HttpContext request = ...

// 重用 scheme, host, port, path, 和 query string...

URI uri = HttpContextUriComponentsBuilder.fromRequest(request)
    .replaceQueryParam("accountId", "{id}")
    .build("123");
```

#### 指向控制器的链接

Web MVC 提供了一种机制来准备指向控制器方法的链接。例如，下面的 MVC 控制器允许创建链接：

```java
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {

  @GET("/bookings/{booking}")
  public ModelAndView getBooking(@PathVariable Long booking) {
    // ...
  }
}
```

#### 视图中的链接

在 Thymeleaf、FreeMarker 或 JSP 等视图中，您可以通过引用每个请求映射的隐式或显式分配的名称， 来构建指向带注解控制器的链接。 考虑以下示例：

---

> 本文件已达到 60 个示例的上限，其余内容请在 `infra-docs/` 原文查阅。
