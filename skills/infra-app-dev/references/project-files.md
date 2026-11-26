# 仓库根文件与示例片段

> 由 `scripts/sync-context7.mjs` 从 context7 的 `llms.txt` 生成，请勿手工编辑。
> 只保留 `infra-docs` 未覆盖的内容（README、CONTRIBUTING、infra-samples 等）；
> 文档正文的权威来源仍是 `infra-docs`，本文件用于补齐“项目怎么跑起来”这类信息。

条目 9 个（跳过：已在 infra-docs 覆盖 70、输出/占位 0、无代码 0）。

## `CONTRIBUTING.md`

### 克隆仓库并构建

克隆你的 Fork 仓库，使用 Gradle Wrapper 构建项目。项目使用 JDK 25 构建，兼容 Java 17+。

```bash
# 1. 克隆你的Fork仓库
git clone https://github.com/YOUR_USERNAME/today-infrastructure.git
cd today-infrastructure

# 2. 使用项目包装器（推荐）
./gradlew build

# 3. 导入IDE
# 项目已包含Gradle配置，可直接用IntelliJ IDEA或Eclipse导入。
```

## `README.md`

### Minimal HTTP application with @InfraApplication and @RestController

Create a minimal HTTP interface in src/main/java/example/DemoApplication.java. Uses infra-starter-webmvc and infra-starter-netty-server, requires JDK 17+. The @InfraApplication enables component scanning and auto-configuration; @GET maps GET requests to the method.

```java
package example;

import infra.app.Application;
import infra.app.InfraApplication;
import infra.web.annotation.GET;
import infra.web.annotation.RestController;

@RestController
@InfraApplication
public class DemoApplication {

  public static void main(String[] args) {
    Application.run(DemoApplication.class, args);
  }

  @GET("/hello")
  public String hello() {
    return "Hello, TODAY!";
  }
}
```

### Curl request to sample index endpoint

After the service starts, run this curl command in another terminal to test the index endpoint. Expected response is 'Hello'.

```bash
curl http://localhost:8080/index
```

### Start the application with infraRun

Run the application from the project root using Gradle. Requires Gradle Wrapper and the infra.application plugin. Use gradlew.bat on Windows. Alternatively, run DemoApplication.main() in the IDE with --server.port=8080 as a program argument.

```bash
./gradlew infraRun --args='--server.port=8080'
```

### Clone and run the sample application

Commands to clone the repository and run the sample application with Gradle. Requires Git and JDK; Gradle downloads dependencies on first run. Use gradlew.bat on Windows.

```bash
git clone https://github.com/TAKETODAY/today-infrastructure.git
cd today-infrastructure/infra-samples
./gradlew infraRun --args='--server.port=8080'
```

### Curl request to hello endpoint

After the service starts, run this curl command in another terminal to verify the hello endpoint. Expected response is 'Hello, TODAY!'.

```bash
curl http://localhost:8080/hello
```

### Submodule build.gradle applying infra.application plugin

After using either of the above configurations, submodules can apply the 'infra.application' plugin. Includes repositories and dependencies for infra-starter-netty-server and infra-starter-webmvc.

```groovy
apply plugin: "java"
apply plugin: 'infra.application'

repositories {
  mavenLocal()
  maven { url "https://central.sonatype.com/repository/maven-snapshots/" }
  maven { url "https://maven.aliyun.com/repository/public" }
  mavenCentral()
}

dependencies {
  implementation 'cn.taketoday:infra-starter-netty-server'
  implementation 'cn.taketoday:infra-starter-webmvc'
}
```

### 复现性能测试（JVM 模式）

克隆 today-blog 示例项目，启动应用（JVM 模式），然后运行低并发和高并发 wrk 压测命令。运行前需准备数据库、初始化数据并安装 wrk。

```bash
# 克隆示例项目
git clone https://github.com/TAKETODAY/today-blog
cd today-blog

# 启动应用（JVM 模式）
./gradlew :blog-api:infraRun

# 低并发延迟测试
wrk -t2 -c10 -d30s --latency 'http://127.0.0.1:8080/api/articles?size=1'

# 高并发吞吐测试
wrk -t20 -c5000 -d1m --latency 'http://127.0.0.1:8080/api/articles?size=1'
```

## `infra-samples/hello-cli/README.md`

### 运行命令行应用

在 infra-samples 目录执行 Gradle 任务启动应用。Windows 使用 gradlew.bat。输出 Hello, World!。

```bash
./gradlew :hello-cli:infraRun
```
