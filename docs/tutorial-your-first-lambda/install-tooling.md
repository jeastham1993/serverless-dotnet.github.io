---
sidebar_position: 1
title: Install Tooling
description: A guide to installing additional .NET tools for developing Lambda functions
keywords: [dotnet cli]
---

# Install Tooling

AWS provides a range of tools to help enable the development of serverless applications with .NET.

## .NET Global Tool Amazon.Lambda.Tools

The [Amazon.Lambda.Tools](https://www.nuget.org/packages/Amazon.Lambda.Tools) .NET CLI tool adds a range of features to the .NET CLI to simplify building serverless applications on AWS.

```shellscript install
dotnet tool install --global Amazon.Lambda.Tools
```

## Amazon.Lambda.Templates

The [Amazon.Lambda.Templates](https://www.nuget.org/packages/Amazon.Lambda.Templates) Nuget package adds templates to the Microsoft Template Engine that are then accessible through the .NET CLI/

```shellscript install
dotnet new --install Amazon.Lambda.Templates
```