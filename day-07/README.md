# Day 07 - EC2 Instance Management: Stop, Modify Type, and Start

## Overview
This day focuses on managing EC2 instances by listing them, stopping an instance, modifying its instance type, and starting it again. This is useful for changing instance types when you need different compute resources.

## Command Explanation

### Step 1: List EC2 Instances

First, you need to get a list of your EC2 instances to find the instance ID you want to modify:

```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], State.Name, InstanceId]' --output table
```

#### What does this command do?

This command lists all EC2 instances in a formatted table showing:
- **Instance Name**: The value of the "Name" tag (if present)
- **State**: Current state of the instance (running, stopped, stopping, etc.)
- **Instance ID**: The unique identifier for the instance

**Alternative simpler command:**
```bash
aws ec2 describe-instances
```
This returns full JSON output with all instance details, but is less readable.

### Step 2: Stop the Instance

Once you've identified the instance ID (e.g., for the `devops-ec2` instance), stop it:

```bash
aws ec2 stop-instances --instance-ids "i-05517985736d67be0"
```

#### What does this command do?

This command stops a running EC2 instance:
- **`aws ec2 stop-instances`**: The AWS CLI command to stop one or more EC2 instances
- **`--instance-ids "i-05517985736d67be0"`**: Specifies the instance ID(s) to stop. You can specify multiple instance IDs separated by spaces

**Important**: Replace `i-05517985736d67be0` with the actual instance ID of your `devops-ec2` instance.

### Step 3: Wait for Instance to Stop

After stopping the instance, you must wait for it to transition from `stopping` to `stopped` state before you can modify its instance type. You can check the status using:

```bash
aws ec2 describe-instances --instance-ids "i-05517985736d67be0" --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], State.Name]' --output table
```

The instance state will change from:
- `running` → `stopping` → `stopped`

### Step 4: Modify Instance Type

Once the instance is in `stopped` state, you can modify its instance type:

```bash
aws ec2 modify-instance-attribute --instance-id "i-05517985736d67be0" --instance-type t2.nano
```

#### What does this command do?

This command changes the instance type of a stopped EC2 instance:
- **`aws ec2 modify-instance-attribute`**: The AWS CLI command to modify instance attributes
- **`--instance-id "i-05517985736d67be0"`**: Specifies the instance ID to modify
- **`--instance-type t2.nano`**: Sets the new instance type. Common types include: t2.nano, t2.micro, t2.small, t2.medium, t3.micro, etc.

**Important**: 
- The instance **must be stopped** before you can change its type
- You cannot change the instance type while it's running
- Replace `t2.nano` with your desired instance type

### Step 5: Start the Instance

After modifying the instance type, start the instance again:

```bash
aws ec2 start-instances --instance-ids "i-05517985736d67be0"
```

#### What does this command do?

This command starts a stopped EC2 instance:
- **`aws ec2 start-instances`**: The AWS CLI command to start one or more EC2 instances
- **`--instance-ids "i-05517985736d67be0"`**: Specifies the instance ID(s) to start

The instance state will change from:
- `stopped` → `pending` → `running`
