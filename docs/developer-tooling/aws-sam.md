---
sidebar_position: 1
title: AWS SAM
---

The [AWS Serverless Application Model (SAM)](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) is a framework for building serverless applications. AWS SAM is split into two parts, a shorthand syntax that adds additional resources on top of CloudFormation and a command line interface (CLI).

## AWS SAM Template

<CH.Scrollycoding>

## Globals

The global sections of the SAM template allows default properties to be set across all Lambda functions. You can also override each of these properties at a specific function level.

```yaml template.yml  focus=4:13
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    MemorySize: 1024
    Architectures: [x86_64]
    Runtime: dotnet6
    Timeout: 30
    Tracing: Active
    Environment:
      Variables:
        PRODUCT_TABLE_NAME: !Ref Table

Resources:
  GetProductsFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./GetProducts/
      Handler: GetProducts::GetProducts.Function::FunctionHandler
      Events:
        Api:
          Type: HttpApi
          Properties:
            Path: /
            Method: GET
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref DynamoDbTable

  DynamoDbTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      BillingMode: PAY_PER_REQUEST
      KeySchema:
        - AttributeName: id
          KeyType: HASH

```

---

## Native AOT support

AWS SAM also provides support for compiling .NET 7 applications with native AOT. To do that, add an additional metadata property to the Lambda function definition and ensure the _`Runtime`_ is set to _`provided.al2`_. Adding this additional metadata property tells SAM to use the _`Amazon.Lambda.Tools`_ global CLI. For more information on native AOT, check out [this walkthrough](./docs/advanced/native-aot).

```yaml template.yml  focus=18:19
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    MemorySize: 1024
    Architectures: [x86_64]
    Runtime: provided.al2
    Timeout: 30
    Tracing: Active
    Environment:
      Variables:
        PRODUCT_TABLE_NAME: !Ref Table

Resources:
  GetProductsFunction:
    Type: AWS::Serverless::Function
    Metadata:
      BuildMethod: dotnet7
    Properties:
      CodeUri: ./GetProducts/
      Handler: GetProducts::GetProducts.Function::FunctionHandler
      Events:
        Api:
          Type: HttpApi
          Properties:
            Path: /
            Method: GET
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref DynamoDbTable

  DynamoDbTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      BillingMode: PAY_PER_REQUEST
      KeySchema:
        - AttributeName: id
          KeyType: HASH

```

---

## Function

Lambda functions are specified using a resource of type _`AWS::Serverless::Function`_. There are 8 supported resources that can be found in the [AWS Docs](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-generated-resources.html).  The _`CodeUri`_ property tells the SAM CLI which folder the code for this function is stored under.

```yaml template.yml  focus=16:26
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    MemorySize: 1024
    Architectures: [x86_64]
    Runtime: dotnet6
    Timeout: 30
    Tracing: Active
    Environment:
      Variables:
        PRODUCT_TABLE_NAME: !Ref Table

Resources:
  GetProductsFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./GetProducts/
      Handler: GetProducts::GetProducts.Function::FunctionHandler
      Events:
        Api:
          Type: HttpApi
          Properties:
            Path: /
            Method: GET
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref DynamoDbTable

  DynamoDbTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      BillingMode: PAY_PER_REQUEST
      KeySchema:
        - AttributeName: id
          KeyType: HASH

```

---

## IAM permissions

SAM provides a set of pre-built [policy templates](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html) to simplify IAM policies. 

```yaml template.yml  focus=27:29
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    MemorySize: 1024
    Architectures: [x86_64]
    Runtime: dotnet6
    Timeout: 30
    Tracing: Active
    Environment:
      Variables:
        PRODUCT_TABLE_NAME: !Ref Table

Resources:
  GetProductsFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./GetProducts/
      Handler: GetProducts::GetProducts.Function::FunctionHandler
      Events:
        Api:
          Type: HttpApi
          Properties:
            Path: /
            Method: GET
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref DynamoDbTable

  DynamoDbTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      BillingMode: PAY_PER_REQUEST
      KeySchema:
        - AttributeName: id
          KeyType: HASH

```

</CH.Scrollycoding>

## SAM CLI

The SAM CLI is used to _`build`_ and _`deploy`_ your serverless application. 

The _`sam build`_ command both compiles the application code and applies generates a CloudFormation ready template. Under the hood, the build command runs a foreach over all of the specified _`AWS::Serverless::Function`_ resources, and compiles your .NET code. The compiled code is then output to the _`.aws-sam`_ folder.

After running _`sam build`_, run _`sma deploy --guided`_ to deploy the application to AWS.

If video is more your thing, there is an [entire playlist on my YouTube channel](https://www.youtube.com/watch?v=0C9KWutITf0&list=PLCOG9xkUD90JWJrqI8S63_MEDIgtF6JFo) going from AWS SAM basics through to complex CICD.