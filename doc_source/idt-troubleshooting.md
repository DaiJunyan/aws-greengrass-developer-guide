--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# IDT for AWS IoT Greengrass troubleshooting<a name="idt-troubleshooting"></a>

IDT for AWS IoT Greengrass writes these errors to various locations based on the type of errors\. Errors are written to the console, log files, and test reports\.

## Error codes<a name="bk-error-codes"></a>

The following table lists the error codes generated by IDT for AWS IoT Greengrass\.


| Error code | Error code name | Possible root cause | Troubleshooting | 
| --- | --- | --- | --- | 
|  101  |  InternalError  |  An internal error occurred\.  |   Check logs under the `<device-tester-extract-location>/results` directory\. If you cannot debug the issue, contact [AWS Developer Support](https://aws.amazon.com/premiumsupport/plans/developers/)\.  | 
|  102  |  TimeoutError  |  The test cannot be completed in a limited time range\. This can happen if: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  | 
|  103  |  PlatformNotSupportError  |  Incorrect OS/architecture combination specified in `device.json`\.  |  Change your configuration to one of the supported combinations: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html) For more information, see [Configure device\.json](set-config.md#device-config)\.  | 
|  104  |  VersionNotSupportError  |  The AWS IoT Greengrass Core software version is not supported by the version of IDT you are using\.  |  Use the device\_tester\_bin version command to find the supported version of the AWS IoT Greengrass Core software\. For example, if you are using macOS, use \./devicetester\_mac\_x86\_64 version\. To find the version of AWS IoT Greengrass Core software that you are using: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html) You can test a different version of the AWS IoT Greengrass Core software\. For more information, see [Getting started with AWS IoT Greengrass](gg-gs.md)\.  | 
|  105  |  LanguageNotSupportError  |  IDT supports Python for AWS IoT Greengrass libraries and SDKs only\.  |  Make sure: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  | 
|  106  |  ValidationError  |  Some fields in `device.json` or `config.json` are invalid\.  |  Check the error message on the right side of the error code in the report\.  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  | 
|  107  |  SSHConnectionFailed  |  The test machine cannot connect to the configured device\.  |  Verify that the following fields in your `device.json` file are correct: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html) For more information, see [Configure device\.json](set-config.md#device-config)\.  | 
|  108  |  RunCommandError  |  A test failed to execute a command on the device under test\.  |  Verify that root access is allowed for the configured user in `device.json`\. A password is required by some devices when executing commands with root access\. Make sure root access is allowed without a password\. For more information, see the documentation for your device\. Try running the failing command manually on your device to see if an error occurs\.  | 
|  109  |  PermissionDeniedError  |  No root access\.  |  Set root access for the configured user on your device\.  | 
|  110  |  CreateFileError  |  Unable to create a file\.  |  Check your device's disk space and directory permissions\.  | 
|  111  |  CreateDirError  |  Unable to create a directory\.  |  Check your device's disk space and directory permissions\.  | 
|  112  |  InvalidPathError  |  The path to the AWS IoT Greengrass Core software is incorrect\.  |  Verify that the path in the error message is valid\. Do not edit any files under the `devicetester_greengrass_<os>` directory\.  | 
|  113  |  InvalidFileError  |  A file is invalid\.  |  Verify that the file in the error message is valid\.  | 
|  114  |  ReadFileError  |  The specified file cannot be read\.  |  Verify the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html) If you are testing on macOS, increase the open files limit\. The default limit is 256, which is enough for testing\.  | 
|  115  |  FileNotFoundError  |  A required file was not found\.  |  Verify the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  | 
|  116  |  OpenFileFailed  |  Unable to open the specified file\.  |  Verify the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html) If you are testing on macOS, increase the open files limit\. The default limit is 256, which is enough for testing\.  | 
|  117  |  WriteFileFailed  |  Failed to write file \(can be the DUT or test machine\)\.  |  Verify that the directory specified in the error message exists and that you have write permission\.    | 
|  118  |  FileCleanUpError  |  A test failed to remove the specified file or directory or to umount the specified file on the remote device\.  |  If the binary file is still running, the file might be locked\. End the process and delete the specified file\.  | 
|  119  |  InvalidInputError  |  Invalid configuration\.  |  Verify that your `suite.json` file is valid\.  | 
|  120  |  InvalidCredentialError  |  Invalid AWS credentials\.  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  | 
|  121  |  AWSSessionError  |  Failed to create an AWS session\.  |  This error can occur if AWS credentials are invalid or the internet connection is unstable\. Try using the AWS CLI to call an AWS API operation\.  | 
|  122  |  AWSApiCallError  |  An AWS API error occurred\.  |  This error might be due to a network issue\. Check your network before retrying the test group\.  | 
|  123  |  IpNotExistError  |   IP address is not included in connectivity information\.  |  Check your internet connection\. You can use the AWS IoT Greengrass console to check the connectivity information for the AWS IoT Greengrass core thing that is being used by the test\. If there are 10 endpoints included in the connectivity information, you can remove some or all of them and rerun the test\. For more information, see [Connectivity information](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-connectivity-info.html)\.  | 
|  124  |  OTAJobNotCompleteError  |  An OTA job did not complete\.  |  Check your internet connection and retry the OTA test group\.  | 
|  125  |  CreateGreengrassServiceRoleError  |  One of the following occurred: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  |  Configure the AWS IoT Greengrass service role\. For more information, see [Greengrass service role](service-role.md)\.  | 
|  126  |  DependenciesNotPresentError  |  One or more dependencies required for the specific test are not present on the device\.  |  Check the test log to see which dependencies are missing on your device: `<device-tester-extract-location>/results/<execution-id>/logs/<test-case-name.log>`  | 
|  127  |  InvalidHSMConfiguration  |  The provided HSM/PKCS configuration is incorrect\.  |  In your `device.json` file, provide the configuration required to interact with the HSM using PKCS\#11\.  | 
|  128  |  OTAJobNotSuccededError  |  The OTA job did not succeed\.  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  | 
|  129  |  NoConnectivityError  |  The host agent is failing to connect to internet\.  |  Check your network connection and firewall settings\. Retry the test group after the connectivity issue is resolved\.  | 
|  130  |  NoPermissionError  |  The IAM user you are using to run IDT for AWS IoT Greengrass does not have permission to create the AWS resources required to run IDT\.  |  See [Permissions policy template](https://docs.aws.amazon.com/greengrass/latest/developerguide/policy-template.html) for the policy template that grants the permissions required to run IDT for AWS IoT Greengrass\.  | 
|  131  |  LeftoverAgentExistError  |  Your device is running AWS IoT Greengrass processes when you attempt to start IDT for AWS IoT Greengrass\.   |  Make sure there is no existing Greengrass daemon running on your device\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/idt-troubleshooting.html)  If you are using an existing installation of AWS IoT Greengrass configured to start automatically after reboot, you must stop the daemon after reboot and before running the test suite\.   | 
|  132  |  DeviceTimeOffsetError  |  The device has the incorrect time\.  |  Set your device to the correct time\.  | 
|  133  |  InvalidMLConfiguration  |  The provided ML configuration is incorrect\.  |  In your `device.json` file, provide the correct configuration required to run ML inference tests\. For more information, see [Optional: Configuring your device for ML qualification](idt-ml-qualification.md)\.  | 

## Resolving IDT for AWS IoT Greengrass errors<a name="idt-gg-resolve-errors"></a>

When you use IDT, you must get the correct configuration files in place before you run IDT for AWS IoT Greengrass\. If you are getting parsing and configuration errors, your first step is to locate and use a configuration template appropriate for your environment\.

If you are still having issues, see the following debugging process\.

**Topics**
+ [Where do I look for errors?](#where-to-look)
+ [Parsing errors](#parse-error)
+ [Required parameter missing error](#param-missing)
+ [Could not start test error](#could-not-start-test)
+ [Not authorized to access resource error](#not-authorized-to-access-resource)
+ [Permission denied errors](#pwd-sudo)
+ [SSH connection errors](#ssh-connect-errors)
+ [Timeout errors](#test-timeout)
+ [Command not found errors while testing](#cmd-not-found)
+ [Security exception on macOS](#macos-notarization-exception)

### Where do I look for errors?<a name="where-to-look"></a>

High\-level errors are displayed on the console during execution, and a summary of the failed tests with the error is displayed when all tests are complete\. `awsiotdevicetester_report.xml` contains a summary of all the errors that caused a test to fail\. The log files for each test run are stored in a directory named with an UUID for the test execution that was displayed on the console during the test run\.

The test logs directory is located in `<device-tester-extract-location>/results/<execution-id>/logs/`\. This directory contains the following files, which are useful for debugging\.


| File | Description | 
| --- | --- | 
| test\_manager\.log |  All of the logs that were written to the console during the test execution\. A summary of the results is located at the end of this file, which includes a list of which tests failed\. The warning and error logs in this file can give you some information about the failures\.   | 
| <test\-group\-id>\_\_<test\-name>\.log | Detailed logs for the specific test\. | 
| <test\-name>\_ggc\_logs\.tar\.gz | A compressed collection of all the logs the AWS IoT Greengrass core daemon generated during the test\. For more information, see [Troubleshooting AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html)\. | 
| <test\-name>\_ota\_logs\.tar\.gz | A compressed collection of logs generated by the AWS IoT Greengrass OTA agent during the test\. For OTA tests only\. | 
| <test\-name>\_basic\_assertion\_publisher\_ggad\_logs\.tar\.gz | A compressed collection of logs generated by the AWS IoT publisher device during the test\. | 
| <test\-name>\_basic\_assertion\_subscriber\_ggad\_logs\.tar\.gz | A compressed collection of logs generated by the AWS IoT subscriber device during the test\. | 

### Parsing errors<a name="parse-error"></a>

Occasionally, a typo in a JSON configuration can lead to parsing errors\. Most of the time, the issue is a result of omitting a bracket, comma, or quotation mark from your JSON file\. IDT performs JSON validation and prints debugging information\. It prints the line where the error occurred, the line number, and the column number of the syntax error\. This information should be enough to help you fix the error, but if you still cannot locate the error, you can perform validation manually in your IDE, a text editor such as Atom or Sublime, or through an online tool like JSONLint\.

### Required parameter missing error<a name="param-missing"></a>

Because new features are being added to IDT, changes to the configuration files might be introduced\. Using an old configuration file might break your configuration\. If this happens, the `<test_case_id>.log` file under `/results/<execution-id>/logs` explicitly lists all missing parameters\. IDT also validates your JSON configuration file schemas to ensure that the latest supported version has been used\.

### Could not start test error<a name="could-not-start-test"></a>

You might encounter errors that point to failures during test start\. There are several possible causes, so do the following:
+ Make sure that the pool name you included in your execution command actually exists\. The pool name is referenced directly from your `device.json` file\.
+ Make sure that the devices in your pool have correct configuration parameters\.

### Not authorized to access resource error<a name="not-authorized-to-access-resource"></a>

You might see the `<user or role> is not authorized to access this resource` error message in the terminal output or in the `test_manager.log` file under `/results/<execution-id>/logs`\. To resolve this issue, attach the `AWSIoTDeviceTesterForGreengrassFullAccess` managed policy to your test user\. For more information, see [Create and configure an AWS account](dev-tst-prereqs.md#config-aws-account-for-idt)\.

### Permission denied errors<a name="pwd-sudo"></a>

IDT performs operations on various directories and files in a device under test\. Some of these operations require root access\. To automate these operations, IDT must be able to run commands with sudo without typing a password\. 

Follow these steps to allow sudo access without typing a password\.

**Note**  
`user` and `username` refer to the SSH user used by IDT to access the device under test\.

1. Use sudo usermod \-aG sudo *<ssh\-username>* to add your SSH user to the sudo group\.

1. Sign out and then sign in for changes to take effect\.

1. Open `/etc/sudoers` file and add the following line to the end of the file: `<ssh-username> ALL=(ALL) NOPASSWD: ALL`
**Note**  
As a best practice, we recommend that you use sudo visudo when you edit `/etc/sudoers`\.

### SSH connection errors<a name="ssh-connect-errors"></a>

When IDT cannot connect to a device under test, connection failures are logged in `/results/<execution-id>/logs/<test-case-id>.log`\. SSH failure messages appear at the top of this log file because connecting to a device under test is one of the first operations that IDT performs\.

Most Windows setups use the PuTTy terminal application to connect to Linux hosts\. This application requires that standard PEM private key files are converted into a proprietary Windows format called PPK\. When IDT is configured in your `device.json` file, use PEM files only\. If you use a PPK file, IDT cannot create an SSH connection with the AWS IoT Greengrass device and cannot run tests\.

### Timeout errors<a name="test-timeout"></a>

You can increase the timeout for each test by specifying a timeout multiplier, which is applied to the default value of each test's timeout\. Any value configured for this flag must be greater than or equal to 1\.0\.

To use the timeout multiplier, use the flag `--timeout-multiplier` when running the tests\. For example:

```
./devicetester_linux run-suite --suite-id GGQ_1.0.0 --pool-id DevicePool1 --timeout-multiplier 2.5
```

For more information, run `run-suite --help`\.

### Command not found errors while testing<a name="cmd-not-found"></a>

You need an older version of the OpenSSL library \(libssl1\.0\.0\) to run tests on AWS IoT Greengrass devices\. Most current Linux distributions use libssl version 1\.0\.2 or later \(v1\.1\.0\)\.

For example, on a Raspberry Pi, run the following commands to install the required version of libssl:

1. 

   ```
   wget http://ftp.us.debian.org/debian/pool/main/o/openssl/libssl1.0.0_1.0.2l-1~bpo8+1_armhf.deb
   ```

1. 

   ```
   sudo dpkg -i libssl1.0.0_1.0.2l-1~bpo8+1_armhf.deb
   ```

### Security exception on macOS<a name="macos-notarization-exception"></a>

When you run IDT on host machine that uses macOS 10\.15, the notarization ticket for IDT is not correctly detected and IDT is blocked from being run\. To run IDT, you will need to grant a security exception to the `devicetester_mac_x86-64` executable\. 

**To grant a security exception to the IDT executable**

1. Launch **System Preferences** from the Apple menu\.

1. Choose **Security & Privacy**, then on the **General** tab, click the lock icon to make changes to security settings\.

1. Look for the message `"devicetester_mac_x86-64" was blocked from use because it is not from an identified developer.` and choose **Allow Anyway**\.

1. Accept the security warning\.

If you have questions about the IDT support policy, contact [AWS Customer Support](https://aws.amazon.com/contact-us/)\.