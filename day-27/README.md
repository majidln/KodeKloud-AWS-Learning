# Day 27: Configuring a Public VPC with an EC2 Instance for Internet Access

Goals:

- VPC **`nautilus-pub-vpc`**
- Subnet **`nautilus-pub-subnet`**
- Public subnet: **Internet Gateway** + route **`0.0.0.0/0` → IGW**, and **auto-assign public IPv4** on the subnet
- EC2 **`nautilus-pub-ec2`**, **`t2.micro`**, **SSH 22** from the internet

Run everything in **one region** (example: **`us-east-1`**). **Resource IDs in the sample output are examples** — yours will differ.

---

### Create VPC

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=nautilus-pub-vpc}]'
```

Example output:

```json
{
    "Vpc": {
        "OwnerId": "839235583241",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-0ed79f331c27dc682",
                "CidrBlock": "10.0.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": false,
        "Tags": [
            {
                "Key": "Name",
                "Value": "nautilus-pub-vpc"
            }
        ],
        "VpcId": "vpc-09dc4de3005bc1897",
        "State": "pending",
        "CidrBlock": "10.0.0.0/16",
        "DhcpOptionsId": "dopt-07f711a395c6aa905"
    }
}
```

Copy **`VpcId`** from the response (below it is `vpc-09dc4de3005bc1897`) and use it in the next commands.

---

### Enable DNS support and DNS hostnames

```bash
aws ec2 modify-vpc-attribute --vpc-id vpc-09dc4de3005bc1897 --enable-dns-support
```

```bash
aws ec2 modify-vpc-attribute --vpc-id vpc-09dc4de3005bc1897 --enable-dns-hostnames
```

---

### Create subnet

Use **`--cidr-block`** (two dashes). This is **wrong** and will error:

```text
aws ec2 create-subnet --vpc-id vpc-09dc4de3005bc1897 cidr-block 10.0.0.0/24 ...
```

Correct:

```bash
aws ec2 create-subnet --vpc-id vpc-09dc4de3005bc1897 --cidr-block 10.0.0.0/24 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=nautilus-pub-subnet}]'
```

Example output:

```json
{
    "Subnet": {
        "AvailabilityZoneId": "use1-az1",
        "MapCustomerOwnedIpOnLaunch": false,
        "OwnerId": "839235583241",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "nautilus-pub-subnet"
            }
        ],
        "SubnetArn": "arn:aws:ec2:us-east-1:839235583241:subnet/subnet-09d3e023b92908d7e",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        },
        "SubnetId": "subnet-09d3e023b92908d7e",
        "State": "available",
        "VpcId": "vpc-09dc4de3005bc1897",
        "CidrBlock": "10.0.0.0/24",
        "AvailableIpAddressCount": 251,
        "AvailabilityZone": "us-east-1a",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false
    }
}
```

---

### Auto-assign public IP on subnet launch

```bash
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-09d3e023b92908d7e \
  --map-public-ip-on-launch
```

---

### Internet Gateway

```bash
aws ec2 create-internet-gateway \
  --query 'InternetGateway.InternetGatewayId' --output text
```

Example:

```text
igw-0a41d1068f2704883
```

Attach it to the VPC:

```bash
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-0a41d1068f2704883 \
  --vpc-id vpc-09dc4de3005bc1897
```

---

### Route table: default route to the Internet Gateway

```bash
aws ec2 create-route-table \
  --vpc-id vpc-09dc4de3005bc1897 \
  --query 'RouteTable.RouteTableId' --output text
```

Example:

```text
rtb-0a62c1d3927579e37
```

```bash
aws ec2 create-route \
  --route-table-id rtb-0a62c1d3927579e37 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0a41d1068f2704883
```

Example output:

```json
{
    "Return": true
}
```

Associate the route table with your subnet:

```bash
aws ec2 associate-route-table \
  --subnet-id subnet-09d3e023b92908d7e \
  --route-table-id rtb-0a62c1d3927579e37
```

Example output:

```json
{
    "AssociationId": "rtbassoc-09ce82194db04df53",
    "AssociationState": {
        "State": "associated"
    }
}
```

---

### Security group (SSH from the internet)

```bash
aws ec2 create-security-group \
  --group-name nautilus-pub-sg \
  --description "SSH for nautilus public lab" \
  --vpc-id vpc-09dc4de3005bc1897 \
  --query 'GroupId' --output text
```

Example:

```text
sg-071e2ee7a320d5bc8
```

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-071e2ee7a320d5bc8 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

Example output:

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-063a8c075d987f1fe",
            "GroupId": "sg-071e2ee7a320d5bc8",
            "GroupOwnerId": "839235583241",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-1:839235583241:security-group-rule/sgr-063a8c075d987f1fe"
        }
    ]
}
```

---

### Latest Ubuntu 22.04 amd64 AMI (Canonical)

```bash
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId" \
  --output text
```

Example:

```text
ami-05e86b3611c60b0b4
```

---

### Key pair (for `ssh` from your machine)

```bash
aws ec2 create-key-pair \
  --key-name nautilus-pub-key \
  --query 'KeyMaterial' \
  --output text > nautilus-pub-key.pem
```

```bash
chmod 400 nautilus-pub-key.pem
```

---

### Launch instance

```bash
aws ec2 run-instances \
  --image-id ami-05e86b3611c60b0b4 \
  --instance-type t2.micro \
  --subnet-id subnet-09d3e023b92908d7e \
  --security-group-ids sg-071e2ee7a320d5bc8 \
  --key-name nautilus-pub-key \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-pub-ec2}]' \
  --query 'Instances[0].InstanceId' --output text
```

Example:

```text
i-0c113f1592faa2418
```

Wait until it is running:

```bash
aws ec2 wait instance-running --instance-ids i-0c113f1592faa2418
```

Check public IP and network:

```bash
aws ec2 describe-instances \
  --instance-ids i-0c113f1592faa2418 \
  --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress,PrivateIpAddress,SubnetId,VpcId]' \
  --output table
```

Example:

```text
------------------------------
|      DescribeInstances     |
+----------------------------+
|  i-0c113f1592faa2418       |
|  running                   |
|  3.237.74.3                |
|  10.0.0.74                 |
|  subnet-09d3e023b92908d7e  |
|  vpc-09dc4de3005bc1897     |
+----------------------------+
```

SSH (Ubuntu user **`ubuntu`**):

```bash
ssh -i nautilus-pub-key.pem ubuntu@3.237.74.3
```

Use the **public IP** from your own `describe-instances` output, not the example IP.

---

### Reference

- A **public subnet** has a route **`0.0.0.0/0`** to an **Internet Gateway** attached to the same VPC, and the subnet is associated with that route table.
- **`--cidr-block`** on `create-subnet` is required; omitting `--` breaks the CLI (as in the failed `cidr-block 10.0.0.0/24` attempt).
