# Step 2 — CloudFront Distribution + Origin Access Control

## What was done
- Created an Origin Access Control (OAC), signing behavior "Sign requests".
- Created a CloudFront distribution with the S3 bucket as origin via OAC
  (not the S3 static-website endpoint — OAC requires the REST endpoint).
- Applied bucket policy granting `s3:GetObject` to the CloudFront service
  principal, scoped to this distribution's ARN via `AWS:SourceArn`.
- Viewer domain: default `*.cloudfront.net` for PoC (no ACM/custom domain yet).
- No authentication at this stage — deliberately isolating origin access
  from the auth layer to be built in later steps.

## Bucket policy (reference)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::BUCKET_NAME/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

## Verification
- `curl -I https://<distribution>.cloudfront.net/finance/Finance%20Image.jpg` → 200
- Direct S3 URL still → 403 (bucket stayed private)

## Notes / gotchas encountered
- Initial `AccessDenied` (not `NoSuchKey`) on a since-confirmed-existing object —
  CloudFront/S3 intentionally returns the same error for "missing" and
  "no permission" to avoid leaking bucket contents to unauthenticated probes.
  Root cause in this project was a path mismatch between the assumed key
  (`reports/finance/index.html`) and the actual key (`finance/finance-index.html`).
