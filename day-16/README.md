## Day 16 - Creating an IAM User

### Overview

This day focuses on **creating a new IAM user** using the AWS CLI.

**IAM (Identity and Access Management)** lets you manage access to AWS services and resources. An IAM user represents a person or application that interacts with AWS. Each user gets unique credentials and can be assigned specific permissions through policies or group memberships.

---

### Command: Create an IAM User

```bash
aws iam create-user --user-name iamuser_jim
```

#### Command Breakdown

- **`aws iam create-user`**: AWS CLI operation that creates a new IAM user in your AWS account.
- **`--user-name iamuser_jim`**:
  - The name for the new IAM user.
  - Must be unique within the AWS account.
  - Can contain letters, digits, and the following characters: `+ = , . @ - _`

#### Example Output

On success, you will get a response similar to:

```json
{
  "User": {
    "Path": "/",
    "UserName": "iamuser_jim",
    "UserId": "AIDAEXAMPLEID1234567",
    "Arn": "arn:aws:iam::123456789012:user/iamuser_jim",
    "CreateDate": "2026-02-18T10:00:00+00:00"
  }
}
```

- **`Path`**: The path for the user (defaults to `/`).
- **`UserName`**: The name you specified.
- **`UserId`**: A unique identifier assigned by AWS.
- **`Arn`**: The Amazon Resource Name, a globally unique identifier for this user.
- **`CreateDate`**: Timestamp of when the user was created.

---

### Practical Notes & Best Practices

- **Permissions**:
  - Your IAM user/role needs the `iam:CreateUser` permission to run this command.
- **New User Has No Permissions**:
  - A freshly created IAM user has **no permissions** by default. You must attach policies or add the user to a group to grant access.
- **Console Access**:
  - Creating a user does not automatically give them AWS Console access. Use `aws iam create-login-profile` to set a console password.
- **Programmatic Access**:
  - To allow API/CLI access, create access keys with `aws iam create-access-key --user-name iamuser_jim`.
- **Naming Convention**:
  - Use a consistent naming pattern (e.g. `iamuser_<name>`, `dev-<name>`) to keep users organized.
- **Least Privilege**:
  - Only grant the minimum permissions the user needs to do their job.

---

### Summary

1. **Create the IAM user**:
   ```bash
   aws iam create-user --user-name iamuser_jim
   ```
2. **Verify** the user was created by checking the output or running:
   ```bash
   aws iam get-user --user-name iamuser_jim
   ```
3. **Next steps**: Attach policies, add to groups, or create access keys/login profiles as needed.
