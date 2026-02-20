## Day 18 - Creating an IAM Policy

### Overview

This day focuses on **creating a new IAM policy** using the AWS CLI.

**IAM (Identity and Access Management)** policies define permissions that specify what actions are allowed or denied on which AWS resources. Policies can be attached to users, groups, or roles to grant them specific permissions. This task creates a custom policy that provides read-only access to EC2 resources, allowing users to view instances, AMIs, and snapshots in the Amazon EC2 console.

---

### Command: Create an IAM Policy

```bash
aws iam create-policy \
  --policy-name iampolicy_siva \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ec2:DescribeInstances",
          "ec2:DescribeImages",
          "ec2:DescribeSnapshots",
          "ec2:DescribeInstanceStatus",
          "ec2:DescribeInstanceAttribute",
          "ec2:DescribeImageAttribute",
          "ec2:DescribeSnapshotAttribute",
          "ec2:DescribeTags",
          "ec2:DescribeVolumes",
          "ec2:DescribeVolumeStatus",
          "ec2:DescribeAvailabilityZones",
          "ec2:DescribeRegions"
        ],
        "Resource": "*"
      }
    ]
  }' \
  --region us-east-1
```

#### Command Breakdown

- **`aws iam create-policy`**: AWS CLI operation that creates a new IAM policy in your AWS account.
- **`--policy-name iampolicy_siva`**:
  - The name for the new IAM policy.
  - Must be unique within the AWS account.
  - Can contain letters, digits, and the following characters: `+ = , . @ - _`
- **`--policy-document`**:
  - The JSON policy document that defines the permissions.
  - Can be provided inline (as shown) or from a file using `file://path/to/policy.json`.
- **`--region us-east-1`**:
  - **Note**: IAM policies are **global resources**, not regional. The `--region` flag is ignored for IAM operations, but including it won't cause an error.

The policy grants read-only access to EC2 resources (instances, AMIs, and snapshots) by allowing various `Describe` actions.

#### Example Output

On success, you will get a response similar to:

```json
{
  "Policy": {
    "PolicyName": "iampolicy_siva",
    "PolicyId": "ANPAEXAMPLEID1234567",
    "Arn": "arn:aws:iam::123456789012:policy/iampolicy_siva",
    "Path": "/",
    "DefaultVersionId": "v1",
    "AttachmentCount": 0,
    "IsAttachable": true,
    "CreateDate": "2026-02-20T10:00:00+00:00",
    "UpdateDate": "2026-02-20T10:00:00+00:00"
  }
}
```

- **`PolicyName`**: The name you specified.
- **`Arn`**: The Amazon Resource Name (ARN), used to reference this policy when attaching it to users, groups, or roles.
- **`AttachmentCount`**: Number of entities this policy is attached to (starts at 0).
