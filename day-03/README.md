# Day 03 - Subnet Creation

## Overview
This day focuses on creating a subnet in an existing VPC using the AWS CLI.

## Command Explanation

The following AWS CLI command creates a new subnet:

```bash
aws ec2 create-subnet --vpc-id <VPC_ID> --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=nautilus-subnet}]' --cidr-block 172.31.96.0/20
```

### What does this command do?

This command creates a new subnet in your VPC with the following parameters:

- **`aws ec2 create-subnet`**: The AWS CLI command to create a new subnet
- **`--vpc-id <VPC_ID>`**: Specifies the VPC ID where the subnet will be created
- **`--cidr-block 172.31.96.0/20`**: Specifies the CIDR block for the subnet (provides 4,096 IP addresses, with 4,091 usable)
- **`--tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=nautilus-subnet}]'`**: Adds a tag with the key "Name" and value "nautilus-subnet" to the subnet

### What is a subnet?

A subnet is a range of IP addresses in your VPC. Subnets allow you to:
- Organize your network resources
- Control routing and network access
- Deploy resources in specific Availability Zones
- Isolate resources for security purposes

### CIDR Block Explanation

The CIDR block `172.31.96.0/20` means:
- **Network address**: 172.31.96.0
- **Subnet mask**: /20 (20 bits for network, 12 bits for hosts)
- **IP range**: 172.31.96.0 - 172.31.111.255
- **Total IPs**: 4,096 addresses
- **Usable IPs**: 4,091 (5 addresses are reserved by AWS)

## How to Use

**Please enter the following command in your terminal:**

```bash
aws ec2 create-subnet --vpc-id <VPC_ID> --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=nautilus-subnet}]' --cidr-block 172.31.96.0/20
```

### Important Notes:

1. **VPC ID**: Replace `<VPC_ID>` with your actual VPC ID.

2. **CIDR Block**: Ensure the CIDR block doesn't overlap with existing subnets in your VPC. The `/20` subnet provides a large address space suitable for many resources.

3. **Availability Zone**: If you need to specify a particular Availability Zone, add `--availability-zone us-east-1a` (or your preferred AZ) to the command.

4. **Tag Specifications Format**: The tag specification must include:
   - `ResourceType=subnet` to indicate you're tagging a subnet
   - `Tags=[{Key=Name,Value=nautilus-subnet}]` for the actual tags

5. **Prerequisites**: 
   - You must have AWS CLI installed and configured
   - You must have appropriate IAM permissions to create subnets
   - The VPC ID must exist and be accessible

6. **Output**: The command will return JSON output containing the subnet details including:
   - `SubnetId`: The unique identifier for your new subnet
   - `State`: The current state of the subnet (usually "available")
   - `AvailabilityZone`: The AZ where the subnet was created

## Verify Subnet Creation

After creating the subnet, you can verify it was created successfully:

```bash
aws ec2 describe-subnets --subnet-ids <SUBNET_ID>
```

Or list all subnets in your VPC:

```bash
aws ec2 describe-subnets --filters "Name=vpc-id,Values=<VPC_ID>"
```

**Note:** Replace `<SUBNET_ID>` with the actual SubnetId from the creation response.

## Understanding CIDR Blocks

### What is a CIDR Block?

CIDR (Classless Inter-Domain Routing) is a method for allocating IP addresses and routing Internet Protocol packets. A CIDR block is a compact representation of an IP address and its associated network mask.

**Format**: `X.X.X.X/Y`
- **X.X.X.X**: The network address (starting IP address)
- **Y**: The prefix length (number of bits used for the network portion)

### How CIDR Notation Works

The prefix length (`/Y`) determines how many IP addresses are available:
- **Smaller number** (e.g., `/16`) = Larger network = More IP addresses
- **Larger number** (e.g., `/24`) = Smaller network = Fewer IP addresses

**Common CIDR block sizes:**
- `/16`: 65,536 IP addresses (e.g., 172.31.0.0/16)
- `/20`: 4,096 IP addresses (e.g., 172.31.96.0/20)
- `/24`: 256 IP addresses (e.g., 172.31.96.0/24)
- `/28`: 16 IP addresses (e.g., 172.31.96.0/28)

### How to Find an Available CIDR Block

Before creating a subnet, you need to find an available CIDR block that doesn't overlap with existing subnets in your VPC. Here's how:

#### Step 1: List All Subnets in Your VPC

```bash
aws ec2 describe-subnets --filters "Name=vpc-id,Values=<VPC_ID>"
```

This command will show you all existing subnets and their CIDR blocks.

#### Step 2: Identify Your VPC's CIDR Block

```bash
aws ec2 describe-vpcs --vpc-ids <VPC_ID>
```

This shows your VPC's main CIDR block (e.g., `172.31.0.0/16`). All subnets must be within this range.

#### Step 3: Analyze Existing Subnets

Look at the `CidrBlock` field in the output. For example, if you see:
- `172.31.0.0/20` (covers 172.31.0.0 - 172.31.15.255)
- `172.31.16.0/20` (covers 172.31.16.0 - 172.31.31.255)
- `172.31.32.0/20` (covers 172.31.32.0 - 172.31.47.255)
- `172.31.48.0/20` (covers 172.31.48.0 - 172.31.63.255)
- `172.31.64.0/20` (covers 172.31.64.0 - 172.31.79.255)
- `172.31.80.0/20` (covers 172.31.80.0 - 172.31.95.255)

The next available `/20` block would be `172.31.96.0/20` (covers 172.31.96.0 - 172.31.111.255).

#### Step 4: Choose an Appropriate Size

Consider your needs:
- **Small applications**: Use `/24` (256 IPs) or `/28` (16 IPs)
- **Medium applications**: Use `/20` (4,096 IPs)
- **Large applications**: Use `/16` (65,536 IPs) - but this is typically the entire VPC

**Remember**: AWS reserves 5 IP addresses in each subnet:
- Network address (first IP)
- VPC router (second IP)
- DNS server (third IP)
- Reserved for future use (fourth IP)
- Broadcast address (last IP)

So a `/24` subnet (256 IPs) actually provides 251 usable IP addresses.

### Example: Finding Available CIDR Blocks

If your VPC uses `172.31.0.0/16` and you have subnets:
- `172.31.0.0/20`
- `172.31.16.0/20`
- `172.31.32.0/20`

Available options include:
- `172.31.48.0/20` (next `/20` block)
- `172.31.48.0/24` (smaller subnet starting at the same address)
- `172.31.96.0/20` (skipping ahead to avoid overlap)

Always ensure your chosen CIDR block:
1. Falls within your VPC's CIDR range
2. Doesn't overlap with any existing subnets
3. Is appropriately sized for your needs
