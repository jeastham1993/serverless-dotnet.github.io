---
sidebar_position: 2
title: AWS CDK
---

The [AWS Cloud Development Kit (CDK)](https://aws.amazon.com/cdk/) is an open-source framework that enables you to define your AWS cloud resources using a familiar programming language. Yep, that's right! You can define AWS resources using .NET.

Deploying .NET Lambda functions using the AWS CDK adds an additional complexity. Your .NET function code needs to be compiled before CDK can deploy your resources. This is less of an issue for interpreted languages like NodeJS and Python as no compilation is required.

There are a number of methods for acheiving this. The CDK does contain some useful built in constructs for performing this pre-compilation.

However, the first thing you'll need to do is [install the AWS CDK.](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_install)

Once installed, create a new CDK project using the below command:

```shellscript
mkdir DotnetCdkWithLambda
cdk init app -l csharp
cd DotnetCdkWithLambda
```

Once generated, open the solution file under src/DotnetCdkWithLambda.sln.

This walkthrough isn't going to deep dive the CDK itself, but focus in purely on deploying Lambda functions. A couple of important points to note:

- CDK generates a CloudFormation template from your .NET code
- On init, you'll get a project with two classes
    - _`Program.cs`_ is the application entry point
    - _`DotnetCdkWithLambdaStack.cs`_ is where you will define the actual resources
- Any classes that inherit from _`Stack`_ will be created as a Stack in CloudFormation

## Generate a Lambda function

Now let's also generate a Lambda function to use as a test. From within the src/ directory run the below CLI commands to create a new _`Functions`_ folder and generate a new AWS Lambda function.

```shellscript
mkdir Functions
cd Functions
dotnet new lambda.EmptyFunction -n Dotnet.CDK.Lambda
```

## CDK with Docker

To deploy a Lambda function using code packaged as a ZIP file, the application bundle must be uploaded to S3. Lambda can then reference the file stored in S3. The CDK provides useful constructs to enable this. This method uses a Docker container for compilation, if you don't have Docker scroll down to the bottom part of this document.

Update the code in _`DotnetCdkWithLambdaStack.cs`_ with the below:

```c#
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using AssetOptions = Amazon.CDK.AWS.S3.Assets.AssetOptions;
using Constructs;

namespace DotnetCdkWithLambda
{
    public class DotnetCdkWithLambdaStack : Stack
    {
        internal DotnetCdkWithLambdaStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            var function = new Function(this,
                "dotnet-cdk-lambda-function",
                new FunctionProps
                {
                    Runtime = Runtime.DOTNET_6,
                    Handler = "Dotnet.CDK.Lambda::Dotnet.CDK.Lambda.Function::FunctionHandler",
                    Code = Code.FromAsset("./src/Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda", new AssetOptions()
                    {
                        Bundling = new BundlingOptions
                        {
                            Image = Runtime.DOTNET_6.BundlingImage,
                            Command = new []
                            {
                                "bash", "-c", string.Join(" && ", new[]
                                {
                                    "cd /asset-input",
                                    "export DOTNET_CLI_HOME=\"/tmp/DOTNET_CLI_HOME\"",
                                    "export PATH=\"$PATH:/tmp/DOTNET_CLI_HOME/.dotnet/tools\"",
                                    "dotnet tool install -g Amazon.Lambda.Tools",
                                    "dotnet lambda package -o output.zip",
                                    "unzip -o -d /asset-output output.zip"
                                };)
                            }
                        }
                    })
                });

            var functionNameOutput = new CfnOutput(
                this,
                "FunctionNameOutput",
                new CfnOutputProps()
                {
                    ExportName = "FunctionName",
                    Value = function.FunctionName
                });
        }
    }
}
```

<CH.Scrollycoding>

### Bundling Commands

CDK provides the ability to bundle your application code as part of the synth process. A set of commands can be specified to run within a Docker container to generate the compiled application code. These commands install the _`Amazon.Lambda.Tools`_ and uses the _`dotnet lambda package`_ command to generate the Lambda output.

```c# cdk-code focus=13:24
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using AssetOptions = Amazon.CDK.AWS.S3.Assets.AssetOptions;
using Constructs;

namespace DotnetCdkWithLambda
{
    public class DotnetCdkWithLambdaStack : Stack
    {
        internal DotnetCdkWithLambdaStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // ...
                            Command = new []
                            {
                                "bash", "-c", string.Join(" && ", new[]
                                {
                                    "cd /asset-input",
                                    "export DOTNET_CLI_HOME=\"/tmp/DOTNET_CLI_HOME\"",
                                    "export PATH=\"$PATH:/tmp/DOTNET_CLI_HOME/.dotnet/tools\"",
                                    "dotnet tool install -g Amazon.Lambda.Tools",
                                    "dotnet lambda package -o output.zip",
                                    "unzip -o -d /asset-output output.zip"
                                };)
                            }
                        }
             // ...
        }
    }
}

```

---

### Function Definition

A Lambda function is defined using the _`Function`_ class, taken from the _`Amazon.CDK.AWS.Lambda`_ namespace. A set of _`FunctionProps`_ are passed into the constructor, defining exactly how the Lambda function will run. In here, you can define timeouts, memory allocation and event event sources.

```c# cdk-code focus=13:19
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using AssetOptions = Amazon.CDK.AWS.S3.Assets.AssetOptions;
using Constructs;

namespace DotnetCdkWithLambda
{
    public class DotnetCdkWithLambdaStack : Stack
    {
        internal DotnetCdkWithLambdaStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // ...
            var function = new Function(this,
                "dotnet-cdk-lambda-function",
                new FunctionProps
                {
                    Runtime = Runtime.DOTNET_6,
                    Handler = "Dotnet.CDK.Lambda::Dotnet.CDK.Lambda.Function::FunctionHandler"
                    Code = Code.FromAsset("./src/Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda", new AssetOptions()
            // ...    
        }
    }
}

```

---

### CDK output

It's also possible to define CloudFormation outputs in the CDK. In this case, the Lambda function name is set as an output. When deployed using the CDK, any defined outputs will display in the terminal window.

```c# cdk-code focus=13:20
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using AssetOptions = Amazon.CDK.AWS.S3.Assets.AssetOptions;
using Constructs;

namespace DotnetCdkWithLambda
{
    public class DotnetCdkWithLambdaStack : Stack
    {
        internal DotnetCdkWithLambdaStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // ...
            var functionNameOutput = new CfnOutput(
                this,
                "FunctionNameOutput",
                new CfnOutputProps()
                {
                    ExportName = "FunctionName",
                    Value = function.FunctionName
                });
            // ...
        }
    }
}

```

</CH.Scrollycoding>

Once defined, run the _`cdk deploy`_ command from the same folder that contains the _`cdk.json`_ file. If this is your first time using the AWS CDK, you will need to run a _`cdk bootstrap`_ command before your first deploy.

Once deployed, invoke the Lambda function in AWS using the below command replacing the _`OUTPUT_FUNCTION_NAME`_ with the value output to your terminal:

```shellscript
dotnet lambda invoke-function -n OUTPUT_FUNCTION_NAME -p hello
```

## CDK without Docker

If you don't have Docker installed on your system, it's still possible to use the AWS CDK to deploy Lambda functions. You will just need to pre-compile your Lambda function code before running the _`cdk deploy`_ command.

In your CDK code, update the Lambda function definition to be the below:

```c# cdk.cs focus=6:6
var function = new Function(this,
    "dotnet-cdk-lambda-function",
    new FunctionProps
    {
        Runtime = Runtime.DOTNET_6,
        Code = Code.FromAsset("./src/Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/bin/Release/net6.0/linux-x64/publish"),
        Handler = "Dotnet.CDK.Lambda::Dotnet.CDK.Lambda.Function::FunctionHandler"
    });
```

Instead of specifying bundling options, we pass the CDK a reference to the bin folder containing the published DLL's. To deploy, use the below commands from the folder containing the _`cdk.json`_ file:

```shellscript deploy.sh
dotnet publish ./src/Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/ -c Release -r linux-x64
cdk deploy
```

Normally, I put these commands inside a _`deploy.ps1`_ or _`deploy.sh`_ script that I can run as part of CICD.