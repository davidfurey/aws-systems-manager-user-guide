# Troubleshooting Systems Manager Run Command<a name="troubleshooting-remote-commands"></a>

Run Command provides status details with each command execution\. For more information about the details of command statuses, see [Understanding Command Statuses](monitor-commands.md)\. You can also use the information in this topic to help troubleshoot problems with Run Command\.

## Where Are My Instances?<a name="where-are-instances"></a>

If you do not see the expected list of instances when you choose **Select Target instances** then verify the following\.

+ You installed the latest version of the SSM Agent on your instance\. Amazon EC2 Windows Amazon Machine Images \(AMIs\) and some Linux AMIs \(for example Amazon Linux\) are pre\-configured with the SSM Agent\. For information about installing the SSM Agent on an instance, see [Installing and Configuring SSM Agent on Linux Instances](sysman-install-ssm-agent.md) \(for Linux\) or [Installing and Configuring SSM Agent on Windows Instances](sysman-install-ssm-win.md) \(for Windows\)\.

+ Your instance is configured with an AWS Identity and Access Management \(IAM\) role that enables the instance to communicate with the Systems Manager API\. Also verify that your user account has an IAM user trust policy that enables your account to communicate with the Systems Manager API\. For more information, see [Configuring Access to Systems Manager](systems-manager-access.md)\. 

**Check Instance Status Using the Health API**  
You can use the Amazon EC2 Health API to quickly determine the following information about Amazon EC2 instances:

+ The status of one or more instances

+ The last time the instance sent a heartbeat value

+ The version of the SSM Agent

+ The operating system 

+ The version of the EC2Config service \(Windows\)

+ The status of the EC2Config service \(Windows\)

## Getting Status Information on Windows Instances<a name="rc-healthapi-win"></a>

Use the following command to get status details about one or more instances:

```
Get-SSMInstanceInformation -InstanceInformationFilterList @{Key="InstanceIds";ValueSet="instance-ID","instance-ID"}
```

Use the following command with no filters to see all instances registered to your account that are currently reporting an online status\. Substitute the ValueSet="Online" with "ConnectionLost" or "Inactive" to view those statuses:

```
Get-SSMInstanceInformation -InstanceInformationFilterList @{Key="PingStatus";ValueSet="Online"}
```

Use the following command to see which instances are running the latest version of the EC2Config service\. Substitute ValueSet="LATEST" with a specific version \(for example, 3\.0\.54 or 3\.10\) to view those details:

```
Get-SSMInstanceInformation -InstanceInformationFilterList @{Key="AgentVersion";ValueSet="LATEST"}
```

## Getting Status Information on Linux Instances<a name="rc-healthapi-linux"></a>

Use the following command to get status details about one or more instances:

```
aws ssm describe-instance-information --instance-information-filter-list key=InstanceIds,valueSet=instance-ID
```

Use the following command with no filters to see all instances registered to your account that are currently reporting an online status\. Substitute the ValueSet="Online" with "ConnectionLost" or "Inactive" to view those statuses:

```
aws ssm describe-instance-information --instance-information-filter-list key=PingStatus,valueSet=Online
```

Use the following command to see which instances are running the latest version of the SSM Agent\. Substitute ValueSet="LATEST" with a specific version \(for example, 1\.0\.145 or 1\.0\) to view those details:

```
aws ssm describe-instance-information --instance-information-filter-list key=AgentVersion,valueSet=LATEST
```

If the describe\-instance\-information API operation returns an AgentStatus of Online, then your instance is ready to be managed using Run Command\. If the status is Inactive, the instance has one or more of the following problems\. 

+ The SSM Agent is not installed\.

+ The instance does not have outbound internet connectivity\.

+ The instance was not launched with an IAM role that enables it to communicate with the SSM API, or the permissions for the IAM role are not correct for Run Command\. For more information, see [Configuring Access to Systems Manager](systems-manager-access.md)\.

## Troubleshooting the SSM Agent<a name="ts-ssmagent-linux"></a>

If you experience problems executing commands using Run Command, there might be a problem with the SSM Agent\. Use the following information to help you troubleshoot the agent\. 

**View Agent Logs**  
The SSM Agent logs information in the following files\. The information in these files can help you troubleshoot problems\.

**On Windows**

+ %PROGRAMDATA%\\Amazon\\SSM\\Logs\\amazon\-ssm\-agent\.log

+ %PROGRAMDATA%\\Amazon\\SSM\\Logs\\error\.log

**Note**  
The default filename of the seelog is seelog\.xml\.template\. If you modify a seelog, you must rename the file to seelog\.xml\.

**On Linux**

+ /var/log/amazon/ssm/amazon\-ssm\-agent\.log

+ /var/log/amazon/ssm/errors\.log

On Linux, you can enable extended logging by updating the seelog\.xml file\. By default, the configuration file is located here: /etc/amazon/ssm/seelog\.xml\.

For more information about cihub/seelog configuration, go to the [cihub/seelog Wiki](https://github.com/cihub/seelog/wiki)\. For examples of cihub/seelog configurations, go to [cihub/seelog examples](https://github.com/cihub/seelog-examples)\. 