--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# How to create a secret resource \(console\)<a name="secrets-console"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

This tutorial shows how to use the AWS Management Console to add a *secret resource* to a Greengrass group\. A secret resource is a reference to a secret from AWS Secrets Manager\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

On the AWS IoT Greengrass core device, connectors and Lambda functions can use the secret resource to authenticate with services and applications, without hard\-coding passwords, tokens, or other credentials\.

In this tutorial, you start by creating a secret in the AWS Secrets Manager console\. Then, in the AWS IoT Greengrass console, you add a secret resource to a Greengrass group from the group's **Resources** page\. This secret resource references the Secrets Manager secret\. Later, you attach the secret resource to a Lambda function, which allows the function to get the value of the local secret\.

**Note**  
Alternatively, the console allows you to create a secret and secret resource when you configure a connector or Lambda function\. You can do this from the connector's **Configure parameters** page or the Lambda function's **Resources** page\.  
Only connectors that contain parameters for secrets can access secrets\. For a tutorial that shows how the Twilio Notifications connector uses a locally stored authentication token, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

The tutorial contains the following high\-level steps:

1. [Create a Secrets Manager secret](#secrets-console-create-secret)

1. [Add a secret resource to a group](#secrets-console-create-resource)

1. [Create a Lambda function deployment package](#secrets-console-create-deployment-package)

1. [Create a Lambda function](#secrets-console-create-function)

1. [Add the function to the group](#secrets-console-create-gg-function)

1. [Attach the secret resource to the function](#secrets-console-affiliate-gg-function)

1. [Add subscriptions to the group](#secrets-console-create-subscription)

1. [Deploy the group](#secrets-console-create-deployment)

1. [Test the Lambda function](#secrets-console-test-solution)

The tutorial should take about 20 minutes to complete\.

## Prerequisites<a name="secrets-console-prerequisites"></a>

To complete this tutorial, you need:
+ A Greengrass group and a Greengrass core \(v1\.7 or later\)\. To learn how to create a Greengrass group and core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\. The Getting Started tutorial also includes steps for installing the AWS IoT Greengrass Core software\.
+ AWS IoT Greengrass must be configured to support local secrets\. For more information, see [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ To get the values of local secrets, your user\-defined Lambda functions must use AWS IoT Greengrass Core SDK v1\.3\.0 or later\.

## Step 1: Create a Secrets Manager secret<a name="secrets-console-create-secret"></a>

In this step, you use the AWS Secrets Manager console to create a secret\.

1. <a name="create-secret-step-signin"></a>Sign in to the [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/)\.
**Note**  
For more information about this process, see [ Step 1: Create and store your secret in AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/tutorials_basic.html) in the *AWS Secrets Manager User Guide*\.

1. <a name="create-secret-step-create"></a>Choose **Store a new secret**\.

1. <a name="create-secret-step-othertype"></a>Under **Choose secret type**, choose **Other type of secret**\.

1. Under **Specify the key\-value pairs to be stored for this secret**:
   + For **Key**, enter **test**\.
   + For **Value**, enter **abcdefghi**\.

1. <a name="create-secret-step-encryption"></a>Keep **aws/secretsmanager** selected for the encryption key, and then choose **Next**\.
**Note**  
You aren't charged by AWS KMS if you use the default AWS managed key that Secrets Manager creates in your account\.

1. For **Secret name**, enter **greengrass\-TestSecret**, and then choose **Next**\.
**Note**  
By default, the Greengrass service role allows AWS IoT Greengrass to get the value of secrets with names that start with *greengrass\-*\. For more information, see [secrets requirements](secrets.md#secrets-reqs)\.

1. <a name="create-secret-step-rotation"></a>This tutorial doesn't require rotation, so choose disable automatic rotation, and then choose **Next**\.

1. <a name="create-secret-step-review"></a>On the **Review** page, review your settings, and then choose **Store**\.

   Next, you create a secret resource in your Greengrass group that references the secret\.

## Step 2: Add a secret resource to a Greengrass group<a name="secrets-console-create-resource"></a>

In this step, you configure a group resource that references the Secrets Manager secret\.

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. <a name="create-secret-resource-step-choosegroup"></a>Choose the group that you want to add the secret resource to\.

1. <a name="create-secret-resource-step-secretstab"></a>On the group configuration page, choose the **Resources** tab, and then scroll down to the **Secrets** section\. The **Secrets** section displays the secret resources that belong to the group\. You can add, edit, and remove secret resources from this section\.
**Note**  
Alternatively, the console allows you to create a secret and secret resource when you configure a connector or Lambda function\. You can do this from the connector's **Configure parameters** page or the Lambda function's **Resources** page\.

1. <a name="create-secret-resource-step-addsecretresource"></a>Choose **Add** under the **Secrets** section\.

1. On the **Add a secret resource** page, enter **MyTestSecret** in the **Resource name**\.

1. Under **Secret**, choose **greengrass\-TestSecret**\.

1. <a name="create-secret-resource-step-selectlabels"></a>In the **Select labels \(Optional\)** section, the AWSCURRENT staging label represents the latest version of the secret\. This label is always included in a secret resource\.
**Note**  
This tutorial requires the AWSCURRENT label only\. You can optionally include labels that are required by your Lambda function or connector\.

1. Choose **Add resource**\.

## Step 3: Create a Lambda function deployment package<a name="secrets-console-create-deployment-package"></a>

To create a Lambda function, you must first create a Lambda function *deployment package* that contains the function code and dependencies\. Greengrass Lambda functions require the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) for tasks such as communicating with MQTT messages in the core environment and accessing local secrets\. This tutorial creates a Python function, so you use the Python version of the SDK in the deployment package\.

**Note**  
To get the values of local secrets, your user\-defined Lambda functions must use AWS IoT Greengrass Core SDK v1\.3\.0 or later\.

1. <a name="download-ggc-sdk"></a> From the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads page, download the AWS IoT Greengrass Core SDK for Python to your computer\.

1. <a name="unzip-ggc-sdk"></a>Unzip the downloaded package to get the SDK\. The SDK is the `greengrasssdk` folder\.

1. Save the following Python code function in a local file named `secret_test.py`\.

   ```
   import greengrasssdk
   
   secrets_client = greengrasssdk.client('secretsmanager')
   iot_client = greengrasssdk.client('iot-data')
   secret_name = 'greengrass-TestSecret'
   send_topic = 'secrets/output'
   
   
   def function_handler(event, context):
       """
       Gets a secret and publishes a message to indicate whether the secret was
       successfully retrieved.
       """
       response = secrets_client.get_secret_value(SecretId=secret_name)
       secret_value = response.get('SecretString')
       message = (f'Failed to retrieve secret {secret_name}.' if secret_value is None else
                  f'Successfully retrieved secret {secret_name}.')
       iot_client.publish(topic=send_topic, payload=message)
       print('Published: ' + message)
   ```

   The `get_secret_value` function supports the name or ARN of the Secrets Manager secret for the `SecretId` value\. This example uses the secret name\. For this example secret, AWS IoT Greengrass returns the key\-value pair: `{"test":"abcdefghi"}`\.
**Important**  
Make sure that your user\-defined Lambda functions handle secrets securely and don't log any any sensitive data that's stored in the secret\. For more information, see [ Mitigate the Risks of Logging and Debugging Your Lambda Function](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html#best-practice_lamda-debug-statements) in the *AWS Secrets Manager User Guide*\. Although this documentation specifically refers to rotation functions, the recommendation also applies to Greengrass Lambda functions\.

1. Zip the following items into a file named `secret_test_python.zip`\. When you create the ZIP file, include only the code and dependencies, not the containing folder\.
   + **secret\_test\.py**\. App logic\.
   + **greengrasssdk**\. Required library for all Python Greengrass Lambda functions\.

   This is your Lambda function deployment package\.

## Step 4: Create a Lambda function<a name="secrets-console-create-function"></a>

In this step, you use the AWS Lambda console to create a Lambda function and configure it to use your deployment package\. Then, you publish a function version and create an alias\.

1. First, create the Lambda function\.

   1. <a name="lambda-console-open"></a>In the AWS Management Console, choose **Services**, and open the AWS Lambda console\.

   1. <a name="lambda-console-create-function"></a>Choose **Create function** and then choose **Author from scratch**\.

   1. In the **Basic information** section, use the following values:
      + For **Function name**, enter **SecretTest**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

   1. <a name="lambda-console-save-function"></a>At the bottom of the page, choose **Create function**\.

1. Next, register the handler and upload your Lambda function deployment package\.

   1. <a name="lambda-console-upload"></a>On the **Code** tab, under **Code source**, choose **Upload from**\. From the dropdown, choose **\.zip file**\.  
![\[The Upload from dropdown with .zip file highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/lra-console/upload-deployment-package.png)

   1. Choose **Upload**, then choose your `secret_test_python.zip` deployment package\. Then, choose **Save**\.

   1. <a name="lambda-console-runtime-settings-para"></a>On the **Code** tab for the function, under **Runtime settings**, choose **Edit**, and then enter the following values\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **secret\_test\.function\_handler**

   1. <a name="lambda-console-save-config"></a>Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. Now, publish the first version of your Lambda function and create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

   1. <a name="shared-publish-function-version"></a>From the **Actions** menu, choose **Publish new version**\.

   1. <a name="shared-publish-function-version-description"></a>For **Version description**, enter **First version**, and then choose **Publish**\.

   1. On the **SecretTest: 1** configuration page, from the **Actions** menu, choose **Create alias**\.

   1. On the **Create a new alias** page, use the following values:
      + For **Name**, enter **GG\_SecretTest**\.
      + For **Version**, choose **1**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

   1. Choose **Create**\.

Now you're ready to add the Lambda function to your Greengrass group and attach the secret resource\.

## Step 5: Add the Lambda function to the Greengrass group<a name="secrets-console-create-gg-function"></a>

In this step, you add the Lambda function to the Greengrass group in the AWS IoT console\.

1. <a name="choose-add-lambda"></a>On the group configuration page, choose the **Lambda functions** tab\.

1. Under the **My Lambda functions** section, choose **Add**\.

1. For the **Lambda function**, choose **SecretTest**\.

1. For the **Lambda function version**, choose the alias to the version that you published\.

Next, configure the lifecycle of the Lambda function\.

1. In the **Lambda function configuration** section, make the following updates\.
**Note**  
 We recommend that you run your Lambda function without containerization unless your business case requires it\. This helps enable access to your device GPU and camera without configuring device resources\. If you run without containerization, you must also grant root access to your AWS IoT Greengrass Lambda functions\. 

   1. **To run without containerization:**
      + For **System user and group**, choose **Another user ID/group ID**\. For **System user ID**, enter **0**\. For **System group ID**, enter **0**\.

        This allows your Lambda function to run as root\. For more information about running as root, see [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.
**Tip**  
You also must update your `config.json` file to grant root access to your Lambda function\. For the procedure, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
      + For **Lambda function containerization**, choose **No container**\.

        For more information about running without containerization, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations)\.
      + For **Timeout**, enter **10 seconds**\.
      + For **Pinned**, choose **True**\.

        For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.
      + Under **Additional Parameter**, for **Read access to /sys directory**, choose **Enabled**\.

   1.  **To run in containerized mode instead:** 
**Note**  
We do not recommend running in containerized mode unless your business case requires it\.
      + For **System user and group**, choose **Use group default**\.
      + For **Lambda function containerization**, choose **Use group default**\.
      + For **Memory limit**, enter **1024 MB**\.
      + For **Timeout**, enter **10 seconds**\.
      + For **Pinned**, choose **True**\.

        For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.
      + Under **Additional Parameters**, for **Read access to /sys directory**, choose **Enabled**\.

1.  Choose **Add Lambda function**\.

Next, associate the secret resource with the function\.

## Step 6: Attach the secret resource to the Lambda function<a name="secrets-console-affiliate-gg-function"></a>

In this step, you associate the secret resource to the Lambda function in your Greengrass group\. This associates the resource with the function, which allows the function to get the value of the local secret\.

1. On the group configuration page, choose the **Lambda functions** tab\.

1. Choose the **SecretTest** function\.

1. On the function's details page, choose **Resources**\.

1. Scroll to the **Secrets** section and choose **Associate**\.

1. Choose **MyTestSecret**, and then choose **Associate**\.

## Step 7: Add subscriptions to the Greengrass group<a name="secrets-console-create-subscription"></a>

In this step, you add subscriptions that allow AWS IoT and the Lambda function to exchange messages\. One subscription allows AWS IoT to invoke the function, and one allows the function to send output data to AWS IoT\.

1. <a name="shared-subscriptions-addsubscription"></a>On the group configuration page, choose the **Subscriptions** tab, and then choose **Add Subscription**\.

1. Create a subscription that allows AWS IoT to publish messages to the function\.

   On the group configuration page, choose the **Subscriptions** tab, and then choose **Add subscription**\.

1. On the **Create a subscription** page, configure the source and target, as follows:

   1. In **Source type**, choose **Lambda function**, and then choose **IoT Cloud**\.

   1. In **Target type**, choose **Service**, and then choose **SecretTest**\.

   1. In the **Topic filter**, enter **secrets/input**, and then choose **Create subscription**\. 

1. Add a second subscription\. Choose the **Subscriptions** tab, choose **Add subscription**, and configure the source and target, as follows: 

   1. In **Source type**, choose **Services**, and then choose **SecretTest**\.

   1. In **Target type**, choose **Lambda function**, and then choose **IoT Cloud**\.

   1. In the **Topic filter**, enter **secrets/output**, and then choose **Create subscription**\.

## Step 8: Deploy the Greengrass group<a name="secrets-console-create-deployment"></a>

Deploy the group to the core device\. During deployment, AWS IoT Greengrass fetches the value of the secret from Secrets Manager and creates a local, encrypted copy on the core\.

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

## Test the Lambda function<a name="secrets-console-test-solution"></a>

1. <a name="choose-test-page"></a>On the AWS IoT console home page, choose **Test**\.

1. For **Subscribe to topic**, use the following values, and then choose **Subscribe**\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/secrets-console.html)

1. For **Publish to topic**, use the following values, and then choose **Publish** to invoke the function\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/secrets-console.html)

   If successful, the function publishes a "Success" message\.

## See also<a name="secrets-console-see-also"></a>
+ [Deploy secrets to the AWS IoT Greengrass core](secrets.md)