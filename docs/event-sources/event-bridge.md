---
sidebar_position: 2
title: Amazon EventBridge
---

## Trigger AWS Lambda from SQS

![AmazonEventBridge to AWS Lambda diagram](/img/event-sources/event-bridge-lambda.png)

In this pattern, we will walk through how to trigger AWS Lambda from an event in Amazon EventBridge.

## Packages Required

*If you're confused about why the package is called CloudWatchEvents, EventBridge used to be part of CloudWatch and was broken out into it's own service.*

```shellscript install
dotnet add package Amazon.Lambda.CloudWatchEvents
```

## Function Code

The Amazon EventBridge to Lambda integration is what is known as a asynchronous invoke. As well as the Lambda runtime deserializing the event payload, for EventBridge events it will also deserialize the event detail as well. Specifying a type in the CloudWatchEvent object enables this automatic deserialization.


```c# Function.cs
public async Task FunctionHandler(CloudWatchEvent<MessageObject> eventBridgeEvent, ILambdaContext context)
{
    try
    {
        var eventType = eventBridgeEvent.DetailType;
        var messageSource = eventBridgeEvent.Source;
        var myMessageObject = eventBridgeEvent.Detail;

        await ProcessMessage(myMessageObject);
    }
    catch (Exception e)
    {
        this._logger.LogError(e, "Failure processing event");
        throw;
    }
}
```

## Best Practices

- Configure **DeadLetterQueues** both on the EventBridge integration and Lambda itself. Without DLQ's, failed messages will be lost.
