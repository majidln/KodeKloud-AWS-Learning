## Day 22 - EC2 SSH: Key Pair, Security Group, and Root Access

### Overview

Create an EC2 key pair with the **AWS CLI**, launch an instance that uses it, open **SSH (port 22)** to your IP, then enable **passwordless SSH as `root`** by installing your **public** key in `/root/.ssh/authorized_keys`.

Keep the **private** key only on the machine you SSH from (e.g. **aws-client**).

---

### 1. Create a key pair and save the private key

```bash
aws ec2 create-key-pair \
  --key-name "your-key-name" \
  --query 'KeyMaterial' \
  --output text > your-key.pem

chmod 400 your-key.pem
```

---

### 2. Allow SSH from your IP (CIDR)

Use your **public** IPv4 with `/32` (single host), e.g. `203.0.113.10/32`. Find it with `curl -s https://checkip.amazonaws.com`.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr YOUR_PUBLIC_IP/32
```

---

### 3. Launch the instance

Use `--key-name` matching the pair you created, plus `--image-id`, `--instance-type`, and (if needed) `--subnet-id` and `--security-group-ids`. Tag the instance for easy lookup:

```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type t3.micro \
  --key-name "your-key-name" \
  --security-group-ids sg-xxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=your-instance-name}]'
```

---

### 4. SSH as the default user, then add your public key for root

On **aws-client**, generate a key if needed: `ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa`.

**Copy only** `~/.ssh/id_rsa.pub` to the server’s root user:

```bash
ssh -i your-key.pem ec2-user@INSTANCE_PUBLIC_IP
# Amazon Linux: ec2-user | Ubuntu: ubuntu

sudo mkdir -p /root/.ssh
sudo chmod 700 /root/.ssh
sudo bash -c 'cat >> /root/.ssh/authorized_keys' < ~/.ssh/id_rsa.pub
sudo chmod 600 /root/.ssh/authorized_keys
```

Allow key-based root login in `sshd_config` (e.g. `PermitRootLogin prohibit-password`), then `sudo systemctl reload sshd`.

---

### 5. SSH as root from aws-client

```bash
ssh -i ~/.ssh/id_rsa root@INSTANCE_PUBLIC_IP
```

---

### Notes

- **Timeout / nothing on SSH**: check security group **22** from current IP, instance in a **public** subnet with a **public IP**, and correct **Linux user** for the AMI.
- **Never** put the **private** key in `authorized_keys`—only the **public** `.pub` line.
