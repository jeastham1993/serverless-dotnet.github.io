---
sidebar_position: 1
title: Native AOT
---

[Native ahead of time (AOT)](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/) compilation is a new feature that went live with the general availability of .NET 7. Native AOT produces an application that is pre-compiled to native code. This allows users of the app to run it on a machine without the .NET runtime being installed.

One of the benefits of native AOT is the start-up speed. As the application is pre-compiled to native code there is no need for just-in-time (JIT) compilati0on, instead the app is ready to run immediately. This is beneficial in environments with lots of deployed instances, like AWS Lambda. Native AOT applications target a specific runtime and must be compiled on the OS and processor architecture that they will run on in production.

> Due to the underlying OS of AWS Lambda, native AOT will only run on the x84 architecture.

There are some limitations to native AOT, the biggest being the lack of support for run-time code generation. This has an impact on any systems that use unconstrained reflection. Both System.Text.Json and Newtonsoft.Json rely heavily on reflection to function, meaning any JSON (de)serialiazation using either of these libraryies will require changes.

Let's dive into how you can run native AOT applications on AWS Lambda.

## AWS Tooling

AWS announched tooling to support to make it easier to build and deploy native AOT applications to Lambda. This tooling makes use of [Docker](https://www.docker.com/), ensure you have Docker running on your system. Also ensure you have version 5.6.0 or later of the [Amazon.Lambda.Tools](https://www.nuget.org/packages/Amazon.Lambda.Tools).

```bash install-tools
dotnet tool install --global Amazon.Lambda.Tools
```

AWS also announched pre-built templates to quickly get started with native AOT. Ensure you have the latest version of the [Amazon.Lambda.Templates](https://www.nuget.org/packages/Amazon.Lambda.Templates).

```bash install-tools
dotnet new --install Amazon.Lambda.Templates
```

## Getting Started

To get started with your first native AOT Lambda function run the following command to start a new project:

```bash new-native-aot
dotnet new lambda.NativeAOT -n LambdaNativeAot
```

Open up the project in the IDE of your choice and let's have a look at the code.

<CH.Scrollycoding>

## Function Code

Native AOT compiles application code down to a single binary. This means the application entry-point needs to be a _`static Main()`_ method. The main method uses the _`LambdaBootstrapBuilder`_ class that comes from the _`Amazon.Lambda.RuntimeSupport`_ Nuget package. The _`.Create`_ method bootstraps the Lambda runtime, passing in the actual FunctionHandler method as well as a serializer to use. The _`.RunAsync()`_ method makes the function ready to receive requests.

```c# Function.cs focus=10:16
using Amazon.Lambda.Core;
using Amazon.Lambda.RuntimeSupport;
using Amazon.Lambda.Serialization.SystemTextJson;
using System.Text.Json.Serialization;

namespace LambdaNativeAot;

public class Function
{
    private static async Task Main()
    {
        Func<string, ILambdaContext, string> handler = FunctionHandler;
        await LambdaBootstrapBuilder.Create(handler, new SourceGeneratorLambdaJsonSerializer<LambdaFunctionJsonSerializerContext>())
            .Build()
            .RunAsync();
    }

    public static string FunctionHandler(string input, ILambdaContext context)
    {
        return input.ToUpper();
    }
}

[JsonSerializable(typeof(string))]
public partial class LambdaFunctionJsonSerializerContext : JsonSerializerContext
{
}

```

---

## (De)Serialization

As mentioned earlier, native AOT removes the support for using common JSON (de)seraizliation libraries. In .NET 6, Microsoft introduced [source generated serializers](https://devblogs.microsoft.com/dotnet/try-the-new-system-text-json-source-generator/). Source generated serialization generates the code required for (de)serailization at compile time. 

To use source generated serializers you need to specify a partial class that inherits from the _`JsonSerializerContext`_ class. Annotations are then added to that class to define which objects to generate compile time code for. In this instance, this is just a string. All objects you need to (de)seriailize need to be added as an annotation, including any Lambda event sources like _`SQSEvent`_ or _`APIGatewayHttpApiV2ProxyRequest`_.

```c# Function.cs focus=24:27
using Amazon.Lambda.Core;
using Amazon.Lambda.RuntimeSupport;
using Amazon.Lambda.Serialization.SystemTextJson;
using System.Text.Json.Serialization;

namespace LambdaNativeAot;

public class Function
{
    private static async Task Main()
    {
        Func<string, ILambdaContext, string> handler = FunctionHandler;
        await LambdaBootstrapBuilder.Create(handler, new SourceGeneratorLambdaJsonSerializer<LambdaFunctionJsonSerializerContext>())
            .Build()
            .RunAsync();
    }

    public static string FunctionHandler(string input, ILambdaContext context)
    {
        return input.ToUpper();
    }
}

[JsonSerializable(typeof(string))]
public partial class LambdaFunctionJsonSerializerContext : JsonSerializerContext
{
}

```

---

## Project File Updates

Updates are required to the csproj file to enable both native AOT and allow the code to run on AWS Lambda. The first is to set the _`TargetFramework`_ to _`net7.0`_.

Native AOT on Lambda makes use of [Lambda custom runtimes](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html). Custom runtimes allow you to bring your own runtime to Lambda. When using a custom runtime the Lambda service looks for a file named _`bootstrap`_. For that reason, the compiled assembly name needs to be output with the name bootstrap. The final change is to set the _`PublishAot`_ flag to true.

```xml Function.cs focus=4:6
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <AssemblyName>bootstrap</AssemblyName>
    <PublishAot>true</PublishAot>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <AWSProjectType>Lambda</AWSProjectType>
    <CopyLocalLockFileAssemblies>true</CopyLocalLockFileAssemblies>
    <StripSymbols>true</StripSymbols>
  </PropertyGroup>
  <ItemGroup Condition="'$(RuntimeIdentifier)' == 'linux-arm64'">
    <RuntimeHostConfigurationOption Include="System.Globalization.AppLocalIcu" Value="68.2.0.9" />
    <PackageReference Include="Microsoft.ICU.ICU4C.Runtime" Version="68.2.0.9" />
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Amazon.Lambda.RuntimeSupport" Version="1.8.2" />
    <PackageReference Include="Amazon.Lambda.Core" Version="2.1.0" />
    <PackageReference Include="Amazon.Lambda.Serialization.SystemTextJson" Version="2.3.0" />
  </ItemGroup>
  <ItemGroup>
    <RdXmlFile Include="rd.xml" />
  </ItemGroup>
</Project>

```

---

## Trimming Options

Native AOT compilation trims your application code, making the bundle size as small as possible. Many Nuget libraries are not yet 'trim friendly', meaning required pieces of code may be removed. Microsoft provide a way to exclude libraries from trimming using the [trimming options](https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/trimming-options?pivots=dotnet-7-0) built into the compiler. To exclude an assembly from the trimming, specify it as a _`<TrimmerRootAssembly>`_.

```xml Function.cs focus=23:31
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <AssemblyName>bootstrap</AssemblyName>
    <PublishAot>true</PublishAot>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <AWSProjectType>Lambda</AWSProjectType>
    <CopyLocalLockFileAssemblies>true</CopyLocalLockFileAssemblies>
    <StripSymbols>true</StripSymbols>
  </PropertyGroup>
  <ItemGroup Condition="'$(RuntimeIdentifier)' == 'linux-arm64'">
    <RuntimeHostConfigurationOption Include="System.Globalization.AppLocalIcu" Value="68.2.0.9" />
    <PackageReference Include="Microsoft.ICU.ICU4C.Runtime" Version="68.2.0.9" />
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Amazon.Lambda.RuntimeSupport" Version="1.8.2" />
    <PackageReference Include="Amazon.Lambda.Core" Version="2.1.0" />
    <PackageReference Include="Amazon.Lambda.Serialization.SystemTextJson" Version="2.3.0" />
  </ItemGroup>
  
  <ItemGroup>
    <TrimmerRootAssembly Include="AWSSDK.Core" />
    <TrimmerRootAssembly Include="AWSSDK.DynamoDBv2" />
    <TrimmerRootAssembly Include="AWSXRayRecorder.Core" />
    <TrimmerRootAssembly Include="AWSXRayRecorder.Handlers.AwsSdk" />
    <TrimmerRootAssembly Include="Shared" />
    <TrimmerRootAssembly Include="Amazon.Lambda.AspNetCoreServer.Hosting" />
    <TrimmerRootAssembly Include="Amazon.Lambda.AspNetCoreServer" />
  </ItemGroup>
</Project>

```

---

</CH.Scrollycoding>

## Deploying

When ready to deploy, it's as simple as using the _`deploy-function`_ command in the CLI global tooling. This command downloads a Docker image built using Amazon Linux 2 (AL2) as a base image. Your local file system is then attached to a running container, your code is compiled within this container. If the deploy-function comamnd is executed on a machine running AL2 then the publish will run as normal.

```bash deploy.sh
dotnet lambda deploy-function
```