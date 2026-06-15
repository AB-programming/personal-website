---
external: false
title: Aspire 入门指南
description: Aspire是一个面向分布式应用的代码优先编排和可观测层，它提供一个平台来编排和管理你的服务、前端、数据库、队列、容器和云依赖。
date: 2026-06-15
---

Aspire是一个面向分布式应用的代码优先编排和可观测层，它提供一个平台来编排和管理你的服务、前端、数据库、队列、容器和云依赖。

# 构建一个MVP
## 安装Aspire
MacOS/Linux可以通过以下方式安装：
```bash
curl -sSL https://aspire.dev/install.sh | bash
```
Windows可以通过以下方式安装：
```powershell
irm https://aspire.dev/install.ps1 | iex
```
安装完成后可以通过以下命令检查：
```bash
aspire --version
```
详细可参照官方文档：[Aspire官方文档](https://aspire.dev/get-started/install-cli/)

## 初始化项目
```bash
aspire new aspire-starter -n AspireApp -o AspireApp
```
过程如下：
```bash
使用 *.dev.localhost URL [y/N]: y
✅ 使用 *.dev.localhost URL 进行本地开发。
使用 Redis 缓存 [Y/n]: y
✅ 使用 Redis 缓存进行缓存。
是否要创建测试项目? [y/N]: y
选择测试框架 MSTest
✅ 使用 MSTest 进行测试。
📦 使用项目模板版本: 13.4.3
📦 Created or updated NuGet.config in the project directory with required package sources.
✅ 已在 C:\project\AspireApp 中成功创建项目。

Would you like to configure AI agent environments for this project? [Y/n]: y
🤖 Installed Aspire agent skills:
     Skills: aspire, aspire-deployment, aspire-init, aspire-monitoring, aspire-orchestration
     Locations: .agents/skills
✅ 智能体环境配置完成。
```

项目结构大致如下：
```plaintext
AspireApp/
├── .agents/skills/
│   ├── aspire/
│   ├── aspire-deployment/
│   ├── aspire-init/
│   ├── aspire-monitoring/
│   └── aspire-orchestration/
├── AspireApp.ApiService/
│   ├── Properties/
│   ├── Program.cs
│   ├── AspireApp.ApiService.csproj
│   ├── AspireApp.ApiService.http
│   ├── appsettings.json
│   └── appsettings.Development.json
├── AspireApp.AppHost/
│   ├── Properties/
│   ├── AppHost.cs
│   ├── AspireApp.AppHost.csproj
│   ├── appsettings.json
│   └── appsettings.Development.json
├── AspireApp.ServiceDefaults/
│   ├── Extensions.cs
│   └── AspireApp.ServiceDefaults.csproj
├── AspireApp.Tests/
│   ├── WebTests.cs
│   └── AspireApp.Tests.csproj
├── AspireApp.Web/
│   ├── Components/
│   │   ├── Layout/
│   │   │   ├── MainLayout.razor
│   │   │   ├── MainLayout.razor.css
│   │   │   ├── NavMenu.razor
│   │   │   └── NavMenu.razor.css
│   │   ├── Pages/
│   │   │   ├── Counter.razor
│   │   │   ├── Error.razor
│   │   │   ├── Home.razor
│   │   │   └── Weather.razor
│   │   ├── App.razor
│   │   ├── Routes.razor
│   │   └── _Imports.razor
│   ├── Properties/
│   ├── wwwroot/
│   │   ├── lib/bootstrap/dist/
│   │   ├── app.css
│   │   └── favicon.png
│   ├── Program.cs
│   ├── WeatherApiClient.cs
│   ├── AspireApp.Web.csproj
│   ├── appsettings.json
│   └── appsettings.Development.json
├── AspireApp.sln
├── aspire.config.json
└── nuget.config
```
将项目初始化好后，进入AspireApp目录，通过以下命令可以启动：
```bash
aspire run
```
点击仪表盘链接进入页面：
![Aspire仪表盘](/images/2026-06-15/1.webp)

注意：创建项目时使用了Redis进行缓存，在Aspire中，这需要依赖本机docker环境

可以看到，当前Aspire应用中有三个资源，一个Redis、一个前端（Blazor）和一个后端（ASP.NET Core Web API）应用，现在尝试添加一个资源。

使用以下命令添加一个PostgreSQL：
```bash
aspire add postgres
```
从 `AspireApp.AppHost/AppHost.cs` 中加入以下代码：
```csharp
var builder = DistributedApplication.CreateBuilder(args);

var cache = builder.AddRedis("cache")
    .WithLifetime(ContainerLifetime.Persistent);

var db = builder.AddPostgres("postgres")
    .WithLifetime(ContainerLifetime.Persistent)
    .AddDatabase("mydb");

var apiService = builder.AddProject<Projects.AspireApp_ApiService>("apiservice")
    .WithHttpHealthCheck("/health");

builder.AddProject<Projects.AspireApp_Web>("webfrontend")
    .WithExternalHttpEndpoints()
    .WithHttpHealthCheck("/health")
    .WithReference(cache)
    .WaitFor(cache)
    .WithReference(apiService)
    .WaitFor(apiService);

builder.Build().Run();
```
再次启动就会看到Resources中多了一个postgres资源，其下有一个mydb的数据库。

注意：postgres也需要有docker环境，会创建一个容器。

## 实现一个简单的Todo
当前Aspire项目中有一个前端服务、一个后端服务、一个数据库和一个缓存。可以构建一个简单的应用了，这里尝试构建一个todo，项目目录下有.agent/skills，可以直接丢给智能体让其帮忙完成。

如下是我给智能体的描述：

当前是一个Aspire项目，相关skill在.agent/skills下，当前项目下管理了一个前端和后端项目，分别是apiservice和webfrontend，还管理了一个Postgres和Redis，根据这些内容帮我写一个简单的Todo应用，要求用上数据库和缓存。


示例代码：[Aspire示例代码仓库](https://github.com/AB-programming/aspireapp-example)

# 相关概念
## AppHost
AppHost用来声明应用程序服务及关系，负责本地编排、服务发现、依赖注入和可观测性接入。Aspire强调一个“代码优先（code-first）”的模型，AppHost就是一个code-first的架构关系描述器。

例如在AppHost.cs中，定义了四个应用服务，并且使用`WithReference`和`WaitFor`来声明相互之间的依赖关系，`WithReference`是需要应用另一个服务，`WaitFor`是需要等待另一个服务启动后启动。例如apiservice如果依赖一个数据库和一个缓存，使用了`WithReference`和`WaitFor`，那么Aspire就会在启动apiservice之前把数据库和缓存的connection string自动注入并且确保数据库和缓存启动起来。

AppHost会自动启动所有服务，管理启动的顺序，进行服务发现，管理容器，统一日志，在Graph中也可以它们之间的依赖关系
![依赖图](/images/2026-06-15/2.webp)

可观测性接入就是指Aspire提供了一个Dashboard来进行分布式追踪、日志、服务拓扑图等，无需进行额外配置，AppHost自动接入Dashboard。

如果想让别的语言服务也接入Aspire的话，可以使用Language & runtime，详见[languages-and-runtimes](https://aspire.dev/languages-and-runtimes/)

## Resources
Aspire中，资源包括

- **人工智能服务**：大型语言模型、人工智能端点和认知服务。
- **缓存**：Redis、内存缓存和分布式缓存解决方案。
- **容器**：用于数据库、消息代理和其他服务的 Docker 容器。
- **数据库**：SQL Server、PostgreSQL、MySQL、MongoDB 和其他数据存储。
- **可执行文件**：控制台应用程序、脚本和后台服务。
- **框架**：使用各种框架构建的 Web 应用程序、API 和微服务。
- **消息服务**：服务总线、RabbitMQ、Kafka 和其他消息系统。
- **项目**：C# 项目、Node.js 应用程序、Python 服务等等。
- **存储**：Blob 存储、文件系统和云存储服务。

资源的整个生命周期包括了服务发现、启动、健康检查和停止。

## Health check
健康检查是一种确定资源是否准备好且正常运行的机制，健康检查有两种形式，一种是AppHost级别的健康检查，WaitFor中会进行健康检查，另一种是用作负载均衡的端点/health或/alive。

## Named endpoints
Named endpoints的格式形如`scheme://_endpointName.serviceName`，endpointName对应的一个端口名，serviceName对应的一个服务名，本例中后端服务名为apiservice，默认端口名为空，其Named endpoints为`https+http://apiservice`

在前端AspireApp.Web模块的Program.cs注册HttpClient
```csharp
builder.Services.AddHttpClient<TodoApiClient>(client =>
    {
        client.BaseAddress = new("https+http://apiservice");
    });
```

## Inner-loop networking
Aspire内部有一个工作的逻辑网络层，其中有四个元素，Endpoints、Proxies、Proxyless endpoints、Container networks，端点配置通过代理进行转发，转发到实际的地址，IsProxied配置为false则为无代理端点，不经过Aspire内部的代理转发，Aspire还为容器创建和管理专用网络，让容器在本地开发中能相互发现和通信。

为了能更好的理解内部的网络，官方也给出了一个示意图，如下：
![内部网络](/images/2026-06-15/3.webp)
图片来源：https://aspire.dev/fundamentals/networking-overview/

## 参考
[Aspire documentation home](https://aspire.dev/docs/)
