# Day 26: Configuring an EC2 Instance as a Web Server with Nginx

### Step 1: User data (`#cloud-config`)

Use **cloud-init** `#cloud-config` so Ubuntu applies **`package_update`**, installs **`nginx`**, then **`runcmd`** starts and enables the service.

Create `user-data.txt` (filename arbitrary) with **line 1 exactly** `#cloud-config` (no leading spaces):

```yaml
#cloud-config
package_update: true
packages:
  - nginx
runcmd:
  - [ systemctl, start, nginx ]
  - [ systemctl, enable, nginx ]
```

---

### Step 2: Find your default VPC (optional)

If you need a **VPC ID** for creating a security group in a specific VPC:

```bash
aws ec2 describe-vpcs
```

Note the **`VpcId`** for the default VPC (or the VPC where the instance will run). Example shape of output:

```json
{
  "Vpcs": [
    {
      "VpcId": "vpc-0d1f27c91f7fb5b68",
      "CidrBlock": "172.31.0.0/16",
      "IsDefault": true
    }
  ]
}
```

Your IDs will differ by account and region.

---

### Step 3: Create a security group and allow HTTP

Create the group in the VPC from Step 2:

```bash
aws ec2 create-security-group \
  --group-name datacenter-sg \
  --description "internet facing sg" \
  --vpc-id <vpc-id>
```

Allow **inbound TCP 80** from anywhere:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <security-group-id> \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

**Egress:** The default security group rule usually allows all outbound traffic. If your environment requires explicit egress for HTTPS (APT) during bootstrap, ensure **outbound** TCP **443** (and often **80**) is allowed so `apt-get` can reach Ubuntu mirrors.

---

### Step 4: Resolve the latest Ubuntu 22.04 AMI ID

Canonical’s owner ID on AWS is **`099720109477`**. This returns the **newest** matching 22.04 **amd64** server image in the current region:

```bash
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId" \
  --output text
```

Example result: `ami-05e86b3611c60b0b4` (region and date dependent).

---

### Step 5: Launch the instance

```bash
aws ec2 run-instances \
  --instance-type t2.micro \
  --image-id <ami-id> \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-ec2}]' \
  --security-group-ids <security-group-id> \
  --user-data file://user-data.txt
```

---

### Step 6: Get the public IP and verify in the browser

Wait until the instance is **running** and **status checks** have passed (console or `describe-instance-status`). Then fetch the **public IP**:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].PublicIpAddress" \
  --output text
```

Open **`http://<public-ip>`** in a browser. You should see the **default Nginx welcome page** once user data has finished installing and starting the service (can take a minute after boot).

Quick CLI check:

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://<public-ip>/
```

Expect **`200`** when Nginx is serving HTTP.

---