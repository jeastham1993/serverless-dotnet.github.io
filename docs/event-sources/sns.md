---
sidebar_position: 4
title: Amazon SNS
---

## Trigger AWS Lambda from SNS

![SNS to AWS Lambda diagram](/img/event-sources/sns-lambda.png)

In this pattern, we will walk through how to trigger AWS Lambda from messages sent to an SNS topic.

## Packages Required

```shellscript install
dotnet add package Amazon.Lambda.SNSEvents
```

## Function Code

The S3 to Lambda integration is what is known as an asynchronous invoke. Events are forwarded to the Lambda service and processed asynchronously, meaning the caller can continue doing other work.

The _`SNSEvent`_ object contains a List of _`SNSRecord`_ objects. The _`SNSRecord`_ objects contains all the message metadata including the Message body itself, the type of message and the ARN of the topic the message came from.

It's important to implement error handling and dead letter queues. If the Lambda functions fails and no dead letter queue is configured, messages will be lost. 

```c# Function.cs
public async Task FunctionHandler(SNSEvent snsEvent, ILambdaContext context)
{
    foreach (var record in snsEvent.Records)
    {
        try
        {
            var eventBody = record.Sns.Message;
            var messageType = record.Sns.Type;
            var topicArn = record.Sns.TopicArn;
        }
        catch (Exception e)
        {
            this._logger.LogError(e, "Failure processing event");
            throw;
        }
    }
}
```

## Best Practices

- Ensure errors are handled. If the Lambda function fails messages will be lost if no dead letter queue is configured
