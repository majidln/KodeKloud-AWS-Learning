# Day 09 - EC2 Instance Termination Protection

## Overview
This day focuses on enabling termination protection for an EC2 instance. Termination protection prevents accidental termination of critical instances through the AWS API, console, or CLI. This is useful for production instances that should not be terminated accidentally.

## Command Explanation

### Step 1: List EC2 Instances

```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], State.Name, InstanceId]' --output table
```

### Step 2: Enable Termination Protection

Once you've identified the instance ID, enable termination protection:

```bash
aws ec2 modify-instance-attribute \
    --instance-id i-1234567890abcdef0 \
    --disable-api-termination
```

#### What does this command do?

This command enables termination protection for an EC2 instance:
- **`aws ec2 modify-instance-attribute`**: The AWS CLI command to modify instance attributes
- **`--instance-id i-1234567890abcdef0`**: Specifies the instance ID to modify
- **`--disable-api-termination`**: Enables termination protection by setting the `disable-api-termination` attribute to `true` (which disables the API termination functionality, thus protecting the instance from being terminated)

**Important**: 
- Replace `i-1234567890abcdef0` with the actual instance ID of your instance
- When termination protection is enabled, the instance cannot be terminated through the AWS API, console, or CLI
- The instance can still be stopped (termination protection only prevents termination, not stopping)
- Termination protection can be enabled on both running and stopped instances

**Note**: The attribute name `disable-api-termination` can be confusing. When set to `true`, it disables the ability to terminate via API, which means termination protection is **enabled**.

### Step 3: Verify Termination Protection Status

To verify that termination protection has been enabled, check the instance attributes:

```bash
aws ec2 describe-instance-attribute --instance-id i-1234567890abcdef0 --attribute disableApiTermination
```

#### What does this command do?

This command retrieves the termination protection status of an instance:
- **`aws ec2 describe-instance-attribute`**: The AWS CLI command to describe instance attributes
- **`--instance-id i-1234567890abcdef0`**: Specifies the instance ID to check
- **`--attribute disableApiTermination`**: Queries the termination protection attribute

**Expected Output:**
- If termination protection is **enabled**: `"Value": true` (meaning API termination is disabled, so termination is protected)
- If termination protection is **disabled**: `"Value": false` (meaning API termination is enabled, so termination is allowed)

### Step 4: Disable Termination Protection (Optional)

If you need to disable termination protection later (e.g., to terminate the instance), use:

```bash
aws ec2 modify-instance-attribute \
    --instance-id i-1234567890abcdef0 \
    --no-disable-api-termination
```

This removes termination protection and allows the instance to be terminated normally.

## Important Notes

1. **Termination Protection vs Stop Protection**: 
   - Termination protection prevents **terminating** an instance
   - Stop protection (a separate feature) prevents **stopping** an instance
   - Both can be enabled independently on the same instance

2. **When to Use Termination Protection**:
   - Production instances that should not be accidentally terminated
   - Critical workloads that must not be permanently deleted
   - Instances that hold important state or data

3. **Limitations**:
   - Termination protection does not prevent stopping
   - Termination protection does not prevent instance failures or hardware issues
   - You must disable termination protection before you can terminate the instance

4. **Prerequisites**: 
   - You must have AWS CLI installed and configured
   - You must have appropriate IAM permissions (`ec2:ModifyInstanceAttribute` and `ec2:DescribeInstanceAttribute`)
