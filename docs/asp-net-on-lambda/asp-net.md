---
sidebar_position: 1
title: ASP.NET on Lambda
description: A guide to taking an ASP.NET WebAPI and running it on AWS Lambda.
keywords: [dotnet cli asp.net api]
---

# Start a new project

It's possible to run a traditional ASP.NET on AWS Lambda without changing a single line of code in your existing application. By traditional, I mean an API that was around before Microsoft got rid of the _`Startup.cs`_ file in new projects created from .NET 6 onwards. If you have an existing API that still has a Startup.cs file, read on. If you have an API that uses a single _`Program.cs`_ file with all the startup logic, check out the documentation on [minimal API's on Lambda](/docs/asp-net-on-lambda/minimal-apis).

To demonstrate how to run do this, let's start with a brand new project. Yes, I know .NET Core 3.1 is out of support! This is the only way to generate a new project that still includes a _`Startup.cs`_ file.

```shellscript new-project
dotnet new webapi -f netcoreapp3.1 -n WebApi.OnLambda
```

Once initialised, open the project in the IDE of your choice. The sample API contains a single controller with a single _`GET /weatherforecast`_ endpoint. Remember, we are going to run this application on Lambda without changing a single line of existing application code.

<CH.Scrollycoding>

## .NET 6

The first thing to do is bump the _`TargetFramework`_ to be net6.0. You can do that in the WebApi.OnLambda.csproj file.

```xml project-file focus=3
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>
</Project>

```

---
## Nuget Package

The next step to enable Lambda support is to install the _`Amazon.Lambda.AspNetCoreServer`_ Nuget package. This package provides classes and functionality to proxy the event payload passed to Lambda into a format that ASP.NET can understand

```shellscript nuget-install
dotnet add package Amazon.Lambda.AspNetCoreServer
```

---

## New Class

The third step is to create a new class in your project. This class is the entry point that Lambda will use when invoking your function. The name of the class is irrelevant, but I'd probably recommend calling it something sensible like _`LambdaEntryPoint.cs`_. This class needs to inherit from one of either _`APIGatewayHttpApiV2ProxyFunction`_, _`APIGatewayProxyFunction`_ or _`ApplicationLoadBalancerFunction`_. Choose the correct base class depending on what you are putting in front of Lambda.

```c# aspnet-on-lambda focus=5:5
namespace WebApi.OnLambda;

using Amazon.Lambda.AspNetCoreServer;

public class LambdaEntryPoint : APIGatewayHttpApiV2ProxyFunction
{
    /// <inheritdoc />
    protected override void Init(IWebHostBuilder builder)
    {
        builder.UseStartup<Startup>();
    }
}
```

---

## Configure Startup

You'll also need to override the _`Init`_ method to configure the correct _`Startup`_ class. You could also use this override to perform any Lambda specific startup logic that you may not want to do when running outside of Lambda. There are more details on this configuration in the [Amazon.Lambda.Tools GitHub repo](https://github.com/aws/aws-lambda-dotnet/tree/master/Libraries/src/Amazon.Lambda.AspNetCoreServer).

```c# aspnet-on-lambda focus=8:12
namespace WebApi.OnLambda;

using Amazon.Lambda.AspNetCoreServer;

public class LambdaEntryPoint : APIGatewayHttpApiV2ProxyFunction
{
    /// <inheritdoc />
    protected override void Init(IWebHostBuilder builder)
    {
        // Add any Lambda specific startup logic.
        builder.UseStartup<Startup>();
    }
}
```

---

## Add deployment configuration

Finally, add a new JSON file to your project named _`aws-lambda-tools-defaults.json`_. For more information on the contents of this file, checkout the [Deployment Configuration tutorial](./docs/tutorial-your-first-lambda/deployment-configuration). The most important setting in here is the _`FunctionHandler`_. The handler will always be in the format _`ASSEMBLYNAME::NAMESPACE.CLASS_NAME_FROM_PREVIOUS_STEP::FunctionHandlerAsync`_. All of the inherited base classes contain a method named _`FunctionHandlerAsync`_, this is what the Lambda service needs to invoke.

```json aws-lambda-tools-defaults.json focus=15:15
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "WebApi.OnLambda::WebApi.OnLambda.LambdaEntryPoint::FunctionHandlerAsync"
}
```

</CH.Scrollycoding>

And that's it, that is all you need to do to enable your existing ASP.NET API to be hosted on AWS Lambda. To deploy this to AWS Lambda, you can follow the instructions in the [deploy tutorial](/docs/tutorial-your-first-lambda/deploy).

Use the below JSON payload to test your Lambda function, either in the AWS Console or using the _`dotnet lambda invoke-function`_ CLI Command.

```json
{
  "version": "2.0",
  "routeKey": "$default",
  "rawPath": "/weatherforecast",
  "requestContext": {
    "http": {
      "method": "GET",
      "path": "/weatherforecast"
    }
  },
  "isBase64Encoded": false
}
```