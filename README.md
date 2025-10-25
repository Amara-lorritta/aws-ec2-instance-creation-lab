## **Creating Amazon EC2 Instances Lab**

## **Overview**

In this lab, I learned how to launch and configure Amazon EC2 instances using both the AWS Management Console and the AWS Command Line Interface (CLI).
You first created a bastion host manually, then connected securely using EC2 Instance Connect and launched a web server instance through the AWS CLI.
This lab demonstrates the fundamental workflow of provisioning compute resources, applying IAM roles, and automating deployment using command-line tools.

## **Objectives & Learning Outcomes**

After completing this lab, I will be able to:

Launch an EC2 instance using the AWS Management Console.

Securely connect to an instance via EC2 Instance Connect.

Use the AWS CLI to automate instance creation.

Retrieve required metadata (AMI ID, Subnet ID, Security Group ID).

Configure and deploy a web server through a user data script.

## **Architecture**

Architecture Summary:
A bastion host is manually launched in a public subnet within a VPC.
I connected to it using EC2 Instance Connect, then use the AWS CLI to programmatically launch another instance — the web server — using metadata and a user-data bootstrap script.
Flow:
User → AWS Console (Bastion Host) → EC2 Instance Connect (SSH access) → AWS CLI commands → Web Server in VPC

<img width="1000" height="600" alt="954edf77-5374-424e-b573-15b9b9bec915" src="https://github.com/user-attachments/assets/be78ba60-6c07-485d-93b2-ed24df7ac7fd" />


## **Commands and Steps**

```bash
# Step 1: Set Region and Retrieve Latest Amazon Linux 2 AMI
AZ=`curl -s http://***********/latest/meta-data/placement/availability-zone`
export AWS_DEFAULT_REGION=${AZ::-1}
AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
echo $AMI

# Step 2: Retrieve Subnet ID for Public Subnet
SUBNET=$(aws ec2 describe-subnets --filters 'Name=tag:Name,Values=Public Subnet' --query Subnets[].SubnetId --output text)
echo $SUBNET

# Step 3: Retrieve Security Group ID for Web Security Group
SG=$(aws ec2 describe-security-groups --filters Name=group-name,Values=WebSecurityGroup --query SecurityGroups[].GroupId --output text)
echo $SG

# Step 4: Download the User Data Script for Web Server Setup
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-1-23732/171-lab-JAWS-create-ec2/s3/UserData.txt
cat UserData.txt

# Step 5: Launch EC2 Web Server Instance
INSTANCE=$(
aws ec2 run-instances \
--image-id $AMI \
--subnet-id $SUBNET \
--security-group-ids $SG \
--user-data file:///home/ec2-user/UserData.txt \
--instance-type t3.micro \
--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Web Server}]' \
--query 'Instances[*].InstanceId' \
--output text \
)
echo $INSTANCE

# Step 6: Wait Until Instance is Running
aws ec2 describe-instances --instance-ids $INSTANCE --query 'Reservations[].Instances[].State.Name' --output text

# Step 7: Get Public DNS of Web Server
aws ec2 describe-instances --instance-ids $INSTANCE --query Reservations[].Instances[].PublicDnsName --output text

```
## **Screenshots**

EC2 Console Launch	EC2 instance launched manually through AWS Management Console.
<img width="1638" height="269" alt="ec2 instance launched 1" src="https://github.com/user-attachments/assets/ec1c9d24-0d8f-4796-a034-c4a822a80222" />

EC2 Instance Connect	Secure connection to Bastion Host using EC2 Instance Connect.
<img width="1299" height="359" alt="ec2 instance connect" src="https://github.com/user-attachments/assets/d4768edc-83c6-42d6-a12e-7368c6f4d687" />

Running Instances	Bastion Host and Web Server visible in EC2 console.
<img width="1552" height="300" alt="Running Instance" src="https://github.com/user-attachments/assets/c1157d22-bf15-4e3c-a80e-83b655109db0" />

Web Server Output	Successfully loaded the hosted web page using Public DNS URL.
<img width="1000" height="468" alt="new web server" src="https://github.com/user-attachments/assets/ccd2690e-00f1-47a1-bd86-6ecf1f1c56f6" />


## **Tools Used**

Amazon EC2 – Compute resource hosting

AWS Management Console – Manual instance provisioning

AWS CLI – Automated instance creation

Amazon VPC – Network isolation

IAM Roles – Permissions and access management

Amazon Linux 2 – OS for Bastion and Web Server


## **What Actually Happened** 

Launched Bastion Host manually in a public subnet using the AWS Management Console.

Connected via EC2 Instance Connect to establish SSH access without key pairs.

Retrieved latest AMI, subnet, and security group details using AWS CLI commands.

Downloaded and reviewed User Data Script, which installed Apache and the sample web application.

Launched Web Server EC2 Instance programmatically using AWS CLI with proper configurations.

Verified Instance Status to confirm it was running successfully.

Accessed Web Application using the instance’s public DNS — confirming successful deployment.


## **Author**

Amarachi Emeziem

Cloud Security Engineer

LinkedIn: https://www.linkedin.com/in/amarachilemeziem/
