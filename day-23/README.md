## Day 23 - S3: Create a Bucket and Sync Objects Between Buckets

### Overview

Create a new S3 bucket, then **sync** objects from an existing bucket into it with `aws s3 sync`. Sync copies or updates objects on the destination to match the source **without** deleting from the source. If you need a true **move** (copy then remove from the source), use `aws s3 mv` instead (see below).

---

### 1. Create the destination bucket

```bash
aws s3 mb s3://nautilus-sync-22719
```

`aws s3 mb` creates an empty bucket in your default Region (unless you pass `--region`).

---

### 2. Sync all objects from the source bucket to the new bucket

```bash
aws s3 sync s3://nautilus-s3-19027 s3://nautilus-sync-22719
```

- **`s3://nautilus-s3-19027`**: source bucket
- **`s3://nautilus-sync-22719`**: destination bucket

`sync` only **transfers what is new or changed** (by size/etag), so re-runs are efficient. It does **not** delete objects from the source. To also remove extra keys on the destination so it mirrors the source exactly, add **`--delete`** (use with care).


### Notes

- **Spelling**: for `mv`, the CLI flag is **`--recursive`**, not `--recersive`.
- **Same Region**: for best performance and to avoid cross-Region charges, create the new bucket in the **same Region** as the source (use `aws s3 mb ... --region <region>` if needed).
- **Verify**: `aws s3 ls s3://nautilus-sync-22719 --recursive` and `aws s3 ls s3://nautilus-s3-19027 --recursive` to confirm counts and prefixes.
- **Preview sync**: `aws s3 sync s3://nautilus-s3-19027 s3://nautilus-sync-22719 --dryrun`
