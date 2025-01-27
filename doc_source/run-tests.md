--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Run the AWS IoT Greengrass qualification suite<a name="run-tests"></a>

After you [set the required configuration](set-config.md), you can start the tests\. The runtime of the full test suite depends on your hardware\. For reference, it takes approximately 30 minutes to complete the full test suite on a Raspberry Pi 3B\.

The following example `run-suite` commands show you how to run the qualification tests for a device pool\. A device pool is a set of identical devices\.

------
#### [ IDT v3\.0\.0 and later ]

Run all test groups in a specified test suite\.  

```
devicetester_[linux | mac | win_x86-64] run-suite --suite-id GGQ_1.0.0 --pool-id <pool-id>
```
Use the `list-suites` command to list the test suites that are in the `tests` folder\.

Run a specific test group in a test suite\.  

```
devicetester_[linux | mac | win_x86-64] run-suite --suite-id GGQ_1.0.0 --group-id <group-id> --pool-id <pool-id>
```
Use the `list-groups` command to list the test groups in a test suite\.

Run a specific test case in a test group\.  

```
devicetester_[linux | mac | win_x86-64] run-suite --group-id <group-id> --test-id <test-id>
```

Run multiple test cases in a test group\.  

```
devicetester_[linux | mac | win_x86-64] run-suite --group-id <group-id> --test-id <test-id1>,<test-id2>
```

List the test cases in a test group\.  

```
devicetester_[linux | mac | win_x86-64] list-test-cases --group-id <group-id>
```

The options for the `run-suite` command are optional\. For example, you can omit `pool-id` if you have only one device pool defined in your `device.json` file\. Or, you can omit `suite-id` if you want to run the latest test suite version in the `tests` folder\.

**Note**  
IDT prompts you if a newer test suite version is available online\. For more information, see [Set the default update behavior](#idt-update-behavior)\.

For more information about `run-suite` and other IDT commands, see [IDT for AWS IoT Greengrass commands](#bk-cli)\.

------
#### [ IDT v2\.3\.0 and earlier ]

Run all test groups in a specified suite\.  

```
devicetester_[linux | mac | win_x86-64] run-suite --suite-id GGQ_1 --pool-id <pool-id>
```

Run a specific test group\.  

```
devicetester_[linux | mac | win_x86-64] run-suite --suite-id GGQ_1 --group-id <group-id> --pool-id <pool-id>
```
`suite-id` and `pool-id` are optional if you are running a single test suite on a single device pool\. This means that you have only one device pool defined in your `device.json` file\.

------

## Check for Greengrass dependencies<a name="idt-dependency-checker"></a>

We recommend that you run the dependency checker test group to make sure all Greengrass dependencies are installed before you run related test groups\. For example:
+ Run `ggcdependencies` before running core qualification test groups\.
+ Run `containerdependencies` before running container\-specific test groups\.
+ Run `dockerdependencies` before running Docker\-specific test groups\.
+ Run `ggcstreammanagementdependencies` before running stream manager\-specific test groups\.

## Set the default update behavior<a name="idt-update-behavior"></a>

When you start a test run, IDT checks online for a newer test suite version\. If one is available, IDT prompts you to update to the latest available version\. You can set the `upgrade-test-suite` \(or `u`\) flag to control the default update behavior\. Valid values are:
+ `y`\. IDT downloads and uses the latest available version\.
+ `n` \(default\)\. IDT uses the version specified in the `suite-id` option\. If `suite-id` is not specified, IDT uses the latest version in the `tests` folder\.

If you don't include the `upgrade-test-suite` flag, IDT prompts you when an update is available and waits 30 seconds for your input \(`y` or `n`\)\. If no input is entered, it defaults to `n` and continues running the tests\.

The following examples show common use cases for this feature:

**Automatically use the latest tests available for a test group\.**  

```
devicetester_linux run-suite -u y --group-id mqtt --pool-id DevicePool1
```

**Run tests in a specific test suite version\.**  

```
devicetester_linux run-suite -u n --suite-id GGQ_1.0.0 --group-id mqtt --pool-id DevicePool1
```

**Prompt for updates at runtime\.**  

```
devicetester_linux run-suite --pool-id DevicePool1
```

## IDT for AWS IoT Greengrass commands<a name="bk-cli"></a>

The IDT commands are located in the `<device-tester-extract-location>/bin` directory\. Use them for the following operations:

------
#### [ IDT v3\.0\.0 and later ]

`help`  <a name="idt-command-help"></a>
Lists information about the specified command\.

`list-groups`  <a name="idt-command-list-groups"></a>
Lists the groups in a given test suite\.

`list-suites`  <a name="idt-command-list-suites"></a>
Lists the available test suites\.

`list-supported-products`  
Lists the supported products, in this case AWS IoT Greengrass versions, and test suite versions for the current IDT version\.

`list-test-cases`  
Lists the test cases in a given test group\. The following option is supported:  
+ `group-id`\. The test group to search for\. This option is required and must specify a single group\.

`run-suite`  
Runs a suite of tests on a pool of devices\. The following are some supported options:  
+ `suite-id`\. The test suite version to run\. If not specified, IDT uses the latest version in the `tests` folder\.
+ `group-id`\. The test groups to run, as a comma\-separated list\. If not specified, IDT runs all test groups in the test suite\.
+ `test-id`\. The test cases to run, as a comma\-separated list\. When specified, `group-id` must specify a single group\.
+ `pool-id`\. The device pool to test\. You must specify a pool if you have multiple device pools defined in your `device.json` file\.
+ `upgrade-test-suite`\. Controls how test suite version updates are handled\. Starting in IDT v3\.0\.0, IDT checks online for updated test suite versions\. For more information, see [Test suite versions](idt-gg-qualification.md#idt-test-suite-versions)\.
+ `stop-on-first-failure`\. Configures IDT to stop execution on the first failure\. This option should be used with `group-id` to debug the specified test groups\. Do not use this option when running a full test\-suite to generate a qualification report\.
+ `update-idt`\. Sets the response for the prompt to update IDT\. `Y` as input stops the test execution if IDT detects there is a newer version\. `N` as input continues the test execution\.
+ `update-managed-policy`\. `Y` as input stops the test execution if IDT detects that the user's managed policy isn’t updated\. `N` as input continues the test execution\.
For more information about `run-suite` options, use the `help` option:  

```
devicetester_[linux | mac | win_x86-64] run-suite -h
```

------
#### [ IDT v2\.3\.0 and earlier ]

`help`  <a name="idt-command-help"></a>
Lists information about the specified command\.

`list-groups`  <a name="idt-command-list-groups"></a>
Lists the groups in a given test suite\.

`list-suites`  <a name="idt-command-list-suites"></a>
Lists the available test suites\.

`run-suite`  
Runs a suite of tests on a pool of devices\.  
For more information about `run-suite` options, use the `help` option:  

```
devicetester_[linux | mac | win_x86-64] run-suite -h
```

------