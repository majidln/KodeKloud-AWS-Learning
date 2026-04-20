# Day 24: Setting Up an Application Load Balancer for an EC2 Instance

Set up an **internet-facing Application Load Balancer** that listens on **HTTP 80** and forwards to **port 80** on an EC2 instance (`datacenter-ec2`) running Nginx. Use a dedicated security group for the ALB (public HTTP), and make **appropriate changes to the default security group** attached to the EC2 instance (or attach the new security group to the instance) so traffic can reach Nginx.

**Why the default security group is not enough by itself:** The **default** security group only allows inbound traffic from other resources that use that **same** default group. It does **not** automatically allow traffic from the **Application Load Balancer**, because the ALB uses your new **`datacenter-sg`**. Until you fix that, users may reach the ALB on the internet, but the ALB cannot successfully connect to the instance on port 80 (health checks fail and the site does not load). You fix it by either **(1)** adding an inbound rule on the **default** security group allowing **TCP 80** from **`datacenter-sg`**, or **(2)** **attaching `datacenter-sg` to the EC2 instance** as well (in addition to or instead of relying only on default—follow your lab’s wording). Option (1) matches *“Make appropriate changes in the default security group … if necessary.”*

Throughout this guide, **replace the placeholders** (for example `<vpc-id>`) with values **from the output of the previous step**. Add `--region your-region` to any command if your default region is not set in the AWS CLI.

---

## Prerequisites

- [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/install-clv2.html) configured (`aws sts get-caller-identity` works).
- An EC2 instance in a VPC with Nginx listening on **port 80** (health checks expect HTTP **200** on `/` by default).

---

## 1. Find the VPC ID

```bash
aws ec2 describe-vpcs
```

**Save:** the `VpcId` of the VPC where your instance lives (often the default VPC). You will use it as `<vpc-id>` below.

To list subnets in that VPC (you need **two subnet IDs in different Availability Zones** for the ALB):

```bash
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=<vpc-id>" \
  --query "Subnets[*].{SubnetId:SubnetId,AZ:AvailabilityZone,Cidr:CidrBlock}" \
  --output table
```

**Save:** two `SubnetId` values from **different** `AZ` columns — call them `<subnet-a>` and `<subnet-b>`.

---

## 2. Create security group `datacenter-sg` and open port 80 to the public

Replace `<vpc-id>` with the value from step 1.

```bash
aws ec2 create-security-group \
  --group-name datacenter-sg \
  --description "internet facing sg" \
  --vpc-id <vpc-id>
```

**Save:** `GroupId` from the response — use it as `<datacenter-sg-id>` below.

Allow inbound HTTP from the internet (lab requirement):

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <datacenter-sg-id> \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

This security group is attached to the **ALB** in a later step.

---

## 3. Create target group `datacenter-tg` (HTTP, port 80)

Replace `<vpc-id>` with the same VPC as in step 1.

For an Application Load Balancer, use **`HTTP`**, not `tcp`.

```bash
aws elbv2 create-target-group \
  --name datacenter-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id <vpc-id>
```

**Save:** `TargetGroupArn` from `TargetGroups[0]` — use it as `<target-group-arn>` below.

---

## 4. Create load balancer `datacenter-alb` and attach `datacenter-sg`

Replace `<subnet-a>` and `<subnet-b>` with the two subnets from step 1 (different AZs). Replace `<datacenter-sg-id>` with the security group from step 2.

```bash
aws elbv2 create-load-balancer \
  --name datacenter-alb \
  --subnets <subnet-a> <subnet-b> \
  --security-groups <datacenter-sg-id>
```

**Save:** `LoadBalancerArn` — use as `<load-balancer-arn>`. Optionally note `DNSName` for testing later.

Wait until the load balancer is ready:

```bash
aws elbv2 wait load-balancer-available \
  --load-balancer-arns <load-balancer-arn>
```

---

## 5. Listener: forward port 80 to the target group

Replace `<load-balancer-arn>` and `<target-group-arn>` with values from steps 4 and 3.

```bash
aws elbv2 create-listener \
  --load-balancer-arn <load-balancer-arn> \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=<target-group-arn>
```

---

## 6. Register the EC2 instance on port 80

Find the instance ID if you do not have it (replace `datacenter-ec2` if your Name tag differs):

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" "Name=instance-state-name,Values=running" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text
```

**Save:** that instance ID as `<instance-id>`.

Register it to the target group:

```bash
aws elbv2 register-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=<instance-id>,Port=80
```

---

## 7. Default security group on the EC2 — allow traffic from the ALB on port 80

The **default** security group usually allows inbound only from **itself** (other members of the same group). It does **not** grant “internet access” to the instance in the sense that matters here: **inbound from the load balancer** still needs an explicit rule, because the ALB is associated with **`datacenter-sg`**, not the default group. Without that, traffic from the ALB to port 80 on the instance is denied.

**Two valid approaches (use what your lab allows):**

1. **Edit the default security group** (matches the lab line above): add **TCP 80** inbound with source **the security group** `datacenter-sg` (`<datacenter-sg-id>`).
2. **Attach `datacenter-sg` to the EC2 instance** (e.g. in the EC2 console: instance → **Security** → **Security groups** → **Change security groups** / add `datacenter-sg`). Then the instance also uses the rules of `datacenter-sg`. Note: some environments block changing instance security groups via API (`modify-instance-attribute`) with an **access denied** error; the **console** may still work, or you may only be allowed to edit **rules** on the default group.

**If AWS CLI returns `UnauthorizedOperation` or similar** when running the commands below (or when modifying the instance’s security groups), complete the same change in the **AWS Console**: **EC2** → **Security Groups** → select the **default** security group for your VPC → **Edit inbound rules** → add **Type** HTTP (or custom TCP **80**), **Source** = the **`datacenter-sg`** security group (or paste its ID). Alternatively, attach **`datacenter-sg`** to the instance on the instance’s **Security** tab as described above.

List the instance’s security groups:

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[0].Instances[0].SecurityGroups[*].{GroupId:GroupId,GroupName:GroupName}" \
  --output table
```

**Save:** the `GroupId` for the **default** security group — use as `<default-sg-id>`.

Add ingress from the ALB security group (replace `<default-sg-id>` and `<datacenter-sg-id>`):

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <default-sg-id> \
  --protocol tcp \
  --port 80 \
  --source-group <datacenter-sg-id>
```

**Alternative (broader):** allow TCP 80 from the whole VPC CIDR. Get the CIDR from step 1:

```bash
aws ec2 describe-vpcs --vpc-ids <vpc-id> --query "Vpcs[0].CidrBlock" --output text
```

Then:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <default-sg-id> \
  --protocol tcp \
  --port 80 \
  --cidr <vpc-cidr>
```

> **Note:** Some lab accounts block `modify-instance-attribute` or other EC2 API calls for security groups. If the CLI fails with an access error, apply the **same** rule (or attach **`datacenter-sg`** to the instance) using the **console UI**, as in the note above.

---

## 8. Verify

- In the AWS console or CLI, the target should become **healthy** (Nginx must return **200** on `/` for the default HTTP health check).
- Open the ALB DNS name in a browser: `http://<alb-dns-name>`.

```bash
aws elbv2 describe-load-balancers \
  --load-balancer-arns <load-balancer-arn> \
  --query "LoadBalancers[0].DNSName" \
  --output text

aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

---
