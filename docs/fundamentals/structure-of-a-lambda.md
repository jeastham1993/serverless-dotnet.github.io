---
sidebar_position: 2
title: Structure of a Lambda function
description: A walkthrough of the key components of a Lambda function.
keywords: [fundamentals]
---

# Lambda Function Structure

Let's discuss the structure of a Lambda function built using .NET. We will dive deeper into these topics in later sections.


<CH.Scrollycoding>


## (De)serialization

Event payloads are passed to Lambda functions as JSON strings, but we are working in an object orientated language! The Lambda runtime for .NET supports the ability to automatically (de)serialize the Lambda payloads and responses on your behalf.

To configure, you need to tell the Lambda runtime what serializer to use. There are default implementations for _`System.Text.Json`_ and `_Newtonsoft.Json`_ as well as the capability to write your own if required.

```c# Function.cs focus=12
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Shared.DataAccess;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace GetProducts
{
    public class Function
    {
        private readonly ProductsDAO dataAccess;

        public Function()
        {
            this.dataAccess = new DynamoDbProducts();
        }

        public async Task<APIGatewayHttpApiV2ProxyResponse> FunctionHandler(APIGatewayHttpApiV2ProxyRequest apigProxyEvent,
            ILambdaContext context)
        {
            if (!apigProxyEvent.RequestContext.Http.Method.Equals(HttpMethod.Get.Method))
            {
                return new APIGatewayHttpApiV2ProxyResponse
                {
                    Body = "Only GET allowed",
                    StatusCode = (int)HttpStatusCode.MethodNotAllowed,
                };
            }
    
            context.Logger.LogLine($"Received {apigProxyEvent}");

            var products = await dataAccess.GetAllProducts();
    
            context.Logger.LogLine($"Found {products.Products.Count} product(s)");
    
            return new APIGatewayHttpApiV2ProxyResponse
            {
                Body = JsonSerializer.Serialize(products),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```
---

## Initialization

The _`FunctionHandler`_ method will be called on every invoke. It's a good practice in Lambda to initialize any objects that can be re-used outside of the handler. Things like database connections, configuration or loading of any static data.

Performing this once outside of the handler ensures this code will only execute once per execution environment. In this example, we are initializing the _`ProductsDAO`_ object in our Function constructor.

The Lambda service **requires** a public, parameterless constructor to initialize the Function class.

```c# Function.cs focus=18:23
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Shared.DataAccess;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace GetProducts
{
    public class Function
    {
        private readonly ProductsDAO dataAccess;

        public Function()
        {
            this.dataAccess = new DynamoDbProducts();
        }

        public async Task<APIGatewayHttpApiV2ProxyResponse> FunctionHandler(APIGatewayHttpApiV2ProxyRequest apigProxyEvent,
            ILambdaContext context)
        {
            if (!apigProxyEvent.RequestContext.Http.Method.Equals(HttpMethod.Get.Method))
            {
                return new APIGatewayHttpApiV2ProxyResponse
                {
                    Body = "Only GET allowed",
                    StatusCode = (int)HttpStatusCode.MethodNotAllowed,
                };
            }
    
            context.Logger.LogLine($"Received {apigProxyEvent}");

            var products = await dataAccess.GetAllProducts();
    
            context.Logger.LogLine($"Found {products.Products.Count} product(s)");
    
            return new APIGatewayHttpApiV2ProxyResponse
            {
                Body = JsonSerializer.Serialize(products),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```
---
## The Function Handler

The Function Handler is the key component of a Lambda function. This is what the Lambda service will invoke when
a new event is received. The name of the method is unimportant (*standardize on something memorable though*), what is important are the two method parameters.

```c# Function.cs focus=25:49
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Shared.DataAccess;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace GetProducts
{
    public class Function
    {
        private readonly ProductsDAO dataAccess;

        public Function()
        {
            this.dataAccess = new DynamoDbProducts();
        }

        public async Task<APIGatewayHttpApiV2ProxyResponse> FunctionHandler(APIGatewayHttpApiV2ProxyRequest apigProxyEvent,
            ILambdaContext context)
        {
            if (!apigProxyEvent.RequestContext.Http.Method.Equals(HttpMethod.Get.Method))
            {
                return new APIGatewayHttpApiV2ProxyResponse
                {
                    Body = "Only GET allowed",
                    StatusCode = (int)HttpStatusCode.MethodNotAllowed,
                };
            }
    
            context.Logger.LogLine($"Received {apigProxyEvent}");

            var products = await dataAccess.GetAllProducts();
    
            context.Logger.LogLine($"Found {products.Products.Count} product(s)");
    
            return new APIGatewayHttpApiV2ProxyResponse
            {
                Body = JsonSerializer.Serialize(products),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```
---

## The Event Payload

The first method parameter is the event payload. This is what the Lambda service passes to your function. The structure of this payload will change depending if you are using API Gateway, SQS or any of the other Lambda event sources.

[In this example](focus://16[34:46]), the Lambda function is being triggered by API Gateway. We will come on to the different event sources in a later section, for now just know what the first parameter is the event payload.

```c# Function.cs focus=25
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Shared.DataAccess;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace GetProducts
{
    public class Function
    {
        private readonly ProductsDAO dataAccess;

        public Function()
        {
            this.dataAccess = new DynamoDbProducts();
        }

        public async Task<APIGatewayHttpApiV2ProxyResponse> FunctionHandler(APIGatewayHttpApiV2ProxyRequest apigProxyEvent,
            ILambdaContext context)
        {
            if (!apigProxyEvent.RequestContext.Http.Method.Equals(HttpMethod.Get.Method))
            {
                return new APIGatewayHttpApiV2ProxyResponse
                {
                    Body = "Only GET allowed",
                    StatusCode = (int)HttpStatusCode.MethodNotAllowed,
                };
            }
    
            context.Logger.LogLine($"Received {apigProxyEvent}");

            var products = await dataAccess.GetAllProducts();
    
            context.Logger.LogLine($"Found {products.Products.Count} product(s)");
    
            return new APIGatewayHttpApiV2ProxyResponse
            {
                Body = JsonSerializer.Serialize(products),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```
---

## The Lambda Context

The second method paramter is the _`ILambdaContext`_ interface. This object contains **contextual** information about this specific invoke of the function. It also includes a handy little logging context for quick and easy logging.

```c# Function.cs focus=25
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Shared.DataAccess;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace GetProducts
{
    public class Function
    {
        private readonly ProductsDAO dataAccess;

        public Function()
        {
            this.dataAccess = new DynamoDbProducts();
        }

        public async Task<APIGatewayHttpApiV2ProxyResponse> FunctionHandler(APIGatewayHttpApiV2ProxyRequest apigProxyEvent,
            ILambdaContext context)
        {
            if (!apigProxyEvent.RequestContext.Http.Method.Equals(HttpMethod.Get.Method))
            {
                return new APIGatewayHttpApiV2ProxyResponse
                {
                    Body = "Only GET allowed",
                    StatusCode = (int)HttpStatusCode.MethodNotAllowed,
                };
            }
    
            context.Logger.LogLine($"Received {apigProxyEvent}");

            var products = await dataAccess.GetAllProducts();
    
            context.Logger.LogLine($"Found {products.Products.Count} product(s)");
    
            return new APIGatewayHttpApiV2ProxyResponse
            {
                Body = JsonSerializer.Serialize(products),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
    }
}
```

</CH.Scrollycoding>