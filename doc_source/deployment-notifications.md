--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Get deployment notifications<a name="deployment-notifications"></a>

Amazon EventBridge event rules provide you with notifications about state changes for your Greengrass group deployments\. EventBridge delivers a near real\-time stream of system events that describes changes in AWS resources\. AWS IoT Greengrass sends these events to EventBridge on an *at least once* basis\. This means that AWS IoT Greengrass might send multiple copies of a given event to ensure delivery\. Additionally, your event listeners might not receive the events in the order that the events occurred\.

**Note**  
Amazon EventBridge is an event bus service that you can use to connect your applications with data from a variety of sources, such as [Greengrass core devices](telemetry.md) and deployment notifications\. For more information, see [What is Amazon EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html) in the *Amazon EventBridge User Guide*\.

AWS IoT Greengrass emits an event when group deployments change state\. You can create an EventBridge rule that runs for all state transitions or transitions to states you specify\. When a deployment enters a state that initiates a rule, EventBridge invokes the target actions defined in the rule\. This allows you to send notifications, capture event information, take corrective action, or initiate other events in response to a state change\. For example, you can create rules for the following use cases:
+ Initiate post\-deployment operations, such as downloading assets and notifying personnel\.
+ Send notifications upon a successful or failed deployment\.
+ Publish custom metrics about deployment events\.

AWS IoT Greengrass emits an event when a deployment enters the following states: `Building`, `InProgress`, `Success`, and `Failure`\.

**Note**  
Monitoring the status of a [bulk deployment](bulk-deploy-cli.md) operation is not currently supported\. However, AWS IoT Greengrass emits state\-change events for individual group deployments that are part of a bulk deployment\.

## Group deployment status change event<a name="events-message-format"></a>

The [event](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/CloudWatchEventsandEventPatterns.html) for a deployment state change uses the following format:

```
{
    "version":"0",
    "id":" cd4d811e-ab12-322b-8255-EXAMPLEb1bc8",
    "detail-type":"Greengrass Deployment Status Change",
    "source":"aws.greengrass",
    "account":"123456789012",
    "time":"2018-03-22T00:38:11Z",
    "region":"us-west-2",
    "resources":[],
    "detail":{    
        "group-id": "284dcd4e-24bc-4c8c-a770-EXAMPLEf03b8",
        "deployment-id": "4f38f1a7-3dd0-42a1-af48-EXAMPLE09681",
        "deployment-type": "NewDeployment|Redeployment|ResetDeployment|ForceResetDeployment",
        "status": "Building|InProgress|Success|Failure"
    }
}
```

You can create rules that apply to one or more groups\. You can filter rules by one or more of the following deployment types and deployment states:

**Deployment types**  
+ `NewDeployment`\. The first deployment of a group version\.
+ `ReDeployment`\. A redeployment of a group version\.
+ `ResetDeployment`\. Deletes deployment information stored in the AWS Cloud and on the AWS IoT Greengrass core\. For more information, see [Reset deployments](reset-deployments-scenario.md)\.
+ `ForceResetDeployment`\. Deletes deployment information stored in the AWS Cloud and reports success without waiting for the core to respond\. Also deletes deployment information stored on the core if the core is connected or when it next connects\.

**Deployment states**  
+ `Building`\. AWS IoT Greengrass is validating the group configuration and building deployment artifacts\.
+ `InProgress`\. The deployment is in progress on the AWS IoT Greengrass core\.
+ `Success`\. The deployment was successful\.
+ `Failure`\. The deployment failed\.

It's possible that events might be duplicated or out of order\. To determine the order of events, use the `time` property\.

**Note**  
AWS IoT Greengrass doesn't use the `resources` property, so it's always empty\.

## Prerequisites for creating EventBridge rules<a name="create-events-rule-prereqs"></a>

Before you create an EventBridge rule for AWS IoT Greengrass, do the following:
+ Familiarize yourself with events, rules, and targets in EventBridge\.
+ Create and configure the targets invoked by your EventBridge rules\. Rules can invoke many types of targets, including:
  + Amazon Simple Notification Service \(Amazon SNS\)
  + AWS Lambda functions
  + Amazon Kinesis Video Streams
  + Amazon Simple Queue Service \(Amazon SQS\) queues

For more information, see [What is Amazon EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html) and [Getting started with Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-getting-set-up.html) in the *Amazon EventBridge User Guide*\.

## Configure deployment notifications \(console\)<a name="create-events-rule-console"></a>

Use the following steps to create an EventBridge rule that publishes an Amazon SNS topic when the deployment state changes for a group\. This allows web servers, email addresses, and other topic subscribers to respond to the event\. For more information, see [Creating a EventBridge rule that triggers on an event from an AWS resource](https://docs.aws.amazon.com/eventbridge/latest/userguide/create-eventbridge-rule.html) in the *Amazon EventBridge User Guide*\.

1. Open the [Amazon EventBridge console](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, choose **Rules**\.

1. Choose **Create rule**\.

1. Enter a name and description for the rule\.

   A rule can't have the same name as another rule in the same Region and on the same event bus\.

1. For **Event bus**, choose the event bus that you want to associate with this rule\. If you want this rule to match events that come from your account, select **AWS default event bus**\. When an AWS service in your account emits an event, it always goes to your account's default event bus\.

1. For **Rule type**, choose **Rule with an event pattern**\.

1. Choose **Next**\.

1. For **Event source**, choose **AWS services**\.

1. For **Event pattern**, choose **AWS services**\.

1. For **AWS service**, choose Greengrass\.

1. For **Event type**, choose **Greengrass Deployment Status Change**\.
**Note**  
The **AWS API Call via CloudTrail** event type is based on AWS IoT Greengrass integration with AWS CloudTrail\. You can use this option to create rules initiated by read or write calls to the AWS IoT Greengrass API\. For more information, see [Logging AWS IoT Greengrass API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

1. Choose the deployment states that initiate a notification\.
   + To receive notifications for all state change events, choose **Any state**\.
   + To receive notifications for some state change events only, choose **Specific state\(s\)**, and then choose the target states\.

1. Choose the deployment types that initiate a notification\.
   + To receive notifications for all deployment types, choose **Any state**\.
   + To receive notifications for some deployment types only, choose **Specific state\(s\)**, and then choose the target deployment types\.

1. Choose **Next**\.

1. For **Target types**, choose **AWS service**\.

1. For **Select a target**, configure your target\. This example uses an Amazon SNS topic, but you can configure other target types to send notifications\.

   1. For **Target**, choose **SNS topic**\.

   1. For **Topic**, choose your target topic\.

   1. Choose **Next**\.

1. Under **Tags**, define tags for the rule or leave the fields empty\.

1. Choose **Next**\.

1. Review the details of the rule and choose **Create rule**\.

## Configure deployment notifications \(CLI\)<a name="create-events-rule-cli"></a>

Use the following steps to create an EventBridge rule that publishes an Amazon SNS topic when the deployment state changes for a group\. This allows web servers, email addresses, and other topic subscribers to respond to the event\.

1. Create the rule\.
   + Replace *group\-id* with the ID of your AWS IoT Greengrass group\.

   ```
   aws events put-rule \
     --name TestRule \
     --event-pattern "{\"source\": [\"aws.greengrass\"], \"detail\": {\"group-id\": [\"group-id\"]}}"
   ```

   Properties that are omitted from the pattern are ignored\.

1. Add the topic as a rule target\.
   + Replace *topic\-arn* with the ARN of your Amazon SNS topic\.

   ```
   aws events put-targets \
     --rule TestRule \
     --targets "Id"="1","Arn"="topic-arn"
   ```
**Note**  
To allow Amazon EventBridge to call your target topic, you must add a resource\-based policy to your topic\. For more information, see [Amazon SNS permissions](https://docs.aws.amazon.com/eventbridge/latest/userguide/resource-based-policies-eventbridge.html#sns-permissions) in the *Amazon EventBridge User Guide*\.

For more information, see [Events and event patterns in EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-and-event-patterns.html) in the *Amazon EventBridge User Guide*\.

## Configure deployment notifications \(AWS CloudFormation\)<a name="create-events-rule-cloudformation"></a>

Use AWS CloudFormation templates to create EventBridge rules that send notifications about state changes for your Greengrass group deployments\. For more information, see [Amazon EventBridge resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_Events.html) in the *AWS CloudFormation User Guide*\.

## See also<a name="deployment-notifications-see-also"></a>
+ [Deploy AWS IoT Greengrass groups to an AWS IoT Greengrass core](deployments.md)
+ [What is Amazon EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html) in the *Amazon EventBridge User Guide*