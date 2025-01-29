[[_TOC_]]
## Overview
Normally we only need to create new .NET solutions at very early stage of an engagement; and it only needs to be done once. 

# Creating the solution

Generally there are two ways to start a new project:
- Create a new one from IDE or command line using pre-defined template
- or copy an existing solution.

We recommend the former approach because:

- Renaming existing projects could be quite tricky without a proper IDE with refactoring feature (because of project references)
- Removing existing code could also take quite some time (depending on the project size)
- Out-of-date package dependencies
- Again, it's not ideal to "inherit" an existing project GUID, which could cause unexpected conflicts in the future

## Solution Structure
Normally I find it more flexible to create the .NET solution under a `src` folder in the repository root for these benefits:
- Make the repository root folder more clean (only for those global files like `.gitignore`)
- Leave space for those contents as important as code in parallel folders (e.g. `Docs`)

Here's an example :
```
[.]
│   README.md
│   .gitignore
|   .gitattribute    
│
└───[src]
│   │   CodingKata.sln
│   │
│   └───[CodingKata.Api]
│       │   CodingKata.Api.csproj
│       │   Program.cs
│       │   ...
│   
└───[infrastructure]
|   │   api-build.yml
|   │   api-deploy.yml
|   |
└───[docs]
    │   environments.md
    │   architecture.md
    |   ...
```

## Create new .NET solution
There are multiple ways to create a new .NET solution, depending on which IDE or tool you are more comfortable with. Here are some typical options:

- Using Visual Studio: [Link](https://learn.microsoft.com/en-us/dotnet/core/tutorials/with-visual-studio)
- Using Visual Studio Code (with C# Dev Kit): [Link](https://learn.microsoft.com/en-us/dotnet/core/tutorials/with-visual-studio-code)
- Using Rider: [Link](https://www.jetbrains.com/help/rider/Creating_and_Opening_Projects_and_Solutions.html)
- Using SDK command line `dotnet` 

### Hints
- When choosing project, make sure to choose `Web API`
- Make sure to choose `.net9.0` as target framework
- You can let IDE to create new git repository for you when creating the new .NET solution, but it's better to do it separately because the solution folder is one level below (as mentioned earlier)
- Uncheck all those optional checkboxes as they can be added in later stages. (e.g. authentication, docker support) Most sample codes added automatically by IDE at this stage are only for reference and you will need to remove them afterwards anyway.
- Most of the new project created by IDE contains some dummy code (e.g. `WeatherForecast` endpoint) for you to verify the build. Don't forget to remove them afterwards.

### Dotnet Command line

You can get quite a way towards the basic structure using the command line.

```
dotnet new gitignore 
mkdir src
cd src
dotnet new sln --name CodingKata
dotnet new classlib --name CodingKata.Domain
dotnet sln add CodingKata.Domain/CodingKata.Domain.csproj 
dotnet new webapi --name CodingKata.API
dotnet sln add CodingKata.API/CodingKata.API.csproj 
dotnet add CodingKata.API/CodingKata.API.csproj reference CodingKata.Domain/CodingKata.Domain.csproj 
dotnet new editorconfig
```


### Minimal hosting model
ASP.NET Core came with a new `Minimal Hosting Model` which simplifies the application bootstrap and configuration code. When you create a new .NET 9 application the minimal hosting model is used by default. (You can always fall back to the old model with separate `Startup.cs` and `Program.cs` if you want). 

We recommend to use the minimal hosting model unless there's any compelling reason not to (e.g. reuse some configuration from existing projects which doesn't fit for some reason)

### Common file templates
- `.gitignore`: [here](https://github.com/github/gitignore/blob/main/VisualStudio.gitignore)
- `editorconfig`: [here](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/code-style-rule-options) - You might need to restart your IDE for the config to be picked up.


## HTTP Security
### Overview
Some essential HTTP headers need to be added to enhance the web security. Read [this article](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html) as an overview of all security-related HTTP headers and their recommended configurations. 

### HSTS and HTTP Redirection
For security, we should enable these in any new projects on non-development environments to enforce HTTPS:
- Issue HTTP response to redirect from `http://` to `https://`
- Add HSTS header to tell browser to always visit the page using `https://`

They can be achieved by adding `.UseHttpsRedirection()` and `.UseHsts()` respectively in ASP.NET pipeline. For more details on this topic, read [this](https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?tabs=visual-studio%2Clinux-sles).

### Other common HTTP security headers for API
You can use [this library](https://github.com/andrewlock/NetEscapades.AspNetCore.SecurityHeaders) to add those common security headers in just one-line-of-code. Please be aware that many security HTTP headers are only applicable to _document_ responses (`text/html`, `text/javascript`, `application/javascript`) hence they won't appear in API responses (`application/json`).


## Add global exception handler
When referring to "Global Exception Handler", we are talking about these features when unhandled exception occurs in controller:
- Catch unhandled exceptions
- Logging unhandled exceptions for diagnosis
- Map error codes/exception types to HTTP status code
- Return standardized response as per RFC7807 specification

Before .NET 8, we used to implement global error/exception handler in a dedicated ASP.NET Core middleware. It's still a valid approach in .NET 8; however there are some new concepts and classes introduced in .NET 8 which helps to simplify the exception handling in Web API. Check [this](https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors) for more details.

Follow [this](https://medium.com/@AntonAntonov88/handling-errors-with-iexceptionhandler-in-asp-net-core-8-0-48c71654cc2e) or [this](https://www.milanjovanovic.tech/blog/global-error-handling-in-aspnetcore-8) to add a global exception handler so that we don't need to repeat the same logic in every single Controller class.

## Dependency Injection
3rd party dependency injection libraries were pretty popular choices before .NET 6, e.g. [Autofac](https://autofac.org/), [NInject](https://github.com/ninject/Ninject). Nowadays more projects lean back on the native dependency injection from .NET framework as it's good enough in most cases.

To learn more about .NET dependency injection, click [here](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)

# API Documentation and Code Generation
In the past, lots of manual work was required in the API documentation process. Nowadays with the presence of [OpenAPI](https://www.openapis.org/) standard and various open source tools, it has become much easier. 

There are two major implementations of the OpenAPI specification: `Swashbuckle` and `NSwag`. Here's an article which summarise the differences: [link](https://code-maze.com/aspnetcore-swashbuckle-vs-nswag/). In a nutshell, `Swashbuckle` provides more flexibility in configuration and `NSwag` provides built-in codegen support. Also, if you create your project from IDE then `Swashbuckle` has (most likely) already been added by default. 

In the sample project, we use `Swashbuckle` for Swagger UI and `NSwag` for typescript codegen.

## How to add Swagger/OpenAPI with UI
Follow [this page](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/overview)

## Typescript codegen

### Preparation
Firstly you need to add `NSwag` into .NET pipeline: [here](https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-nswag?tabs=visual-studio#add-and-configure-swagger-middleware)

### To generate DTO types in typescript
1. You need to add NSwag configuration document (`nswag.json`). Here's the [definition](https://github.com/RicoSuter/NSwag/wiki/NSwag-Configuration-Document). You can generate it using [NSwagStudio](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio) from scratch.
1. Integrate NSwag codegen in MSBuild to keep the typescript DTO always up-to-date: [follow this](https://github.com/RicoSuter/NSwag/wiki/NSwag.MSBuild)

### To generate API clients in typescript
Follow [this](https://github.com/RicoSuter/NSwag/wiki/TypeScriptClientGenerator)

### Using NSwag on a mac

Some of the tools above don't work on a mac, however the npm CLI commands work.

#### Install the tools:
```
dotnet add CodingKata.API/CodingKata.API.csproj package NSwag.AspNetCore
sudo npm install nswag -g
```
#### Update the aspnet code

Change from 
 `app.UseSwagger();`
to 
 `app.UseOpenApi();`

Add 
```
builder.Services.AddOpenApiDocument(config => {
    //TODO: config goes here
});
```
#### Generate the ts client
```
nswag aspnetcore2openapi /output:swagger.json  
nswag openapi2tsclient /input:swagger.json /output:ApiModule.ts
```
#### Integrate with the build

```
<Target Name="swaggergen" AfterTargets="Build" Outputs="$(RootFolder)\swagger.json">
    <Exec Command="nswag aspnetcore2openapi /nobuild:true /output:swagger.json" ConsoleToMSBuild="true"/>
    <Exec Command="nswag openapi2tsclient /input:swagger.json /output:ApiModule.ts" ConsoleToMSBuild="true"/>
</Target>
```

## Swift Codegen
Apple has provided official library for generating both DTO types and client code in Swift based on OpenAPI document.

[Swift-OpenAPI-Generator](https://www.swift.org/blog/introducing-swift-openapi-generator/#swift-openapi-generator)

Follow [this](https://swiftpackageindex.com/apple/swift-openapi-generator/1.3.0/tutorials/swift-openapi-generator/clientswiftpm) tutorial to create a new local Swift Package for generated code.

Please be aware that you need to run this on MacOS. In theory it should work under any Swift runtime (e.g. Windows, WSL2, Linux).

# Logging

## Key Concepts
**Logging API**: the interface which our code calls to log contents. Normally we use the default `Microsoft.Extensions.Logging` from .NET framework to maximise code reusability.

**Logging Framework**: the library which converts the log content to structured log messages. The most popular choice in .NET world is `Serilog`. 

**Logging Sink**: where the structured log messages flow to eventually. It could be a text file in local file system, a web service (via RPC/HTTP call), or even a database.

**Structured logging**: Instead of using arbitrary strings in log messages, use an organised log format to make it easier to search, filter and analyse log data. Check [this article](https://newrelic.com/blog/how-to-relic/structured-logging) and [this](https://github.com/serilog/serilog/wiki/Structured-Data) for more details. Most modern logging frameworks support structured logging.

::: mermaid
flowchart LR
    subgraph api[Logging API]
    mel[Microsoft.Extensions.Logging]
    end
    subgraph framework[Logging Framework]
    serilog[Serilog]
    end
    subgraph sink[Logging Sink]
    file[Text file]
    console[Console]
    seq[Seq Service]
    db[Database]
    appinsights[Azure App Insights]
    end
    mel --> serilog
    serilog --> file
    serilog --> console
    serilog --> seq
    serilog --> appinsights
    serilog --> db
:::

## Configuration
Here's an overview on .NET logging structure and how to enable it in .NET 9 pipeline: [link](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging?tabs=command-line)

Follow [this page](https://github.com/serilog/serilog/wiki/Configuration-Basics) for instructions on how to add Serilog as logging framework.

Here's a [list of sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks) which Serilog supports.

Serilog can be configured entirely in configuration file, from code, or both(mixed). It's up to the team's preference in choosing which approach. In the sample project, only default log level (and package-level overrides) is exposed in `appsettings.json`; all the other configurations are applied in code. 

## .NET High performance logging
In order to reduce computational cost in logging messages, Microsoft provides a guideline and tools known as [High-performance logging](https://learn.microsoft.com/en-us/dotnet/core/extensions/high-performance-logging). Combined with [Compile-time logging source generation](https://learn.microsoft.com/en-us/dotnet/core/extensions/logger-message-generator), you will end up having cleaner, faster and more organised and consistent logging codes than before.

# Add MediatR with global error handler

CQRS stands for _Command and Query Responsibility Segregation_, an architectural pattern which is used to separate the data-reading operations from the data-writing ones for better performance, scalability and security. For more information about CQRS pattern, check [this page](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs).

## Add MediatR in project
One common choice when applying CQRS in .NET world is the MediatR library. Follow [this blog](https://medium.com/@EdsonMZ/implementing-the-cqrs-and-mediator-pattern-in-a-net-8-web-api-8c0319a4525c) for detailed instructions.

## Add validators via MediatR pipeline
It's always a non-trivial work to set up a mechanism for validating http requests and surfacing validation violations in a consistent and neat way. Luckily, it becomes easier with the help from MediatR pipeline and `FluentValidation` library.  

Check [this blog](https://code-maze.com/cqrs-mediatr-fluentvalidation/) for more details about how to add input validation via MediatR.

# Add basic CRUD with API controllers

## Controller-based APIs v.s. Minimal APIs
A new way to define Web API was added in .NET 6 called minimal API, which defines endpoints with logical handlers in lambdas instead of Controller classes. In the sample project we still use the traditional Controller-based approach. To learn more about the differences, read [this](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/apis).

Please be aware that most new projects created from templates in IDE use minimal API for brevity. If you decide to only use controller-based API, it's better to remove those configurations specifically for minimal API (e.g. `AddEndpointsApiExplorer()`)

## Routing
Routing in .NET 9 Web API is pretty straightforward but quite flexible if you want more customised control over the default offerings. For more information about routing, click [here](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing)

Routing is configured in ASP.NET Core pipeline where we specify those middleware components (including their orders and behaviour) that we want to execute on every HTTP request. For more information on how to configure pipeline, read [this](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/)

## JSON Serialisation
.NET 8 uses `System.Text.Json` as the default JSON serialiser when handling HTTP requests. You can customise the serialisation behaviour in Web API via `JsonSerialiserOptions` if you want to:
- Customise how it handles Enum types
- Customise how it handles NULL values
- Customise how it handles property name cases (e.g. `camelCase` v.s. `kebab-case`)
- Support serialisation of types from 3rd party library (e.g. [NodaTime](https://nodatime.org/3.1.x/userguide/serialization))
- or even use another JSON serialiser e.g. [Json.NET](https://www.newtonsoft.com/json)

For more details, click [here](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.jsonserializeroptions?view=net-9.0)

## Entities, DTOs and Models
There are many ways in designing a domain model for a project; and like many other problems, the answers are normally different depending on project size and complicity. It looks like an overkill for a small project, but it plays an important role in keeping code consistency and business logic integrity in medium to large project. 

[Domain-drive design](https://en.wikipedia.org/wiki/Domain-driven_design) is one of the most popular approaches. It's a very big topic which can actually be a separate Kata. 

In the sample project we follow a minimum convention:
- All the types defined in domain layer are called `Entities`
- All the types returned from the API (as output) are called `Models` or `ViewModels`
- All the types the API accepts (as input) are called `DTOs`

Again, this is just ONE way, not THE ONLY way. Actually in many projects the boundary between Models and DTOs is very vague, which is totally fine. There's no 100% correct answer in modelling.  

It's recommended to define entities, models and DTOs in a separate `Domain` project for better readability and reusability.

## .http files
In order to test our Web API during development, various REST clients can be used including Postman, Insomnia, [PowerShell script](https://www.twilio.com/docs/usage/tutorials/how-to-make-http-basic-request-twilio-powershell) or even Google Chrome with plugins. A new `.http` file type was introduced in Visual Studio 2022 which provides another way to define, execute and share HTTP requests. 

For more information about `.http` syntax and how to use them (including Visual Studio Code), click [here](https://learn.microsoft.com/en-us/aspnet/core/test/http-files)

# Testing

## Unit Test 
To ensure the robustness of your application, it’s crucial to focus on the business logic. This involves creating unit tests that thoroughly validate the core functionality and behavior of your services. Here’s how you can achieve this:

1. Add Mocking
Mocking is a technique used in unit testing to simulate the behavior of complex objects or external dependencies. This allows you to isolate the business logic and test it independently.
Moq is a popular mocking framework for .NET, which provides a simple API for creating mock objects and verifying interactions.

2. Add Tests on the Business Logic
Add unit tests ensure that all public methods and properties are covered


### Best Practice
https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices

## Integrated Test 
Integration tests evaluate an app's components on a broader level than unit tests. Integration tests confirm that two or more app components work together to produce an expected result, possibly including every component required to fully process a request. Separate unit tests from integration tests into different projects to allow controls over which set of tests are run. 

1. Add Microsoft.AspNetCore.Mvc.Testing to allow mocking of the request 

2. Add Tests on the Endpoint
Add integrated tests should cover the interaction between multiple components

Read [ASP.NET Core integration tests](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests) for details of how to use TestServer for your API.

# Health Checks
Health checks are a useful way to quickly determine if all the key parts of an application are operational. In more complex or high availability systems, these checks can be used to notify load balancers of unhealthy nodes, or allow new instances to be started and unhealthy ones stopped by the cluster manager.

https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks

## Health Check Endpoint
Adding a simple availability endpoint can be added with the following lines in Program.cs

```
builder.Services.AddHealthChecks();
//...
app.MapHealthChecks("/health");
```

If the app is running, then the endpoint e.g. http://localhost:5201/health will return a successful result.

For SQL connection check use 'AspNetCore.HealthChecks.SqlServer' nuget package and

```
builder.Services.AddHealthChecks()
                .AddSqlServer(connectionString);
```
Implement _IHealthCheck_ interface for custom health checks as per https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks#create-health-checks
## Health Check UI
The UI provided by https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks consists of a few parts:
Add these packages to the API project
```
dotnet add package AspNetCore.HealthChecks.UI
dotnet add package AspNetCore.HealthChecks.UI.Client
dotnet add package AspNetCore.HealthChecks.UI.InMemory.Storage
```
1. An endpoint that returns the details each health check
```
app.MapHealthChecks("/health-detail",new HealthCheckOptions{
    Predicate = _ => true,
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
});
```
2. Services to aggregate the health check endpoints and store the results
```
builder.Services
            .AddHealthChecksUI(setupSettings: setup =>
                {
                    setup.AddHealthCheckEndpoint("endpoint1", "http://localhost:5201/health-detail");
                })
            .AddInMemoryStorage();
```
3. The HealthCheck UI endpoint
```
app.UseRouting()
    .UseEndpoints(config => config.MapHealthChecksUI());
```

# Blogs and Further Reading
1. https://www.kaels-kabbage.com/posts/mediatr-caching/
2. [Vite and Aspnet core integration](https://github.com/Eptagone/Vite.AspNetCore)
