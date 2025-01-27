--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Resilience in AWS IoT Greengrass<a name="disaster-recovery-resiliency"></a>

The AWS global infrastructure is built around Amazon Web Services Regions and Availability Zones\. Each AWS Region provides multiple physically separated and isolated Availability Zones, which are connected with low\-latency, high\-throughput, and highly redundant networking\. With Availability Zones, you can design and operate applications and databases that automatically fail over between zones without interruption\. Availability Zones are more highly available, fault tolerant, and scalable than traditional single or multiple data center infrastructures\. 

For more information about Amazon Web Services Regions and Availability Zones, see [AWS Global Infrastructure](http://aws.amazon.com/about-aws/global-infrastructure/)\.

In addition to the AWS global infrastructure, AWS IoT Greengrass offers several features to help support your data resiliency and backup needs\.
+ If the core loses internet connectivity, client devices can continue to communicate over the local network\.
+ You can configure the core to store unprocessed messages destined for AWS Cloud targets in a local storage cache instead of in\-memory storage\. The local storage cache can persist across core restarts \(for example, after a group deployment or a device reboot\), so AWS IoT Greengrass can continue to process messages destined for AWS IoT Core\. For more information, see [MQTT message queue for cloud targets](gg-core.md#mqtt-message-queue)\.
+ You can configure the core to establish a persistent session with the AWS IoT Core message broker\. This allows the core to receive messages sent while the core is offline\. For more information, see [MQTT persistent sessions with AWS IoT Core](gg-core.md#mqtt-persistent-sessions)\.
+ You can configure a Greengrass group to write logs to the local file system and to CloudWatch Logs\. If the core loses connectivity, local logging can continue, but CloudWatch logs are sent with a limited number of retries\. After the retries are exhausted, the event is dropped\. You should also be aware of [logging limitations](greengrass-logs-overview.md#gg-log-limits)\.
+ You can author Lambda functions that read [stream manager](stream-manager.md) streams and send the data to local storage destinations\.