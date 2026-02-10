## Day 13 - Creating an AMI from an EC2 Instance

### Overview

This day focuses on creating an Amazon Machine Image (AMI) from an existing EC2 instance using the AWS CLI, and then checking the status of that AMI until it becomes **available**.

An **AMI (Amazon Machine Image)** is a template that contains the software configuration (OS, application server, and applications) required to launch EC2 instances. Once you create an AMI from a configured instance, you can launch new instances with the exact same setup.

---

### Command 1: Create an AMI from an EC2 Instance

The following AWS CLI command creates an AMI from an existing EC2 instance:

```bash
aws ec2 create-image --instance-id i-0a9ea810d2eb6b46b --name nautilus-ec2-ami
```

#### Command Breakdown

- **`aws ec2 create-image`**: AWS CLI operation that creates a new AMI from the specified EC2 instance.
- **`--instance-id i-0a9ea810d2eb6b46b`**:
  - The ID of the EC2 instance you want to capture as an image.
  - Replace this with your own instance ID if different.
- **`--name nautilus-ec2-ami`**:
  - A human-readable name for the AMI.
  - Choose a descriptive name that reflects the purpose or configuration of the instance.

#### Example Output

On success, you will get a response similar to:

```json
{
  "ImageId": "ami-045b6ef5458efcccf"
}
```

Save this **ImageId** (`ami-045b6ef5458efcccf` in this example), because you will use it to check the status of the AMI and to launch instances from it later.

> **Note**: When you run `create-image`, AWS may briefly reboot the instance (depending on options) to ensure file system consistency.

---

### Command 2: Check AMI Status Until It Is Available

After creating the AMI, you can check its status using:

```bash
aws ec2 describe-images --image-ids ami-045b6ef5458efcccf
```

#### Command Breakdown

- **`aws ec2 describe-images`**: Describes one or more AMIs.
- **`--image-ids ami-045b6ef5458efcccf`**:
  - The ID of the AMI you created in the previous step.
  - Replace this with the actual `ImageId` returned by `create-image`.

The output will include the **state** of the image, for example:

```json
{
  "Images": [
    {
      "ImageId": "ami-045b6ef5458efcccf",
      "State": "pending",
      ...
    }
  ]
}
```

You should wait until the state becomes:

```json
"State": "available"
```

Once the AMI is **available**, it is ready to be used to launch new EC2 instances.

---

### Practical Notes & Best Practices

- **Region**:
  - Ensure you are running these commands in the same Region where your instance lives (via `--region` or your AWS CLI configuration).
- **Permissions**:
  - Your IAM user/role must have permissions such as:
    - `ec2:CreateImage`
    - `ec2:DescribeImages`
- **Instance State**:
  - Typically, the instance should be in the **running** or **stopped** state before creating an image.
- **Costs**:
  - AMIs are stored as EBS snapshots; they incur storage charges.
  - Delete unused AMIs and their snapshots to avoid unnecessary costs.
- **Naming Convention**:
  - Use meaningful AMI names (e.g. including app name, environment, date) to keep them organized.

---

### Summary of Steps

1. **Create an AMI** from your EC2 instance:
   ```bash
   aws ec2 create-image --instance-id i-0a9ea810d2eb6b46b --name nautilus-ec2-ami
   ```
2. **Copy the `ImageId`** from the output (e.g. `ami-045b6ef5458efcccf`).
3. **Check the AMI status** until it becomes `available`:
   ```bash
   aws ec2 describe-images --image-ids ami-045b6ef5458efcccf
   ```

