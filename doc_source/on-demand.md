--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Test on\-demand Lambda functions<a name="on-demand"></a>

An *[on\-demand](lambda-functions.md#lambda-lifecycle)* Lambda function is similar in functionality to a cloud\-based AWS Lambda function\. Multiple invocations of an on\-demand Lambda function can run in parallel\. An invocation of the Lambda function creates a separate container to process invocations or reuses an existing container, if resources permit\. Any variables or preprocessing that are defined outside of the function handler are not retained when containers are created\.

1. On the group configuration page, choose the **Lambda functions** tab\.

1. Under **My Lambda functions**, choose the **Greengrass\_HelloWorld\_Counter** Lambda function\.

1. On the **Greengrass\_HelloWorld\_Counter** details page, choose **Edit**\.

1. For **Pinned**, choose **False**, and then choose **Save**\.

1. <a name="console-actions-deploy"></a>On the group configuration page, choose **Deploy**\.

1. <a name="console-test-after-deploy"></a>After your deployment is complete, return to the AWS IoT console home page and choose **Test**\.

1. Configure the following fields:
   + For **Subscription topic**, enter **hello/world/counter**\.
   + For **Quality of Service**, choose **0**\.
   + For **MQTT payload display**, choose **Display payloads as strings**\.  
![\[Screenshot of Subscriptions test page.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-056.png)

1. Choose **Subscribe**\.
**Note**  
You should not see any messages after you subscribe\.

1. To test the on\-demand lifecycle, invoke the function by publishing a message to the `hello/world/counter/trigger` topic\. You can use the default message\.

   1. Choose **Publish** three times quickly, within five seconds of each press of the button\.  
![\[Screenshot showing the Publish to topic button, which must be clicked rapidly three times.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-063.png)

      Each publish invokes the function handler and creates a container for each invocation\. The invocation count is not incremented for the three times you triggered the function because each on\-demand Lambda function has its own container/sandbox\.  
![\[Screenshot showing Invocation Count fixed at 1.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-064.png)

   1. After approximately 30 seconds, choose **Publish to topic**\. The invocation count should be incremented to 2\. This shows that a container created from an earlier invocation is being reused, and that preprocessing variables outside of the function handler were stored\.  
![\[Screenshot showing Invocation Count now at 2.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-065.png)

You should now understand the two types of Lambda functions that can run on the AWS IoT Greengrass core\. The next module, [Module 4](module4.md), shows you how local IoT devices can interact in an AWS IoT Greengrass group\.