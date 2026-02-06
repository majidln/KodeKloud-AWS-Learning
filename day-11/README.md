## Day 11 - Attaching an Existing Network Interface (ENI) to an EC2 Instance

### Overview

This day focuses on attaching an existing Elastic Network Interface (ENI) to an EC2 instance using the AWS CLI, and verifying that the attachment is successful.

### Attach Command

The following AWS CLI command attaches an existing ENI to an EC2 instance:

```bash
aws ec2 attach-network-interface \
  --network-interface-id eni-0e38ed8b4b807579a \
  --instance-id i-0ff75ec06bd060dd1 \
  --device-index 1
```

### Command Breakdown

- **`aws ec2 attach-network-interface`**: AWS CLI operation that attaches an existing network interface to an EC2 instance.
- **`--network-interface-id`**: The ID of the ENI you want to attach (example: `eni-0e38ed8b4b807579a`).
- **`--instance-id`**: The ID of the EC2 instance to attach the ENI to (example: `i-0ff75ec06bd060dd1`).
- **`--device-index`**:
  - Determines the index of this network interface on the instance.
  - `0` is normally the primary interface created with the instance.
  - `1`, `2`, etc. are used for additional interfaces.
  - In this example, `1` means this ENI will be the **second** network interface on the instance.

### Prerequisites

Before running the attach command, ensure:

- **AWS CLI** is installed and configured (`aws configure` completed).
- Your IAM principal has permissions:
  - `ec2:AttachNetworkInterface`
  - `ec2:DescribeNetworkInterfaces`
  - `ec2:DescribeInstances`
- The **ENI** (`eni-0e38ed8b4b807579a`) and **instance** (`i-0ff75ec06bd060dd1`) are:
  - In the **same AWS Region**.
  - In the **same Availability Zone**.
- The instance has a free **device index** available (e.g. `1`).

### Step 1: Confirm ENI Status and Availability Zone

```bash
aws ec2 describe-network-interfaces \
  --network-interface-ids eni-0e38ed8b4b807579a
```

Verify that:

- `Status` is **`available`** (not already attached).
- `AvailabilityZone` matches the Availability Zone of your EC2 instance.

### Step 2: Confirm Instance Availability Zone

```bash
aws ec2 describe-instances \
  --instance-ids i-0ff75ec06bd060dd1 \
  --query "Reservations[0].Instances[0].Placement.AvailabilityZone" \
  --output text
```

Make sure this Availability Zone is the same as the ENI’s.

### Step 3: Attach the ENI

Run the attach command:

```bash
aws ec2 attach-network-interface \
  --network-interface-id eni-0e38ed8b4b807579a \
  --instance-id i-0ff75ec06bd060dd1 \
  --device-index 1
```

If successful, the output will include an attachment ID, for example:

```json
{
  "AttachmentId": "eni-attach-0123456789abcdef0"
}
```

Save this **AttachmentId** if you might need to detach the ENI later.

### Step 4: Verify the ENI is Attached

#### Option A: Check from the ENI side

```bash
aws ec2 describe-network-interfaces \
  --network-interface-ids eni-0e38ed8b4b807579a \
  --query "NetworkInterfaces[0].{Status:Status,InstanceId:Attachment.InstanceId,DeviceIndex:Attachment.DeviceIndex}" \
  --output table
```

You should see:

- **Status**: `in-use`
- **InstanceId**: `i-0ff75ec06bd060dd1`
- **DeviceIndex**: `1`

#### Option B: Check from the instance side

```bash
aws ec2 describe-instances \
  --instance-ids i-0ff75ec06bd060dd1 \
  --query "Reservations[0].Instances[0].NetworkInterfaces[].{Id:NetworkInterfaceId,DeviceIndex:Attachment.DeviceIndex}" \
  --output table
```

Look for your ENI (`eni-0e38ed8b4b807579a`) with `DeviceIndex` set to `1`.

#### Option C: Check in the AWS Console

- Go to **EC2 Console → Instances → select your instance**.
- Open the **Networking** tab.
- Under **Network interfaces**, confirm:
  - The ENI `eni-0e38ed8b4b807579a` is listed.
  - Its **Status** is `in-use`.
  - Its **Device index** is `1`.

### Detaching the Network Interface (Optional)

If you later need to detach the ENI:

1. **Get the attachment ID** (if you didn’t save it earlier):

   ```bash
   aws ec2 describe-network-interfaces \
     --network-interface-ids eni-0e38ed8b4b807579a \
     --query "NetworkInterfaces[0].Attachment.AttachmentId" \
     --output text
   ```

2. **Detach the ENI**:

   ```bash
   aws ec2 detach-network-interface \
     --attachment-id eni-attach-0123456789abcdef0
   ```

   You can add `--force` if necessary, but be aware this may disrupt traffic.

### Common Errors and Fixes

- **`InvalidParameterValue: networkInterface and instance are in different availability zones`**
  - **Cause**: ENI and instance are not in the same AZ.
  - **Fix**: Use an ENI in the same AZ as the instance, or choose an instance in the ENI’s AZ.

- **`InvalidParameterCombination: Network interface is currently in use`**
  - **Cause**: ENI is already attached to another instance.
  - **Fix**: Detach the ENI from its current instance before re-attaching.

- **`Resource.AlreadyAssociated` or `InvalidDeviceIndex`**
  - **Cause**: The chosen `device-index` is already taken, or not allowed for this instance type.
  - **Fix**: Choose another device index (e.g. `2`, `3`) within the limits of the instance type.

- **`UnauthorizedOperation`**
  - **Cause**: Your IAM role/user lacks necessary EC2 permissions.
  - **Fix**: Ensure your IAM policy includes `ec2:AttachNetworkInterface` and relevant `Describe*` actions.

### Notes & Best Practices

- **Security groups and subnet** associated with the ENI define its network behavior. Review these before attaching to production instances.
- ENIs can be used for **high availability** and **failover** by moving them between instances (detach from one, attach to another).
- Always make sure you are operating in the **correct Region** using `aws configure list` or by setting `AWS_REGION` / `--region` explicitly.

