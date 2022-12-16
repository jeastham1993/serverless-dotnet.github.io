---
sidebar_position: 1
title: API Gateway HTTP API
---

## Trigger AWS Lambda with API Gateway HTTP API

![API Gateway to AWS Lambda diagram](/img/event-sources/api-gw-http-lambda.png)

In this pattern, we will walk through how to trigger AWS Lambda from an API call to an API Gateway HTTP API endpoint.

## Packages Required

```shellscript install
dotnet add package Amazon.Lambda.APIGatewayEvents
```

## Function Code

The API Gateway to Lambda integration is what is known as a synchronous invoke. API Gateway will wait for Lambda to return a response that is then passed back to the caller. For that reason, we have a _`APIGatewayHttpApiV2ProxyRequest`_ as our event payload and _`APIGatewayHttpApiV2ProxyResponse`_ as our method response.

```c# Function.cs
public class Function
{
    public async Task<APIGatewayHttpApiV2ProxyResponse> FunctionHandler(APIGatewayHttpApiV2ProxyRequest apigProxyEvent, ILambdaContext context)
    {
        try
        {
            if (!apigProxyEvent.RequestContext.Http.Method.Equals(HttpMethod.Get.Method))
            {
                return new APIGatewayHttpApiV2ProxyResponse
                {
                    Body = "Only GET allowed",
                    StatusCode = (int)HttpStatusCode.MethodNotAllowed,
                };
            }
    
            // Perform business logic
            // apigProxyEvent.Body to access the body of the request sent to API Gateway.
    
            return new APIGatewayHttpApiV2ProxyResponse
            {
                Body = JsonSerializer.Serialize(new {
                    "hello": "world"
                }),
                StatusCode = 200,
                Headers = new Dictionary<string, string> {{"Content-Type", "application/json"}}
            };
        }
        catch (Exception)
        {
            // Log exception, incremement failure metrics etc.
            
            return new APIGatewayHttpApiV2ProxyResponse
            {
                StatusCode = (int)HttpStatusCode.InternalServerError,
            };
        }
    }
}
```

## Best Practices

- Catch exceptions and return a useful response to API Gateway
