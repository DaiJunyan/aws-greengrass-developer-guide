--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Encryption in transit<a name="encryption-in-transit"></a>

AWS IoT Greengrass has three modes of communication where data is in transit:
+ [Data in transit over the internet](#data-in-transit-internet)\. Communication between a Greengrass core and AWS IoT Greengrass over the internet is encrypted\.
+ [Data in transit over the local network](#data-in-transit-local-network)\. Communication between a Greengrass core and client devices over a local network is encrypted\.
+ [Data on the core device](#data-in-transit-locally)\. Communication between components on the Greengrass core device is not encrypted\.

## Data in transit over the internet<a name="data-in-transit-internet"></a>

AWS IoT Greengrass uses Transport Layer Security \(TLS\) to encrypt all communication over the internet\. All data sent to the AWS Cloud is sent over a TLS connection using MQTT or HTTPS protocols, so it is secure by default\. AWS IoT Greengrass uses the AWS IoT transport security model\. For more information, see [Transport security](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html) in the *AWS IoT Core Developer Guide*\.

## Data in transit over the local network<a name="data-in-transit-local-network"></a>

AWS IoT Greengrass uses TLS to encrypt all communication over the local network between the Greengrass core and client devices\. For more information, see [Supported Cipher Suites for Local Network Communication](gg-sec.md#gg-cipher-suites)\.

It is your responsibility to protect the local network and private keys\.<a name="customer-responsibility-device-security"></a>

For Greengrass core devices, it's your responsibility to:  
+ Keep the kernel updated with the latest security patches\.
+ Keep system libraries updated with the latest security patches\.
+ Protect private keys\. For more information, see [Key management for the Greengrass core device](key-management.md)\.

For client devices, it's your responsibility to:  
+ Keep the TLS stack up to date\.
+ Protect private keys\.

## Data on the core device<a name="data-in-transit-locally"></a>

AWS IoT Greengrass doesn't encrypt data exchanged locally on the Greengrass core device because the data doesn't leave the device\. This includes communication between user\-defined Lambda functions, connectors, the AWS IoT Greengrass Core SDK, and system components, such as stream manager\.