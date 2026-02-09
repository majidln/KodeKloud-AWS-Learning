# Day 12 - Attach EBS Volume to EC2 Instance

## Overview

Attach an existing EBS volume to an EC2 instance in the same Availability Zone. The volume appears as a block device (e.g. `/dev/sdb`) on the instance.

## Command

```bash
aws ec2 attach-volume --device "/dev/sdb" --instance-id "<instance-id>" --volume-id "<volume-id>"
```

- **`--device`**: Block device name on the instance (e.g. `/dev/sdb`).
- **`--instance-id`**: EC2 instance ID.
- **`--volume-id`**: EBS volume ID.

Volume and instance must be in the **same Availability Zone**. The volume must be in `available` state.

## Verify

```bash
aws ec2 describe-volumes --volume-ids <volume-id> --query 'Volumes[*].Attachments[*].[State, InstanceId, Device]' --output table
```

## Detach (optional)

```bash
aws ec2 detach-volume --volume-id <volume-id>
```

Unmount the volume on the instance before detaching.
