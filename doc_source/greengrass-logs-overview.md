--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Monitoring with AWS IoT Greengrass logs<a name="greengrass-logs-overview"></a>

AWS IoT Greengrass consists of the cloud service and the AWS IoT Greengrass Core software\. The AWS IoT Greengrass Core software can write logs to Amazon CloudWatch and to the local file system of your core device\. Lambda functions and connectors running on the core can also write logs to CloudWatch Logs and the local file system\. You can use logs to monitor events and troubleshoot issues\. All AWS IoT Greengrass log entries include a timestamp, log level, and information about the event\. Changes to logging settings take effect after you deploy the group\.

Logging is configured at the group level\. For steps that show how to configure logging for a Greengrass group, see [Configure logging for AWS IoT Greengrass](#config-logs)\.

## Accessing CloudWatch Logs<a name="gg-logs-cloudwatch"></a>

If you configure CloudWatch logging, you can view the logs on the **Logs** page of the Amazon CloudWatch console\. Log groups for AWS IoT Greengrass logs use the following naming conventions:

```
/aws/greengrass/GreengrassSystem/greengrass-system-component-name
/aws/greengrass/Lambda/aws-region/account-id/lambda-function-name
```

Each log group contains log streams that use the following naming convention:

```
date/account-id/greengrass-group-id/name-of-core-that-generated-log
```

The following considerations apply when you use CloudWatch Logs:
+ Logs are sent to CloudWatch Logs with a limited number of retries in case there's no internet connectivity\. After the retries are exhausted, the event is dropped\.
+ Transaction, memory, and other limitations apply\. For more information, see [Logging limitations](#gg-log-limits)\.
+ <a name="gg-logs-cloudwatch-perms"></a>Your Greengrass group role must allow AWS IoT Greengrass to write to CloudWatch Logs\. To grant permissions, [ embed the following inline policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#embed-inline-policy-console) in your group role\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "logs:CreateLogGroup",
                  "logs:CreateLogStream",
                  "logs:PutLogEvents",
                  "logs:DescribeLogStreams"
              ],
              "Resource": [
                  "arn:aws:logs:*:*:*"
              ]
          }
      ]
  }
  ```
**Note**  
You can grant more granular access to your log resources\. For more information, see [ Using identity\-based policies \(IAM policies\) for CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/iam-identity-based-access-control-cwl.html) in the *Amazon CloudWatch User Guide*\.

  The group role is an IAM role that you create and attach to your Greengrass group\. You can use the console or the AWS IoT Greengrass API to manage the group role\.

     
**Using the console**  

  1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

  1. <a name="group-choose-target-group"></a>Choose the target group\.

  1. Choose **View settings**\. Under **Group role**, you can view, associate, or disassociate the group role\.

     For steps that show you how to attach the group role, see [group role](group-role.md)\.
   
**Using the CLI**  
  + To find the group role, use the [get\-associated\-role](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-associated-role.html) command\.
  + To attach the group role, use the [associate\-role\-to\-group](https://docs.aws.amazon.com/cli/latest/reference/greengrass/associate-role-to-group.html) command\.
  + To remove the group role, use the [disassociate\-role\-from\-group](https://docs.aws.amazon.com/cli/latest/reference/greengrass/disassociate-role-from-group.html) command\.
   
To learn how to get the group ID to use with these commands, see [Getting the group ID](deployments.md#api-get-group-id)\.

## Accessing file system logs<a name="gg-logs-local"></a>

If you configure file system logging, the log files are stored under `greengrass-root/ggc/var/log` on the core device\. The following is the high\-level directory structure:

```
greengrass-root/ggc/var/log
    - crash.log
    - system
        - log files for each Greengrass system component
    - user
        - region
            - account-id
                - log files generated by each user-defined Lambda function
            - aws
                - log files generated by each connector
```

**Note**  
By default, *greengrass\-root* is the `/greengrass` directory\. If a [write directory](gg-core.md#write-directory) is configured, then the logs are under that directory\.

The following considerations apply when you use file system logs:
+ Reading AWS IoT Greengrass logs on the file system requires root permissions\.
+ AWS IoT Greengrass supports size\-based rotation and automatic cleanup when the amount of log data is close to the configured limit\.
+ The `crash.log` file is available in file system logs only\. This log isn't written to CloudWatch Logs\.
+ Disk usage limitations apply\. For more information, see [Logging limitations](#gg-log-limits)\.

**Note**  
Logs for AWS IoT Greengrass Core software v1\.0 are stored under the `greengrass-root/var/log` directory\.

## Default logging configuration<a name="config-logs-default"></a>

If logging settings aren't explicitly configured, AWS IoT Greengrass uses the following default logging configuration after the first group deployment\.

AWS IoT Greengrass System Components  
+ Type \- `FileSystem`
+ Component \- `GreengrassSystem`
+ Level \- `INFO`
+ Space \- `128 KB`

User\-defined Lambda Functions  
+ Type \- `FileSystem`
+ Component \- `Lambda`
+ Level \- `INFO`
+ Space \- `128 KB`

**Note**  
Before the first deployment, only system components write logs to the file system because no user\-defined Lambda functions are deployed\.

## Configure logging for AWS IoT Greengrass<a name="config-logs"></a>

You can use the AWS IoT console or the [AWS IoT Greengrass APIs](#config-logs-api) to configure AWS IoT Greengrass logging\.

**Note**  
To allow AWS IoT Greengrass to write logs to CloudWatch Logs, your group role must allow the [required CloudWatch Logs actions](#gg-logs-cloudwatch-perms)\.

### Configure logging \(console\)<a name="config-logs-console"></a>

You can configure logging on the group's **Settings** page\.

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. Choose the group where you want to configure logging\.

1. On the group configuration page, choose the **Logs** tab\.

1. Choose the logging location, as follows:
   + To configure CloudWatch logging, for **CloudWatch logs configuration**, choose **Edit**\.
   + To configure file system logging, for **Local logs configuration**, choose **Edit**\.

   You can configure logging for one location or both locations\.

1. In the edit logs configuration modal, select **Greengrass system log level** or **User Lambda functions log level**\. You can choose one component or both components\.

1. Choose the lowest level of events that you want to log\. Events below this threshold are filtered out and aren't stored\.

1. Choose **Save**\. Changes take effect after you deploy the group\.

### Configure logging \(API\)<a name="config-logs-api"></a>

You can use AWS IoT Greengrass logger APIs to configure logging programmatically\. For example, use the [https://docs.aws.amazon.com/greengrass/latest/apireference/createloggerdefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createloggerdefinition-post.html) action to create a logger definition based on a [https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-loggerdefinitionversion.html](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-loggerdefinitionversion.html) payload, which uses the following syntax:

```
{
  "Loggers": [
    {
      "Id": "string",
      "Type": "FileSystem|AWSCloudWatch",
      "Component": "GreengrassSystem|Lambda",
      "Level": "DEBUG|INFO|WARN|ERROR|FATAL",
      "Space": "integer"
    },
    {
      "Id": "string",
      ...
    }
  ]
}
```

`LoggerDefinitionVersion` is an array of one or more [https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-logger.html](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-logger.html) objects that have the following properties:

`Id`  
An identifier for the logger\.

`Type`  
The storage mechanism for log events\. When `AWSCloudWatch` is used, log events are sent to CloudWatch Logs\. When `FileSystem` is used, log events are stored on the local file system\.  
Valid values: `AWSCloudWatch`, `FileSystem`

`Component`  
The source of the log event\. When `GreengrassSystem` is used, events from Greengrass system components are logged\. When `Lambda` is used, events from user\-defined Lambda functions are logged\.  
Valid values: `GreengrassSystem`, `Lambda`

`Level`  
The log\-level threshold\. Log events below this threshold are filtered out and aren't stored\.  
Valid values: `DEBUG`, `INFO` \(recommended\), `WARN`, `ERROR`, `FATAL`

`Space`  
The maximum amount of local storage, in KB, to use for storing logs\. This field applies only when `Type` is set to `FileSystem`\.

### Configuration example<a name="config-logs-examples"></a>

The following `LoggerDefinitionVersion` example specifies a logging configuration that:
+ Turns on file system `ERROR` and above logging for AWS IoT Greengrass system components\.
+ Turns on file system `INFO` \(and above\) logging for user\-defined Lambda functions\.
+ Turns on CloudWatch `INFO` \(and above\) logging for user\-defined Lambda functions\.

```
{
  "Name": "LoggingExample",
  "InitialVersion": {
    "Loggers": [
      {
        "Id": "1",
        "Component": "GreengrassSystem",
        "Level": "ERROR",
        "Space": 10240,
        "Type": "FileSystem"
      },
      {
        "Id": "2",
        "Component": "Lambda",
        "Level": "INFO",
        "Space": 10240,
        "Type": "FileSystem"
      },
      {
        "Id": "3",
        "Component": "Lambda",
        "Level": "INFO",
        "Type": "AWSCloudWatch"
      }
    ]
  }
}
```

After you create a logger definition version, you can use its version ARN to create a group version before [deploying the group](deployments.md)\.

## Logging limitations<a name="gg-log-limits"></a>

AWS IoT Greengrass has the following logging limitations\.

### Transactions per second<a name="gg-log-limit-tps"></a>

When logging to CloudWatch is enabled, the logging component batches log events locally before sending them to CloudWatch, so you can log at a rate higher than five requests per second per log stream\.

### Memory<a name="gg-log-limit-mem"></a>



If AWS IoT Greengrass is configured to send logs to CloudWatch and a Lambda function logs more than 5 MB/second for a prolonged period of time, the internal processing pipeline eventually fills up\. The theoretical worst case is 6 MB per Lambda function\. 

### Clock skew<a name="gg-log-limit-skew"></a>

When logging to CloudWatch is enabled, the logging component signs requests to CloudWatch using the normal Signature Version 4 signing process\. If the system time on the AWS IoT Greengrass core device is out of sync by more than [15 minutes](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html), then the requests are rejected\.

### Disk usage<a name="gg-log-limit-disk"></a>

Use the following formula to calculate the total maximum amount of disk usage for logging\. 

```
greengrass-system-component-space * 8    // 7 if automatic IP detection is disabled
  + 128KB                                // the internal log for the local logging component
  + lambda-space * lambda-count          // different versions of a Lambda function are treated as one
```

Where:

`greengrass-system-component-space`  
The maximum amount of local storage for the AWS IoT Greengrass system component logs\.

`lambda-space`  
The maximum amount of local storage for Lambda function logs\.

`lambda-count`  
The number of deployed Lambda functions\.

### Log loss<a name="gg-log-loss"></a>

If your AWS IoT Greengrass core device is configured to log only to CloudWatch and there's no internet connectivity, you have no way to retrieve the logs currently in the memory\.

When Lambda functions are terminated \(for example, during deployment\), a few seconds' worth of logs are not written to CloudWatch\.

## CloudTrail logs<a name="cloudtrail-integration"></a>

AWS IoT Greengrass runs with AWS CloudTrail, a service that provides a record of actions taken by a user, role, or an AWS service in AWS IoT Greengrass\. For more information, see [Logging AWS IoT Greengrass API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.