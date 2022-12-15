---
sidebar_position: 2
title: Minimal API with native AOT
description: A guide to taking an ASP.NET Minimal API and running it on AWS Lambda with .NET 7 native AOT.
keywords: [dotnet cli asp.net api nativeaot]
---

Ok, now we've got the basics out of the way. Let's take a look at how to supercharge your cold start performance using .NET 7 native AOT.

**Big disclaimer here! Microsoft DO NOT officially support ASP.NET for native AOT in .NET 7 (maybe in .NET 8). This approach is functional, but your mileage may vary**

# Start a new project

To demonstrate how to use native AOT with ASP.NET minimal API's on AWS Lambda let's continue right from where we left off in the last section. Adding native AOT support is pretty straightforward.

<CH.Scrollycoding>

## Package Versions

The first thing you'll need to do is ensure you're using version 1.5.0 or later of the _`Amazon.Lambda.AspNetCoreServer.Hosting`_ package.

```xml minimal-native-aot focus=8:8
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Amazon.Lambda.AspNetCoreServer.Hosting" Version="1.5.0" />
  </ItemGroup>
</Project>

```

---

## Source Generated Serialization

The second thing you'll need to do is add a new partial class to your project. This class will configure a [source generated serializer](https://devblogs.microsoft.com/dotnet/try-the-new-system-text-json-source-generator/). Source generated serializers generate classes at compile time to work with JSON, as opposed to the reflection based approach used in System.Text.Json and Newtonsoft.Json.

```c# minimal-native-aot 
using System.Text.Json;
using Amazon.Lambda.APIGatewayEvents;

namespace MinimalApi.OnLambda;

[JsonSerializable(typeof(WeatherForecast))]
[JsonSerializable(typeof(APIGatewayHttpApiV2ProxyRequest))]
[JsonSerializable(typeof(APIGatewayHttpApiV2ProxyResponse))]
public partial class ApiSerializationContext : JsonSerializerContext
{
}

```

---

## Using Statements

Add two additional _`using`_ statements. One for the Amazon Lambda serialization libraries, and one to ensure the _`ApiSerializationContext`_ is available to the _`Program.cs`_ file.


```c# minimal-native-aot focus=1:2
using Amazon.Lambda.Serialization.SystemTextJson;
using MinimalApi.OnLambda;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAWSLambdaHosting(LambdaEventSource.HttpApi, options =>
{
    options.Serializer = new SourceGeneratorLambdaJsonSerializer<ApiSerializationContext>();
});
            
builder.Services.ConfigureHttpJsonOptions(options =>
{
    options.SerializerOptions.AddContext<ApiSerializationContext>();
});

var app = builder.Build();

// Configure the HTTP request pipeline.

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast =  Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
});

app.Run();

public record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}

```

---

## AWS Lambda Hosting Configuration

Second, you'll need to add an additional parameter to the _`AddAWSLambdaHosting`_ method. With this parameter you can configure the serializer for the AWS tooling to use. This allows the inbound payload from API Gateway to be (de)serialized into a request ASP.NET understands. The _`SourceGeneratorLambdaJsonSerializer`_ class is provided by the _`Amazon.Lambda.Serialization`_ package.


```c# minimal-native-aot focus=6:9
using Amazon.Lambda.Serialization.SystemTextJson;
using MinimalApi.OnLambda;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAWSLambdaHosting(LambdaEventSource.HttpApi, options =>
{
    options.Serializer = new SourceGeneratorLambdaJsonSerializer<ApiSerializationContext>();
});
            
builder.Services.ConfigureHttpJsonOptions(options =>
{
    options.SerializerOptions.AddContext<ApiSerializationContext>();
});

var app = builder.Build();

// Configure the HTTP request pipeline.

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast =  Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
});

app.Run();

public record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}

```

---

## Pass Serialization Context to ASP.NET

The serialization context is also passed into ASP.NET's HTTP JSON options. This let's the ASP.NET model binding use the source generated serializers generated at compile time.


```c# minimal-native-aot focus=11:14
using Amazon.Lambda.Serialization.SystemTextJson;
using MinimalApi.OnLambda;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAWSLambdaHosting(LambdaEventSource.HttpApi, options =>
{
    options.Serializer = new SourceGeneratorLambdaJsonSerializer<ApiSerializationContext>();
});
            
builder.Services.ConfigureHttpJsonOptions(options =>
{
    options.SerializerOptions.AddContext<ApiSerializationContext>();
});

var app = builder.Build();

// Configure the HTTP request pipeline.

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast =  Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
});

app.Run();

public record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}

```

</CH.Scrollycoding>

And that's it! That is all you need to do to enable native AOT for your ASP.NET minimal API's, and run them on AWS Lambda.

If your video is more your thing, I have a [walkthrough on my YouTube channel](https://www.youtube.com/watch?v=exRLCRHpvQE) that covers these same concepts.