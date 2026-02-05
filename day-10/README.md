# Day 10 - Associate Elastic IP with EC2 Instance

## Overview
This day focuses on associating an Elastic IP (EIP) address with an EC2 instance using the AWS CLI. An Elastic IP is a static, public IPv4 address that you can allocate to your account and associate with an instance. Unlike a regular public IP that may change when an instance is stopped and started, an Elastic IP remains the same until you disassociate or release it.

## Command Explanation

### Step 1: List EC2 Instances (Optional)

To identify the instance you want to associate an EIP with:

```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], State.Name, InstanceId, PublicIpAddress]' --output table
```

### Step 2: List Elastic IPs (Optional)

To see allocated Elastic IPs and their allocation IDs:

```bash
aws ec2 describe-addresses --query 'Addresses[*].[PublicIp, AllocationId, AssociationId, InstanceId]' --output table
```

### Step 3: Associate Elastic IP with Instance

Associate an Elastic IP with your EC2 instance:

```bash
aws ec2 associate-address --instance-id i-06e137dfb949eb55e --allocation-id eipalloc-0305a9055062d3b38
```

#### What does this command do?

This command associates an Elastic IP address with an EC2 instance:
- **`aws ec2 associate-address`**: The AWS CLI command to associate an EIP with an instance (or network interface)
- **`--instance-id i-06e137dfb949eb55e`**: The ID of the EC2 instance to which the Elastic IP will be assigned
- **`--allocation-id eipalloc-0305a9055062d3b38`**: The allocation ID of the Elastic IP (returned when you allocate an EIP in a VPC; required for VPC EIPs)

**Important**:
- Replace `i-06e137dfb949eb55e` with your target instance ID
- Replace `eipalloc-0305a9055062d3b38` with your Elastic IP allocation ID
- The instance and the EIP must be in the same region
- If the instance already has an Elastic IP, you must disassociate it first (or use a different EIP)
- In EC2-Classic (legacy), you use `--public-ip` instead of `--allocation-id`; in VPC you use `--allocation-id`

**Expected output:** The command returns an `AssociationId` if successful, which you can use later to disassociate the address.

### Step 4: Verify the Association

Check that the EIP is associated with the instance:

```bash
aws ec2 describe-addresses --allocation-ids eipalloc-0305a9055062d3b38
```

Or describe the instance to see its public IP:

```bash
aws ec2 describe-instances --instance-ids i-06e137dfb949eb55e --query 'Reservations[*].Instances[*].[PublicIpAddress, InstanceId]' --output table
```

### Step 5: Disassociate Elastic IP (Optional)

To disassociate the Elastic IP from the instance (the EIP remains allocated to your account):

```bash
aws ec2 disassociate-address --association-id <association-id>
```

Get the `association-id` from the output of `associate-address` or from `describe-addresses`.

## Important Notes

1. **Elastic IP behavior**:
   - One Elastic IP can be associated with one instance (or network interface) at a time
   - Associating a new EIP to an instance that already has an EIP will replace the previous association
   - Elastic IPs are charged when allocated but not associated with a running instance

2. **When to use Elastic IPs**:
   - When you need a stable public IP for DNS, firewalls, or whitelisting
   - For instances that may be stopped and started but must keep the same public IP
   - For failover scenarios (associate the same EIP to a different instance)

3. **Limitations**:
   - Default limit of 5 Elastic IPs per region (can be increased via service quota)
   - EIP and instance must be in the same region and (for VPC) same network scope

4. **Prerequisites**:
   - AWS CLI installed and configured
   - Appropriate IAM permissions (e.g. `ec2:AssociateAddress`, `ec2:DescribeAddresses`, `ec2:DescribeInstances`)
