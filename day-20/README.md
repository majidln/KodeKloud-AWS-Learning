## Day 20 - Creating an IAM Role for EC2

### Overview

This day focuses on **creating an IAM role** for the **EC2** service using the AWS CLI, then **attaching a managed policy** to that role.

**IAM roles** let AWS services (here, EC2) assume a set of permissions without long-lived credentials. The **trust policy** declares that `ec2.amazonaws.com` may assume the role. A separate **permissions policy** (`iampolicy_anita`) defines what the role is allowed to do once assumed.

**Requirements covered:**

1. IAM role name: **`iamrole_anita`**
2. **Entity type**: AWS service · **Use case**: EC2 (trust policy uses `ec2.amazonaws.com`)
3. Attach managed policy **`iampolicy_anita`**

---

### Step 1: Create the trust policy file

Create a JSON file that allows EC2 to assume the role. From the `day-20` directory you can run:

```bash
touch trust-ec2.json
```

Edit `trust-ec2.json` and add:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This is the **trust relationship** document (who may assume the role).

---

### Step 2: Create the IAM role

```bash
aws iam create-role \
  --role-name iamrole_anita \
  --assume-role-policy-document file://trust-ec2.json
```

Run this from the directory that contains `trust-ec2.json`, or pass the full path: `file:///path/to/trust-ec2.json`.

#### Command breakdown

- **`aws iam create-role`**: Creates a new IAM role in your account.
- **`--role-name iamrole_anita`**: Name of the role (must match your requirement).
- **`--assume-role-policy-document file://trust-ec2.json`**: Trust policy JSON; `ec2.amazonaws.com` satisfies **AWS Service / EC2** as the use case.

#### Example output (success)

The response includes `Role.Arn`, `Role.RoleName`, `Role.CreateDate`, and the embedded `AssumeRolePolicyDocument`.

---

### Step 3: Attach the managed policy

Attach the existing customer-managed policy by ARN (get the ARN from `create-policy` output, `list-policies`, or the console):

```bash
aws iam attach-role-policy \
  --role-name iamrole_anita \
  --policy-arn arn:aws:iam::194055855324:policy/iampolicy_anita
```

Replace `194055855324` and the policy name segment with your account ID and policy name if they differ.

#### Command breakdown

- **`aws iam attach-role-policy`**: Attaches a **managed** policy to the role.
- **`--role-name iamrole_anita`**: Target role.
- **`--policy-arn`**: Full ARN of `iampolicy_anita` (or your policy).

On success, the command returns no output.

---

### Verify

```bash
aws iam get-role --role-name iamrole_anita
aws iam list-attached-role-policies --role-name iamrole_anita
```

---

### Practical notes

- **Policy must exist**: `iampolicy_anita` must already exist in the account before `attach-role-policy`.
- **EC2 instance profile**: To assign this role to an EC2 instance in the console or API, create an **instance profile**, add `iamrole_anita` to it, and select that profile when launching the instance. The role and trust policy alone do not create the instance profile.
- **Permissions**: Your CLI identity needs `iam:CreateRole`, `iam:AttachRolePolicy`, and permission to read the policy ARN.
