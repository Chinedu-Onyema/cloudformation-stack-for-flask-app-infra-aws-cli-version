# AWS CloudFormation Deployment Guide with AWS CLI: Flask App Infrastructure

This is a complete step-by-step guide for deploying and managing the AWS infrastructure for a Flask Employee Directory Application using the AWS CLI, WSL (Windows Subsystem for Linux), and CloudFormation templates.

### PDF GUIDE: [CREATE A CLOUDFORMATION STACK FOR YOUR FLASK APP INFRASTRUCTURE WITH THE COMMAND LINE.pdf](https://github.com/user-attachments/files/32284547/CREATE.A.CLOUDFORMATION.STACK.FOR.YOUR.FLASK.APP.INFRASTRUCTURE.WITH.THE.COMMAND.LINE.pdf)

### WATCH VIDEO WALKTHROUGH HERE: https://youtu.be/BNAkguhBMYk

## PREREQUISITES
OS: Windows Subsystem for Linux (WSL) or Linux/macOS terminal environment.

AWS CLI: Version 2.x installed locally.

IAM Permissions: Access key and secret key for an IAM user (e.g., onyema_2) authorized to manage EC2, RDS, VPC, S3, and CloudFormation resources.


## ENVIRONMENT SETUP & IAM AUTHENTICATION
1) Launch WSL on your machine.

2) Verify AWS CLI installation and location:
<PRE>aws --version</PRE>
<PRE>which aws</PRE>

3) Configure your AWS credentials:
<PRE>aws configure</PRE>

4) Enter your Access Key, Secret Key, Default Region (e.g., eu-north-1), and Output Format (json).

5) Validate identity and API access:


## GATHERING INFRASTRUCTURE PARAMETERS
Run the following CLI commands to query your target AWS environment for necessary resource IDs:

1) Retrieve Latest Amazon Linux 2023 AMI ID

<PRE>aws ec2 describe-images --owners amazon --filters "Name=name,Values=al2023-ami-2023*" "Name=architecture,Values=x86_64" "Name=virtualization-type,Values=hvm" --query 'Images | sort_by(@, &CreationDate) | [-1].[ImageId,Name,CreationDate]' --output table --region eu-north-1</PRE>


2) Retrieve VPC ID

<PRE>aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId, CidrBlock, Tags[?Key==`Name`].Value|[0]]' --output table</PRE>


3) Identify Public Subnet ID (For EC2)
Select a public subnet with MapPublicIpOnLaunch set to true:

<PRE>aws ec2 describe-subnets --region eu-north-1 --query 'Subnets[*].{SubnetId:SubnetId, VPC:VpcId, AZ:AvailabilityZone, CIDR:CidrBlock, AutoAssignPublicIP:MapPublicIpOnLaunch, Name:Tags[?Key==`Name`].Value|[0]}' --output table</PRE>


4) Identify Private Subnet IDs (For RDS Multi-AZ / Subnet Group)
Select two private subnets (where AutoAssignPublicIP is false):

<PRE>aws ec2 describe-subnets --region eu-north-1 --query 'Subnets[*].{SubnetId:SubnetId, VPC:VpcId, AZ:AvailabilityZone, CIDR:CidrBlock, AutoAssignPublicIP:MapPublicIpOnLaunch, Name:Tags[?Key==`Name`].Value|[0]}' --output table</PRE>


5) Create or update your parameters_file.json in your working directory with the extracted IDs:


## STACK DEPLOYMENT & VALIDATION

1) Execute Stack Provisioning
Deploy the stack using create-stack. Set --on-failure DELETE to automatically roll back if errors occur during creation:

<PRE>aws cloudformation create-stack --stack-name employee-application-infrastructure --template-body file://'application_infrastructure_deployment_CLI.yml' --parameters file://'parameters_file.json' --capabilities CAPABILITY_IAM --region eu-north-1 --on-failure DELETE</PRE>


2) Monitor Creation Progress
Block and pause the terminal until provisioning is fully completed:

<PRE>aws cloudformation wait stack-create-complete --stack-name employee-application-infrastructure --region eu-north-1</PRE>


3) Confirm Stack Status
Verify that the stack output reads CREATE_COMPLETE:


4) Inspect Provisioned Resources
Check active compute and database endpoints:

<PRE>aws ec2 describe-instances --output table</PRE>
<PRE>aws rds describe-db-instances --output table</PRE>


## INFRASTRUCTURE TEARDOWN & SECURITY CLEANUP

1) Delete CloudFormation Stack
Decommission all stack resources automatically:

<PRE>aws cloudformation delete-stack --stack-name employee-application-infrastructure --region eu-north-1</PRE>


2) Verify Resource Deletion
Confirm active EC2 and RDS instances are terminated:

<PRE>aws ec2 describe-instances</PRE>
<PRE>aws rds describe-db-instances</PRE>


3) Logout & Purge AWS CLI Session
Remove stored IAM credentials from your local machine:

<PRE>rm ~/.aws/credentials</PRE>

Edit ~/.aws/config to clean up remaining profile blocks, then verify identity revokement:



