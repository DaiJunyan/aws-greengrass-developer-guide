# Troubleshooting AWS Greengrass Applications<a name="gg-troubleshooting"></a>

Use the following information to help troubleshoot issues in AWS Greengrass\.

## <a name="gg-troubleshooting-issues"></a>


**Issues**  

| Symptom | Solution | 
| --- | --- | 
|  You see 403 Forbidden error on deployment in the logs\.  |  Make sure the policy of the AWS Greengrass core in the cloud includes "greengrass:\*" as an allowed action\.   | 
|  Device's shadow does not sync with the cloud\.  |  Check that the AWS Greengrass core has permissions for "iot:UpdateThingShadow" and "iot:GetThingShadow" actions\. Also see [Troubleshooting Shadow Synchronization Timeout Issues](#troubleshooting-shadow-sync)\.  | 
| The AWS Greengrass core software does not run on Raspberry Pi because user namespace is not enabled\. | Run `rpi-update` to update\. Raspbian has released a new kernel 4\.9 that has user namespace enabled\. | 
| A `ConcurrentDeployment` error occurs when you run `create-deployment` for the first time\. | A deployment might be in progress\. You can run `get-deployment-history` to see if a deployment was created\. If not, try creating the deployment again\.  | 
| The AWS Greengrass core software does not start successfully\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html) | 
| The AWS Greengrass core software does not start on a Raspberry Pi, and you receive the following error in the terminal: `Failed to invoke PutLogEvents on local Cloudwatch, logGroup: /GreengrassSystem/connection_manager, error: RequestError: send request failed caused by: Post http://path/cloudwatch/logs/: dial tcp address: getsockopt: connection refused, response: { }`\. | This can occur if the OS version is Raspbian Stretch\. You can run the following command in the terminal to see the OS version that's installed on your Pi: <pre>cat /etc/os-release</pre> We recommend that you use the [supported platform](what-is-gg.md#gg-platforms) \(currently [Raspian Jessie, 2017\-03\-02](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/)\)\. However, if you choose to run the AWS Greengrass core software on Raspbian Stretch, add `cgroup_enable=memory` to your `/boot/cmdline.txt` file and then restart the device\.  You might encounter other issues if you run AWS Greengrass core software on an unsupported platform\.   | 
| The greengrassd script displays: `unable to accept TCP connection. accept tcp [::]:8000: accept4: too many open files`\. | The file descriptor limit for the AWS Greengrass core software has reached the threshold and must be increased\. Use the following command: <pre>ulimit -n 2048</pre> and restart the AWS Greengrass core software\.  In this example, the limit is increased to 2048\. Choose a value appropriate for your use case\.  | 
| You receive the following error: `Runtime execution error: unable to start lambda container. container_linux.go:259: starting container process caused "process_linux.go:345: container init caused \"rootfs_linux.go:50: preparing rootfs caused \\\"permission denied\\\"\""` | Either install AWS Greengrass directly under the root directory, or ensure that the /greengrass directory and its parent directories have `execute` permissions for everyone\. | 
| The deployment fails with the following error message: Greengrass is not authorized to assume the Service Role associated with this account\. | Using the AWS CLI, check that an appropriate service role is associated with your account by using [GetServiceRoleForAccount](http://docs.aws.amazon.com/greengrass/latest/apireference/getserviceroleforaccount-get.html) and ensure that the role has at least the AWSGreengrassResourceAccessRolePolicy permission applied\. If you need to associate a Greengrass service role with your account, use [AssociateServiceRoleToAccount](http://docs.aws.amazon.com/greengrass/latest/apireference/associateserviceroletoaccount-put.html)\. | 
| The deployment doesn't finish\. | Make sure that the AWS Greengrass daemon is running on your core device\. Run the following commands in your core device terminal to check whether the daemon is running and start it, if needed\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html)  | 
| The deployment doesn't finish, and runtime\.log contains multiple `wait 1s for container to stop` entries\. | Restart the AWS Greengrass daemon by running the following commands in your core device terminal\. <pre>cd /greengrass/ggc/core/<br />sudo ./greengrassd stop<br />sudo ./greengrassd start</pre>  | 
| You receive the following error: `Deployment guid of type NewDeployment for group guid failed error: Error while processing. group config is invalid: 112 or [119 0] don't have rw permission on the file: path` | Ensure that the owner group of the *path* directory has read and write permissions to the directory\. | 

## Troubleshooting with Logs<a name="troubleshooting-logs"></a>

------
#### [ GGC v1\.5\.0 ]

If logs are configured to be stored on the local file system, start looking in the following locations\. Reading the logs on the file system requires root privileges\.

`greengrass-root/ggc/var/log/crash.log`  
Shows messages generated when an AWS Greengrass core crashes\.

`greengrass-root/ggc/var/log/system/runtime.log`  
Shows messages about which component failed\.

`greengrass-root/ggc/var/log/system/`  
Contains all logs from AWS Greengrass system components, such as the certificate manager and the connection manager\. By using the messages in `ggc/var/log/system/` and `ggc/var/log/system/runtime.log`, you should be able to find out which error occurred in AWS Greengrass system components\.

`greengrass-root/ggc/var/log/user/`  
Contains all logs from user\-defined Lambda functions\. Check this folder to find error messages from your local Lambda functions\.

**Note**  
By default, *greengrass\-root* is the `/greengrass` directory\.

If the logs are configured to be stored on the cloud, use CloudWatch Logs to view log messages\. Note that `crash.log` is found only in file system logs on the AWS Greengrass core device\.

If AWS IoT is configured to write logs to CloudWatch, check those logs for information if connection errors occur when system components attempt to connect to AWS IoT\.

For more information about AWS Greengrass logging, see [Monitoring with AWS Greengrass Logs](greengrass-logs-overview.md)\.

------
#### [ GGC v1\.3\.0 ]

If logs are configured to be stored on the local file system, start looking in the following locations\. Reading the logs on the file system requires root privileges\.

`greengrass-root/ggc/var/log/crash.log`  
Shows messages generated when an AWS Greengrass core crashes\.

`greengrass-root/ggc/var/log/system/runtime.log`  
Shows messages about which component failed\.

`greengrass-root/ggc/var/log/system/`  
Contains all logs from AWS Greengrass system components, such as the certificate manager and the connection manager\. By using the messages in `ggc/var/log/system/` and `ggc/var/log/system/runtime.log`, you should be able to find out which error occurred in AWS Greengrass system components\.

`greengrass-root/ggc/var/log/user/`  
Contains all logs from user\-defined Lambda functions\. Check this folder to find error messages from your local Lambda functions\.

**Note**  
By default, *greengrass\-root* is the `/greengrass` directory\.

If the logs are configured to be stored on the cloud, use CloudWatch Logs to view log messages\. Note that `crash.log` is found only in file system logs on the AWS Greengrass core device\.

If AWS IoT is configured to write logs to CloudWatch, check those logs for information if connection errors occur when system components attempt to connect to AWS IoT\.

For more information about AWS Greengrass logging, see [Monitoring with AWS Greengrass Logs](greengrass-logs-overview.md)\.

------
#### [ GGC v1\.1\.0 ]

If logs are configured to be stored on the local file system, start looking in the following locations\. Reading the logs on the file system requires root privileges\.

`greengrass-root/ggc/var/log/crash.log`  
Shows messages generated when an AWS Greengrass core crashes\.

`greengrass-root/ggc/var/log/system/runtime.log`  
Shows messages about which component failed\.

`greengrass-root/ggc/var/log/system/`  
Contains all logs from AWS Greengrass system components, such as the certificate manager and the connection manager\. By using the messages in `ggc/var/log/system/` and `ggc/var/log/system/runtime.log`, you should be able to find out which error occurred in AWS Greengrass system components\.

`greengrass-root/ggc/var/log/user/`  
Contains all logs from user\-defined Lambda functions\. Check this folder to find error messages from your local Lambda functions\.

**Note**  
By default, *greengrass\-root* is the `/greengrass` directory\.

If the logs are configured to be stored on the cloud, use CloudWatch Logs to view log messages\. Note that `crash.log` is found only in file system logs on the AWS Greengrass core device\.

If AWS IoT is configured to write logs to CloudWatch, check those logs for information if connection errors occur when system components attempt to connect to AWS IoT\.

For more information about AWS Greengrass logging, see [Monitoring with AWS Greengrass Logs](greengrass-logs-overview.md)\.

------
#### [ GGC v1\.0\.0 ]

If logs are configured to be stored on the local file system, start looking in the following locations\. Reading the logs on the file system requires root privileges\.

`greengrass-root/var/log/crash.log`  
Shows messages generated when an AWS Greengrass core crashes\.

`greengrass-root/var/log/system/runtime.log`  
Shows messages about which component failed\.

`greengrass-root/var/log/system/`  
Contains all logs from AWS Greengrass system components, such as the certificate manager and the connection manager\. Using the messages in `var/log/system/` and `var/log/system/runtime.log`, you should be able to find out which error occurred in AWS Greengrass system components\.

`greengrass-root/var/log/user/`  
Contains all logs from user\-defined Lambda functions\. Check this folder to find error messages from your local Lambda functions\.

**Note**  
By default, *greengrass\-root* is the `/greengrass` directory\.

If AWS Greengrass is configured to write logs to CloudWatch, you can [view log messages in the CloudWatch console](greengrass-logs-overview.md#gg-logs-cloudwatch)\. Note that `crash.log` is found only in file system logs on the AWS Greengrass core device\.

If AWS IoT is configured to write logs to CloudWatch, check those logs for information if connection errors occur when system components attempt to connect to AWS IoT\.

------

## Troubleshooting Storage Issues<a name="troubleshooting-storage"></a>

When the local file storage is full, some components might start failing: 
+ Local shadow updates do not occur\.
+ New AWS Greengrass core MQTT server certificates cannot be downloaded locally\.
+ Deployments fail\.

You should always be aware of the amount of free space available locally\. This can be calculated based on the sizes of deployed Lambda functions, the logging configuration \(see [Troubleshooting with Logs](#troubleshooting-logs)\), and the number of shadows stored locally\. 

## Troubleshooting Messages<a name="troubleshooting-messages"></a>

All messages sent within AWS Greengrass are sent with QoS 0\. If an AWS Greengrass core is restarted, messages that have not been processed yet are lost\. For this reason, restart the AWS Greengrass core when the service disruption is the lowest\. The AWS Greengrass core is restarted by a deployment, too\.

## Troubleshooting Shadow Synchronization Timeout Issues<a name="troubleshooting-shadow-sync"></a>

------
#### [ GGC v1\.5\.0 ]

If there is a significant delay in communication between a Greengrass core device and the cloud, then shadow synchronization may fail due to a timeout\. You may see something like this in your log files:

```
[2017-07-20T10:01:58.006Z][ERROR]-cloud_shadow_client.go:57,Cloud shadow client error: unable to get cloud shadow what_the_thing_is_named for synchronization. Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][WARN]-sync_manager.go:263,Failed to get cloud copy: Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][ERROR]-sync_manager.go:375,Failed to execute sync operation {what_the_thing_is_named VersionDiscontinued []}"
```

A possible fix is to configure the amount of time your Greengrass core device waits for a host response\. Open the `greengrass-root/config/config.json` file and add a system\.shadowSyncTimeout field with a timeout value in seconds\. For example:

```
{
  "coreThing": {
    "caPath": "root-ca.pem",
    "certPath": "cloud.pem.crt",
    "keyPath": "cloud.pem.key",
    "thingArn": "arn:aws:iot:us-west-2:049039099382:thing/GGTestGroup42_Core",
    "iotHost": "your-AWS-IoT-endpoint", 
    "ggHost": "greengrass.iot.us-west-2.amazonaws.com",
    "keepAlive": 600
  },
  "runtime": {
    "cgroup": {
      "useSystemd": "yes"
    }
  },
  "system": {
    "shadowSyncTimeout": 10
  }
}
```

If no shadowSyncTimeout value is specified in the config\.json file, the default is 1 second\.

------
#### [ GGC v1\.3\.0 ]

If there is a significant delay in communication between a Greengrass core device and the cloud, then shadow synchronization may fail due to a timeout\. You may see something like this in your log files:

```
[2017-07-20T10:01:58.006Z][ERROR]-cloud_shadow_client.go:57,Cloud shadow client error: unable to get cloud shadow what_the_thing_is_named for synchronization. Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][WARN]-sync_manager.go:263,Failed to get cloud copy: Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][ERROR]-sync_manager.go:375,Failed to execute sync operation {what_the_thing_is_named VersionDiscontinued []}"
```

A possible fix is to configure the amount of time your Greengrass core device waits for a host response\. Open the `greengrass-root/config/config.json` file and add a system\.shadowSyncTimeout field with a timeout value in seconds\. For example:

```
{
  "coreThing": {
    "caPath": "root-ca.pem",
    "certPath": "cloud.pem.crt",
    "keyPath": "cloud.pem.key",
    "thingArn": "arn:aws:iot:us-west-2:049039099382:thing/GGTestGroup42_Core",
    "iotHost": "your-AWS-IoT-endpoint", 
    "ggHost": "greengrass.iot.us-west-2.amazonaws.com",
    "keepAlive": 600
  },
  "runtime": {
    "cgroup": {
      "useSystemd": "yes"
    }
  },
  "system": {
    "shadowSyncTimeout": 10
  }
}
```

If no shadowSyncTimeout value is specified in the config\.json file, the default is 1 second\.

------
#### [ GGC v1\.1\.0 ]

If there is a significant delay in communication between a Greengrass core device and the cloud, then shadow synchronization may fail due to a timeout\. You may see something like this in your log files:

```
[2017-07-20T10:01:58.006Z][ERROR]-cloud_shadow_client.go:57,Cloud shadow client error: unable to get cloud shadow what_the_thing_is_named for synchronization. Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][WARN]-sync_manager.go:263,Failed to get cloud copy: Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][ERROR]-sync_manager.go:375,Failed to execute sync operation {what_the_thing_is_named VersionDiscontinued []}"
```

A possible fix is to configure the amount of time your Greengrass core device waits for a host response\. Open the `greengrass-root/config/config.json` file and add a system\.shadowSyncTimeout field with a timeout value in seconds\. For example:

```
{
  "coreThing": {
    "caPath": "root-ca.pem",
    "certPath": "cloud.pem.crt",
    "keyPath": "cloud.pem.key",
    "thingArn": "arn:aws:iot:us-west-2:049039099382:thing/GGTestGroup42_Core",
    "iotHost": "your-AWS-IoT-endpoint", 
    "ggHost": "greengrass.iot.us-west-2.amazonaws.com",
    "keepAlive": 600
  },
  "runtime": {
    "cgroup": {
      "useSystemd": "yes"
    }
  },
  "system": {
    "shadowSyncTimeout": 10
  }
}
```

If no shadowSyncTimeout value is specified in the config\.json file, the default is 1 second\.

------
#### [ GGC v1\.0\.0 ]

Not supported\.

------