# Systems Manager

Hello, this is a self-paced workshop designed to explore the main features inside AWS Systems Manager.

## Let's begin preparing the environment

1\. Log into the AWS Management Console and choose the preferred [AWS region](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html).

2\. [We need to set up AWS Config to record configuration changes of our resources.](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html)

3\. [We need to create a Keypair to log in to the ec2 instances.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair)

5\. Launch the CloudFormation template two times for your selected region, the first one for the production environment and the second one for the development environment, [download the template to create the resources stacks](https://raw.githubusercontent.com/aurbac/aws-systems-manager/master/AURBAC-AWS-Instances-For-SSM.yaml).

  a\. Go to CloudFormation service and click on **Create Stack**.

  b\. Choose your template downloaded **AURBAC-AWS-Instances-For-SSM.yaml** and click **Next**.

  c\. Use the name `production-resources` or `development-resources` to identify the cloudformation stacks.

  d\. Select your KeyPair created previously, click **Next**.

  e\. For the Options, add the tag key `Environment` and the value `Production` or `Development` depending on the stack to be created, click **Next**.

  f\. Check the box `I acknowledge that AWS CloudFormation might create IAM resources.` and click **Create**.

6\. Launch the CloudFormation template to create the S3 bucket and the role for the maintenance window task, [download the template to create the resources stack](https://raw.githubusercontent.com/aurbac/aws-systems-manager/master/AURBAC-AWS-SSM-Requirements.yaml).

  a\. Go to CloudFormation service and click on **Create Stack**.

  b\. Choose your template downloaded **AURBAC-AWS-SSM-Requirements.yaml** and click **Next**.

  c\. Use the name `ssm-requirements` identify the cloudformation stacks.

  d\. Click **Next**.

  e\. For the Options, click **Next**.

  f\. Check the box `I acknowledge that AWS CloudFormation might create IAM resources.` and click **Create**.

### AWS Services created

* Two amazon VPC environments with two public subnets, each one with:
  * EC2 instance Amazon Linux 2 (t2.micro) with IAM Role and Tags (Environment and Patch Group).
  * EC2 instance Centos 7 (t2.micro) with IAM Role and Tag (Environment).
  * EC2 instance Windows 2012 (t2.medium) with IAM Role and Tag (Environment).
  * EC2 instance Windows 2016 (t2.medium) with IAM Role and Tag (Environment).
  * Each instance has an IAM Role with the policy required for Systems Manager: **AmazonEC2RoleforSSM**
* An S3 bucket for the maintenance window task.
* An IAM Role for the maintenance window task.

## Resource Groups for Production and Development services

1\. Go to Systems Manager service and click on **Find Resources** under Resource Groups section.

2\. In **Grouping criteria** apply a filter using the tag key `Environment` and the value `Production` or `Development` and click **View query results**.

3\. You will see the resources related to the environment, click on **Save query as group** and name the group as `Production-Services` or `Development-Services` and **Create group**.

In **Saved Resource Groups** under Resource Groups section you will see the resources groups created to explore.

## Inventory for Production and Development instances

1\. Go to Systems Manager service and click on **Inventory** under Insights section.

2\. Click on **Setup Inventory**.

3\. Use the name `Inventory-Association-For-Production` or `Inventory-Association-For-Development`.

4\. In Targets section click on **Specifying a tag** and use the tag key `Environment` and the value `Production` or `Development` and click **Setup Inventory**.

5\. Click on **State Manager** under Actions section, you will see the Associations created to collect the information for every environment and the Last execution.

**NOTE:** Aditionally, you can create an inventory that search for **Files** or the **Windows Registry**, [example configuration in the Configuring Collection section in step 6](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-inventory-configuring.html).


## Update the SSM Agent with State Manager

1\. Go to Systems Manager service and click on **State Manager** under Actions section and click on **Create association** button.

2\. Use the name `UpdateSSMAgent`.

3\. In the Command Document, click in the search bar and select, **Document name prefix**, then click on **Equal**, then type in `AWS-UpdateSSMAgent` and enter.

3\. Now select the **AWS-UpdateSSMAgent** document name that will upgrade Systems Management agent on the instances.

4\. On Targets, select **Selecting all managed instances in this account**.

5\. Specify schedule for **Every Day** at your preference time.

6\. Scroll down and click **Create association**.

#### AWS Config: Configuration change

1\. Once the status for **UpdateSSMAgent** association is **Success** go to **Inventory** under Insights section, scroll down and on Corresponding managed instances and click in the AWS Config button for the `development-resources-AmazonLinux2` instance.

![Inventory instances](https://github.com/aurbac/aws-systems-manager/raw/master/images/inventory-instances-config.png)

2\. In the next page you will see a timeline changes for the instance, the last change will be selected, scroll down and expand the **Changes** section to see the change version for the SSM Agent, you will see something similar like the following:

![SSM Agent change](https://github.com/aurbac/aws-systems-manager/raw/master/images/config-change.png)

## Enabling a WebServer in Linux and Windows using a Document and Run Command

### Documents

1\. Go to Systems Manager service and click on **Documents** under Shared Resources section and click on **Create document** button.

2\. Use the name `ActivateWebServer` for the new document.

3\. On Document type select **Command document** and content **JSON** type.

4\. Copy and replace in the field text for **Content** with the follwing following structure:

    {
    "schemaVersion": "2.2",
    "description": "Activate WebServer.",
    "parameters": {},
    "mainSteps": [
        {
        "action": "aws:runPowerShellScript",
        "precondition": {
            "StringEquals": [
            "platformType",
            "Windows"
            ]
        },
        "name": "WindowsWebServer",
        "inputs": {
            "runCommand": [
                "Set-ExecutionPolicy Unrestricted -Force",
                "New-Item -ItemType directory -Path 'C:\temp'",
                "Import-Module ServerManager",
                "install-windowsfeature web-server, web-webserver -IncludeAllSubFeature",
                "install-windowsfeature web-mgmt-tools"
            ]
        }
        },
        {
        "action": "aws:runShellScript",
        "precondition": {
            "StringEquals": [
            "platformType",
            "Linux"
            ]
        },
        "name": "LinuxWebServer",
        "inputs": {
            "runCommand": [
            "#!/bin/bash",
            "sudo yum update -y",
            "sudo yum install httpd -y",
            "sudo systemctl enable httpd",
            "sudo systemctl start httpd"
            ]
        }
        }
    ]
    }

5\. Scroll down and click **Create document**.

### Run Command

1\. Go to Systems Manager service and click on **Run Command** under Actions section and click on **Run Command** button.

2\. On the Run a command page, click in the search bar and select, **Owner**, then click on **Owned by me**.

3\. Now select the **ActivateWebServer** document name that we created previously.

4\. For the command paramters we are going to use the default values, where Password is referenced for the Paramater Store created.

4\. On Targets, select **Specifying a tag** and apply a filter using the tag key `Environment` and the value `Development`, click on **Add**.

5\. Scroll down and click **Run**.

6\. Next you will see page documenting your running command then and overall success in green.

**Once each instance task is finished you can check the web server by accessing by the public IP in your browser.**

## Patching Linux and Windows with Maintenance Windows

We are going to create a special PatchBaseline that will only apply to Amazon Linux 2 instances based on the tag **Patch Group**, for rest of the instances will be applied the default Patch Baselines provided by AWS.

### Patch Baseline

1\. Go to Systems Manager service and click on **Patch Manager** under Actions section and click on **Create Maintenance Window** button.

2\. Use the name `AmazonLinux2`.

3\. For the **Operating system** select `Amazon Linux 2`.

4\. For aproval rules leave the configuration for **All** or be specific in the clasification and severity.

5\. Click on **Create patch baseline**.

6\. Refesh the page until you can see the new patch baseline, select it and click on **Actions** and **Modify patch groups**.

7\. In the **Patch groups** field type `AmazonLinux2` and click on **Add**.

8\. Click in **Close**.

**Remember that this new patch baseline will only apply to instances tagged with "Patch Group" and the value of "AmazonLinux2".**

### Maintenance Windows

1\. Go to Systems Manager service and click on **Maintenance Windows** under Actions section, click on **Create maintenance window**.

2\. Use the name `MaitenanceWindowForDevelopmentInstances`.

3\. Schedule the maitenance for every day at the desired time.

4\. Specify a duration of `2` and stop initiating tasks of `1`.

5\. You can select your **Schedule timezone** prefered.

6\. Click on **Create maintenance window**.

7\. To configure the targets click on the maintenance window created, go to **Targets** and click on **Register target**.

8\. For the **Target Name** type `DevelopmentInstances`.

9\. In Targets, select **Specifying tags** and in Tags type the key `Environment` and the value of `Development`.

10\. Click on **Register target**.

11\. Now go to **Tasks** of the maintenance window and click on **Register tasks** and **Register Run command task**.

12\. For the **Name** type `DevelopmentInstances`.

13\. In the Command Document, click in the search bar and select, **Document name prefix**, then click on **Equal**, then type in `AWS-RunPatchBaseline` and enter.

14\. Now select the **AWS-RunPatchBaseline** document name that will apply the patches depending of the operating systen and Patch Group tag.

15\. In Targets select the target created **DevelopmentInstances**.

16\. Use a **Concurrency** of `10` and an **Error threshold** of `1`.

17\. For the **IAM service role** select **Use a custom service role** and select `ssm-requirements-RoleMaintenanceWindowTask-XXXXXXXXXXXXX`, it correspond to the output value of **ArnRoleMaintenanceWindowTask** for the **ssm-requirements** cloudformation stack.

18\. For the **Output options** check **Enable writing to S3** and type output value of **BucketMaintenanceWindowTaskOutput** for the **ssm-requirements** cloudformation stack.

19\. In Parameters, for the **Operation** select **Install**.

20\. Click on **Register Run command task**.

**For testing purpose, you can edit the Maitenance Window to be executed in a nearby time, and go to History and refresh the page until you can see the task in progress and completed, explore the details and outputs to verify the patch baseline applied. Also you can go to Run Command and you will see command executed for the AWS-RunPatchBaseline.**

For example, in the following image you can see the patch baseline applied for the Amazon Linux 2 instance, it should correspond to the patch baseline id created to be applied to instances with the tag **Patch Group** and value of **AmazonLinux2**.
![SSM Agent change](https://github.com/aurbac/aws-systems-manager/raw/master/images/output-patch-amazon-linux-2.png)