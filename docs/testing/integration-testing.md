---
sidebar_position: 5
title: Integration Test In The Cloud
---

Our unit tests have all passed (Yay!) and we are confident our business logic works. Now it's time to test if the integrations themselves work, in this case an interaction with the S3 API's.

Run these tests against **deployed cloud resources**. A first pass against an emulated version of S3 might be useful, but running against S3 itself as early as possible will catch issues quickly.

<CH.Scrollycoding>

## Integration Test Setup

[xUnit](https://xunit.net/) is used for all testing. xUnit supports the idea of [test fixtures](https://xunit.net/docs/shared-context). Fixtures provide a shared context between different test runs. More on this configuration in a second, but for now just know that the inheritance from _`IClassFixture<Setup>`_ allows us to perform setup and teardown logic.

```c# IntegrationTest.cs focus=1
public class IntegrationTest : IClassFixture<Setup>
{
    private HttpClient _httpClient;

    public IntegrationTest()
    {
        _httpClient = new HttpClient();
        _httpClient.DefaultRequestHeaders.Add("INTEGRATION_TEST", "true");
    }

    [Fact]
    public async Task ListStorageAreas_ShouldReturnSuccess()
    {
        var result = await _httpClient.GetAsync($"{Setup.ApiUrl}storage");

        result.StatusCode.Should().Be(HttpStatusCode.OK);

        var responseBody = await result.Content.ReadAsStringAsync();

        var storageAreasResult = JsonSerializer.Deserialize<ListStorageAreasResult>(responseBody);

        storageAreasResult.Should().NotBeNull();
        storageAreasResult.StorageAreas.Should().NotBeNull();
        storageAreasResult.IsSuccess.Should().BeTrue();
    }
}

```
---

## Make HTTP Calls

Remember, we are testing integrations. That includes the integration with S3 but also with API Gateway. Running these tests against the actual deployed API gives the truest possible integration test. If our users are going to interact with this API *(even via a frontend)* then that should form the basis of our test. 

Note the _`INTEGRATION_TEST`_ request header being passed. This gives us the possibility to change our function execution if it is an integration test or not.

```c# IntegrationTest.cs focus=5:9
public class IntegrationTest : IClassFixture<Setup>
{
    private HttpClient _httpClient;

    public IntegrationTest()
    {
        _httpClient = new HttpClient();
        _httpClient.DefaultRequestHeaders.Add("INTEGRATION_TEST", "true");
    }

    [Fact]
    public async Task ListStorageAreas_ShouldReturnSuccess()
    {
        var result = await _httpClient.GetAsync($"{Setup.ApiUrl}storage");

        result.StatusCode.Should().Be(HttpStatusCode.OK);

        var responseBody = await result.Content.ReadAsStringAsync();

        var storageAreasResult = JsonSerializer.Deserialize<ListStorageAreasResult>(responseBody);

        storageAreasResult.Should().NotBeNull();
        storageAreasResult.StorageAreas.Should().NotBeNull();
        storageAreasResult.IsSuccess.Should().BeTrue();
    }
}

```
---

## Assert On What Your Users Want

Fundamentally, we write tests to ensure that the code we deploy into production services the needs of our customers and doesn't break any existing functionality. Working backwards from this statement tells us to assert based on what our users are expecting. Testing that the API call to S3 actually happened may be useful, but our users don't care about the internals if the service itself isn't working.

```c# IntegrationTest.cs focus=18:24
public class IntegrationTest : IClassFixture<Setup>
{
    private HttpClient _httpClient;

    public IntegrationTest()
    {
        _httpClient = new HttpClient();
        _httpClient.DefaultRequestHeaders.Add("INTEGRATION_TEST", "true");
    }

    [Fact]
    public async Task ListStorageAreas_ShouldReturnSuccess()
    {
        var result = await _httpClient.GetAsync($"{Setup.ApiUrl}storage");

        result.StatusCode.Should().Be(HttpStatusCode.OK);

        var responseBody = await result.Content.ReadAsStringAsync();

        var storageAreasResult = JsonSerializer.Deserialize<ListStorageAreasResult>(responseBody);

        storageAreasResult.Should().NotBeNull();
        storageAreasResult.StorageAreas.Should().NotBeNull();
        storageAreasResult.IsSuccess.Should().BeTrue();
    }
}

```
---

## Test Setup

Remember the _`IClassFixture<Setup>`_ interface we saw earlier. Here is an implementation of the _`Setup`_ class. The application is deployed using AWS SAM and CloudFormation. This enables us to run API calls against CloudFormation itself to access the API endpoint for the system under test. This same integration test will run on a developer machine **and** in a CICD environment, it's just a case of setting the environment variable for stack and region name.

```c# IntegrationTest.cs
public class Setup : IDisposable
{
    public static string ApiUrl { get; set; }

    public Setup()
    {
        var stackName = Environment.GetEnvironmentVariable("AWS_SAM_STACK_NAME") ?? "dotnet-intro-test-samples";
        var region = Environment.GetEnvironmentVariable("AWS_SAM_REGION_NAME") ?? "us-east-1";

        if (string.IsNullOrEmpty(stackName))
        {
            throw new Exception("Cannot find env var AWS_SAM_STACK_NAME. Please setup this environment variable with the stack name where we are running integration tests.");
        }

        var cloudFormationClient = new AmazonCloudFormationClient(new AmazonCloudFormationConfig()
        {
            RegionEndpoint = RegionEndpoint.GetBySystemName(region)
        });

        var response = cloudFormationClient.DescribeStacksAsync(new DescribeStacksRequest()
        {
            StackName = stackName
        }).Result;

        var output = response.Stacks[0].Outputs.FirstOrDefault(p => p.OutputKey == "ApiEndpoint");

        ApiUrl = output.OutputValue;
    }

    public void Dispose()
    {
        // Do "global" teardown here; Only called once.
    }
}

```

</CH.Scrollycoding>

## Further Reading

- The AWS Samples GitHub organisation contains a [serverless-test-samples repository](https://github.com/aws-samples/serverless-test-samples)