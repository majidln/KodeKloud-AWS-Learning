## Day 19 - Attaching an IAM Policy to a User

### Overview

This day focuses on **attaching an existing IAM policy to a user** using the AWS CLI.

**IAM (Identity and Access Management)** policies define permissions for AWS resources. When you have a policy and a user already created, you can attach the policy directly to the user to grant them the defined permissions. This task covers how to look up a policy's ARN by name and then attach it to a specific user.

---

### Step 1: Find the Policy ARN

Before attaching a policy, you need its ARN. If you only know the policy name, use `list-policies` with a query filter:

```bash
aws iam list-policies --query "Policies[?PolicyName=='iampolicy_yousuf'].[Arn,PolicyId]" --output table
```

#### Command Breakdown

- **`aws iam list-policies`**: Lists all IAM policies available in your account (both AWS-managed and customer-managed).
- **`--query "Policies[?PolicyName=='iampolicy_yousuf'].[Arn,PolicyId]"`**:
  - Uses JMESPath to filter the results to only the policy matching the given name.
  - Projects only the `Arn` and `PolicyId` fields from the result.
- **`--output table`**: Formats the output as a readable table instead of JSON.

#### Example Output

```
----------------------------------------------------------------------
|                          ListPolicies                              |
+--------------------------------------------------+-----------------+
|  arn:aws:iam::912430769224:policy/iampolicy_yousuf  |  ANPA...XYZ  |
+--------------------------------------------------+-----------------+
```

Copy the ARN from the output — you will need it in the next step.

---

### Step 2: Attach the Policy to the User

Once you have the policy ARN, attach it to the target user:

```bash
aws iam attach-user-policy \
  --user-name iamuser_yousuf \
  --policy-arn arn:aws:iam::912430769224:policy/iampolicy_yousuf
```

#### Command Breakdown

- **`aws iam attach-user-policy`**: AWS CLI operation that attaches a managed policy to an IAM user.
- **`--user-name iamuser_yousuf`**: The name of the IAM user who will receive the permissions.
- **`--policy-arn arn:aws:iam::912430769224:policy/iampolicy_yousuf`**:
  - The full ARN of the policy to attach.
  - Replace `912430769224` with your actual AWS account ID if different.

#### Example Output

This command produces no output on success. A successful run returns to the shell prompt with no error.

---

### Practical Notes

- **Required permissions**: Your IAM user/role needs `iam:ListPolicies` and `iam:AttachUserPolicy` permissions to run these commands.
- **Managed vs. inline policies**: `attach-user-policy` works with **managed policies** (customer-managed or AWS-managed). Inline policies use a different command (`put-user-policy`).
- **Policy limits**: A single IAM user can have up to 10 managed policies attached directly.
- **Verify the attachment**:
  ```bash
  aws iam list-attached-user-policies --user-name iamuser_yousuf
  ```
