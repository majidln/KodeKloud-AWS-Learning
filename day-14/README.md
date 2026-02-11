## Day 14 - Stopping and Terminating an EC2 Instance

### Overview

This day focuses on **safely stopping** an EC2 instance and then **terminating** it using the AWS CLI.

Stopping an instance shuts it down (like powering off a server) but keeps the instance and its attached EBS volumes so you can start it again later.  
Terminating an instance permanently deletes it; you cannot start it again after termination.

---

### Step 1: Stop the EC2 Instance

Use the following command to **stop** the instance:

```bash
aws ec2 stop-instances --instance-ids i-02bd22cb3985309b1
```

#### Command Breakdown

- **`aws ec2 stop-instances`**: Stops one or more running EC2 instances.
- **`--instance-ids i-02bd22cb3985309b1`**:
  - The ID of the EC2 instance you want to stop.
  - Replace this with your own instance ID if it’s different.

#### Example Output

```json
{
  "StoppingInstances": [
    {
      "InstanceId": "i-02bd22cb3985309b1",
      "CurrentState": {
        "Code": 64,
        "Name": "stopping"
      },
      "PreviousState": {
        "Code": 16,
        "Name": "running"
      }
    }
  ]
}
```

You should wait until the instance state changes from `stopping` to `stopped`.

---

### Step 2: Terminate the EC2 Instance

Once the instance is in the **stopped** state (and you are sure you no longer need it), you can **terminate** it:

```bash
aws ec2 terminate-instances --instance-ids i-02bd22cb3985309b1
```

#### Command Breakdown

- **`aws ec2 terminate-instances`**: Permanently deletes one or more EC2 instances.
- **`--instance-ids i-02bd22cb3985309b1`**:
  - The ID of the EC2 instance you want to terminate.
  - Replace this with your own instance ID if needed.

#### Example Output

```json
{
  "TerminatingInstances": [
    {
      "InstanceId": "i-02bd22cb3985309b1",
      "CurrentState": {
        "Code": 32,
        "Name": "shutting-down"
      },
      "PreviousState": {
        "Code": 80,
        "Name": "stopped"
      }
    }
  ]
}
```

After some time, the state will move from `shutting-down` to `terminated`.

---

### Practical Notes & Best Practices

- **Region**:
  - Ensure you run these commands in the same Region as your instance (via `--region` or AWS CLI configuration).
- **Permissions**:
  - Your IAM user/role needs permissions like:
    - `ec2:StopInstances`
    - `ec2:TerminateInstances`
    - `ec2:DescribeInstances`
- **Billing Impact**:
  - Stopped instances:
    - You stop paying for compute, but still pay for attached EBS storage.
  - Terminated instances:
    - You stop paying for the instance.
    - You may still pay for any EBS volumes or snapshots that weren’t set to be deleted on termination.
- **Double-Check Before Terminating**:
  - Make sure:
    - You have backups/AMIs/snapshots of anything important.
    - The instance is not in use by any production system.

---

### Summary of Steps

1. **Stop the instance** (optional safety step before terminate):
   ```bash
   aws ec2 stop-instances --instance-ids i-02bd22cb3985309b1
   ```
2. **Verify it is stopped** (via `describe-instances` or the AWS Console).
3. **Terminate the instance** when you are sure you no longer need it:
   ```bash
   aws ec2 terminate-instances --instance-ids i-02bd22cb3985309b1
   ```

