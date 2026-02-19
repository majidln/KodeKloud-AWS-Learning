## Day 17 - Creating an IAM Group

### Overview

This day focuses on **creating a new IAM group** using the AWS CLI.

**IAM (Identity and Access Management)** groups let you organize IAM users and assign permissions to multiple users at once. Instead of attaching policies to each user individually, you attach policies to a group and add users to that group. This simplifies access management and keeps permissions consistent across team members.

---

### Command: Create an IAM Group

```bash
aws iam create-group --group-name iamgroup_kirsty
```

#### Command Breakdown

- **`aws iam create-group`**: AWS CLI operation that creates a new IAM group in your AWS account.
- **`--group-name iamgroup_kirsty`**:
  - The name for the new IAM group.
  - Must be unique within the AWS account.
  - Can contain letters, digits, and the following characters: `+ = , . @ - _`

#### Example Output

On success, you will get a response similar to:

```json
{
  "Group": {
    "Path": "/",
    "GroupName": "iamgroup_kirsty",
    "GroupId": "AGPAEXAMPLEID1234567",
    "Arn": "arn:aws:iam::123456789012:group/iamgroup_kirsty",
    "CreateDate": "2026-02-19T10:00:00+00:00"
  }
}
```

- **`Path`**: The path for the group (defaults to `/`).
- **`GroupName`**: The name you specified.
- **`GroupId`**: A unique identifier assigned by AWS.
- **`Arn`**: The Amazon Resource Name, a globally unique identifier for this group.
- **`CreateDate`**: Timestamp of when the group was created.

---

### Practical Notes & Best Practices

- **Permissions**:
  - Your IAM user/role needs the `iam:CreateGroup` permission to run this command.
- **New Group Has No Permissions**:
  - A freshly created IAM group has **no permissions** by default. Attach managed policies or inline policies to the group to grant access.
- **Add Users to the Group**:
  - After creating the group, add users with:
    ```bash
    aws iam add-user-to-group --group-name iamgroup_kirsty --user-name iamuser_jim
    ```
- **Attach Policies**:
  - Grant permissions by attaching policies to the group, e.g.:
    ```bash
    aws iam attach-group-policy --group-name iamgroup_kirsty --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
    ```
- **Naming Convention**:
  - Using a pattern like `iamgroup_<name>` helps keep groups organized and easy to identify.
- **Least Privilege**:
  - Only attach the policies that the group’s members need for their role.

---

### Summary

1. **Create the IAM group**:
   ```bash
   aws iam create-group --group-name iamgroup_kirsty
   ```
2. **Verify** the group was created by checking the output or running:
   ```bash
   aws iam get-group --group-name iamgroup_kirsty
   ```
3. **Next steps**: Add users to the group and attach policies as needed.
