---
title: Azure Application Insights 遥测关联 | Azure Docs
description: Application Insights 遥测关联
services: application-insights
documentationcenter: .net
author: lingliw
manager: digimobile
ms.service: application-insights
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 6/4/2019
ms.reviewer: sergkanz
ms.author: v-lingwu
ms.openlocfilehash: 3f20e9a7c086bfb230417d6c6682a237b07b0621
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562705"
---
# <a name="telemetry-correlation-in-application-insights"></a>Application Insights 中的遥测关联

在微服务的世界中，每次逻辑操作都需要在服务的不同组件中完成工作。 这些组件都可由 [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md) 单独监视。 web-app 组件与身份验证提供程序组件通信以验证用户凭据，并与 API 组件通信以获取要可视化的数据。 API 组件可从其他服务查询数据，还可使用 cache-provider 组件来通知有关此调用的计费组件。 Application Insights 支持分布式遥测关联，可用来检测哪个组件要对故障或性能下降问题负责。

本文介绍了 Application Insights 用于关联由多个组件发送的遥测的数据模型。 其中阐述了 context-propagation 技术和协议， 以及如何在不同的语言和平台上实现相关的概念。

## <a name="data-model-for-telemetry-correlation"></a>遥测关联的数据模型

Application Insights 定义了用于分配遥测关联的[数据模型](../../azure-monitor/app/data-model.md)。 要将遥测与逻辑操作关联，每个遥测项都应包含名为 `operation_Id` 的上下文字段。 此标识符由分布式跟踪中的每个遥测项共享。 因此，即使单个层失去了遥测功能，也仍可关联其他组件报告的遥测。

分布式逻辑操作通常由一系列小规模操作（某个组件处理的请求）构成。 这些操作由[请求遥测](../../azure-monitor/app/data-model-request-telemetry.md)定义。 每个请求遥测都具有自身的 `id`，用于对自身进行唯一全局标识。 与此请求关联的所有遥测项（例如跟踪和异常）应将 `operation_parentId` 设置为请求 `id` 的值。

每个传出操作（例如，对另一个组件的 HTTP 调用）是由[依赖项遥测](../../azure-monitor/app/data-model-dependency-telemetry.md)表示的。 依赖项遥测也定义了自身的全局唯一的 `id`。 此依赖项调用发起的请求遥测将此 `id` 用作其 `operation_parentId`。

可以结合 `dependency.id` 使用 `operation_Id`、`operation_parentId` 和 `request.id`，生成分布式逻辑操作的视图。 这些字段还定义了遥测调用的因果关系顺序。

在微服务环境中，来自组件的跟踪可能会进入不同的存储项。 每个组件可能在 Application Insights 中具有其自身的检测密钥。 为了获取逻辑操作的遥测数据，Application Insights UX 会查询每个存储项中的数据。 如果存储项的数目极大，需要提示后续查找位置。 Application Insights 数据模型定义了以下两个字段来解决此问题：`request.source` 和 `dependency.target`。 第一个字段定义发起依赖项请求的组件，第二个字段定义哪个组件返回依赖项调用的响应。

## <a name="example"></a>示例

我们以名为 Stock Prices 的应用程序为例。该应用程序使用名为 `Stock` 的外部 API 显示某只股票的当前市价。 Stock Prices 应用程序有一个名为 `Stock page` 的页面，可以由客户端 Web 浏览器通过 `GET /Home/Stock` 打开。 应用程序使用 HTTP 调用 `GET /api/stock/value` 来查询 `Stock` API。

可以运行一个查询来分析生成的遥测数据：

```kusto
(requests | union dependencies | union pageViews)
| where operation_Id == "STYz"
| project timestamp, itemType, name, id, operation_ParentId, operation_Id
```

在结果中可以看到，所有遥测项共享根 `operation_Id`。 从该页面发出 Ajax 调用后，会将新的唯一 ID (`qJSXU`) 分配给依赖项遥测，并将 pageView 的 ID 用作 `operation_ParentId`。 接着，服务器请求将 Ajax ID 用作 `operation_ParentId`。

| itemType   | name                      | ID           | operation_ParentId | operation_Id |
|------------|---------------------------|--------------|--------------------|--------------|
| pageView   | Stock page                |              | STYz               | STYz         |
| dependency | GET /Home/Stock           | qJSXU        | STYz               | STYz         |
| 请求    | GET Home/Stock            | KqKwlrSt9PA= | qJSXU              | STYz         |
| dependency | GET /api/stock/value      | bBrf2L7mm2g= | KqKwlrSt9PA=       | STYz         |

在对外部服务发出 `GET /api/stock/value` 调用时，需要知道该服务器的标识，以便对 `dependency.target` 字段进行相应的设置。 如果外部服务不支持监视，则会将 `target` 设置为服务的主机名（例如 `stock-prices-api.com`）。 但是，如果该服务通过返回预定义的 HTTP 标头来标识自身，则 `target` 会包含服务标识，使 Application Insights 能够通过查询该服务中的遥测数据来生成分布式跟踪。

## <a name="correlation-headers"></a>关联标头

我们正在开发[关联 HTTP 协议](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md)的 RFC 提案。 此提案定义两个标头：

- `Request-Id`：承载调用的全局唯一 ID。
- `Correlation-Context`：承载分布式跟踪属性的名称值对集合。

该标准还定义了 `Request-Id` 生成项的两个架构：平面和分层。 使用平面架构时，将为 `Correlation-Context` 集合定义一个已知的 `Id` 键。

Application Insights 为关联 HTTP 协议定义了[扩展](https://github.com/lmolkova/correlation/blob/master/http_protocol_proposal_v2.md)。 它使用 `Request-Context` 名称值对来传播直接调用方或被调用方使用的属性集合。 Application Insights SDK 使用此标头设置 `dependency.target` 和 `request.source` 字段。

### <a name="w3c-distributed-tracing"></a>W3C 分布式跟踪

我们正在转换为 [W3C 分布式跟踪格式](https://w3c.github.io/trace-context/)。 定义的内容：

- `traceparent`：承载调用的全局唯一操作 ID 和唯一标识符。
- `tracestate`：承载跟踪系统特定的上下文。

#### <a name="enable-w3c-distributed-tracing-support-for-classic-aspnet-apps"></a>启用对经典 ASP.NET 应用的 W3C 分布式跟踪支持

从版本 2.8.0-beta1 开始，`Microsoft.ApplicationInsights.Web` 和 `Microsoft.ApplicationInsights.DependencyCollector` 包中提供此功能。
此项默认禁用。 若要启用该项，请更改 `ApplicationInsights.config`：

- 在 `RequestTrackingTelemetryModule` 下，添加 `EnableW3CHeadersExtraction` 元素，并将值设为 `true`。
- 在 `DependencyTrackingTelemetryModule` 下，添加 `EnableW3CHeadersInjection` 元素，并将值设为 `true`。

#### <a name="enable-w3c-distributed-tracing-support-for-aspnet-core-apps"></a>启用对 ASP.NET Core 应用的 W3C 分布式跟踪支持

此功能在 `Microsoft.ApplicationInsights.AspNetCore` 的版本 2.5.0-beta1 以及 `Microsoft.ApplicationInsights.DependencyCollector` 的版本 2.8.0-beta1 中提供。
此项默认禁用。 若要启用该项，请将 `ApplicationInsightsServiceOptions.RequestCollectionOptions.EnableW3CDistributedTracing` 设置为 `true`：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry(o => 
        o.RequestCollectionOptions.EnableW3CDistributedTracing = true );
    // ....
}
```

#### <a name="enable-w3c-distributed-tracing-support-for-java-apps"></a>启用对 Java 应用的 W3C 分布式跟踪支持

- **传入配置**

  - 对于 Java EE 应用，请将以下内容添加到 ApplicationInsights.xml 内的 `<TelemetryModules>` 标记中：

    ```xml
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule>
       <Param name = "W3CEnabled" value ="true"/>
       <Param name ="enableW3CBackCompat" value = "true" />
    </Add>
    ```
  - 对于 Spring Boot 应用，请添加以下属性：

    - `azure.application-insights.web.enable-W3C=true`
    - `azure.application-insights.web.enable-W3C-backcompat-mode=true`

- **传出配置**

  将以下代码添加到 AI-Agent.xml：

  ```xml
  <Instrumentation>
    <BuiltIn enabled="true">
      <HTTP enabled="true" W3C="true" enableW3CBackCompat="true"/>
    </BuiltIn>
  </Instrumentation>
  ```

  > [!NOTE]
  > 默认情况下启用后向兼容性模式，并且 `enableW3CBackCompat` 参数是可选的， 仅在要将后向兼容性关闭时使用。
  >
  > 理想情况下，当所有服务都已更新为支持 W3C 协议的较新版 SDK 时，应将该功能关闭。 强烈建议你尽快迁移到这些更新的 SDK。

> [!IMPORTANT]
> 请确保传入和传出配置完全相同。

## <a name="opentracing-and-application-insights"></a>OpenTracing 和 Application Insights

[OpenTracing 数据模型规范](https://opentracing.io/)和 Application Insights 数据模型按以下方式映射：

| Application Insights                  | OpenTracing                                       |
|------------------------------------   |-------------------------------------------------  |
| `Request`, `PageView`                 | 带 `span.kind = server` 的 `Span`                  |
| `Dependency`                          | 带 `span.kind = client` 的 `Span`                  |
| `Request` 和 `Dependency` 的 `Id`    | `SpanId`                                          |
| `Operation_Id`                        | `TraceId`                                         |
| `Operation_ParentId`                  | `ChildOf` 类型的 `Reference`（父级范围）   |

有关详细信息，请参阅 [Application Insights 遥测数据模型](../../azure-monitor/app/data-model.md)。 

有关 OpenTracing 概念的定义，请参阅 OpenTracing [规范](https://github.com/opentracing/specification/blob/master/specification.md)和 [semantic_conventions](https://github.com/opentracing/specification/blob/master/semantic_conventions.md)。

## <a name="telemetry-correlation-in-net"></a>.NET 中的遥测关联

.NET 至今已定义了多种方式来关联遥测和诊断日志：

- `System.Diagnostics.CorrelationManager` 允许跟踪 [LogicalOperationStack 和 ActivityId](https://msdn.microsoft.com/library/system.diagnostics.correlationmanager.aspx)。 
- `System.Diagnostics.Tracing.EventSource` 和 Windows 事件跟踪 (ETW) 定义了 [SetCurrentThreadActivityId](https://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource.setcurrentthreadactivityid.aspx) 方法。
- `ILogger` 使用[日志范围](https://docs.microsoft.com/aspnet/core/fundamentals/logging#log-scopes)。 
- Windows Communication Foundation (WCF) 和 HTTP 将“当前”上下文传播关联到一起。

但是，这些方法并未实现自动分布式跟踪支持。 `DiagnosticSource` 是支持跨计算机自动关联的一种方式。 .NET 库支持“DiagnosticSource”，并允许通过 HTTP 等传输方法自动跨计算机传播关联上下文。

`DiagnosticSource` 中的[指南活动](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md)解释了跟踪活动的基础知识。

ASP.NET Core 2.0 支持提取 HTTP 标头和启动新的活动。

从版本 4.1.0 开始，`System.Net.HttpClient` 支持自动注入关联 HTTP 标头和以活动形式跟踪 HTTP 调用。

经典 ASP.NET 有一个新的 HTTP 模块 [Microsoft.AspNet.TelemetryCorrelation](https://www.nuget.org/packages/Microsoft.AspNet.TelemetryCorrelation/)。 此模块使用 `DiagnosticSource` 实现遥测关联。 它会基于传入的请求标头启动活动。 它还会关联不同请求处理阶段的遥测，即使 Internet Information Services (IIS) 处理的每个阶段在不同的托管线程上运行。

从版本 2.4.0-beta1 开始，Application Insights SDK 使用 `DiagnosticSource` 和 `Activity` 收集遥测数据并将其与当前活动相关联。

<a name="java-correlation"></a>
## <a name="telemetry-correlation-in-the-java-sdk"></a>Java SDK 中的遥测关联

从版本 2.0.0 开始，[用于 Java 的 Application Insights SDK](../../azure-monitor/app/java-get-started.md) 支持自动关联遥测。 对于所有在请求范围内发出的遥测（例如跟踪、异常、自定义事件），它会自动填充 `operation_id`。 对于通过 HTTP 进行的服务到服务调用，它还负责传播关联标头（如前所述），前提是 [Java SDK 代理](../../azure-monitor/app/java-agent.md)已配置。

> [!NOTE]
> 只有通过 Apache HTTPClient 进行的调用才能使用关联功能。 如果使用 Spring RestTemplate 或 Feign，则二者实际上都可以与 Apache HTTPClient 配合使用。

目前不支持跨消息传送技术（例如，Kafka、RabbitMQ 或 Azure 服务总线）自动进行上下文传播。 但是，可以通过 `trackDependency` 和 `trackRequest` API 手动编码此类方案。 在这些 API 中，让依赖项遥测代表由生成者排队的消息，让请求代表由使用者处理的消息。 在这种情况下，`operation_id` 和 `operation_parentId` 都应在消息的属性中传播。

### <a name="telemetry-correlation-in-asynchronous-java-application"></a>异步 Java 应用程序中的遥测关联

若要在异步 Spring Boot 应用程序中关联遥测，请遵循[此](https://github.com/Microsoft/ApplicationInsights-Java/wiki/Distributed-Tracing-in-Asynchronous-Java-Applications)深入文章。 它为检测 Spring 的 [ThreadPoolTaskExecutor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html) 和 [ThreadPoolTaskScheduler](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskScheduler.html) 提供了指导。 


<a name="java-role-name"></a>
## <a name="role-name"></a>角色名称

有时候，可能需要对组件名称在[应用程序映射](../../azure-monitor/app/app-map.md)中的显示方式进行自定义。 为此，可执行以下操作之一，以便手动设置 `cloud_RoleName`：

- 如果你通过 Application Insights Spring Boot 入门版使用 Spring Boot，则唯一需要执行的更改是在 application.properties 文件中为应用程序设置自定义名称。

  `spring.application.name=<name-of-app>`

  Spring Boot 入门版会自动将 `cloudRoleName` 分配给你为 `spring.application.name` 属性输入的值。

- 如果使用的是 `WebRequestTrackingFilter`，则 `WebAppNameContextInitializer` 将自动设置应用程序名称。 将以下内容添加到配置文件 (ApplicationInsights.xml)：

  ```XML
  <ContextInitializers>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebAppNameContextInitializer" />
  </ContextInitializers>
  ```

- 如果使用云上下文类，则执行以下命令：

  ```Java
  telemetryClient.getContext().getCloud().setRole("My Component Name");
  ```

## <a name="next-steps"></a>后续步骤

- 编写[自定义遥测](../../azure-monitor/app/api-custom-events-metrics.md)。
- 详细了解如何为其他 SDK [设置 cloud_RoleName](../../azure-monitor/app/app-map.md#set-cloud-role-name)。
- 在 Application Insights 中载入微服务的所有组件。 查看[支持的平台](../../azure-monitor/app/platforms.md)。
- 有关 Application Insights 的类型，请参阅[数据模型](../../azure-monitor/app/data-model.md)。
- 了解如何[扩展和筛选遥测](../../azure-monitor/app/api-filtering-sampling.md)。
- 参阅 [Application Insights 配置参考](configuration-with-applicationinsights-config.md)。




