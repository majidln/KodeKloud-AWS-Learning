# Day 06 - EC2 Instance Creation

## Overview
This day focuses on creating an EC2 (Elastic Compute Cloud) instance using the AWS CLI.

## Command Explanation

The following AWS CLI command creates a new EC2 instance:

```bash
aws ec2 run-instances --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-ec2}]' --image-id "ami-0532be01f26a3de55" --instance-type "t2.micro" --key-name "datacenter-kp"
```

### What does this command do?

This command launches a new EC2 instance in your AWS account with the following parameters:

- **`aws ec2 run-instances`**: The AWS CLI command to launch a new EC2 instance
- **`--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-ec2}]'`**: Adds a tag with the key "Name" and value "datacenter-ec2" to the instance, which will display as the instance name in the AWS Console
- **`--image-id "ami-0532be01f26a3de55"`**: Specifies the Amazon Machine Image (AMI) ID to use for the instance. This AMI contains the operating system and any pre-configured software
- **`--instance-type "t2.micro"`**: Specifies the instance type as t2.micro, which is a low-cost, general-purpose instance type eligible for the AWS Free Tier
- **`--key-name "datacenter-kp"`**: Specifies the name of the key pair to use for SSH access to the instance. This key pair must already exist in your AWS account

### What is an EC2 Instance?

An EC2 (Elastic Compute Cloud) instance is a virtual server in AWS's cloud. EC2 instances provide resizable compute capacity in the cloud and can be used for a wide variety of applications, from hosting websites to running complex applications.

## How to Use

**Please enter the following command in your terminal:**

```bash
aws ec2 run-instances --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-ec2}]' --image-id "ami-0532be01f26a3de55" --instance-type "t2.micro" --key-name "datacenter-kp"
```

**Note:** 
- Replace `ami-0532be01f26a3de55` with a valid AMI ID for your region if this AMI doesn't exist in your region
- Replace `datacenter-kp` with your actual key pair name if different
- The `--tag-specifications` parameter sets the instance name to "datacenter-ec2". You can change this name by modifying the `Value=datacenter-ec2` part

### Step 1: Verify Prerequisites

Before creating an instance, ensure you have:

1. **A valid key pair**: The key pair specified in `--key-name` must exist in your AWS account
   ```bash
   aws ec2 describe-key-pairs --key-names datacenter-kp
   ```

2. **A valid AMI ID**: The AMI ID must exist in your region. You can find available AMIs:
   ```bash
   aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*" --query 'Images[*].[ImageId,Name]' --output table
   ```

### Step 2: Launch the Instance

Run the command to launch your EC2 instance.

### Step 3: Verify Instance Creation

After launching the instance, you can list all instances to verify it was created:

```bash
aws ec2 describe-instances
```

This command returns information about all EC2 instances in your account, including InstanceId, InstanceType, State, and Tags.

**To filter instances by specific criteria:**

```bash
# List instances by name tag
aws ec2 describe-instances --filters "Name=tag:Name,Values=datacenter-ec2"

# List running instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"

# List instances by instance type
aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro"
```

### Important Notes:

1. **AMI ID Region Specificity**: AMI IDs are region-specific. The AMI ID `ami-0532be01f26a3de55` may not exist in all regions. Make sure to use an AMI ID that exists in your current region, or specify the region:
   ```bash
   aws ec2 run-instances --region us-east-1 --image-id "ami-0532be01f26a3de55" ...
   ```

2. **Key Pair Requirement**: The key pair specified must already exist in your AWS account. If you haven't created it yet, refer to Day 01 for instructions on creating a key pair.

3. **Instance State**: After creation, the instance will be in `pending` state, then transition to `running` when ready. It may take a few minutes for the instance to fully boot.

4. **Security Groups**: This command doesn't specify a security group. AWS will use the default security group for your VPC. You may want to specify a security group using `--security-group-ids` for better control.

5. **Cost Considerations**: 
   - t2.micro instances are eligible for the AWS Free Tier (750 hours/month for 12 months for new AWS accounts)
   - You are charged for instances while they are running, even if idle
   - Consider stopping or terminating instances when not in use to avoid unnecessary costs

6. **Prerequisites**: 
   - You must have AWS CLI installed and configured
   - You must have appropriate IAM permissions to launch EC2 instances (`ec2:RunInstances`)
   - The key pair must exist in your AWS account
   - The AMI ID must be valid in your region

7. **Output**: The command will return JSON output containing the instance details. Save the `InstanceId` for future reference and management.

## Additional Commands

### Getting Instance Details

```bash
# Get details of a specific instance
aws ec2 describe-instances --instance-ids <INSTANCE_ID>
```

### Starting a Stopped Instance

```bash
aws ec2 start-instances --instance-ids <INSTANCE_ID>
```

### Stopping a Running Instance

```bash
aws ec2 stop-instances --instance-ids <INSTANCE_ID>
```

### Terminating an Instance

```bash
aws ec2 terminate-instances --instance-ids <INSTANCE_ID>
```

**Warning**: This permanently terminates the instance and all its data. Make sure you have backups if needed!

### Connecting to Your Instance via SSH

Once your instance is running, you can connect to it using SSH:

```bash
ssh -i /path/to/your-key.pem ec2-user@<PUBLIC_IP_OR_DNS>
```

Replace:
- `/path/to/your-key.pem` with the path to your private key file
- `<PUBLIC_IP_OR_DNS>` with your instance's public IP address or public DNS name
- `ec2-user` with the appropriate username for your AMI (common usernames: `ec2-user` for Amazon Linux, `ubuntu` for Ubuntu, `admin` for Debian)
