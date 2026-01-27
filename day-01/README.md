# Day 01 - EC2 Key Pair Creation

## Overview
This day focuses on creating an EC2 key pair using the AWS CLI.

## Command Explanation

The script contains the following AWS CLI command:

```bash
aws ec2 create-key-pair --key-name datacenter-kp --key-type rsa
```

### What does this command do?

This command creates a new EC2 key pair in your AWS account with the following parameters:

- **`aws ec2 create-key-pair`**: The AWS CLI command to create a new EC2 key pair
- **`--key-name datacenter-kp`**: Specifies the name of the key pair as "datacenter-kp"
- **`--key-type rsa`**: Specifies the key type as RSA (Rivest-Shamir-Adleman), which is a widely-used encryption algorithm

### Why do we need key pairs?

EC2 key pairs are used to securely connect to your EC2 instances via SSH. When you create a key pair:
- AWS generates a public and private key
- The public key is stored in AWS
- You receive the private key (which you must save securely)
- You use the private key to authenticate when connecting to EC2 instances

## How to Use

**Please enter the following command in your terminal:**

```bash
aws ec2 create-key-pair --key-name datacenter-kp --key-type rsa
```

### Important Notes:

1. **Save the private key**: After running this command, AWS will return the private key material. **You must save this securely** as it won't be shown again. Without the private key, you won't be able to connect to instances using this key pair.

2. **Prerequisites**: 
   - You must have AWS CLI installed and configured
   - You must have appropriate IAM permissions to create EC2 key pairs

3. **Output**: The command will return JSON output containing the key material. Save this to a file (e.g., `datacenter-kp.pem`) and set appropriate permissions:
   ```bash
   chmod 400 datacenter-kp.pem
   ```

## Alternative: Using the Script

You can also execute the script directly:

```bash
bash script.sh
```

However, remember to save the private key output from the command!
