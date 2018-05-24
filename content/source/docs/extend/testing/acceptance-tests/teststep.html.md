---
layout: "extend-testing"
page_title: "Extending Terraform - Acceptance Testing Part 2 TestStep"
sidebar_current: "docs-extend-testing-acceptance-teststep"
description: |-
  Extending Terraform is a section for content dedicated to developing Plugins
  to extend Terraform's core offering.
---

# Acceptance Tests Part 2: TestStep
`TestStep`s represent the application of an actual Terraform configuration file to a given state. Each step requires a configuration as input and provides developers several means of validating the behavior of the specific resource under test. 

## Test Modes
Terraform’s test framework facilitates two distinct modes of acceptance tests, *Lifecycle* and *Import*. 

**Lifecycle** mode is the most common mode, and is used for testing plugins by providing one or more configuration files with the same logic as would be used when running `terraform apply`. 

**Import** mode is used for testing resource functionality to import existing infrastructure into a Terraform statefile, using the same logic as would be used when running `terraform import`. 

An acceptance test’s mode is implicitly determined by the fields provided in the `TestStep` definition. The applicable fields are defined below in the [TestStep Reference API][#teststep-reference-api]. 
## Steps
`Steps` is slice property of [TestCase](/docs/extend/testing/acceptance-tests/testcase.html), the object used to construct acceptance tests. Each step represents a full `terraform apply` of a given configuration language, followed by zero or more checks (defined later) to verify the application. Each `Step` is applied in order, and require its own configuration and optional check functions. 

Below is a code example of a lifecycle test that provides two `TestStep` objects: 

```golang
package example

// example.Widget represents a concrete Go type that represents an API resource
func TestAccExampleWidget_basic(t *testing.T) {
	var widgetBefore, widgetAfter example.Widget
	rName := acctest.RandStringFromCharSet(10, acctest.CharSetAlphaNum)

	resource.Test(t, resource.TestCase{
		PreCheck:     func() { testAccPreCheck(t) },
		Providers:    testAccProviders,
		CheckDestroy: testAccCheckExampleResourceDestroy,
		Steps: []resource.TestStep{
			{
				Config: testAccExampleResource(rName),
				Check: resource.ComposeTestCheckFunc(
					testAccCheckExampleResourceExists("example_widget.foo", &widgetBefore),
				),
			},
			{
				Config: testAccExampleResource_removedPolicy(rName),
				Check: resource.ComposeTestCheckFunc(
					testAccCheckExampleResourceExists("example_widget.foo", &widgetAfter),
				),
			},
		},
	})
}
```

In the above example each `TestCase` invokes a function to retrieve it’s desired configuration, based on a randomized name provided, however an in-line string or constant string would work as well, so long as they contain valid Terraform configuration for the plugin or resource under test. This pattern of a basic configuration followed by a second, modified configuration to test update functionality is a common pattern and covered in our [Best Practices][1] section.

## Check Functions
After the configuration for a `TestStep` is applied, Terraform’s testing framework provides developers an opportunity to check the results by providing a “Check” function. While possible to only supply a single function, it is recommended you use multiple functions to validate specific information about the results of the `terraform apply` ran in each `TestStep`. The `Check` attribute is of `TestStep` is singular, so in order to include multiple checks developers should use either `ComposeTestCheckFunc` or `ComposeAggregateTestCheckFunc` (defined below) to group multiple check functions, defined below:

### ComposeTestCheckFunc 

ComposeTestCheckFunc lets you compose multiple TestCheckFunc functions into a single check. As a user testing their provider, this lets you decompose your checks into smaller pieces more easily, with individual methods for checking specific attributes. Each check is ran in the order provided, and on failure the entire `TestCase` is stopped, and Terraform attempts to destroy any resources created.

Example:

```go 
Steps: []resource.TestStep{
  {
    Config: testAccExampleResource(rName),
    Check: resource.ComposeTestCheckFunc(
      testAccCheckExampleResourceExists("example_widget.foo", &widgetBefore), // if testAccCheckExampleResourceExists fails to find the resource, the parent TestStep and TestCase fail
      resource.TestCheckResourceAttr("example_widget.foo", "size", "expected size"),
    ),
  },
},
```

### ComposeAggregateTestCheckFunc 

ComposeAggregateTestCheckFunc lets you compose multiple TestCheckFunc functions into a single check. It’s purpose and usage is identical to ComposeTestCheckFunc, however each check is ran in order even if a previous check failed, collecting the errors returned from any checks and returning a single aggregate error. The entire `TestCase` is still stopped, and Terraform attempts to destroy any resources created. 

Example:

```
Steps: []resource.TestStep{
  {
    Config: testAccExampleResource(rName),
    Check: resource.ComposeAggregateTestCheckFunc(
      testAccCheckExampleResourceExists("example_widget.foo", &widgetBefore), // if testAccCheckExampleResourceExists fails to find the resource, the following TestCheckResourceAttr is still ran, with any errors aggregated
      resource.TestCheckResourceAttr("example_widget.foo", "active", "true"),
    ),
  },
},
```

## Builtin check functions
Terraform has several TestCheckFunc functions builtin for developers to use for common checks, such as verifying the status and value of a specific attribute in the resulting state. Developers are encouraged to use as many as reasonable to verify the behavior of the plugin/resource, and should combine them with the above mentioned `ComposeTestCheckFunc` or `ComposeAggregateTestCheckFunc` functions.

Most builtin functions accept `name`, `key`, and/or `value` fields, derived from the typical Terraform configuration stanzas:

```hcl
resource "example_widget" "foo" {
  active = true
}
```

Here the `name` represents the resource name in state (`example_widget.foo`), the `key` represents the attribute to check (`active`), and `value` represents the desired value to check against (`true`). Not all functions accept all three inputs.
### TestCheckResourceAttrSet(name, key string)

TestCheckResourceAttrSet ensures a value exists in state for the given name/key combination. It is useful when testing that computed values were set, when it is not possible to know ahead of time what the values will be.

Example:

```go
Steps: []resource.TestStep{
  {
    Config: testAccExampleResource(rName),
    Check: resource.ComposeAggregateTestCheckFunc(
        resource.TestCheckResourceAttrSet("example_widget.foo", "created_at"), // Computed
    ),
  },
},
```

### TestCheckResourceAttr(name, key, value string)

TestCheckResourceAttr checks the value for a given attribute. 

Example:

```go
Steps: []resource.TestStep{
  {
    Config: testAccExampleResource(rName),
    Check: resource.ComposeAggregateTestCheckFunc(
      resource.TestCheckResourceAttrSet("example_widget.foo", "created_at"), // Computed
      resource.TestCheckResourceAttr("example_widget.foo", "name", rName),
      resource.TestCheckResourceAttr("example_widget.foo", "active", "true"),
    ),
  },
},
```

### TestCheckNoResourceAttr(name, key, value string)

TestCheckNoResourceAttr is an inversion of TestCheckResourceAttr, checking that NO value for the given key exists.

Example:

```go
Steps: []resource.TestStep{
  {
    Config: testAccExampleResource(rName),
    Check: resource.ComposeAggregateTestCheckFunc(
      resource.TestCheckResourceAttr("example_widget.foo", "active", "true"),
      // still active, should not have deleted_at date
      resource.TestCheckNoResourceAttr("example_widget.foo", "deleted_at"), 
    ),
  },
},

```
## Custom check functions 

The `Check` attribute of `TestStep` accepts any function of type [TestCheckFunc](https://godoc.org/github.com/hashicorp/terraform/helper/resource#TestCheckFunc). 

## TestStep Reference API
The fields `TestStep` supports are documented below, including the mode (lifecycle or import) that they apply to.

## Next Steps
TestCases are a powerful way to test Terraform plugins using real configurations and verifying the expected results. In the next section [Best Practices][1] we’ll cover some best practice testing scenarios to expand on the simple checks above, and verify actual day-to-day usage and behavior. 




[1]: /docs/extend/testing/acceptance-testing/bestpractices.html

