---
layout: "extend-testing"
page_title: "Extending Terraform - Acceptance Testing"
sidebar_current: "docs-extend-testing-acceptance"
description: |-
  Extending Terraform is a section for content dedicated to developing Plugins
  to extend Terraform's core offering.
---

# Acceptance Tests 

In order to deliver on our promise to be safe and predictable,
we need to be able to easily and routinely verify that Terraform Plugins produce
the expected outcome. The most common usage of an acceptance test is in
Terraform Providers, where each Resource is tested with configuration files and
the resulting infrastructure is verified. Terraform includes a framework for
constructing acceptance tests that imitate the execution of one or more steps of
applying one or more configuration files, allowing multiple scenarios to be
tested.

## Test files

Terraform follows many of the Go programming language conventions with regards
to testing, with both acceptance tests and unit tests being placed in a files
that matches the file under test, with an added `_test.go` suffix. Here’s an
example file structure:

```
terraform-plugin-example/ 
├── provider.go 
├── provider_test.go 
├── example/
│   ├── resource_example_compute.go 
│   ├── resource_example_compute_test.go 
```

To create an acceptance test in the example `resource_example_compute_test.go`
file, the function name must begin with `TestAccXxx`, and have the following
signature:

```go
func TestAccXxx(*testing.T)
```

## Running Acceptance Tests

The easiest way to run acceptance tests is to use the built in `make` step
`testacc`, however, Terraform must be explicitly authorized to run acceptance
tests with the environment variable `TF_ACC`. Example:

```shell
$ TF_ACC=true make testacc 
```

**It’s important to reiterate that acceptance tests create actual resources**,
with possible expenses incurred, and are the responsibility of the user running
the tests. This explicit authorization is to prevent surprising users by
creating real infrastructure at their expense. Creating real infrastructure in
tests verifies the described behavior of Terraform Plugins in real world use
cases against the actual APIs,  and verifies both local state and remote values
match. Acceptance tests require a network connection and often require
credentials to access an account for the given API.

~> **Note: When developing or testing Terraform plugins, it is highly
recommended to run acceptance tests with an account dedicated to testing. This
ensures no infrastructure is created or destroyed in error during development or
validation of any Provider Resources in any environment that cannot be
completely and safely destroyed.**

## Next Steps

Terraform relies heavily on acceptance tests to ensure we keep our promise of
helping users  safely and predictably create, change, and improve
infrastructure. In our next section we detail how to create **Test Cases**,
individual acceptance tests using Terraform’s testing framework, in order to
build and verify real infrastructure. [Proceed to Part 1: Test
Cases](/docs/extend/testing/acceptance-tests/testcase.html)
