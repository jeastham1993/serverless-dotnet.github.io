---
sidebar_position: 2
title: Test Ready Lambda Functions
---

To enable easy testing of our serverless applications there are some slight tweaks to make to our Lambda function. Instead of having a single constructor and performing all of our initiailization there, we are leveraging an internal constructor and dependency injection to give us a flexible easy to test function.

<CH.Scrollycoding>

## The Default Constructor

Remember, the Lambda service **always** needs a public, parameterless constructor. Without this, the Lambda service won't be able to initialize your handler class.

```c# Function.cs focus=6:8
public class Function
{
    private static ListStorageAreasQueryHandler _queryHandler;
    private static ILogger<Function> _logger;

    public Function() : this(null, null)
    {
    }

    internal Function(ListStorageAreasQueryHandler handler, ILogger<Function> logger)
    {
        AWSSDKHandler.RegisterXRayForAllServices();
        
        _queryHandler = handler ?? Startup.ServiceProvider.GetRequiredService<ListStorageAreasQueryHandler>();
        _logger = logger ?? Startup.ServiceProvider.GetRequiredService<ILogger<Function>>();
    }

    public async Task<APIGatewayProxyResponse> Handler(APIGatewayProxyRequest apigProxyEvent,
        ILambdaContext context)
    {
        try
        {
            var queryResult = await _queryHandler.Handle(new ListStorageAreasQuery());

            return new APIGatewayProxyResponse
            {
                Body = JsonSerializer.Serialize(queryResult),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
        catch (AmazonS3Exception e)
        {
            context.Logger.LogLine(e.Message);
            context.Logger.LogLine(e.StackTrace);

            return new APIGatewayProxyResponse
            {
                Body = "[]",
                StatusCode = 500,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```

---

## An internal constructor

All of our initialization logic is seperated into an internal constructor. Notice how the public constructor Lambda will use calls out internal constructor, passing in null values for the services required.

This gives us an entrypoint in which we can pass in mock implementations of our services to test how the logic of our Lambda functions functions with different event payloads.

```c# Function.cs focus=10:16
public class Function
{
    private static ListStorageAreasQueryHandler _queryHandler;
    private static ILogger<Function> _logger;

    public Function() : this(null, null)
    {
    }

    internal Function(ListStorageAreasQueryHandler handler, ILogger<Function> logger)
    {
        AWSSDKHandler.RegisterXRayForAllServices();
        
        _queryHandler = handler ?? Startup.ServiceProvider.GetRequiredService<ListStorageAreasQueryHandler>();
        _logger = logger ?? Startup.ServiceProvider.GetRequiredService<ILogger<Function>>();
    }

    public async Task<APIGatewayProxyResponse> Handler(APIGatewayProxyRequest apigProxyEvent,
        ILambdaContext context)
    {
        try
        {
            var queryResult = await _queryHandler.Handle(new ListStorageAreasQuery());

            return new APIGatewayProxyResponse
            {
                Body = JsonSerializer.Serialize(queryResult),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
        catch (AmazonS3Exception e)
        {
            context.Logger.LogLine(e.Message);
            context.Logger.LogLine(e.StackTrace);

            return new APIGatewayProxyResponse
            {
                Body = "[]",
                StatusCode = 500,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```

---

## Dependency Injection

When deployed, the handler will be null. The function code will then fall back to the _`Startup.ServiceProvider`_ dependency injection container and load the _`ListStorageAreasQueryHandler`_ service. This gives us further control of how an application will run both under test and in the cloud.


```c# Function.cs focus=14
public class Function
{
    private static ListStorageAreasQueryHandler _queryHandler;
    private static ILogger<Function> _logger;

    public Function() : this(null, null)
    {
    }

    internal Function(ListStorageAreasQueryHandler handler, ILogger<Function> logger)
    {
        AWSSDKHandler.RegisterXRayForAllServices();
        
        _queryHandler = handler ?? Startup.ServiceProvider.GetRequiredService<ListStorageAreasQueryHandler>();
        _logger = logger ?? Startup.ServiceProvider.GetRequiredService<ILogger<Function>>();
    }

    public async Task<APIGatewayProxyResponse> Handler(APIGatewayProxyRequest apigProxyEvent,
        ILambdaContext context)
    {
        try
        {
            var queryResult = await _queryHandler.Handle(new ListStorageAreasQuery());

            return new APIGatewayProxyResponse
            {
                Body = JsonSerializer.Serialize(queryResult),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
        catch (AmazonS3Exception e)
        {
            context.Logger.LogLine(e.Message);
            context.Logger.LogLine(e.StackTrace);

            return new APIGatewayProxyResponse
            {
                Body = "[]",
                StatusCode = 500,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```

---


## Configuring Dependency Injection

A quick peek at the Startup class. A lazy loaded _`ServiceProvider`_ means that dependency injection will only be setup if required. If we had another Lambda function in our application that didn't require any of these services, this setup would never run.


```c# Function.cs focus=3:17
public static class Startup
{
    private static ServiceProvider? _serviceProvider;

    public static ServiceProvider ServiceProvider
    {
        get
        {
            if (_serviceProvider == null)
            {
                InitializeServiceProvider();
            }

            return _serviceProvider;
        }
        private set => _serviceProvider = value;
    }

    private static void InitializeServiceProvider()
    {
        var services = new ServiceCollection();
        
        var logger = new LoggerConfiguration()
            .WriteTo.Console(new RenderedCompactJsonFormatter())
            .CreateLogger();
        
        services.AddLogging();

        services.AddSingleton<IAmazonS3>(new AmazonS3Client());
        services.AddSingleton<IStorageService, S3StorageService>();
        services.AddSingleton<ListStorageAreasQueryHandler>();

        _serviceProvider = services.BuildServiceProvider();
    }
}
```

---

</CH.Scrollycoding>

A final note, you'll need to add the below code to an _`AssemblyInfo.cs`_ file in your Lambda function project. This allows a unit test project to access and use the internal constructor.

```c# AssemblyInfo.cs
using System.Runtime.CompilerServices;

[assembly:InternalsVisibleTo("ServerlessTestSamples.UnitTest")]
```

## Further Reading

- The AWS Samples GitHub organisation contains a [serverless-test-samples repository](https://github.com/aws-samples/serverless-test-samples)