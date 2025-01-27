--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Use IDT to run the AWS IoT Greengrass qualification suite<a name="idt-gg-qualification"></a>

You can use AWS IoT Device Tester \(IDT\) for AWS IoT Greengrass to verify that the AWS IoT Greengrass Core software runs on your hardware and can communicate with the AWS Cloud\. It also performs end\-to\-end tests with AWS IoT Core\. For example, it verifies that your device can send and receive MQTT messages and process them correctly\. 

Because AWS IoT Greengrass Version 1 has been moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html), IDT for AWS IoT Greengrass V1 no longer generates signed qualification reports\. If you want to add your hardware to the AWS Partner Device Catalog, run the AWS IoT Greengrass V2 qualification suite to generate test reports that you can submit to AWS IoT\. For more information, see [AWS Device Qualification Program](https://aws.amazon.com/partners/dqp/) and [Supported versions of IDT for AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/dev-test-versions.html)\. 

In addition to testing devices, IDT for AWS IoT Greengrass creates resources \(for example, AWS IoT things, AWS IoT Greengrass groups, Lambda functions, and so on\) in your AWS account to facilitate the qualification process\.

<a name="idt-aws-credentials"></a>To create these resources, IDT for AWS IoT Greengrass uses the AWS credentials configured in the `config.json` file to make API calls on your behalf\. These resources are provisioned at various times during a test\.

When you use IDT for AWS IoT Greengrass to run the AWS IoT Greengrass qualification suite, IDT performs the following steps:

1. Loads and validates your device and credential configurations\.

1. Performs selected tests with the required local and cloud resources\.

1. Cleans up local and cloud resources\.

1. Generates tests reports that indicate if your device passed the tests required for qualification\.

## Test suite versions<a name="idt-test-suite-versions"></a>

IDT for AWS IoT Greengrass organizes tests into test suites and test groups\.<a name="idt-test-suites-groups"></a>
+ A test suite is the set of test groups used to verify that a device works with particular versions of AWS IoT Greengrass\.
+ A test group is the set of individual tests related to a particular feature, such as Greengrass group deployments and MQTT messaging\.

Starting in IDT v3\.0\.0, test suites are versioned using a `major.minor.patch` format, for example `GGQ_1.0.0`\. When you download IDT, the package includes the latest test suite version\.

**Important**  
IDT supports the three latest test suite versions for device qualification\. For more information, see [Support policy for AWS IoT Device Tester for AWS IoT Greengrass V1](idt-support-policy.md)\.  
You can run `list-supported-products` to list the versions of AWS IoT Greengrass and test suites that are supported by your current version of IDT\. Tests from unsupported test suite versions are not valid for device qualification\. IDT doesn't print qualification reports for unsupported versions\.

### Updates to IDT configuration settings<a name="idt-test-suite-versions-config-changes"></a>

New tests might introduce new IDT configuration settings\.
+ If the settings are optional, IDT continues running the tests\.
+ If the settings are required, IDT notifies you and stops running\. After you configure the settings, restart the test run\.

  Configuration settings are located in the `<device-tester-extract-location>/configs` folder\. For more information, see [Configure IDT settings to run the AWS IoT Greengrass qualification suite](set-config.md)\.

If an updated test suite version adds configuration settings, IDT creates a copy of the original configuration file in `<device-tester-extract-location>/configs`\.

## Test group descriptions<a name="dt-test-groups"></a>

------
#### [ IDT v2\.0\.0 and later ]

**Required Test Groups for Core Qualification**  
These test groups are required to qualify your AWS IoT Greengrass device for the AWS Partner Device Catalog\.    
AWS IoT Greengrass Core Dependencies  
Validates that your device meets all software and hardware requirements for the AWS IoT Greengrass Core software\.  
The `Software Packages Dependencies` test case in this test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Deployment  
Validates that Lambda functions can be deployed on your device\.  
MQTT  
Verifies the AWS IoT Greengrass message router functionality by checking local communication between the Greengrass core and client devices, which are local IoT devices\.  
Over\-the\-Air \(OTA\)  
Validates that your device can successfully perform an OTA update of the AWS IoT Greengrass Core software\.  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Version  
Checks that the version of AWS IoT Greengrass provided is compatible with the AWS IoT Device Tester version you are using\.

**Optional Test Groups**  
These test groups are optional\. If you choose to qualify for optional tests, your device is listed with additional capabilities in the AWS Partner Device Catalog\.    
Container Dependencies  
<a name="description-container"></a>Validates that the device meets all of the software and hardware requirements to run Lambda functions in container mode on a Greengrass core\.  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Deployment Container  
<a name="description-deployment-container"></a>Validates that Lambda functions can be deployed on the device and run in container mode on a Greengrass core\.  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Docker Dependencies \(Supported for IDT v2\.2\.0 and later\)  
<a name="description-docker"></a>Validates that the device meets all the required technical dependencies to use the Greengrass Docker application deployment connector to run containers  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Hardware Security Integration \(HSI\)  
<a name="description-hsi"></a>Verifies that the provided HSI shared library can interface with the hardware security module \(HSM\) and implements the required PKCS\#11 APIs correctly\. The HSM and shared library must be able to sign a CSR, perform TLS operations, and provide the correct key lengths and public key algorithm\.  
Stream Manager Dependencies \(Supported for IDT v2\.2\.0 and later\)  
<a name="description-sm"></a>Validates that the device meets all of the required technical dependencies to run AWS IoT Greengrass stream manager\.  
Machine Learning Dependencies \(Supported for IDT v3\.1\.0 and later\)  
<a name="description-ml"></a>Validates that the device meets all of the required technical dependencies to perform ML inference locally\.  
Machine Learning Inference Tests \(Supported for IDT v3\.1\.0 and later\)  
<a name="description-mlit"></a>Validates that ML inference can be performed on the given device under test\. For more information, see [Optional: Configuring your device for ML qualification](idt-ml-qualification.md)\.  
Machine Learning Inference Container Tests \(Supported for IDT v3\.1\.0 and later\)  
<a name="description-mlict"></a>Validates that ML inference can be performed on the given device under test and run in container mode on a Greengrass core\. For more information, see [Optional: Configuring your device for ML qualification](idt-ml-qualification.md)\.

------
#### [ IDT v1\.3\.3 and earlier ]

**Required Test Groups for Core Qualification**  
These tests are required to qualify your AWS IoT Greengrass device for the AWS Partner Device Catalog\.    
AWS IoT Greengrass Core Dependencies  
Validates that your device meets all software and hardware requirements for the AWS IoT Greengrass Core software\.  
Combination \(Device Security Interaction\)  
Verifies the functionality of the device certificate manager and IP detection on the Greengrass core device by changing connectivity information on the Greengrass group in the cloud\. The test group rotates the AWS IoT Greengrass server certificate and verifies that AWS IoT Greengrass allows connections\.  
Deployment \(Required for IDT v1\.2 and earlier\)  
Validates that Lambda functions can be deployed on your device\.  
Device Certificate Manager \(DCM\)  
Verifies that the AWS IoT Greengrass device certificate manager can generate a server certificate on startup and rotate certificates if they are close to expiration\.  
IP Detection \(IPD\)  
Verifies that core connectivity information is updated when there are IP address changes in a Greengrass core device\. For more information, see [Activate automatic IP detection](gg-core.md#ip-auto-detect)\.  
Logging  
Verifies that the AWS IoT Greengrass logging service can write to a log file using a user Lambda function written in Python\.  
MQTT  
Verifies the AWS IoT Greengrass message router functionality by sending messages on a topic that is routed to two Lambda functions\.   
Native  
Verifies that AWS IoT Greengrass can run native \(compiled\) Lambda functions\.  
Over\-the\-Air \(OTA\)  
Validates that your device can successfully perform a OTA update of the AWS IoT Greengrass Core software\.  
Penetration  
Validates that the AWS IoT Greengrass Core software fails to start if hard link/soft link protection and [seccomp](https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt) are not enabled\. It is also used to verify other security\-related features\.  
Shadow  
Verifies local shadow and shadow cloud\-syncing functionality\.  
Spooler  
Validates that the MQTT messages are queued with the default spooler configuration\.  
Token Exchange Service \(TES\)  
Verifies that AWS IoT Greengrass can exchange its core certificate for valid AWS credentials\.  
Version  
Checks that the version of AWS IoT Greengrass provided is compatible with the AWS IoT Device Tester version you are using\.

**Optional Test Groups**  
These tests are optional\. If you choose to qualify for optional tests, your device is listed with additional capabilities in the AWS Partner Device Catalog\.    
Container Dependencies  
Checks that the device meets all of the required dependencies to run Lambda functions in container mode\.  
Hardware Security Integration \(HSI\)  
Verifies that the provided HSI shared library can interface with the hardware security module \(HSM\) and implements the required PKCS\#11 APIs correctly\. The HSM and shared library must be able to sign a CSR, perform TLS operations, and provide the correct key lengths and public key algorithm\.  
Local Resource Access  
Verifies the local resource access \(LRA\) feature of AWS IoT Greengrass by providing access to local files and directories owned by various Linux users and groups to containerized Lambda functions through AWS IoT Greengrass LRA APIs\. Lambda functions should be allowed or denied access to local resources based on local resource access configuration\.  
Network  
Verifies that socket connections can be established from a Lambda function\. These socket connections should be allowed or denied based on the Greengrass core configuration\.

------