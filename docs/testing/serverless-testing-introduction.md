---
sidebar_position: 1
title: An Introduction to Serverless Testing
---

A variety of different testing techniques are required to build production ready serverless applications. Some of which require a paradigm shift from what you as a developer might be used to. I'm a .NET developer to, so I know how much you love running up an application on local host and stepping through with your debugger. With serverless, this can be a challenge.

Serverless applications are made up of lots of small, loosely joined pieces of application functionality. Each piece of your application code has a single responsibility, and work is done through integrations. Serverless applications are commonly heavily reliant on other cloud services to handle these integrations. Services like SQS, SNS, EventBridge and Step Functions to name a few.

Because of how heavily integrations are leveraged, I would always recommend to **test in the cloud** as early as possible. Add IAM and permissions into the mix as well and trying to fake/emulate all of this locally is difficult. Test against actual cloud services! Unit test your business logic locally, but run integration tests in the cloud itself.

[Spotify engineering wrote a fantastic blog article](https://engineering.atspotify.com/2018/01/testing-of-microservices/) around how they test microservices, including the introduction of the term 'testing honeycomb'. In the testing honeycomb, there are a small number of unit tests to test business logic. A large number of integration tests to test how the service in question works in a deployed environment, and then a minimal number of integrated or system tests.

![Testing pyramid and honeycomg](/img/testing/testing-shapes.png)

In this section, we will walk through different testing patterns you can use when building serverless applications.

## Mocks vs Emulators

A quick word on mocking vs emulation. When building serverless app, the recommendation for local development is to mock where possible and use emulators sparingly. Whilst emulating the whole of AWS locally on your machine may seem like a good idea, make them an edge case. Lack of feature parity (+ IAM), configuration cost and limited service coverage are some of the reasons why relying on emulators as a test strategy may not provide a great developer experience.

The same applies to mocking. Although mocking does enable you to control the functionality of an AWS SDK, there is still a cost in configuration and lack of feature parity. Mocks are useful for testing your business logic without needing to worry about external integrations, but for truly valuable tests get in the cloud as quickly as possible.

## Further Reading

- The AWS Samples GitHub organisation contains a [serverless-test-samples repository](https://github.com/aws-samples/serverless-test-samples)