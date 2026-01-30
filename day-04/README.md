# Day 04 - S3 Bucket Versioning

## Overview
This day focuses on enabling versioning on an S3 bucket using the AWS CLI.

## Command Explanation

The following AWS CLI command enables versioning on an S3 bucket:

```bash
aws s3api put-bucket-versioning --bucket <BUCKET_NAME> --versioning-configuration Status=Enabled
```

### What does this command do?

This command enables versioning on an S3 bucket with the following parameters:

- **`aws s3api put-bucket-versioning`**: The AWS CLI command to configure bucket versioning
- **`--bucket <BUCKET_NAME>`**: Specifies the name of the S3 bucket (e.g., `datacenter-s3-3665`)
- **`--versioning-configuration Status=Enabled`**: Enables versioning on the bucket

### What is S3 Bucket Versioning?

S3 bucket versioning is a feature that keeps multiple variants of an object in the same bucket. When versioning is enabled:

- **Multiple versions**: Each time you upload an object with the same key, S3 creates a new version instead of overwriting the existing one
- **Version IDs**: Each version has a unique version ID that you can use to retrieve specific versions
- **Accidental deletion protection**: When you delete an object, S3 doesn't actually remove it; instead, it adds a delete marker
- **Recovery**: You can restore previous versions of objects or recover from accidental deletions

### Why Enable Versioning?

Versioning provides several benefits:

- **Backup and recovery**: Restore previous versions of files if they're accidentally modified or deleted
- **Compliance**: Maintain a history of object changes for audit and compliance requirements
- **Data protection**: Protect against accidental overwrites or deletions
- **Application rollback**: Revert to previous versions of application files or configurations

## How to Use

**Please enter the following command in your terminal:**

```bash
aws s3api put-bucket-versioning --bucket <BUCKET_NAME> --versioning-configuration Status=Enabled
```

### Important Notes:

1. **Bucket Name**: Replace `<BUCKET_NAME>` with your actual S3 bucket name. Bucket names must be globally unique across all AWS accounts.

2. **Bucket Must Exist**: The bucket must already exist before you can enable versioning. If you need to create a bucket first:
   ```bash
   aws s3 mb s3://<BUCKET_NAME> --region <REGION>
   ```

3. **Once Enabled**: Once versioning is enabled on a bucket, it cannot be disabled (only suspended). This is an important consideration.

4. **Storage Costs**: Versioning can increase storage costs since multiple versions of objects are stored. Be aware of this when enabling versioning.

5. **Prerequisites**: 
   - You must have AWS CLI installed and configured
   - You must have appropriate IAM permissions (`s3:PutBucketVersioning`) to enable versioning
   - The bucket must exist and be accessible

6. **Output**: The command typically returns no output on success. You can verify versioning status using the verification commands below.

## Verify Versioning Status

After enabling versioning, you can verify it was enabled successfully:

```bash
aws s3api get-bucket-versioning --bucket <BUCKET_NAME>
```

This command will return:
```json
{
    "Status": "Enabled"
}
```

If versioning is suspended (not disabled), you'll see:
```json
{
    "Status": "Suspended"
}
```

## Working with Versioned Objects

### Upload an Object

When you upload an object to a versioned bucket:
```bash
aws s3 cp file.txt s3://<BUCKET_NAME>/file.txt
```

### List Object Versions

To see all versions of an object:
```bash
aws s3api list-object-versions --bucket <BUCKET_NAME> --prefix file.txt
```

### Retrieve a Specific Version

To download a specific version of an object:
```bash
aws s3api get-object --bucket <BUCKET_NAME> --key file.txt --version-id <VERSION_ID> output.txt
```

### Delete an Object (Creates Delete Marker)

When you delete an object in a versioned bucket:
```bash
aws s3 rm s3://<BUCKET_NAME>/file.txt
```

This doesn't actually delete the object; it adds a delete marker. The object versions remain but are hidden.

### Permanently Delete a Version

To permanently delete a specific version:
```bash
aws s3api delete-object --bucket <BUCKET_NAME> --key file.txt --version-id <VERSION_ID>
```

### Restore a Previous Version

To restore a previous version, copy it over the current version:
```bash
aws s3api copy-object --bucket <BUCKET_NAME> --copy-source <BUCKET_NAME>/file.txt?versionId=<VERSION_ID> --key file.txt
```

## Suspending Versioning

If you need to stop creating new versions (but cannot disable versioning entirely), you can suspend it:

```bash
aws s3api put-bucket-versioning --bucket <BUCKET_NAME> --versioning-configuration Status=Suspended
```

**Note**: Suspending versioning:
- Stops creating new versions for new uploads
- Does not delete existing versions
- Does not remove delete markers
- Can be re-enabled later

## Best Practices

1. **Cost Management**: Monitor storage costs as versioning can significantly increase storage usage
2. **Lifecycle Policies**: Use S3 lifecycle policies to automatically transition old versions to cheaper storage classes or delete them after a certain period
3. **MFA Delete**: Consider enabling MFA (Multi-Factor Authentication) delete for additional protection against accidental deletions
4. **Versioning Strategy**: Decide on a versioning strategy before enabling it, as it cannot be fully disabled once enabled
