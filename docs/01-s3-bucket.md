# Step 1 — Private S3 Bucket

## What was done
- Created a private S3 bucket (`my-test-bucket-<account-id>`).
- Block Public Access: all 4 settings enabled.
- Default encryption: SSE-S3.
- Uploaded folder structure:
  ```
  finance/Finance Image.jpg
  finance/Finance2.jpg
  hr/HR.jpg
  ```

## Verification
- `aws s3 ls` confirms objects present.
- Direct `https://<bucket>.s3.amazonaws.com/...` request returns 403 —
  confirms bucket is not publicly accessible before CloudFront is wired up.
