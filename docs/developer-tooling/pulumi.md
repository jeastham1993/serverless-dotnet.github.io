---
sidebar_position: 4
title: Pulumi
---

[Pulumi](https://www.pulumi.com/) is an open-source infrastructure as code SDK that allows you to define and deploy your cloud resources using a programming language of your choice. It's possible to define any kind of cloud resource, but given this is serverlessdotnet.dev let's build an AWS Lambda function.

The first thing you'll need to do is [install Pulumi.](https://www.pulumi.com/docs/get-started/aws/begin/).

Once installed, create a new directory and initialize a project:

```shellscript start-pulumi
mkdir PulumiServerless
cd PulumiServerless
mkdir PulumiDotnet.Serverless
cd PulumiDotnet.Serverless
pulumi new aws-csharp
```

Take note of the stack name used in the _`pulumi new`_ wizard, you'll need this for deployment.

Let's also create a new Lambda function to use as an example.

```shellscript
cd ..
mkdir Functions
cd Functions
dotnet new lambda.EmptyFunction -n Dotnet.CDK.Lambda
```

Open up the generated project file called _`PulumiDotnet.Serverless.csproj`_.

Replace the _`Program.cs`_ file with the below code:

```c# Program.cs
using Pulumi;

using PulumiDotnet.Serverless;

return await Deployment.RunAsync<LambdaFunctionStack>();
```

Then add an additional class to the project named _`LambdaFunctionStack`_.

Update the contents with the below code:

<details>
<summary>Click Here for Code Sample</summary>

```c# LambdaFunctionStack.cs
using Pulumi;
using Pulumi.Aws.Iam;
using Pulumi.Aws.Lambda;

namespace PulumiDotnet.Serverless;

public class LambdaFunctionStack : Stack
{
    [Output] public Output<string> LambdaFunctionName { get; set; }
    
    public LambdaFunctionStack()
    {
        var lambdaRole = new Role("lambdaRole", new RoleArgs
        {
            AssumeRolePolicy =
                @"{
                ""Version"": ""2012-10-17"",
                ""Statement"": [
                    {
                        ""Action"": ""sts:AssumeRole"",
                        ""Principal"": {
                            ""Service"": ""lambda.amazonaws.com""
                        },
                        ""Effect"": ""Allow"",
                        ""Sid"": """"
                    }
                ]
            }"
        });

        var logPolicy = new RolePolicy("lambdaLogPolicy", new RolePolicyArgs
        {
            Role = lambdaRole.Id,
            Policy =
                @"{
                ""Version"": ""2012-10-17"",
                ""Statement"": [{
                    ""Effect"": ""Allow"",
                    ""Action"": [
                        ""logs:CreateLogGroup"",
                        ""logs:CreateLogStream"",
                        ""logs:PutLogEvents""
                    ],
                    ""Resource"": ""arn:aws:logs:*:*:*""
                }]
            }"
        });
        
        // Create an AWS resource (S3 Bucket)
        var lambda = new Function(
            "PulumiDotnetLambda",
            new FunctionArgs
            {
                Runtime = "dotnet6",
                Code = new FileArchive("Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/bin/Release/net6.0/linux-x64/publish"),
                Handler = "Dotnet.CDK.Lambda::Dotnet.CDK.Lambda.Function::FunctionHandler",
                Role = lambdaRole.Arn
            });

        LambdaFunctionName = lambda.Name;
    }
}
```

</details>

<CH.Scrollycoding>

## IAM Role

The first thing defined is an IAM role for the Lambda function to use. The role simply allows the Lambda service to assume it.

```c# LambdaFunctionStack.cs
// ...
var lambdaRole = new Role("lambdaRole", new RoleArgs
{
    AssumeRolePolicy =
        @"{
        ""Version"": ""2012-10-17"",
        ""Statement"": [
            {
                ""Action"": ""sts:AssumeRole"",
                ""Principal"": {
                    ""Service"": ""lambda.amazonaws.com""
                },
                ""Effect"": ""Allow"",
                ""Sid"": """"
            }
        ]
    }"
});
// ...
```
---

## IAM Policy

Next, an IAM policy is defined and attached to the role using the _`RolePolicy`_ class.

```c# LambdaFunctionStack.cs
// ...

var logPolicy = new RolePolicy(
    "lambdaLogPolicy",
    new RolePolicyArgs
    {
        Role = lambdaRole.Id,
        Policy =
            @"{
        ""Version"": ""2012-10-17"",
        ""Statement"": [{
            ""Effect"": ""Allow"",
            ""Action"": [
                ""logs:CreateLogGroup"",
                ""logs:CreateLogStream"",
                ""logs:PutLogEvents""
            ],
            ""Resource"": ""arn:aws:logs:*:*:*""
        }]
    }"
    });

// ...
```
---

## Lambda Function

Finally, the Lambda function is defined and the name of it set to the Output name. Pulumi provides the _`FileArchive`_ class to specify the files to be included in the ZIP and upload it to Amazon S3.

```c# LambdaFunctionStack.cs focus=5:15

// ...

// Create an AWS resource (S3 Bucket)
var lambda = new Function(
    "PulumiDotnetLambda",
    new FunctionArgs
    {
        Runtime = "dotnet6",
        Code = new FileArchive(
            "Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/bin/Release/net6.0/linux-x64/publish"),
        Handler = "Dotnet.CDK.Lambda::Dotnet.CDK.Lambda.Function::FunctionHandler",
        Role = lambdaRole.Arn
    });

this.LambdaFunctionName = lambda.Name;

// ...
```

</CH.Scrollycoding>

Once defined, run the below commands from a terminal window in the _`PulumiServerless`_ folder.

```shellscript deploy
dotnet publish ./Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/ -c Release -r linux-x64
cd PulumiDotnet.Serverless
pulumi stack select STACK_NAME_SPECIFIED_EARLIER
pulumi up
```

Once deployed, invoke the Lambda function in AWS using the below command replacing the _`OUTPUT_FUNCTION_NAME`_ with the value output to your terminal:

```shellscript
dotnet lambda invoke-function -n OUTPUT_FUNCTION_NAME -p hello
```


