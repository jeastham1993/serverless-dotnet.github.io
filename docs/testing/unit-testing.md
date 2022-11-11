---
sidebar_position: 2
title: Unit Test Your Business Logic
---

Let's kick this section off with a quote from Martin Fowler around unit testing:

> Even a classic tester like myself uses test doubles when there's an awkward collaboration. They are invaluable to **remove non-determinism when talking to remote services**. Indeed some classicist xunit testers also argue that any collaboration with external resources, such as a database or filesystem, should use doubles. Partly this is due to non-determinism risk, partly due to speed. While I think this is a useful guideline, I don't treat using doubles for external resources as an absolute rule. **If talking to the resource is stable and fast enough for you then there's no reason not to do it in your unit tests**.

    *- Martin Fowler*

There are two opposing views here, both of which are valid in the right scenario. The first, using mocks to remove non-determnism when talking to remove services, is one of the most important when writing unit tests against serverless applications. Unit tests could quite easily interact with the cloud services themselves, but there are a number of potential blockers here. 

- Does your company allow each developer to have their own resources/AWS account?
- Do you have permission to connect directly from your developer machine to AWS?
- What if another engineer changes the resources you've perfectly aligned for your test run?

There are a number of factors that can make your unit tests non-detminisitic, making them brittle.

Intead, **focus on your business logic** when developing locally. Let's look at an example:

<CH.Scrollycoding>

## Mock Integrations

This example uses the [Moq](https://github.com/moq/moq4) library to create mock implementations of our SDK calls. This Lambda function is setup to list all buckets in S3, yeah it's trivial I know. What it does show us is how easily we can test how our Lambda function deals with different responses from S3.

```c# UnitTest.cs focus=5:17

[Fact]
public async Task TestLambdaHandlerWithValidS3Response_ShouldReturnSuccess()
{
    var mockedS3Client = new Mock<IAmazonS3>();
    var mockHttpClient = new Mock<HttpClient>();
    
    mockedS3Client.Setup(p => p.ListBucketsAsync(It.IsAny<CancellationToken>())).ReturnsAsync(new ListBucketsResponse()
    {
        Buckets = new List<S3Bucket>()
        {
            new S3Bucket(){BucketName = "bucket1"},
            new S3Bucket(){BucketName = "bucket2"},
            new S3Bucket(){BucketName = "bucket3"},
        },
        HttpStatusCode = HttpStatusCode.OK
    });
    
    var storageService = new S3StorageService(mockedS3Client.Object);
    var handler = new ListStorageAreasQueryHandler(storageService, _mockHandlerLogger.Object);

    var function = new Function(handler, _mockLogger.Object);

    var result = await function.Handler(new APIGatewayProxyRequest(), new TestLambdaContext());

    result.StatusCode.Should().Be(200);

    var responseBody = JsonSerializer.Deserialize<ListStorageAreaResponseBody>(result.Body);

    responseBody.Should().NotBeNull();
    responseBody?.StorageAreas.Count().Should().Be(3);
    responseBody?.StorageAreas.FirstOrDefault().Should().Be("bucket1");
}

```
---

## Use The Internal Constructor

When initializing our _`Function`_ under test the internal constructor is used. This allows the mocks that have just been created to be passed into the Function. Our function code is none the wiser, it runs it's logic as normal.

```c# UnitTest.cs focus=21:21
[Fact]
public async Task TestLambdaHandlerWithValidS3Response_ShouldReturnSuccess()
{
    var mockedS3Client = new Mock<IAmazonS3>();
    var mockHttpClient = new Mock<HttpClient>();
    
    mockedS3Client.Setup(p => p.ListBucketsAsync(It.IsAny<CancellationToken>())).ReturnsAsync(new ListBucketsResponse()
    {
        Buckets = new List<S3Bucket>()
        {
            new S3Bucket(){BucketName = "bucket1"},
            new S3Bucket(){BucketName = "bucket2"},
            new S3Bucket(){BucketName = "bucket3"},
        },
        HttpStatusCode = HttpStatusCode.OK
    });
    
    var storageService = new S3StorageService(mockedS3Client.Object);
    var handler = new ListStorageAreasQueryHandler(storageService, _mockHandlerLogger.Object);

    var function = new Function(handler, _mockLogger.Object);

    var result = await function.Handler(new APIGatewayProxyRequest(), new TestLambdaContext());

    result.StatusCode.Should().Be(200);

    var responseBody = JsonSerializer.Deserialize<ListStorageAreaResponseBody>(result.Body);

    responseBody.Should().NotBeNull();
    responseBody?.StorageAreas.Count().Should().Be(3);
    responseBody?.StorageAreas.FirstOrDefault().Should().Be("bucket1");
}

```
---

## Assert

Now that we know what the response from S3 is going to be, we can assert on what the expected response is from our Lambda function. This Lambda function responds to API calls so our assertions are based around the API response that Lambda returns. 

At this point we could test status code, response contents and even the schema of the response.

```c# UnitTest.cs focus=22:31
[Fact]
public async Task TestLambdaHandlerWithValidS3Response_ShouldReturnSuccess()
{
    var mockedS3Client = new Mock<IAmazonS3>();
    var mockHttpClient = new Mock<HttpClient>();
    
    mockedS3Client.Setup(p => p.ListBucketsAsync(It.IsAny<CancellationToken>())).ReturnsAsync(new ListBucketsResponse()
    {
        Buckets = new List<S3Bucket>()
        {
            new S3Bucket(){BucketName = "bucket1"},
            new S3Bucket(){BucketName = "bucket2"},
            new S3Bucket(){BucketName = "bucket3"},
        },
        HttpStatusCode = HttpStatusCode.OK
    });
    
    var storageService = new S3StorageService(mockedS3Client.Object);
    var handler = new ListStorageAreasQueryHandler(storageService, _mockHandlerLogger.Object);

    var function = new Function(handler, _mockLogger.Object);

    var result = await function.Handler(new APIGatewayProxyRequest(), new TestLambdaContext());

    result.StatusCode.Should().Be(200);

    var responseBody = JsonSerializer.Deserialize<ListStorageAreaResponseBody>(result.Body);

    responseBody.Should().NotBeNull();
    responseBody?.StorageAreas.Count().Should().Be(3);
    responseBody?.StorageAreas.FirstOrDefault().Should().Be("bucket1");
}

```

---

## Testing Problems

We can use this same framework to test for errors. In this example our S3 client is mocked to return _`null`_ buckets and a _`BadRequest`_ status code. The business case defines that the API call itself always returns a 200 status code. If there are errors in S3 then the API returns an empty array for the bucket list.

```c# UnitTest.cs focus=7:11
[Fact]
public async Task TestLambdaHandlerWithS3NullResponse_ShouldReturnEmpty()
{
    var mockedS3Client = new Mock<IAmazonS3>();
    var mockHttpClient = new Mock<HttpClient>();
    
    mockedS3Client.Setup(p => p.ListBucketsAsync(It.IsAny<CancellationToken>())).ReturnsAsync(new ListBucketsResponse()
    {
        Buckets = null,
        HttpStatusCode = HttpStatusCode.BadRequest
    });
    
    var storageService = new S3StorageService(mockedS3Client.Object);
    var handler = new ListStorageAreasQueryHandler(storageService, _mockHandlerLogger.Object);

    var function = new Function(handler, _mockLogger.Object);

    var result = await function.Handler(new APIGatewayProxyRequest(), new TestLambdaContext());

    result.StatusCode.Should().Be(200);

    var responseBody = JsonSerializer.Deserialize<ListStorageAreaResponseBody>(result.Body);

    responseBody.Should().NotBeNull();
    responseBody?.StorageAreas.Count().Should().Be(0);
}
```

---

## Asserting On Problems

That can be seen in the assertions. Mock implementations are passed into the Function in the same way and we are testing that a 200 status and empty array is received.

```c# UnitTest.cs focus=18:25
[Fact]
public async Task TestLambdaHandlerWithS3NullResponse_ShouldReturnEmpty()
{
    var mockedS3Client = new Mock<IAmazonS3>();
    var mockHttpClient = new Mock<HttpClient>();
    
    mockedS3Client.Setup(p => p.ListBucketsAsync(It.IsAny<CancellationToken>())).ReturnsAsync(new ListBucketsResponse()
    {
        Buckets = null,
        HttpStatusCode = HttpStatusCode.BadRequest
    });
    
    var storageService = new S3StorageService(mockedS3Client.Object);
    var handler = new ListStorageAreasQueryHandler(storageService, _mockHandlerLogger.Object);

    var function = new Function(handler, _mockLogger.Object);

    var result = await function.Handler(new APIGatewayProxyRequest(), new TestLambdaContext());

    result.StatusCode.Should().Be(200);

    var responseBody = JsonSerializer.Deserialize<ListStorageAreaResponseBody>(result.Body);

    responseBody.Should().NotBeNull();
    responseBody?.StorageAreas.Count().Should().Be(0);
}
```

</CH.Scrollycoding>

## Debugging

Remember, when running unit tests in either Visual Studio, VS Code or JetBrains Rider you can attach a debugger to your test run. As .NET developers expecting that 'localhost' development experience this is a powerful alternative. Use the test framework as the harness for our debugger.

The second benefit of this, we now have a full suite of unit tests that can be re-used by other developers and CICD pipelines. It removes 'intuition based' debugging on local host and makes it something repeatable.

## Further Reading

- The AWS Samples GitHub organisation contains a [serverless-test-samples repository](https://github.com/aws-samples/serverless-test-samples)