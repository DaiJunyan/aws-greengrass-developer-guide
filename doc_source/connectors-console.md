--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Getting started with Greengrass connectors \(console\)<a name="connectors-console"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

This tutorial shows how to use the AWS Management Console to work with connectors\.

Use connectors to accelerate your development life cycle\. Connectors are prebuilt, reusable modules that can make it easier to interact with services, protocols, and resources\. They can help you deploy business logic to Greengrass devices more quickly\. For more information, see [Integrate with services and protocols using Greengrass connectors](connectors.md)\.

In this tutorial, you configure and deploy the [ Twilio Notifications](twilio-notifications-connector.md) connector\. The connector receives Twilio message information as input data, and then triggers a Twilio text message\. The data flow is shown in following diagram\.

![\[Data flow from Lambda function to Twilio Notifications connector to Twilio.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/connectors/twilio-solution.png)

After you configure the connector, you create a Lambda function and a subscription\.
+ The function evaluates simulated data from a temperature sensor\. It conditionally publishes the Twilio message information to an MQTT topic\. This is the topic that the connector subscribes to\.
+ The subscription allows the function to publish to the topic and the connector to receive data from the topic\.

The Twilio Notifications connector requires a Twilio auth token to interact with the Twilio API\. The token is a text type secret created in AWS Secrets Manager and referenced from a group resource\. This enables AWS IoT Greengrass to create a local copy of the secret on the Greengrass core, where it is encrypted and made available to the connector\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

The tutorial contains the following high\-level steps:

1. [Create a Secrets Manager secret](#connectors-console-create-secret)

1. [Add a secret resource to a group](#connectors-console-create-resource)

1. [Add a connector to the group](#connectors-console-create-connector)

1. [Create a Lambda function deployment package](#connectors-console-create-deployment-package)

1. [Create a Lambda function](#connectors-console-create-function)

1. [Add a function to the group](#connectors-console-create-gg-function)

1. [Add subscriptions to the group](#connectors-console-create-subscription)

1. [Deploy the group](#connectors-console-create-deployment)

1. [Test the solution](#connectors-console-test-solution)

The tutorial should take about 20 minutes to complete\.

## Prerequisites<a name="connectors-console-prerequisites"></a>

To complete this tutorial, you need:
+ A Greengrass group and a Greengrass core \(v1\.9\.3 or later\)\. To learn how to create a Greengrass group and core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\. The Getting Started tutorial also includes steps for installing the AWS IoT Greengrass Core software\.
+ Python 3\.7 installed on the AWS IoT Greengrass core device\.
+  AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ A Twilio account SID, auth token, and Twilio\-enabled phone number\. After you create a Twilio project, these values are available on the project dashboard\.
**Note**  
You can use a Twilio trial account\. If you're using a trial account, you must add non\-Twilio recipient phone numbers to a list of verified phone numbers\. For more information, see [ How to Work with your Free Twilio Trial Account](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account)\.

## Step 1: Create a Secrets Manager secret<a name="connectors-console-create-secret"></a>

In this step, you use the AWS Secrets Manager console to create a text type secret for your Twilio auth token\.

1. <a name="create-secret-step-signin"></a>Sign in to the [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/)\.
**Note**  
For more information about this process, see [ Step 1: Create and store your secret in AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/tutorials_basic.html) in the *AWS Secrets Manager User Guide*\.

1. <a name="create-secret-step-create"></a>Choose **Store a new secret**\.

1. <a name="create-secret-step-othertype"></a>Under **Choose secret type**, choose **Other type of secret**\.

1. Under **Specify the key/value pairs to be stored for this secret**, on the **Plaintext** tab, enter your Twilio auth token\. Remove all of the JSON formatting and enter only the token value\.

1. <a name="create-secret-step-encryption"></a>Keep **aws/secretsmanager** selected for the encryption key, and then choose **Next**\.
**Note**  
You aren't charged by AWS KMS if you use the default AWS managed key that Secrets Manager creates in your account\.

1. For **Secret name**, enter **greengrass\-TwilioAuthToken**, and then choose **Next**\.
**Note**  
By default, the Greengrass service role allows AWS IoT Greengrass to get the value of secrets with names that start with *greengrass\-*\. For more information, see [secrets requirements](secrets.md#secrets-reqs)\.

1. <a name="create-secret-step-rotation"></a>This tutorial doesn't require rotation, so choose disable automatic rotation, and then choose **Next**\.

1. <a name="create-secret-step-review"></a>On the **Review** page, review your settings, and then choose **Store**\.

   Next, you create a secret resource in your Greengrass group that references the secret\.

## Step 2: Add a secret resource to a Greengrass group<a name="connectors-console-create-resource"></a>

In this step, you add a *secret resource* to the Greengrass group\. This resource is a reference to the secret that you created in the previous step\.

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. <a name="create-secret-resource-step-choosegroup"></a>Choose the group that you want to add the secret resource to\.

1. <a name="create-secret-resource-step-secretstab"></a>On the group configuration page, choose the **Resources** tab, and then scroll down to the **Secrets** section\. The **Secrets** section displays the secret resources that belong to the group\. You can add, edit, and remove secret resources from this section\.
**Note**  
Alternatively, the console allows you to create a secret and secret resource when you configure a connector or Lambda function\. You can do this from the connector's **Configure parameters** page or the Lambda function's **Resources** page\.

1. <a name="create-secret-resource-step-addsecretresource"></a>Choose **Add** under the **Secrets** section\.

1. On the **Add a secret resource** page, enter **MyTwilioAuthToken** for the **Resource name**\.

1. For the **Secret**, choose **greengrass\-TwilioAuthToken**\.

1. <a name="create-secret-resource-step-selectlabels"></a>In the **Select labels \(Optional\)** section, the AWSCURRENT staging label represents the latest version of the secret\. This label is always included in a secret resource\.
**Note**  
This tutorial requires the AWSCURRENT label only\. You can optionally include labels that are required by your Lambda function or connector\.

1. Choose **Add resource**\.

## Step 3: Add a connector to the Greengrass group<a name="connectors-console-create-connector"></a>

In this step, you configure parameters for the [Twilio Notifications connector](twilio-notifications-connector.md) and add it to the group\.

1. On the group configuration page, choose **Connectors**, and then choose **Add a connector**\.

1. On the **Add connector** page, choose **Twilio Notifications**\.

1. Choose the version\.

1. In the **Configuration** section:
   + For **Twilio auth token resource**, enter the resource that you created in the previous step\.
**Note**  
When you enter the resource, the **ARN of Twilio auth token secret** property is populated for you\.
   + For **Default from phone number**, enter your Twilio\-enabled phone number\.
   + For **Twilio account SID**, enter your Twilio account SID\.

1. Choose **Add resource**\.

## Step 4: Create a Lambda function deployment package<a name="connectors-console-create-deployment-package"></a>

To create a Lambda function, you must first create a Lambda function *deployment package* that contains the function code and dependencies\. Greengrass Lambda functions require the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) for tasks such as communicating with MQTT messages in the core environment and accessing local secrets\. This tutorial creates a Python function, so you use the Python version of the SDK in the deployment package\.

1. <a name="download-ggc-sdk"></a> From the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads page, download the AWS IoT Greengrass Core SDK for Python to your computer\.

1. <a name="unzip-ggc-sdk"></a>Unzip the downloaded package to get the SDK\. The SDK is the `greengrasssdk` folder\.

1. Save the following Python code function in a local file named `temp_monitor.py`\.

   ```
   import greengrasssdk
   import json
   import random
   
   client = greengrasssdk.client('iot-data')
   
   # publish to the Twilio Notifications connector through the twilio/txt topic
   def function_handler(event, context):
       temp = event['temperature']
       
       # check the temperature
       # if greater than 30C, send a notification
       if temp > 30:
           data = build_request(event)
           client.publish(topic='twilio/txt', payload=json.dumps(data))
           print('published:' + str(data))
           
       print('temperature:' + str(temp))
       return
   
   # build the Twilio request from the input data
   def build_request(event):
       to_name = event['to_name']
       to_number = event['to_number']
       temp_report = 'temperature:' + str(event['temperature'])
   
       return {
           "request": {
               "recipient": {
                   "name": to_name,
                   "phone_number": to_number,
                   "message": temp_report
               }
           },
           "id": "request_" + str(random.randint(1,101))
       }
   ```

1. Zip the following items into a file named `temp_monitor_python.zip`\. When creating the ZIP file, include only the code and dependencies, not the containing folder\.
   + **temp\_monitor\.py**\. App logic\.
   + **greengrasssdk**\. Required library for Python Greengrass Lambda functions that publish MQTT messages\.

   This is your Lambda function deployment package\.

Now, create a Lambda function that uses the deployment package\.

## Step 5: Create a Lambda function in the AWS Lambda console<a name="connectors-console-create-function"></a>

In this step, you use the AWS Lambda console to create a Lambda function and configure it to use your deployment package\. Then, you publish a function version and create an alias\.

1. First, create the Lambda function\.

   1. <a name="lambda-console-open"></a>In the AWS Management Console, choose **Services**, and open the AWS Lambda console\.

   1. <a name="lambda-console-create-function"></a>Choose **Create function** and then choose **Author from scratch**\.

   1. In the **Basic information** section, use the following values:
      + For **Function name**, enter **TempMonitor**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

   1. <a name="lambda-console-save-function"></a>At the bottom of the page, choose **Create function**\.

1. Next, register the handler and upload your Lambda function deployment package\.

   1. <a name="lambda-console-upload"></a>On the **Code** tab, under **Code source**, choose **Upload from**\. From the dropdown, choose **\.zip file**\.  
![\[The Upload from dropdown with .zip file highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/lra-console/upload-deployment-package.png)

   1. Choose **Upload**, and then choose your `temp_monitor_python.zip` deployment package\. Then, choose **Save**\.

   1. <a name="lambda-console-runtime-settings-para"></a>On the **Code** tab for the function, under **Runtime settings**, choose **Edit**, and then enter the following values\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **temp\_monitor\.function\_handler**

   1. <a name="lambda-console-save-config"></a>Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. Now, publish the first version of your Lambda function and create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

   1. <a name="shared-publish-function-version"></a>From the **Actions** menu, choose **Publish new version**\.

   1. <a name="shared-publish-function-version-description"></a>For **Version description**, enter **First version**, and then choose **Publish**\.

   1. On the **TempMonitor: 1** configuration page, from the **Actions** menu, choose **Create alias**\.

   1. On the **Create a new alias** page, use the following values:
      + For **Name**, enter **GG\_TempMonitor**\.
      + For **Version**, choose **1**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

   1. Choose **Create**\.

Now you're ready to add the Lambda function to your Greengrass group\.

## Step 6: Add a Lambda function to the Greengrass group<a name="connectors-console-create-gg-function"></a>

In this step, you add the Lambda function to the group and then configure its lifecycle and environment variables\. For more information, see [Controlling execution of Greengrass Lambda functions by using group\-specific configuration](lambda-group-config.md)\.

1. <a name="choose-add-lambda"></a>On the group configuration page, choose the **Lambda functions** tab\.

1. Under **My Lambda functions**, choose **Add**\.

1. On the **Add Lambda function** page, choose **TempMonitor** for your Lambda function\.

1. For **Lambda function version**, choose **Alias: GG\_TempMonitor**\.

1. Choose **Add Lambda function**\.

## Step 7: Add subscriptions to the Greengrass group<a name="connectors-console-create-subscription"></a>

<a name="connectors-how-to-add-subscriptions-p1"></a>In this step, you add a subscription that enables the Lambda function to send input data to the connector\. The connector defines the MQTT topics that it subscribes to, so this subscription uses one of the topics\. This is the same topic that the example function publishes to\.

<a name="connectors-how-to-add-subscriptions-p2"></a>For this tutorial, you also create subscriptions that allow the function to receive simulated temperature readings from AWS IoT and allow AWS IoT to receive status information from the connector\.

1. <a name="shared-subscriptions-addsubscription"></a>On the group configuration page, choose the **Subscriptions** tab, and then choose **Add Subscription**\.

1. On the **Create a subscription** page, configure the source and target, as follows:

   1. For **Source type**, choose **Lambda function**, and then choose **TempMonitor**\.

   1. For **Target type**, choose **Connector**, and then choose **Twilio Notifications**\.

1. For the **Topic filter**, choose **twilio/txt**\.

1. Choose **Create subscription**\.

1. Repeat steps 1 \- 4 to create a subscription that allows AWS IoT to publish messages to the function\.

   1. For **Source type**, choose **Service**, and then choose **IoT Cloud**\.

   1. For **Select a target**, choose **Lambda function**, and then choose **TempMonitor**\.

   1. For **Topic filter**, enter **temperature/input**\.

1. Repeat steps 1 \- 4 to create a subscription that allows the connector to publish messages to AWS IoT\.

   1. For **Source type**, choose **Connector**, and then choose **Twilio Notifications**\.

   1. For **Target type**, choose **Service**, and then choose **IoT Cloud**\.

   1. For **Topic filter**, **twilio/message/status** is entered for you\. This is the predefined topic that the connector publishes to\.

## Step 8: Deploy the Greengrass group<a name="connectors-console-create-deployment"></a>

Deploy the group to the core device\.

1. <a name="shared-deploy-group-checkggc"></a>Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/ggc-version/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. <a name="shared-deploy-group-deploy"></a>On the group configuration page, choose **Deploy**\.

1. <a name="shared-deploy-group-ipconfig"></a>

   1. In the **Lambda functions** tab, under the **System Lambda functions** section, select **IP detector** and choose **Edit**\.

   1. In the **Edit IP detector settings** dialog box, select ** Automatically detect and override MQTT broker endpoints**\.

   1. Choose **Save**\.

      This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.
**Note**  
If prompted, grant permission to create the [Greengrass service role](service-role.md) and associate it with your AWS account in the current AWS Region\. This role allows AWS IoT Greengrass to access your resources in AWS services\.

      The **Deployments** page shows the deployment timestamp, version ID, and status\. When completed, the status displayed for the deployment should be **Completed**\.

      For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

**Note**  
<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## Test the solution<a name="connectors-console-test-solution"></a>

1. <a name="choose-test-page"></a>On the AWS IoT console home page, choose **Test**\.

1. For **Subscribe to topic**, use the following values, and then choose **Subscribe**\. The Twilio Notifications connector publishes status information to this topic\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-console.html)

1. For **Publish to topic**, use the following values, and then choose **Publish** to invoke the function\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-console.html)

   If successful, the recipient receives the text message and the console displays the `success` status from the [output data](twilio-notifications-connector.md#twilio-notifications-connector-data-output)\.

   Now, change the `temperature` in the input message to **29** and publish\. Because this is less than 30, the TempMonitor function doesn't trigger a Twilio message\.

## See also<a name="connectors-console-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+  [AWS\-provided Greengrass connectors](connectors-list.md)