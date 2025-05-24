# 模型上下文协议（MCP）服务器

Spring AI MCP 模块提供了对模型上下文协议（Model Context Protocol, MCP）的集成，使你能够通过标准化协议暴露你的 AI 工具和资源。当你希望让 Spring AI 工具和资源可被 MCP 兼容客户端访问时，该模块尤为有用。

## 依赖

要使用 MCP 服务器功能，请在项目中添加如下依赖：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-server</artifactId>
    <version>${spring-ai.version}</version>
</dependency>
```

## 配置属性

MCP 服务器可通过 `spring.ai.mcp.server` 前缀下的以下属性进行配置：

| 属性 | 默认值 | 描述 |
|----------|---------|-------------|
| `enabled` | `false` | 启用/禁用 MCP 服务器 |
| `name` | `"mcp-server"` | MCP 服务器名称 |
| `version` | `"1.0.0"` | MCP 服务器版本 |
| `type` | `SYNC` | 服务器类型（`SYNC` 或 `ASYNC`） |
| `resource-change-notification` | `true` | 启用/禁用资源变更通知 |
| `tool-change-notification` | `true` | 启用/禁用工具变更通知 |
| `prompt-change-notification` | `true` | 启用/禁用提示词变更通知 |
| `transport` | `STDIO` | 传输类型（`STDIO`、`WEBMVC` 或 `WEBFLUX`） |
| `sse-message-endpoint` | `"/mcp/message"` | Web 传输下的 SSE 消息端点 |

## 服务器类型

MCP 服务器支持两种运行模式：

### 1. 同步模式（默认）

同步模式为默认选项，适用于大多数顺序访问工具和资源的场景：

```yaml
spring:
  ai:
    mcp:
      server:
        type: SYNC
```

### 2. 异步模式

异步模式适用于需要响应式、非阻塞操作的应用：

```yaml
spring:
  ai:
    mcp:
      server:
        type: ASYNC
```

## 传输选项

MCP 服务器支持三种传输类型：

### 1. STDIO 传输（默认）

标准输入/输出传输为默认选项，适合命令行工具和本地开发：

```yaml
spring:
  ai:
    mcp:
      server:
        transport: STDIO
```

### 2. WebMvc 传输

WebMvc 传输使用 Spring MVC 的服务器推送事件（SSE）进行通信：

```yaml
spring:
  ai:
    mcp:
      server:
        transport: WEBMVC
        sse-message-endpoint: /mcp/message  # 可选，默认为 /mcp/message
```

所需依赖：
```xml
<dependency>
    <groupId>io.modelcontextprotocol.sdk</groupId>
    <artifactId>mcp-spring-webmvc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 3. WebFlux 传输

WebFlux 传输使用 Spring WebFlux 的 SSE 进行响应式通信：

```yaml
spring:
  ai:
    mcp:
      server:
        transport: WEBFLUX
        sse-message-endpoint: /mcp/message  # 可选，默认为 /mcp/message
```

所需依赖：
```xml
<dependency>
    <groupId>io.modelcontextprotocol.sdk</groupId>
    <artifactId>mcp-spring-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

## 核心特性

MCP 服务器提供以下核心功能：

### 工具

- 支持同步与异步执行的可扩展工具注册系统
- 通过 Spring 组件扫描自动发现和注册工具
- 工具更新的变更通知支持

### 资源

- 静态与动态资源管理
- 可选的资源变更通知
- 支持同步与异步资源访问

### 提示词

- 可配置的提示词模板
- 模板更新的变更通知支持
- 与 Spring AI 提示词系统集成

## 使用示例

以下为使用 WebMvc 传输和自定义设置配置 MCP 服务器的示例：

```yaml
spring:
  ai:
    mcp:
      server:
        enabled: true
        name: "My AI Tools Server"
        version: "1.0.0"
        type: SYNC
        transport: WEBMVC
        sse-message-endpoint: /ai/mcp/events
        resource-change-notification: true
        tool-change-notification: true
        prompt-change-notification: false
```

## 自动配置

MCP 服务器的自动配置包括：

1. `McpServerAutoConfiguration`：核心服务器配置，支持同步与异步模式
2. `McpWebMvcServerAutoConfiguration`：WebMvc 传输配置（存在 WebMvc 依赖时激活）
3. `McpWebFluxServerAutoConfiguration`：WebFlux 传输配置（存在 WebFlux 依赖时激活）

自动配置会根据你的配置和可用依赖自动设置合适的服务器类型和传输方式。

## 实现工具与资源

要通过 MCP 服务器暴露你的 Spring AI 工具和资源：

1. 为你的 AI 工具实现 `ToolCallback` 接口：
```java
@Component
public class MyAiTool implements ToolCallback {
    // 实现
}
```

2. 自动配置会根据服务器类型配置自动发现并注册你的工具，转换为同步或异步实现。

## 监控

MCP 服务器可对以下变更提供通知：
- 工具（添加或移除时）
- 资源（更新时）
- 提示词（模板变更时）

你可以通过配置属性启用或禁用这些通知。通知系统适用于同步和异步服务器类型，无论选择哪种运行模式都能提供一致的变更跟踪。
