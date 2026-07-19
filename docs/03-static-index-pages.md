# Step 2.5 — Static Index Pages

## What was done
Since S3+CloudFront via OAC grants `GetObject` only (no `ListBucket`/browsing),
plain static `index.html` pages were added per folder so a user can discover
available files without knowing exact object keys in advance.

Uploaded to S3 with explicit content-type so browsers render instead of
downloading:

```bash
aws s3 cp web/finance/finance-index.html s3://<bucket>/finance/finance-index.html --content-type "text/html"
aws s3 cp web/hr/hr-index.html s3://<bucket>/hr/hr-index.html --content-type "text/html"
```

## Known limitation (accepted for now)
This is a placeholder. It will be replaced by a dynamic listing (Lambda +
`s3:ListObjectsV2` scoped by prefix) once the authentication flow works
end-to-end — the same "what can this user see" logic will double as the
folder-level authorization check called out as a future enhancement.

## Verification
- `https://<distribution>.cloudfront.net/finance/finance-index.html` renders
  as HTML with working links to both finance images.
- Same for `/hr/hr-index.html`.
