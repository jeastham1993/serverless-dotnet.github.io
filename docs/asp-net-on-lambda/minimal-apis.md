---
sidebar_position: 1
title: Minimal API's
description: A guide to taking an ASP.NET Minimal API and running it on AWS Lambda.
keywords: [dotnet cli asp.net api]
---

# Start a new project

To demonstrate how to run an ASP.NET minimal API on AWS Lambda let's start with a brand new project. It's possible to do this by adding a single line of code to your application.

To create a new minimal API run the below command:

```shellscript new-project
dotnet new webapi -minimal --no-https --no-openapi -n MinimalApi.OnLambda
```

Once initialised, open the project in the IDE of your choice. The sample API contains a single endpoint with a _`/weatherforecast`_ endpoint.

```c# webapi
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

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

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
[]
```

## Adding Lambda support

AWS provides a Nuget package that makes it simple to run ASP.NET on Lambda. In fact, for minimal API's you can do that by adding a single line of code to the application. To start, let's add the Nuget package. Navigate to the folder containing the created _`.csproj`_ file and run:

```shellscript add-package
dotnet add package Amazon.Lambda.AspNetCoreServer.Hosting
```

Once installed, all you need to do is add a single line of code to the _`Program.cs`_ file. That line is:

```c# add-startup
builder.Services.AddAWSLambdaHosting(LambdaEventSource.HttpApi);
```

The call to AddAWSLambdaHosting takes a single parameter. This parameter will differ based on which if the API options is sourcing your Lambda function. The options are API Gateway HTTP or REST API's, as well as an Application Load Balancer.

One of the interesting things about the AWS tooling is that it understands the context in which the API is running. If you run the application on your local machine the built in Kestrel web server will be used. Once running in Lambda, an in memory web server is used.

Under the hood, the tooling takes the event payload Lambda receives and converts that to a HTTP request that ASP.NET understands. The ASP.NET response is then translated back to the response format Lambda expects.

Ensure that line is added before the call to _`builder.Build();`_.

And that's it, that is all you need to do to enable your minimal API to be hosted on AWS Lambda. To deploy this to AWS Lambda, you can follow the instructions in the [deploy tutorial](/docs/tutorial-your-first-lambda/deploy).