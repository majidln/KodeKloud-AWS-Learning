## Day 21 - Launch EC2 Instance and Assign Elastic IP

### Overview

This day covers **launching an EC2 instance with the latest Ubuntu 22.04 AMI** and **associating an Elastic IP** to it using the AWS CLI.

The steps are:
1. Find the latest Ubuntu 22.04 AMI ID
2. Launch an EC2 instance with that AMI
3. Allocate an Elastic IP address
4. Associate the Elastic IP with the instance

---

### Step 1: Find the Latest Ubuntu 22.04 AMI

```bash
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId" \
  --output text
```

#### Command Breakdown

- **`--owners 099720109477`**: Filters images owned by Canonical (the official Ubuntu publisher on AWS).
- **`--filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"`**: Narrows results to HVM SSD-backed Ubuntu 22.04 64-bit server images.
- **`--query "sort_by(Images, &CreationDate)[-1].ImageId"`**: Sorts all matching images by creation date and returns only the newest one's ID.
- **`--output text`**: Returns a plain string (e.g. `ami-09757e8c9b2ba5eef`) ready to use in the next command.

---

### Step 2: Launch the EC2 Instance

```bash
aws ec2 run-instances \
  --instance-type t2.micro \
  --image-id ami-09757e8c9b2ba5eef \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]'
```

Note the **instance ID** from the output (e.g. `i-0dcc7aada477b517d`) — you will need it in Step 4.

---

### Step 3: Allocate an Elastic IP

```bash
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=xfusion-eip}]' \
  --query "AllocationId" \
  --output text
```


### Step 4: Associate the Elastic IP with the Instance

```bash
aws ec2 associate-address \
  --instance-id i-0dcc7aada477b517d \
  --allocation-id eipalloc-0d958f59383ca3dd7
```

#### Example Output

```json
{
    "AssociationId": "eipassoc-0922dac95ecababaf"
}
```

The `AssociationId` confirms the Elastic IP is now linked to the instance.

---

### Important Notes

- **Elastic IP cost**: An Elastic IP is free while associated with a running instance. You are charged if it is allocated but unattached, or if the instance is stopped.
- **Default VPC**: If no `--subnet-id` or `--security-group-ids` are specified, `run-instances` uses the default VPC and security group.

---

### Summary

1. **Get the latest Ubuntu 22.04 AMI ID**:
   ```bash
   aws ec2 describe-images \
     --owners 099720109477 \
     --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*" \
     --query "sort_by(Images, &CreationDate)[-1].ImageId" \
     --output text
   ```
2. **Launch the instance** using that AMI ID.
3. **Allocate an Elastic IP** and note the `AllocationId`.
4. **Associate the Elastic IP** with the instance using its `InstanceId` and the `AllocationId`.
