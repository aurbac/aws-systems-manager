# Systems Manager

Hello, this is a self-paced workshop designed explore the main features in AWS Systems Manager.

## Let's begin preparing the environment

a\. Log into the AWS Management Console and choose the preferred [AWS region](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html).

b\. [Firstly, we need to set up AWS Config to record configuration changes of our resources.](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html)

c\. [We need to create a Keypair to log in to the ec2 instances.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair)

d\. Launch the CloudFormation template two times for your selected region, the first one for the production environment and the second one for the development environment, [download the template to create the resources stacks](https://raw.githubusercontent.com/aurbac/aws-systems-manager/master/AURBAC-AWS-Systems-Manager-Resources.yaml).

### Creating two CloudFormation stacks

1\. Go to CloudFormation service and click on **Create Stack**.

2\. Use the name `production-resources` or `development-resources` to identify the cloudformation stacks.

3\. Select your KeyPair created previously, click **Next**.

4\. For the Options, add the tag key `Environment` and the value `Production` or `Development` depending on the stack to be created, click **Next**.

5\. Check the box `I acknowledge that AWS CloudFormation might create IAM resources.` and click **Create**.

### AWS Services created

* Amazon VPC environment with two public subnets
* EC2 instance Amazon Linux 2 (t2.micro) with IAM Role
* EC2 instance Windows 2012 (t2.medium) with IAM Role
* EC2 instance Windows 2016 (t2.medium) with IAM Role
* The IAM Role has the policy required for Systems Manager: **AmazonEC2RoleforSSM**

## Resource Groups

1\. Go to Systems Manager service en click on **Find Resources** under Resource Groups section.

2\. On the resource apply a filter using the tag key `Environment` and the value `Production` or `Development` and click **View query results**.

3\. You will see the resources related to the environment, click on **Save query as group** and name the group as `Production-Services` or `Development-Services` and **Create group**.

## Inventory

1\. Go to Systems Manager service en click on **Inventory** under Insights section.

2\. Click on **Setup Inventory**.

3\. Use the name `Inventory-Association-For-Production` or `Inventory-Association-For-Development`.

4\. In Targets section click on **Specifying a tag** and use the tag key `Environment` and the value `Production` or `Development` and click **Setup Inventory**.

5\. Click on **State Manager** under Actions section, you will see the Associations created to collect the information for every environment and the Last execution.

**NOTE:** Aditionally, you can create an inventory that search for **Files** or the **Windows Registry**, [example configuration in the Configuring Collection section in step 6](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-inventory-configuring.html).

## Run Command to Update SSM Agent

1\. Go to Systems Manager service en click on **Run Command** under Actions section and click on **Run Command** button.

2\. On the Run a command page, click in the search bar and select, **Document name prefix**, then click on **Equal**, then type in `AWS-UpdateSSMAgent` and enter.

3\. Now select the **AWS-UpdateSSMAgent** document name that will upgrade Systems Management agent on the instances.

4\. On Targets, select **Specifying a tag** and apply a filter using the tag key `Environment` and the value `Development`, click on **Add**.

5\. Scroll down and click **Run**.

6\. Next you will see page documenting your running command then and overall success in green.

#### AWS Config: Configuration change

1\. Go to **Inventory** under Insights section, scroll down and on Corresponding managed instances and click in the AWS Config button for the `development-resources-AmazonLinux2` instance.

![Inventory instances](https://github.com/aurbac/aws-systems-manager/raw/master/images/inventory-instances-config.png)

2\. In the next page you will see a timeline changes for the instance, the last change will be selected, scroll down and expand the **Changes** section to see the change version for the SSM Agent.

![SSM Agent change](https://github.com/aurbac/aws-systems-manager/raw/master/images/config-change.png)

## Patching

1\. Go to Systems Manager service en click on **Maintenance Windows** under Actions section and click on **Create Maintenance Window** button.

2\. Use the name `MaitenanceWindows2012Prod`.

3\. Schedule the maitenance for every day at the desired time.

4\. Specify a duration of `4` and stop initiating tasks of `1`.

5\. Click on **Create maintenance window**.

6\. Click on **Patch manager** under Actions section and click on **Configure patching** button.

7\. On instances to patch check **Select instances manually** and select the **production-resources-Windows2012** instance.

8\. On Patching schedule select the existing Maintenance Window created before **MaitenanceWindows2012Prod**.

9\. For the patching operation use **Scan and install** and click on **Configure patching**.

10\. Now you can come back to **Maintenance Windows** and click on the Windows ID for **MaitenanceWindows2012Prod**.

11\. You can see the details for the maintenance window, the tasks, targets and the history of executions.
