---
sidebar_position: 3
title: Terraform
---

[Terraform](https://www.terraform.io/) is one of the most popular infrastructure as code (IaC) tools available today. It enables the automation of infrastructure across any cloud. Terraform allows you to build any kind of infrastructure, not just serverless. For this example, let's focus on deploying a Lambda function.

Deploying .NET Lambda functions using Terraform adds an additional complexity. Your .NET function code needs to be compiled before Terraform can deploy your resources. This is less of an issue for interpreted languages like NodeJS and Python as no compilation is required.

However, the first thing you'll need to do is [install Terraform.](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli).

Once installed, create a new directory:

```shellscript start-terraform
mkdir terraform-dotnet-serverless
cd terraform-dotnet-serverless
```

Then create a file called _`main.tf`_. 

Copy in the below code. There's a lot, don't worry I'll walk through it step by step:

<details>
<summary>Click Here for Code Sample</summary>

```terraform main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

resource "random_uuid" "bucket_random_id" {
}

# Configure the AWS Provider
provider "aws" {
  region = "eu-west-1"
}

# Create S3 bucket to store our application source code.
resource "aws_s3_bucket" "lambda_bucket" {
  bucket        = "${random_uuid.bucket_random_id.result}-dotnet-tf-bucket"
  acl           = "private"
  force_destroy = true
}

data "archive_file" "lambda_archive" {
  type = "zip"

  source_dir  = "Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/bin/Release/net6.0/linux-x64/publish"
  output_path = "Dotnet.CDK.Lambda.zip"
}

resource "aws_s3_object" "lambda_bundle" {
  bucket = aws_s3_bucket.lambda_bucket.id

  key    = "Dotnet.CDK.Lambda.zip"
  source = data.archive_file.lambda_archive.output_path

  etag = filemd5(data.archive_file.lambda_archive.output_path)
}

resource "aws_cloudwatch_log_group" "aggregator" {
  name = "/aws/lambda/${aws_lambda_function.function.function_name}"

  retention_in_days = 30
}

resource "aws_iam_role" "lambda_function_role" {
  name = "FunctionIamRole_dotnet-terraform-lambda"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Sid    = ""
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_policy_attach" {
  role       = aws_iam_role.lambda_function_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_lambda_function" "function" {
  function_name    = "dotnet-terraform-lambda"
  s3_bucket        = aws_s3_bucket.lambda_bucket.id
  s3_key           = aws_s3_object.lambda_bundle.key
  runtime          = "dotnet6"
  handler          = "Dotnet.CDK.Lambda::Dotnet.CDK.Lambda.Function::FunctionHandler"
  source_code_hash = data.archive_file.lambda_archive.output_base64sha256
  role             = aws_iam_role.lambda_function_role.arn
  timeout          = 30
}

output "function_name" {
  value = aws_lambda_function.function.function_name
}
```

</details>

Let's also create a new Lambda function to use as an example.

```shellscript
mkdir Functions
cd Functions
dotnet new lambda.EmptyFunction -n Dotnet.CDK.Lambda
```

<CH.Scrollycoding>

## Terraform Provider

The opening section of the file configures the AWS provider for Terraform. More documentation on this is found in the [Terraform documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs).

```terraform main.tf focus=1:13
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "eu-west-1"
}

#...
```
---

## S3 Bucket for Code Storage

When packaging Lambda applications as ZIP files an S3 bucket is required to store the code. This code creates a new S3 bucket using a random identifier. You can update the _`bucket`_ property if you wish to use a different bucket name.

```terraform main.tf focus=3:11
#...

resource "random_uuid" "bucket_random_id" {
}

# Create S3 bucket to store our application source code.
resource "aws_s3_bucket" "lambda_bucket" {
  bucket        = "${random_uuid.bucket_random_id.result}-dotnet-tf-bucket"
  acl           = "private"
  force_destroy = true
}

#...
```
---

## Lambda ZIP Package

Terraform [provides a data source](https://registry.terraform.io/providers/hashicorp/archive/latest/docs/data-sources/archive_file) for generating ZIP files. The publish directory of the Lambda function is specified as the source directory and a ZIP file is generated. This ZIP file is then uploaded to S3 using the _`aws_s3_object`_ resource. 

```terraform main.tf focus=3:17
#...

data "archive_file" "lambda_archive" {
  type = "zip"

  source_dir  = "Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/bin/Release/net6.0/linux-x64/publish"
  output_path = "Dotnet.CDK.Lambda.zip"
}

resource "aws_s3_object" "lambda_bundle" {
  bucket = aws_s3_bucket.lambda_bucket.id

  key    = "Dotnet.CDK.Lambda.zip"
  source = data.archive_file.lambda_archive.output_path

  etag = filemd5(data.archive_file.lambda_archive.output_path)
}

#...
```
---

## CloudWatch

Additionally, a Amazon CloudWatch log group is defined to store logs generated by the Lambda function.

```terraform main.tf
#...

resource "aws_cloudwatch_log_group" "aggregator" {
  name = "/aws/lambda/${aws_lambda_function.function.function_name}"

  retention_in_days = 30
}

#...
```
---

## IAM

An IAM role and policy allow Lambda execution is then created to provide Lambda with the _`AWSLambdaBasicExecutionRole`_ required to run.

```terraform main.tf
#...

resource "aws_iam_role" "lambda_function_role" {
  name = "FunctionIamRole_dotnet-terraform-lambda"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Sid    = ""
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_policy_attach" {
  role       = aws_iam_role.lambda_function_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

#...
```
---

## Lambda Function Resource

A Lambda function is defined using the _`aws_lambda_function`_ resource. Notice that the _`s3_bucket`_, _`s3_key`_ and _`source_code_hash`_ properties are using values from the S3 and asset objects defined earlier.

```terraform main.tf focus=5:6,9
#...

resource "aws_lambda_function" "function" {
  function_name    = "dotnet-terraform-lambda"
  s3_bucket        = aws_s3_bucket.lambda_bucket.id
  s3_key           = aws_s3_object.lambda_bundle.key
  runtime          = "dotnet6"
  handler          = "Dotnet.CDK.Lambda::Dotnet.CDK.Lambda.Function::FunctionHandler"
  source_code_hash = data.archive_file.lambda_archive.output_base64sha256
  role             = aws_iam_role.lambda_function_role.arn
  timeout          = 30
}

#...
```
---

## Output Name

And finally the name of the Lambda function is output to the console. 

```terraform main.tf focus=3:5
#...

output "function_name" {
  value = aws_lambda_function.function.function_name
}

#...
```
</CH.Scrollycoding>

Once defined, run the below commands from a terminal window in the same folder as the _`main.tf`_ file.

```shellscript deploy
dotnet publish ./Functions/Dotnet.CDK.Lambda/src/Dotnet.CDK.Lambda/ -c Release -r linux-x64
terraform init
terraform apply
```

Once deployed, invoke the Lambda function in AWS using the below command replacing the _`OUTPUT_FUNCTION_NAME`_ with the value output to your terminal:

```shellscript
dotnet lambda invoke-function -n OUTPUT_FUNCTION_NAME -p hello
```
