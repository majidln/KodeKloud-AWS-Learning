## Day 15 - Creating EBS Snapshots with AWS CLI

### Overview

This day focuses on creating an **EBS snapshot** from an existing EBS volume using the AWS CLI.  
An EBS snapshot is a **point-in-time backup** of your EBS volume, stored in Amazon S3 and managed by AWS.  
You can use snapshots to restore data, create new volumes, or migrate data across Availability Zones or Regions.

---

### Step 1: List Existing EBS Volumes

First, list your existing EBS volumes so you can identify the **Volume ID** you want to back up.

```bash
aws ec2 describe-volumes
```

From the output, locate the `VolumeId` of the volume you want to snapshot (for example: `vol-0202bda43700b5ac9`).

---

### Step 2: Create a Snapshot from the Volume

Use the following command to create a snapshot from the selected volume:

```bash
aws ec2 create-snapshot \
  --volume-id vol-0202bda43700b5ac9 \
  --description "datacenter Snapshot" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=datacenter-vol-ss}]'
```

#### Command Breakdown

- **`aws ec2 create-snapshot`**: Creates a snapshot of an existing EBS volume.
- **`--volume-id vol-0202bda43700b5ac9`**:
  - The ID of the EBS volume to back up.
  - Replace this with the `VolumeId` you obtained from `describe-volumes`.
- **`--description "datacenter Snapshot"`**:
  - A human-readable description to identify the snapshot.
- **`--tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=datacenter-vol-ss}]'`**:
  - Adds a `Name` tag to the snapshot so it’s easier to find in the AWS Console and CLI.

The command returns JSON output that includes the new `SnapshotId`.  
**Save this `SnapshotId`**, because you will need it in the next step.

---

### Step 3: Wait Until the Snapshot is Completed

Creating a snapshot is an **asynchronous** operation.  
Use the `wait` command to block until the snapshot reaches the `completed` state:

```bash
aws ec2 wait snapshot-completed \
  --snapshot-ids <SNAPSHOT_ID>
```

Replace `<SNAPSHOT_ID>` with the ID returned by `create-snapshot`, for example:

```bash
aws ec2 wait snapshot-completed \
  --snapshot-ids snap-0123456789abcdef0
```

Once this command returns without errors, your snapshot is fully created and ready to use.

---

### Step 4: Verify the Snapshot

You can verify the snapshot using:

```bash
aws ec2 describe-snapshots --snapshot-ids <SNAPSHOT_ID>
```

This will show details such as:
- `State` (should be `completed`)
- `VolumeId`
- `Description`
- `Tags`

---

### Practical Notes & Best Practices

- **Region**:
  - Ensure your AWS CLI is configured for the correct Region (or pass `--region` explicitly).
- **Permissions**:
  - Your IAM user/role needs permissions such as:
    - `ec2:DescribeVolumes`
    - `ec2:CreateSnapshot`
    - `ec2:DescribeSnapshots`
    - `ec2:CreateTags` (for tagging snapshots)
- **Impact on Performance**:
  - The first snapshot of a volume is a full copy; subsequent snapshots are incremental.
  - There may be a **slight performance impact** on the volume while the snapshot is being created.
- **Cost**:
  - You are charged for the storage used by your snapshots.
  - Deleting a snapshot can reduce costs, but be careful—removing snapshots may affect your backup chain.

---

### Summary of Commands

1. **List volumes** to find the Volume ID:

   ```bash
   aws ec2 describe-volumes
   ```

2. **Create a snapshot** with description and tags:

   ```bash
   aws ec2 create-snapshot \
     --volume-id vol-0202bda43700b5ac9 \
     --description "datacenter Snapshot" \
     --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=datacenter-vol-ss}]'
   ```

3. **Wait for completion**:

   ```bash
   aws ec2 wait snapshot-completed \
     --snapshot-ids <SNAPSHOT_ID>
   ```

4. **Verify the snapshot**:

   ```bash
   aws ec2 describe-snapshots --snapshot-ids <SNAPSHOT_ID>
   ```

