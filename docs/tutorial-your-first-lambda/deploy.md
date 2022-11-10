---
sidebar_position: 4
title: Deploy
description: A guide deploying a Lambda function with the .NET CLI
keywords: [dotnet cli]
---

# Deploy

Ok, now it's actually time to deploy. I promise! Now that we have configured our Lambda function, deploying it is as simple as running a single command in your terminal window. Ensure you are in the same directory as the _`MyFirstLambda.csproj`_ file at this point.

```bash deploy
dotnet lambda deploy-function
```

You'll now be asked to specify any values you didn't configure manually in _`aws-lambda-tools-defaults.json`_. 

During the deployment wizard you will be asked to select an IAM role. At this stage, I'd suggest creating a new IAM role with a name that reflects the name you chose for the Lambda function itself. If you called your Lambda _`my-first-lambda`_ then call the role _`my-first-lambda-role`_. Give the role _`AWSLambdaBasicExecutionRole`_ permissions and away you go 🚀.

## Test

Now that we have a Lambda function running in the cloud we can even invoke it right from our terminal window. Run the below command, substituing in the `LAMBDA_FUNCTION_NAME` placeholder with the name chosen in the deploy step.

```bash
dotnet lambda invoke-function LAMBDA_FUNCTION_NAME -p hello, world!
```

The response back from the command will give you something along the lines of

```bash
Amazon Lambda Tools for .NET Core applications (5.6.1)
Project Home: https://github.com/aws/aws-extensions-for-dotnet-cli, https://github.com/aws/aws-lambda-dotnet

Payload:
"HELLO"

Log Tail:
START RequestId: ea7191e7-b800-48a8-a165-3b6bc0c45533 Version: $LATEST
END RequestId: ea7191e7-b800-48a8-a165-3b6bc0c45533
REPORT RequestId: ea7191e7-b800-48a8-a165-3b6bc0c45533  Duration: 173.23 ms     Billed Duration: 174 ms Memory Size: 256 MB     Max Memory Used: 67 MB
```

## Congratulations

You've just deployed your first Lambda function.