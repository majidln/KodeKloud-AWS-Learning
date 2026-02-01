# Day 05 - EBS Volume Creation

## Overview
This day focuses on creating an EBS (Elastic Block Store) volume using the AWS CLI, and learning how to list availability zones and existing volumes.

## Command Explanation

The following AWS CLI command creates a new EBS volume:

```bash
aws ec2 create-volume --size 2 --volume-type "gp3" --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=nautilus-volume}]'
```

### What does this command do?

This command creates a new EBS volume in your AWS account with the following parameters:

- **`aws ec2 create-volume`**: The AWS CLI command to create a new EBS volume
- **`--size 2`**: Specifies the size of the volume in GiB (2 GiB in this case)
- **`--volume-type "gp3"`**: Specifies the volume type as gp3 (General Purpose SSD)
- **`--availability-zone us-east-1a`**: Specifies the Availability Zone where the volume will be created
- **`--tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=nautilus-volume}]'`**: Adds a tag with the key "Name" and value "nautilus-volume" to the volume, which will display as the volume name in the AWS Console

### What is an EBS Volume?

An EBS (Elastic Block Store) volume is a durable, block-level storage device that you can attach to your EC2 instances. EBS volumes provide persistent storage that persists independently of the instance lifecycle.

## How to Use

### Step 1: List Available Availability Zones

Before creating a volume, you may want to see which Availability Zones are available in your region:

```bash
aws ec2 describe-availability-zones
```

This command returns a list of all Availability Zones in your current region.

### Step 2: Create the EBS Volume

**Please enter the following command in your terminal:**

```bash
aws ec2 create-volume --size 2 --volume-type "gp3" --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=nautilus-volume}]'
```

**Note:** 
- Replace `us-east-1a` with your desired Availability Zone if different.
- The `--tag-specifications` parameter sets the volume name to "nautilus-volume". You can change this name by modifying the `Value=nautilus-volume` part.

### Step 3: Verify Volume Creation

After creating the volume, you can list all volumes to verify it was created:

```bash
aws ec2 describe-volumes
```

This command returns information about all EBS volumes in your account, including VolumeId, Size, VolumeType, State, and AvailabilityZone.

**To filter volumes by specific criteria:**

```bash
# List volumes in a specific Availability Zone
aws ec2 describe-volumes --filters "Name=availability-zone,Values=us-east-1a"

# List volumes of a specific type
aws ec2 describe-volumes --filters "Name=volume-type,Values=gp3"

# List available (unattached) volumes
aws ec2 describe-volumes --filters "Name=status,Values=available"
```

### Important Notes:

1. **Availability Zone Matching**: EBS volumes can only be attached to EC2 instances in the same Availability Zone. Make sure the volume's AZ matches your instance's AZ.

2. **Volume Size**: Minimum size is 1 GiB, maximum is 16 TiB. The size can be increased after creation, but not decreased.

3. **Volume State**: After creation, the volume will be in `creating` state, then transition to `available` when ready. Only `available` volumes can be attached to instances.

4. **Cost Considerations**: You are charged for EBS volumes as long as they exist, even if not attached to an instance. Consider deleting unused volumes to avoid unnecessary costs.

5. **Prerequisites**: 
   - You must have AWS CLI installed and configured
   - You must have appropriate IAM permissions to create EBS volumes (`ec2:CreateVolume`)
   - You must specify a valid Availability Zone in your region

6. **Output**: The command will return JSON output containing the volume details. Save the `VolumeId` for attaching the volume to an instance.

## Additional Commands

### Attaching a Volume to an EC2 Instance

```bash
aws ec2 attach-volume --volume-id <VOLUME_ID> --instance-id <INSTANCE_ID> --device /dev/sdf
```

**Important**: The instance and volume must be in the same Availability Zone!

### Detaching a Volume

```bash
aws ec2 detach-volume --volume-id <VOLUME_ID>
```

### Deleting a Volume

```bash
aws ec2 delete-volume --volume-id <VOLUME_ID>
```

**Warning**: This permanently deletes the volume and all its data. Make sure you have backups (snapshots) if needed!
