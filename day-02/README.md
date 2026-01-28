# Day 02 - Security Group Setup

## Steps to Create Security Group and Add Inbound Rules

### Step 1: Create Security Group

Create a security group for Nautilus App Servers:

```bash
aws ec2 create-security-group --description "Security group for Nautilus App Servers" --group-name "nautilus-sg"
```

**Note:** Save the `GroupId` from the response. You'll need it for the next steps.

### Step 2: Verify Security Group Creation

Verify the security group was created successfully by describing it using the GroupId from the previous step:

```bash
aws ec2 describe-security-groups --group-id <GROUP_ID>
```

**Note:** Replace `<GROUP_ID>` with the actual GroupId from Step 1.

### Step 3: Add Inbound Rules

Add inbound rules to allow HTTP (port 80) and SSH (port 22) traffic from anywhere:

#### Allow HTTP Traffic (Port 80)

```bash
aws ec2 authorize-security-group-ingress --group-id <GROUP_ID> --protocol tcp --port 80 --cidr 0.0.0.0/0
```

#### Allow SSH Traffic (Port 22)

```bash
aws ec2 authorize-security-group-ingress --group-id <GROUP_ID> --protocol tcp --port 22 --cidr 0.0.0.0/0
```

**Note:** Replace `<GROUP_ID>` with the actual GroupId from Step 1 in both commands above.
