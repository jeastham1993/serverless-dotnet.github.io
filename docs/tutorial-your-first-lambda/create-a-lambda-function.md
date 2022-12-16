---
sidebar_position: 2
title: Create A Lambda Function
description: A guide to creating a Lambda function from the .NET Global CLI
keywords: [dotnet cli]
---

# Create a Lambda Function

Now it's time to create your first Lambda function. You can use the dotnet new command to create a new application from a range of available templates. For this example, we will create an empty Lambda function.

```shellscript dotnet-new
dotnet new lambda.EmptyFunction -n MyFirstLambda
```

The created folder structure contains the following files and folders:

```
MyFirstLambda
└── src
  ├── MyFirstLambda
    ├── Function.cs
    ├── MyFirstLambda.csproj
└── test
   ├── MyFirstLambda.Test
    ├── FunctionTest.cs
```

We will come to tests in a later section, let's first focus on the Lambda function code itself in `Function.cs`. Open up the `MyFirstLambda.csproj` in the IDE of your choice.

<CH.Scrollycoding>

## Lambda Functions in .NET

In it's simplest form, a Lambda function is made up of a single public method. This is the method that will be invoked by the Lambda service. It can be called whatever you want, call it banana if you like.

Typically, it's a good idea to standardize on something along the lines of _`Handler`_ or _`FunctionHandler`_.

```c# Function.cs focus=1:20
using Amazon.Lambda.Core;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MyFirstLambda;

public class Function
{
    /// <summary>
    /// A simple function that takes a string and does a ToUpper
    /// </summary>
    /// <param name="input"></param>
    /// <param name="context"></param>
    /// <returns></returns>
    public string FunctionHandler(string input, ILambdaContext context)
    {
        return input.ToUpper();
    }
}

```

---

## The Function Handler

The handler itself is a simple method call. It be sync or async, that's up to you. The important parts are the method parameters. A Lambda function handler can take up to 2 parameters. 

[The first is the payload](focus://16[34:46]). This is the event payload the Lambda service will pass into your function. In this example, we are passing in a simple string. But as you will see in later examples this can also be a POCO.

[The second is the `ILambdaContext`](focus://16[48:64]). This is an optional parameter, delete it if you wish. The _`ILambdaContext`_ object holds contextual information about this specific invoke. Things like the `RequestId`, `FunctionName` and the `FunctionVersion`.

```c# Function.cs focus=16:20
using Amazon.Lambda.Core;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MyFirstLambda;

public class Function
{
    /// <summary>
    /// A simple function that takes a string and does a ToUpper
    /// </summary>
    /// <param name="input"></param>
    /// <param name="context"></param>
    /// <returns></returns>
    public string FunctionHandler(string input, ILambdaContext context)
    {
        return input.ToUpper();
    }
}

```

---

## The Lambda Serializer

The .NET runtime for Lambda can automatically de/serialize Lambda requests and responses to objects and back again. Specifying this line at the top of your function tells Lambda how to perform this de/serialization. There is built in support for _`System.Text.Json`_ and _`Newtonsoft.Json`_ as well as the ability to provide your own custom serializer.

```c# Function.cs focus=4:4
using Amazon.Lambda.Core;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MyFirstLambda;

public class Function
{
    /// <summary>
    /// A simple function that takes a string and does a ToUpper
    /// </summary>
    /// <param name="input"></param>
    /// <param name="context"></param>
    /// <returns></returns>
    public string FunctionHandler(string input, ILambdaContext context)
    {
        return input.ToUpper();
    }
}

```

</CH.Scrollycoding>

And that is all there is to your first Lambda function. Feeling good? Now let's go and deploy it to your AWS account.