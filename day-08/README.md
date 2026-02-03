# Day 08 - EC2 Instance Stop Protection

## Overview
This day focuses on enabling stop protection for an EC2 instance. Stop protection prevents accidental stopping of critical instances through the AWS API, console, or CLI. This is useful for production instances that should not be stopped accidentally.

## Command Explanation

### Step 1: List EC2 Instances

```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], State.Name, InstanceId]' --output table
```

### Step 2: Enable Stop Protection

Once you've identified the instance ID, enable stop protection:

```bash
aws ec2 modify-instance-attribute \
    --instance-id i-1234567890abcdef0 \
    --disable-api-stop
```

#### What does this command do?

This command enables stop protection for an EC2 instance:
- **`aws ec2 modify-instance-attribute`**: The AWS CLI command to modify instance attributes
- **`--instance-id i-1234567890abcdef0`**: Specifies the instance ID to modify
- **`--disable-api-stop`**: Enables stop protection by setting the `disable-api-stop` attribute to `true` (which disables the API stop functionality, thus protecting the instance from being stopped)

**Important**: 
- Replace `i-1234567890abcdef0` with the actual instance ID of your instance
- When stop protection is enabled, the instance cannot be stopped through the AWS API, console, or CLI
- The instance can still be terminated (stop protection only prevents stopping, not termination)
- Stop protection can be enabled on both running and stopped instances

**Note**: The attribute name `disable-api-stop` can be confusing. When set to `true`, it disables the ability to stop via API, which means stop protection is **enabled**.

### Step 3: Verify Stop Protection Status

To verify that stop protection has been enabled, check the instance attributes:

```bash
aws ec2 describe-instance-attribute --instance-id i-1234567890abcdef0 --attribute disableApiStop
```

#### What does this command do?

This command retrieves the stop protection status of an instance:
- **`aws ec2 describe-instance-attribute`**: The AWS CLI command to describe instance attributes
- **`--instance-id i-1234567890abcdef0`**: Specifies the instance ID to check
- **`--attribute disableApiStop`**: Queries the stop protection attribute

**Expected Output:**
- If stop protection is **enabled**: `"Value": true` (meaning API stop is disabled, so stopping is protected)
- If stop protection is **disabled**: `"Value": false` (meaning API stop is enabled, so stopping is allowed)

### Step 4: Disable Stop Protection (Optional)

If you need to disable stop protection later, use:

```bash
aws ec2 modify-instance-attribute \
    --instance-id i-1234567890abcdef0 \
    --no-disable-api-stop
```

This removes stop protection and allows the instance to be stopped normally.

## Important Notes

1. **Stop Protection vs Termination Protection**: 
   - Stop protection prevents **stopping** an instance
   - Termination protection (a separate feature) prevents **terminating** an instance
   - Both can be enabled independently

2. **When to Use Stop Protection**:
   - Production instances that should not be accidentally stopped
   - Critical workloads that must remain running
   - Instances that are part of high-availability configurations

3. **Limitations**:
   - Stop protection does not prevent termination
   - Stop protection does not prevent instance failures or hardware issues
   - You can still modify other instance attributes when stop protection is enabled

4. **Prerequisites**: 
   - You must have AWS CLI installed and configured
   - You must have appropriate IAM permissions (`ec2:ModifyInstanceAttribute` and `ec2:DescribeInstanceAttribute`)
