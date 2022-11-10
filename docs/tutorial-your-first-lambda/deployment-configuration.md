---
sidebar_position: 3
title: Deployment Configuration
description: A guide to configuring a Lambda function once deployed into the cloud
keywords: [dotnet cli]
---

# Deployment Configuration

Now it's time to deploy your first Lambda function into AWS. The observant amongst you will have noticed within the function code directory there is a file called _`aws-lambda-tools-defaults.json`_. Open that up and let's have a look inside and walk through each property step by step:

<CH.Scrollycoding>


### Function Handler
**By far the most important of the settings in this file**. The function handler. This setting is how the Lambda service knows what to invoke. This setting is split into 3 distinct parts:

1. MyFirstLambda - The name of your assembly
2. MyFirstLambda.Function - The class name, including the entire namespace
3. FunctionHandler - The name of the method call. You can see now why banana would have been confusing.

```json aws-lambda-tools-defaults.json focus=15
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```

---

### Profile
Specify the profile to use from your AWS credentials file.

```json aws-lambda-tools-defaults.json focus=8
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```

---

### Region
Specify the AWS region to deploy into.

```json aws-lambda-tools-defaults.json focus=9
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```
---

### Configuration
The configuration to use on _`dotnet publish`_.

```json aws-lambda-tools-defaults.json focus=10
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```

---

### Function Architecture
The processor architecture to use on AWS. Choose betweenm `x86_64` and `arm64`.

```json aws-lambda-tools-defaults.json focus=11
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```

---

### Function Runtime
The runtime to use. Given .NET Core 3.1 is on it's way out of support this will likely be `dotnet6` for any new development.

```json aws-lambda-tools-defaults.json focus=12
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```

---

### Function Memory Size
The amount of memory to allocate to your Lambda function. As we discovered in the introduction, Lambda is charged **per ms** of execution and that per ms charge changes based on the memory allocated. Choose a value from 128mb to 10gb.

```json aws-lambda-tools-defaults.json focus=13
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```

---

### Function Timeout
You can control the timeout of your Lambda function. Choose anything from 1s through to 15 minutes. The choice is yours. Always specify the value in seconds though. Hint, 15 minutes is 900 seconds.

```json aws-lambda-tools-defaults.json focus=14
{
  "Information": [
    "This file provides default values for the deployment wizard inside Visual Studio and the AWS Lambda commands added to the .NET Core CLI.",
    "To learn more about the Lambda commands with the .NET Core CLI execute the following command at the command line in the project root directory.",
    "dotnet lambda help",
    "All the command line options for the Lambda command can be specified in this file."
  ],
  "profile": "",
  "region": "",
  "configuration": "Release",
  "function-architecture": "x86_64",
  "function-runtime": "dotnet6",
  "function-memory-size": 256,
  "function-timeout": 30,
  "function-handler": "MyFirstLambda::MyFirstLambda.Function::FunctionHandler"
}
```
</CH.Scrollycoding>