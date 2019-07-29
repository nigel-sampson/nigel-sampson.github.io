---
layout: post
title: Approval Tests
tags: csharp
---

Approval Tests or Snapshot Tests are in my opinion a vital part of the testing ecosystem that can service a number of uses cases. In this post I'll introduce the concepts and discuss some of the scenarios we may want to employ them to best effect.

## What are approval tests?
Approval tests work by creating an output that needs human approval / verification. This could be because of a number of reasons:

- The output is visual such as a screenshot.
- The output is too large for regular unit testing assertions (a complex object graph etc.).
- The output has no way to programmatically defined.

Once the initial output has been defined and "approved" then as long as the test provides consistent output then the test will continue to pass. Once the test provides output that is different to the approved output then the test will fail. The developer then has two choices:

1. If the change in the output was unintended then fix the bug that's causing the change in the output.
2. "Approve" the new output as the baseline for future tests.

I've been very vague here about "output" intentionally because that output can be anything you want really, as long as it can be compared to another copy in a consistent manner.

## When should I use approval tests?
Here are some scenarios where we use approval tests at [Pushpay][pp].

### Website Visuals
We use an automated testing process that takes screenshots (using headless Chrome) of most pages in our web applications with static data. If the page has changed for any reason then the test fails. If that change is intended the developer can simply mark the new visual as approved and carry on, otherwise it gives a clear indidication that the changes they're testing are having unintended consequences on pages they didn't attend.

### API Contracts
We use the SDL of GraphQL API's and Swagger for the REST API's as the output for an approval test. What this means is that any change that results in the change to the public API contract will result in a failed test. We want to be very deliberate in any changes we make in the contracts we expose to other teams and this guards against unintended changes. It also forces the developer to think very closely about the contract as they approve it.

### Exceptions to the rule
We may have a conventions or policies that will almost always have exceptions. An example of this are our feature flags, ideally our Sandbox and Production environments should be very similar in terms of which flags are on. We can have an approval test where the output is the list of all flags where the state is different between environments, this way we can capture and approve exceptions to our rule and ensure that any of these exceptions are intentional and approved.

## Approval Tests in .NET

Some good frameworks I've used for approval tests in .NET are [Approval Tests.NET][at] and [Snapshooter][ss].

## Summary

I hope you can see from the examples above that approval tests fill a very valuable niche where the output of the test requires human intelligence to assert corrections. They can capture impossible to assert rules as well as exceptions to the rules. They can help to ensure that changes to critical parts of your system such as the API and the visuals are intentional and approved.

[pp]: https://pushpay.com
[at]: https://github.com/approvals/ApprovalTests.Net
[ss]: https://github.com/SwissLife-OSS/snapshooter