---
layout: "extend-testing"
page_title: "Extending Terraform - Acceptance Testing Part 2 TestStep"
sidebar_current: "docs-extend-testing-acceptance-teststep"
description: |-
  Extending Terraform is a section for content dedicated to developing Plugins
  to extend Terraform's core offering.
---

# Acceptance Testing Part 2: TestStep

`TestStep`s represent the application of an actual Terraform configuration file
to a given state. Each step requires a configuration as input and provides
developers several means of validating the behavior of the specific resource
under test. 

## Test Modes

 - Lifecycle vs import

## Step Checks

- Compose and aggregate
- All the TestChec funcs

## TestStep Reference API

### Lifecycle tests

### Import tests

## Next Steps

TestCases are a powerful way to test Terraform plugins using real configurations
and verifying the expected results. In the next section [Best
Practices](/docs/extend/testing/acceptance-tests/bestpractices.html) we’ll
cover some best practice testing scenarios to expand on the simple checks above,
and verify actual day-to-day usage and behavior. 

