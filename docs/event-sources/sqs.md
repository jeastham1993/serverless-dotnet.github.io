---
sidebar_position: 5
title: Amazon SQS
---

## Trigger AWS Lambda from SQS

![SQS to AWS Lambda diagram](/img/event-sources/sqs-lambda.png)

In this pattern, we will walk through how to trigger AWS Lambda from messages placed in an SQS queue.

## Packages Required

```bash install
dotnet add package Amazon.Lambda.SQSEvents
```

## Function Code

The SQS to Lambda integration is what is known as a poll based invoke. The Lambda service will poll SQS on your behalf and pass batches of messages into Lambda.

The _`SQSEvent`_ object contains an array of _`SQSMessage`_ objects. The number of messages per batch is configured at the event source.

You can also return an _`SQSBatchResponse`_ object to handle a situation in which some of the messages in a batch fail. An SQS Handler will also work with a _`void`_ or `_Task`_ return type.

```c# Function.cs
public SQSBatchResponse FunctionHandler(SQSEvent sqsEvent, ILambdaContext context)
{
    var batchResponse = new SQSBatchResponse();
    
    foreach (var record in sqsEvent.Records)
    {
        try
        {
            var payload = JsonSerializer.Deserialize<MessageObject>(record.Body);
        }
        catch (Exception)
        {
            batchResponse.BatchItemFailures.Add(new SQSBatchResponse.BatchItemFailure()
            {
                ItemIdentifier = record.MessageId
            });
        }
    }

    return batchResponse;
}
```

## Best Practices

- Ensure errors are handled. If the Lambda function fails **all** messages will go back on to the SQS queue. Try/catch and return an _`SQSBatchResponse`_ object to handle any partial batch failures.
