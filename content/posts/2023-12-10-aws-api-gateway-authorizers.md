---
title: AWS API Gateway Authorizer Patterns
date: "2023-12-10"
comments: true
buyMeACoffee: true
draft: false
toc: true
Cover:
tags:
  - aws
  - cloud

---

## Introduction

AWS API Gateway Authorizers are a powerful tool to secure API Gateway endpoints.

The configuration of Authorizers can be a bit overwhelming at first. There are many options which have an impact 
on the security, performance and programming model of your API Gateway.

This article is a collection of three patterns that I have identified from my experience with AWS API Gateway Authorizers,
from discussions with peers and colleagues and from existing documentation online.


Please note that this is not a comprehensive guide to API Gateway Authorizers. It's a collection of abstract concepts
that can be used to reason about the different patterns:

1. [**No-Authorizer Pattern**](#no-authorizer-pattern)
2. [**Binary Authorizer Pattern**](#binary-authorizer-pattern)
3. [**Context Authorizer Pattern**](#context-authorizer-pattern)

Note that the naming of the patterns is not official. I made them up to make it easier to talk about them.

## No-Authorizer Pattern

The first pattern consists of an API Gateway and integration Lambdas in the background.

![diagram](/img/generated/no-authorizer.svg)

There is no authorizer Lambda configured in the API Gateway. All requests to the API Gateway are authorized by default
and forwarded to the backend Lambdas. The Lambdas are responsible for authorization directly.

In the diagram above, there is an exemplary "External User Service" that is queried by the backend Lambdas to authorize
the request.

### When to use

- When there is a only handful of Lambdas to integrate with the API Gateway
- When it's possible to re-use the same authorization code across Lambdas in a maintainable way (same code repository or
  shared library)
- When the authorization call does not take a considerable amount of time to execute
- When authorization logic should be testable together with the business logic
- When each invocation should use the up-to-date authorization information

## Cacheable vs Non-Cacheable Authorizers

Before we dive into the next patterns, let's talk about caching.

API Gateway Authorizers can be configured to cache the authorization result for a given request for a given amount of
time.
This is useful when the authorization call takes a considerable amount of time to execute and the authorization result
is not expected to change within the cache time.

Caching the authorization result can significantly improve the response times of the authorizer but it comes with a
trade-off.
The cached authorization result is not updated until the cache expires. This means that the authorization result may not
be in sync with the actual authorization state of the user.

This needs to be taken into account when choosing the right authorizer pattern.

### Example

1. There's a role based authorization system with 2 roles: `admin` and `user`
2. User A has an admin role and requests a resource in our AWS backend
3. The authorization result ("user A has admin role and is allowed to do admin things") is cached for a given amount of
   time
4. In the meantime, user A's role is changed to `user`
5. User A requests the same resource again where the authorization result is cached => The cached authorization result
   is used and user A is allowed to do admin things even though he shouldn't be allowed to

## Binary Authorizer Pattern

This pattern consists of an API Gateway, a single authorizer and integration Lambdas.

![diagram](/img/generated/binary-authorizer.svg)

The authorizer is responsible for the authorization based on a binary decision (e.g. "does the request contain a valid
token"). In the diagram above, an exemplary "External User Service" is queried by the authorizer to authorize the request.

Authorized requests are forwarded to backend Lambdas. The Lambdas assume that the requester is authorized to perform the
call and can focus on the business logic.

### When to use

- When separation of concerns between authorization and business logic is needed (code, test, deploy)

### When to cache

- When the authorization call takes a considerable amount of time to execute
- When the security requirements allow the use of caching to improve response times of Authorizer and Lambda invocations

## Context Authorizer Pattern

This pattern consists of an API Gateway, a single authorizer and integration Lambdas.

![diagram](/img/generated/context-authorizer.svg)

The authorizer is responsible for the authorization based on a binary decision (e.g. "does the request contain a valid
token")
and for providing the authorization information to the backend Lambdas.

The Lambdas assume that the requester is authorized to perform the call and receive additional context information to
make more nuanced authorization decisions.

In the diagram above, an exemplary "External User Service" is queried by the authorizer to authorize the request and 
get the role of the user. The role is then forwarded to the backend Lambdas as context information.

### When to use

- When the authorization conditions are complex and require evaluation on a per-Lambda basis
- When it's possible to re-use the same authorization evaluation code across Lambdas in a maintainable way 
  (same code repository or shared library)

### When to cache

- When the authorization call takes a considerable amount of time to execute
- When the security requirements allow the use of caching to improve response times of Authorizer and Lambda invocations

## Choosing the right authorizer pattern

Here are some questions to ask yourself when choosing the right authorizer pattern.

Please note that these are based on the identity source being a authorization token.
This does not take into account other identity sources like request parameters where caching behaves differently (I hope
to cover this in a future article).

1. Does my use-case need strict security requirements?

    - Yes:
        - No-Authorizer Pattern
        - Non-Cacheable Binary Authorizer Pattern
        - Non-Cacheable Context Authorizer Pattern

2. Do my authorizer calls take a considerable amount of time to execute?

    - Yes:
        - Cacheable Binary Authorizer Pattern
        - Cacheable Context Authorizer Pattern

3. Do all my Lambdas require the same authorization information?

    - Yes:
        - Binary Authorizer Pattern

4. Do my Lambdas require different authorization information?

    - Yes:
        - Context Authorizer Pattern

## Conclusion

We have looked at different patterns for API Gateway Authorizers and learned when to use which pattern.

There is no one-size-fits-all solution. It's important to understand the trade-offs of each pattern and choose the right
one for your use-case.

## Further reading and mentions

- [AWS Lambda Authorizer Patterns and Caching](https://www.linkedin.com/pulse/aws-lambda-authorizer-patterns-caching-harshit-pandey/)
- [API Gateway Authorizer and Logout (Performance/Security Considerations)](https://stackoverflow.com/q/53813947/1510659)
