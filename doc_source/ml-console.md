--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# How to configure machine learning inference using the AWS Management Console<a name="ml-console"></a>

To follow the steps in this tutorial, you need AWS IoT Greengrass Core v1\.10 or later\.

You can perform machine learning \(ML\) inference locally on a Greengrass core device using locally generated data\. For information, including requirements and constraints, see [Perform machine learning inference](ml-inference.md)\.

This tutorial describes how to use the AWS Management Console to configure a Greengrass group to run a Lambda inference app that recognizes images from a camera locally, without sending data to the cloud\. The inference app accesses the camera module on a Raspberry Pi and runs inference using the open source [SqueezeNet](https://github.com/DeepScale/SqueezeNet) model\.

The tutorial contains the following high\-level steps:

1. [Configure the Raspberry Pi](#config-raspberry-pi)

1. [Install the MXNet framework](#install-mxnet)

1. [Create a model package](#package-ml-model)

1. [Create and publish a Lambda function](#ml-console-create-lambda)

1. [Add the Lambda function to the group](#ml-console-config-lambda)

1. [Add resources to the group](#ml-console-add-resources)

1. [Add a subscription to the group](#ml-console-add-subscription)

1. [Deploy the group](#ml-console-deploy-group)

1. [Test the app](#ml-console-test-app)

## Prerequisites<a name="ml-inference-prerequisites"></a>

To complete this tutorial, you need:
+ Raspberry Pi 4 Model B, or Raspberry Pi 3 Model B/B\+, set up and configured for use with AWS IoT Greengrass\. To set up your Raspberry Pi with AWS IoT Greengrass, run the [Greengrass Device Setup](quick-start.md) script, or make sure that you have completed [Module 1](https://docs.aws.amazon.com/greengrass/latest/developerguide/module1.html) and [Module 2](https://docs.aws.amazon.com/greengrass/latest/developerguide/module2.html) of [Getting started with AWS IoT Greengrass](gg-gs.md)\.
**Note**  
The Raspberry Pi might require a 2\.5A [power supply](https://www.raspberrypi.org/documentation/hardware/raspberrypi/power/) to run the deep learning frameworks that are typically used for image classification\. A power supply with a lower rating might cause the device to reboot\.
+ [Raspberry Pi Camera Module V2 \- 8 megapixel, 1080p](https://www.amazon.com/Raspberry-Pi-Camera-Module-Megapixel/dp/B01ER2SKFS)\. For information about how to set up the camera, see [Connecting the camera](https://www.raspberrypi.org/documentation/usage/camera/) in the Raspberry Pi documentation\. 
+ A Greengrass group and a Greengrass core\. For information about how to create a Greengrass group or core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\.

**Note**  
This tutorial uses a Raspberry Pi, but AWS IoT Greengrass supports other platforms, such as [Intel Atom](#atom-lambda-config) and [NVIDIA Jetson TX2](#jetson-lambda-config)\. In the example for Jetson TX2, you can use static images instead of images streamed from a camera\. If using the Jetson TX2 example, you might need to install Python 3\.6 instead of Python 3\.7\. For information about configuring your device so that you can install the AWS IoT Greengrass Core software, see [Setting up other devices](setup-filter.other.md)\.  
For third party platforms that AWS IoT Greengrass does not support, you must run your Lambda function in non\-containerized mode\. To run in non\-containerized mode, you must run your Lambda function as root\. For more information, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations) and [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.

## Step 1: Configure the Raspberry Pi<a name="config-raspberry-pi"></a>

In this step, install updates to the Raspbian operating system, install the camera module software and Python dependencies, and enable the camera interface\.

Run the following commands in your Raspberry Pi terminal\.

1. Install updates to Raspbian\.

   ```
   sudo apt-get update
   sudo apt-get dist-upgrade
   ```

1. <a name="install-picamera-step"></a>Install the `picamera` interface for the camera module and other Python libraries that are required for this tutorial\.

   ```
   sudo apt-get install -y python3-dev python3-setuptools python3-pip python3-picamera
   ```

   Validate the installation:
   + Make sure that your Python 3\.7 installation includes pip\.

     ```
     python3 -m pip
     ```

     If pip isn't installed, download it from the [pip website](https://pip.pypa.io/en/stable/installing/) and then run the following command\.

     ```
     python3 get-pip.py
     ```
   + Make sure that your Python version is 3\.7 or higher\.

     ```
     python3 --version
     ```

     If the output lists an earlier version, run the following command\.

     ```
     sudo apt-get install -y python3.7-dev
     ```
   + Make sure that Setuptools and Picamera installed successfully\.

     ```
     sudo -u ggc_user bash -c 'python3 -c "import setuptools"'
     sudo -u ggc_user bash -c 'python3 -c "import picamera"'
     ```

     If the output doesn't contain errors, the validation is successful\.
**Note**  
If the Python executable installed on your device is `python3.7`, use `python3.7` instead of `python3` for the commands in this tutorial\. Make sure that your pip installation maps to the correct `python3.7` or `python3` version to avoid dependency errors\.

1. Reboot the Raspberry Pi\.

   ```
   sudo reboot
   ```

1. Open the Raspberry Pi configuration tool\.

   ```
   sudo raspi-config
   ```

1. Use the arrow keys to open **Interfacing Options** and enable the camera interface\. If prompted, allow the device to reboot\.

1. Use the following command to test the camera setup\.

   ```
   raspistill -v -o test.jpg
   ```

   This opens a preview window on the Raspberry Pi, saves a picture named `test.jpg` to your current directory, and displays information about the camera in the Raspberry Pi terminal\.

## Step 2: Install the MXNet framework<a name="install-mxnet"></a>

In this step, install MXNet libraries on your Raspberry Pi\.

1. <a name="ssh-rpi-step"></a>Sign in to your Raspberry Pi remotely\.

   ```
   ssh pi@your-device-ip-address
   ```

1. Open the MXNet documentation, open [Installing MXNet](https://mxnet.apache.org/get_started/?), and follow the instructions to install MXNet on the device\.
**Note**  
We recommend installing version 1\.5\.0 and building MXNet from source for this tutorial to avoid device conflicts\.

1. After you install MXNet, validate the following configuration:
   + Make sure the `ggc_user` system account can use the MXNet framework\.

     ```
     sudo -u ggc_user bash -c 'python3 -c "import mxnet"'
     ```
   + Make sure NumPy is installed\.

     ```
     sudo -u ggc_user bash -c 'python3 -c "import numpy"'
     ```

## Step 3: Create an MXNet model package<a name="package-ml-model"></a>

In this step, create a model package that contains a sample pretrained MXNet model to upload to Amazon Simple Storage Service \(Amazon S3\)\. AWS IoT Greengrass can use a model package from Amazon S3, provided that you use the tar\.gz or zip format\.

1. On your computer, download the MXNet sample for Raspberry Pi from [Machine learning samples](what-is-gg.md#gg-ml-samples)\.

1.  Unzip the downloaded `mxnet-py3-armv7l.tar.gz` file\. 

1. Navigate to the `squeezenet` directory\.

   ```
   cd path-to-downloaded-sample/mxnet-py3-armv7l/models/squeezenet
   ```

   The `squeezenet.zip` file in this directory is your model package\. It contains SqueezeNet open source model artifacts for an image classification model\. Later, you upload this model package to Amazon S3\.

## Step 4: Create and publish a Lambda function<a name="ml-console-create-lambda"></a>

In this step, create a Lambda function deployment package and Lambda function\. Then, publish a function version and create an alias\.

First, create the Lambda function deployment package\.

1. On your computer, navigate to the `examples` directory in the sample package that you unzipped in [Step 3: Create an MXNet model package](#package-ml-model)\.

   ```
   cd path-to-downloaded-sample/mxnet-py3-armv7l/examples
   ```

   The `examples` directory contains function code and dependencies\.
   + `greengrassObjectClassification.py` is the inference code used in this tutorial\. You can use this code as a template to create your own inference function\.
   + <a name="ml-samples-ggc-sdk"></a>`greengrasssdk` is version 1\.5\.0 of the AWS IoT Greengrass Core SDK for Python\.
**Note**  <a name="ml-samples-ggc-sdk-upgrade"></a>
If a new version is available, you can download it and upgrade the SDK version in your deployment package\. For more information, see [AWS IoT Greengrass Core SDK for Python](https://github.com/aws/aws-greengrass-core-sdk-python/) on GitHub\.

1.  Compress the contents of the `examples` directory into a file named `greengrassObjectClassification.zip`\. This is your deployment package\. 

   ```
   zip -r greengrassObjectClassification.zip .
   ```
**Note**  <a name="ml-samples-function-zip"></a>
 Make sure the `.py` ﬁles and dependencies are in the root of the directory\. 

    

   Next, create the Lambda function\.

1. From the AWS IoT console, choose **Functions** and **Create function**\.

1. Choose **Author from scratch** and use the following values to create your function:
   + For **Function name**, enter **greengrassObjectClassification**\.
   + For **Runtime**, choose **Python 3\.7**\.

   For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

1. Choose **Create function**\.

    

   Now, upload your Lambda function deployment package and register the handler\.

1. Choose your Lambda function and upload your Lambda function deployment package\.

   1. <a name="lambda-console-upload"></a>On the **Code** tab, under **Code source**, choose **Upload from**\. From the dropdown, choose **\.zip file**\.  
![\[The Upload from dropdown with .zip file highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/lra-console/upload-deployment-package.png)

   1. Choose **Upload**, and then choose your `greengrassObjectClassification.zip` deployment package\. Then, choose **Save**\.

   1. <a name="lambda-console-runtime-settings-para"></a>On the **Code** tab for the function, under **Runtime settings**, choose **Edit**, and then enter the following values\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **greengrassObjectClassification\.function\_handler**\.

      Choose **Save**\.

   Next, publish the first version of your Lambda function\. Then, create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

1. From the **Actions** menu, choose **Publish new version**\.  
![\[The Publish new version option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/ml-inference/lambda-publish-version.png)

1. For **Version description**, enter **First version**, and then choose **Publish**\.

1. On the **greengrassObjectClassification: 1** configuration page, from the **Actions** menu, choose **Create alias**\.  
![\[The Create alias option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/ml-inference/lambda-create-alias.png)

1. On the **Create a new alias** page, use the following values:
   + For **Name**, enter **mlTest**\.
   + For **Version**, enter **1**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. Choose **Save**\.

    

   Now, add the Lambda function to your Greengrass group\.

## Step 5: Add the Lambda function to the Greengrass group<a name="ml-console-config-lambda"></a>

In this step, add the Lambda function to the group and then configure its lifecycle and environment variables\.

First, add the Lambda function to your Greengrass group\.

1. In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. From the group configuration page, choose the **Lambda functions** tab\.

1. Under the **My Lambda functions** section, choose **Add**\.

1. For the **Lambda function**, choose **greengrassObjectClassification**\.

1. For the **Lambda function version**, choose **Alias:mlTest**\.

    

   Next, configure the lifecycle and environment variables of the Lambda function\.

1. On the **Lambda function configuration** section, make the following updates\.
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

   1.  **To run in containerized mode instead:** 
**Note**  
We do not recommend running in containerized mode unless your business case requires it\.
      + For **System user and group**, choose **Use group default**\.
      + For **Lambda function containerization**, choose **Use group default**\.
      + For **Memory limit**, enter **96 MB**\.
      + For **Timeout**, enter **10 seconds**\.
      + For **Pinned**, choose **True**\. 

        For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.

1. Under **Environment variables**, create a key\-value pair\. A key\-value pair is required by functions that interact with MXNet models on a Raspberry Pi\.

   For the key, use MXNET\_ENGINE\_TYPE\. For the value, use NaiveEngine\. 
**Note**  
In your own user\-defined Lambda functions, you can optionally set the environment variable in your function code\.

1. Keep the default values for all other properties and choose **Add Lambda function**\.

## Step 6: Add resources to the Greengrass group<a name="ml-console-add-resources"></a>

In this step, create resources for the camera module and the ML inference model and affiliate the resources with the Lambda function\. This makes it possible for the Lambda function to access the resources on the core device\.

**Note**  
If you run in non\-containerized mode, AWS IoT Greengrass can access your device GPU and camera without configuring these device resources\. 

First, create two local device resources for the camera: one for shared memory and one for the device interface\. For more information about local resource access, see [Access local resources with Lambda functions and connectors](access-local-resources.md)\.

1. On the group configuration page, choose the **Resources** tab\.

1. In the **Local resources** section, choose **Add local resource**\.

1. On the **Add a local resource** page, use the following values:
   + For **Resource name**, enter **videoCoreSharedMemory**\.
   + For **Resource type**, choose **Device**\.
   + For **Local device path**, enter **/dev/vcsm**\.

     The device path is the local absolute path of the device resource\. This path can only refer to a character device or block device under `/dev`\.
   + For **System group owner and file access permissions**, choose **Automatically add file system permissions of the system group that owns the resource**\.

     The **System group owner and file access permissions** option lets you grant additional file access permissions to the Lambda process\. For more information, see [Group owner file access permission](access-local-resources.md#lra-group-owner)\.

1. Next, you add a local device resource for the camera interface\.

1. Choose **Add local resource**\.

1. On the **Add a local resource** page, use the following values:
   + For **Resource name**, enter **videoCoreInterface**\.
   + For **Resource type**, choose **Device**\.
   + For **Local device path**, enter **/dev/vchiq**\.
   + For **System group owner and file access permissions**, choose **Automatically add file system permissions of the system group that owns the resource**\. 

1. At the bottom of the page, choose **Add resource**\.

 

Now, add the inference model as a machine learning resource\. This step includes uploading the `squeezenet.zip` model package to Amazon S3\.

1. On the **Resources** tab for your group, under the **Machine Learning** section, choose **Add machine learning resource**\.

1. On the **Add a machine learning resource** page, for **Resource name**, enter **squeezenet\_model**\.

1. For **Model source**, choose **Use a model stored in S3, such as a model optimized through Deep Learning Compiler**\.

1. For **S3 URI**, enter a path where the S3 bucket is saved\. 

1.  Choose **Browse S3**\. This opens up a new tab to the Amazon S3 console\. 

1.  On the Amazon S3 console tab, upload the `squeezenet.zip` file to an S3 bucket\. For information, see [ How do I upload files and folders to an S3 Bucket? ](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/upload-objects.html)in the *Amazon Simple Storage Service User Guide*\. 
**Note**  
For the S3 bucket to be accessible, your bucket name must contain the string **greengrass** and the bucket must be in the same region that you use for AWS IoT Greengrass\. Choose a unique name \(such as **greengrass\-bucket\-*user\-id*\-*epoch\-time***\)\. Don't use a period \(`.`\) in the bucket name\. 

1. On the AWS IoT Greengrass console tab, locate and choose your S3 bucket\. Locate your uploaded `squeezenet.zip` file, and choose **Select**\. You might need to choose **Refresh** to update the list of available buckets and files\. 

1. For **Destination path**, enter **/greengrass\-machine\-learning/mxnet/squeezenet**\.

   This is the destination for the local model in the Lambda runtime namespace\. When you deploy the group, AWS IoT Greengrass retrieves the source model package and then extracts the contents to the specified directory\. The sample Lambda function for this tutorial is already configured to use this path \(in the `model_path` variable\)\.

1. Under **System group owner and file access permissions**, choose **No system group**\.

1. Choose **Add resource**\.

### Using SageMaker trained models<a name="sm-models"></a>

This tutorial uses a model that's stored in Amazon S3, but you can easily use SageMaker models too\. The AWS IoT Greengrass console has built\-in SageMaker integration, so you don't need to manually upload these models to Amazon S3\. For requirements and limitations for using SageMaker models, see [Supported model sources](ml-inference.md#supported-model-sources)\.

To use an SageMaker model:
+ For **Model source**, choose **Use a model trained in AWS SageMaker**, and then choose the name of the model's training job\.
+ For **Destination path**, enter the path to the directory where your Lambda function looks for the model\.

## Step 7: Add a subscription to the Greengrass group<a name="ml-console-add-subscription"></a>

In this step, add a subscription to the group\. This subscription enables the Lambda function to send prediction results to AWS IoT by publishing to an MQTT topic\.

1. On the group configuration page, choose the **Subscriptions** tab, and then choose **Add Subscription**\.

1. On the **Subscription details** page, configure the source and target, as follows:

   1. In **Source type**, choose **Lambda function**, and then choose **greengrassObjectClassification**\.

   1. In **Target type**, choose **Service**, and then choose **IoT Cloud**\.

1. In **Topic filter**, enter **hello/world**, and then choose **Create subscription**\.

## Step 8: Deploy the Greengrass group<a name="ml-console-deploy-group"></a>

In this step, deploy the current version of the group definition to the Greengrass core device\. The definition contains the Lambda function, resources, and subscription configurations that you added\.

1. Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/1.11.6/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. On the group configuration page, choose **Deploy**\.  
![\[The group page with Deployments and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/console-group-deployments-deploy.png)

1. In the **Lambda functions** tab, under the **System Lambda functions** section, select **IP detector** and choose **Edit**\.

1. In the **Edit IP detector settings** dialog box, select **Automatically detect and override MQTT broker endpoints**\.

1. Choose **Save**\.

   This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.
**Note**  
If prompted, grant permission to create the [Greengrass service role](service-role.md) and associate it with your AWS account in the current AWS Region\. This role allows AWS IoT Greengrass to access your resources in AWS services\.

   The **Deployments** page shows the deployment timestamp, version ID, and status\. When completed, the status displayed for the deployment should be **Completed**\.

   For more information about deployments, see [Deploy AWS IoT Greengrass groups to an AWS IoT Greengrass core](deployments.md)\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Step 9: Test the inference app<a name="ml-console-test-app"></a>

Now you can verify whether the deployment is configured correctly\. To test, you subscribe to the `hello/world` topic and view the prediction results that are published by the Lambda function\.

**Note**  
If a monitor is attached to the Raspberry Pi, the live camera feed is displayed in a preview window\.

1. In the AWS IoT console, under **Test**, choose **MQTT test client**\.

1. For **Subscriptions**, use the following values:
   + For the subscription topic, use hello/world\.
   + Under **Additional configuration**, for **MQTT payload display**, choose **Display payloads as strings**\.

1. Choose **Subscribe**\.

   If the test is successful, the messages from the Lambda function appear at the bottom of the page\. Each message contains the top five prediction results of the image, using the format: probability, predicted class ID, and corresponding class name\.  
![\[The Subscriptions page showing test results with message data.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/ml-inference/prediction-results.png)

### Troubleshooting AWS IoT Greengrass ML inference<a name="ml-inference-troubleshooting"></a>

If the test is not successful, you can try the following troubleshooting steps\. Run the commands in your Raspberry Pi terminal\.

#### Check error logs<a name="troubleshooting-check-logs"></a>

1. <a name="root-access-logs"></a>Switch to the root user and navigate to the `log` directory\. Access to AWS IoT Greengrass logs requires root permissions\.

   ```
   sudo su
   cd /greengrass/ggc/var/log
   ```

1. In the `system` directory, check `runtime.log` or `python_runtime.log`\.

   In the `user/region/account-id` directory, check `greengrassObjectClassification.log`\.

   For more information, see [Troubleshooting with logs](gg-troubleshooting.md#troubleshooting-logs)\.

##### Unpacking error in runtime\.log<a name="troubleshooting-targz-unpacking"></a>

If `runtime.log` contains an error similar to the following, make sure that your `tar.gz` source model package has a parent directory\.

```
Greengrass deployment error: unable to download the artifact model-arn: Error while processing. 
Error while unpacking the file from /tmp/greengrass/artifacts/model-arn/path to /greengrass/ggc/deployment/path/model-arn,
error: open /greengrass/ggc/deployment/path/model-arn/squeezenet/squeezenet_v1.1-0000.params: no such file or directory
```

If your package doesn't have a parent directory that contains the model files, use the following command to repackage the model:

```
tar -zcvf model.tar.gz ./model
```

For example:

```
─$ tar -zcvf test.tar.gz ./test
./test
./test/some.file
./test/some.file2
./test/some.file3
```

**Note**  
Don't include trailing `/*` characters in this command\.

 

#### Verify that the Lambda function is successfully deployed<a name="troubleshooting-check-lambda"></a>

1. List the contents of the deployed Lambda in the `/lambda` directory\. Replace the placeholder values before you run the command\.

   ```
   cd /greengrass/ggc/deployment/lambda/arn:aws:lambda:region:account:function:function-name:function-version
   ls -la
   ```

1. Verify that the directory contains the same content as the `greengrassObjectClassification.zip` deployment package that you uploaded in [Step 4: Create and publish a Lambda function](#ml-console-create-lambda)\.

   Make sure that the `.py` files and dependencies are in the root of the directory\.

 

#### Verify that the inference model is successfully deployed<a name="troubleshooting-check-model"></a>

1. Find the process identification number \(PID\) of the Lambda runtime process:

   ```
   ps aux | grep 'lambda-function-name*'
   ```

   In the output, the PID appears in the second column of the line for the Lambda runtime process\.

1. Enter the Lambda runtime namespace\. Be sure to replace the placeholder *pid* value before you run the command\.
**Note**  
This directory and its contents are in the Lambda runtime namespace, so they aren't visible in a regular Linux namespace\.

   ```
   sudo nsenter -t pid -m /bin/bash
   ```

1. List the contents of the local directory that you specified for the ML resource\.

   ```
   cd /greengrass-machine-learning/mxnet/squeezenet/
   ls -ls
   ```

   You should see the following files:

   ```
   32 -rw-r--r-- 1 ggc_user ggc_group   31675 Nov 18 15:19 synset.txt
   32 -rw-r--r-- 1 ggc_user ggc_group   28707 Nov 18 15:19 squeezenet_v1.1-symbol.json
   4832 -rw-r--r-- 1 ggc_user ggc_group 4945062 Nov 18 15:19 squeezenet_v1.1-0000.params
   ```

## Next steps<a name="next-steps"></a>

Next, explore other inference apps\. AWS IoT Greengrass provides other Lambda functions that you can use to try out local inference\. You can find the examples package in the precompiled libraries folder that you downloaded in [Step 2: Install the MXNet framework](#install-mxnet)\.

## Configuring an Intel Atom<a name="atom-lambda-config"></a>

 To run this tutorial on an Intel Atom device, you must provide source images, configure the Lambda function, and add another local device resource\. To use the GPU for inference, make sure the following software is installed on your device:
+ OpenCL version 1\.0 or later
+ Python 3\.7 and pip
**Note**  
If your device is prebuilt with Python 3\.6, you can create a symlink to Python 3\.7 instead\. For more information, see [Step 2](#python-symlink)\.
+ [NumPy](https://pypi.org/project/numpy/)
+ [OpenCV on Wheels](https://pypi.org/project/opencv-python/)

1. Download static PNG or JPG images for the Lambda function to use for image classification\. The example works best with small image files\. 

   Save your image files in the directory that contains the `greengrassObjectClassification.py` file \(or in a subdirectory of this directory\)\. This is in the Lambda function deployment package that you upload in [Step 4: Create and publish a Lambda function](#ml-console-create-lambda)\.
**Note**  
 If you're using AWS DeepLens, you can use the onboard camera or mount your own camera to perform inference on captured images instead of static images\. However, we strongly recommend you start with static images first\.   
If you use a camera, make sure that the `awscam` APT package is installed and up to date\. For more information, see [Update your AWS DeepLens device](https://docs.aws.amazon.com/deeplens/latest/dg/deeplens-manual-updates.html) in the *AWS DeepLens Developer Guide*\.

1. <a name="python-symlink"></a>If you aren't using Python 3\.7, make sure to create a symlink from Python 3\.x to Python 3\.7\. This configures your device to use Python 3 with AWS IoT Greengrass\. Run the following command to locate your Python installation:

   ```
   which python3
   ```

   Run the following command to create the symlink:

   ```
   sudo ln -s path-to-python-3.x/python3.x path-to-python-3.7/python3.7
   ```

   Reboot the device\.

1. Edit the configuration of the Lambda function\. Follow the procedure in [Step 5: Add the Lambda function to the Greengrass group](#ml-console-config-lambda)\. 
**Note**  
 We recommend that you run your Lambda function without containerization unless your business case requires it\. This helps enable access to your device GPU and camera without configuring device resources\. If you run without containerization, you must also grant root access to your AWS IoT Greengrass Lambda functions\. 

   1. **To run without containerization:**
      + For **System user and group**, choose **Another user ID/group ID**\. For **System user ID**, enter **0**\. For **System group ID**, enter **0**\.

        This allows your Lambda function to run as root\. For more information about running as root, see [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.
**Tip**  
You also must update your `config.json` file to grant root access to your Lambda function\. For the procedure , see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
      + For **Lambda function containerization**, choose **No container**\.

        For more information about running without containerization, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations)\.
      + Update the **Timeout** value to 5 seconds\. This ensures that the request does not time out too early\. It takes a few minutes after setup to run inference\.
      + Under **Pinned**, choose **True**\.
      + Under **Additional Parameters**, for **Read access to /sys directory**, choose **Enabled**\. 
      +  For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\. 

   1.  **To run in containerized mode instead:** 
**Note**  
We do not recommend running in containerized mode unless your business case requires it\.
      + Update the **Timeout** value to 5 seconds\. This ensures that the request does not time out too early\. It takes a few minutes after setup to run inference\.
      +  For **Pinned**, choose **True**\. 
      + Under **Additional Parameters**, for **Read access to /sys directory**, choose **Enabled**\. 

1. **If running in containerized mode**, add the required local device resource to grant access to your device GPU\.
**Note**  
If you run in non\-containerized mode, AWS IoT Greengrass can access your device GPU without configuring device resources\. 

   1. On the group configuration page, choose the **Resources** tab\.

   1. Choose **Add local resource**\.

   1. Define the resource:
      + For **Resource name**, enter **renderD128**\.
      + For **Resource type**, choose **Local device**\.
      + For **Device path**, enter **/dev/dri/renderD128**\.
      + For **System group owner and file access permissions**, choose **Automatically add file system permissions of the system group that owns the resource**\.
      + For **Lambda function affiliations**, grant **Read and write access** to your Lambda function\.

## Configuring an NVIDIA Jetson TX2<a name="jetson-lambda-config"></a>

To run this tutorial on an NVIDIA Jetson TX2, provide source images and configure the Lambda function\. If you're using the GPU, you must also add local device resources\.

1.  Make sure your Jetson device is configured so you can install the AWS IoT Greengrass Core software\. For more information about configuring your device, see [Setting up other devices](setup-filter.other.md)\. 

1. Open the MXNet documentation, go to [Installing MXNet on a Jetson](https://mxnet.apache.org/get_started/jetson_setup), and follow the instructions to install MXNet on the Jetson device\.
**Note**  
 If you want to build MXNet from source, follow the instructions to build the shared library\. Edit the following settings in your `config.mk` file to work with a Jetson TX2 device:   
Add `-gencode arch=compute-62, code=sm_62` to the `CUDA_ARCH` setting\.
Turn on CUDA\.  

     ```
     USE_CUDA = 1
     ```

1. Download static PNG or JPG images for the Lambda function to use for image classification\. The app works best with small image files\. Alternatively, you can instrument a camera on the Jetson board to capture the source images\.

   Save your image files in the directory that contains the `greengrassObjectClassification.py` file\. You can also save them in a subdirectory of this directory\. This directory is in the Lambda function deployment package that you upload in [Step 4: Create and publish a Lambda function](#ml-console-create-lambda)\.

1. Create a symlink from Python 3\.7 to Python 3\.6 to use Python 3 with AWS IoT Greengrass\. Run the following command to locate your Python installation:

   ```
   which python3
   ```

   Run the following command to create the symlink:

   ```
   sudo ln -s path-to-python-3.6/python3.6 path-to-python-3.7/python3.7
   ```

   Reboot the device\.

1. Make sure the `ggc_user` system account can use the MXNet framework:

   ```
   “sudo -u ggc_user bash -c 'python3 -c "import mxnet"'
   ```

1. Edit the configuration of the Lambda function\. Follow the procedure in [Step 5: Add the Lambda function to the Greengrass group](#ml-console-config-lambda)\.
**Note**  
 We recommend that you run your Lambda function without containerization unless your business case requires it\. This helps enable access to your device GPU and camera without configuring device resources\. If you run without containerization, you must also grant root access to your AWS IoT Greengrass Lambda functions\. 

   1. **To run without containerization:**
      + For **System user and group**, choose **Another user ID/group ID**\. For **System user ID**, enter **0**\. For **System group ID**, enter **0**\.

        This allows your Lambda function to run as root\. For more information about running as root, see [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\.
**Tip**  
You also must update your `config.json` file to grant root access to your Lambda function\. For the procedure, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.
      + For **Lambda function containerization**, choose **No container**\.

        For more information about running without containerization, see [Considerations when choosing Lambda function containerization](lambda-group-config.md#lambda-containerization-considerations)\.
      + Under **Additional Parameters**, for **Read access to /sys directory**, choose **Enabled**\. 
      +  Under **Environment variables**, add the following key\-value pairs to your Lambda function\. This configures AWS IoT Greengrass to use the MXNet framework\.     
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/ml-console.html)

   1.  **To run in containerized mode instead:** 
**Note**  
We do not recommend running in containerized mode unless your business case requires it\.
      + Increase the **Memory limit** value\. Use 500 MB for CPU, or at least 2000 MB for GPU\. 
      + Under **Additional Parameters**, for **Read access to /sys directory**, choose **Enabled**\. 
      +  Under **Environment variables**, add the following key\-value pairs to your Lambda function\. This configures AWS IoT Greengrass to use the MXNet framework\.     
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/ml-console.html)

1. **If running in containerized mode**, add the following local device resources to grant access to your device GPU\. Follow the procedure in [Step 6: Add resources to the Greengrass group](#ml-console-add-resources)\.
**Note**  
 If you run in non\-containerized mode, AWS IoT Greengrass can access your device GPU without configuring device resources\. 

   For each resource:
   + For **Resource type**, choose **Device**\.
   + For **System group owner and file access permissions**, choose **Automatically add file system permissions of the system group that owns the resource**\.

          
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/ml-console.html)

1. **If running in containerized mode**, add the following local volume resource to grant access to your device camera\. Follow the procedure in [Step 6: Add resources to the Greengrass group](#ml-console-add-resources)\.
**Note**  
 If you run in non\-containerized mode, AWS IoT Greengrass can access your device camera without configuring volume resources\. 
   + For **Resource type**, choose **Volume**\.
   + For **System group owner and file access permissions**, choose **Automatically add file system permissions of the system group that owns the resource**\.

          
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/ml-console.html)