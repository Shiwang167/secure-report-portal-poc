# Secure Report Portal — PoC

Proof of concept for serving files from a private S3 bucket through CloudFront,
authenticated via AWS IAM Identity Center (SAML 2.0) using Lambda@Edge —
no Amazon Cognito.

## Status

Work in progress. See `docs/` for the step-by-step build log.

## Architecture (target)

```
User Browser
    │
    ▼
CloudFront (OAC → private S3 origin)
    │
    ├─ Lambda@Edge (viewer-request): checks JWT cookie
    │       │
    │       ├─ valid   → pass through to S3 origin
    │       └─ missing/invalid → redirect to IAM Identity Center SAML SSO
    │
    ▼
S3 (private bucket, reports/finance/*, reports/hr/*)

Separate (non-edge) endpoint:
IAM Identity Center → SAML assertion POST → ACS Lambda (API Gateway)
    → validates SAML signature → issues self-signed JWT → sets cookie → redirects to CloudFront
```

## Why not native OIDC with Identity Center?

IAM Identity Center's OIDC endpoints are built for the CLI/SDK device-code flow,
not browser-based authorization-code flows for arbitrary web apps. For a custom
web portal, IdC integrates via **SAML 2.0** (customer-managed application).
This repo issues its own signed JWT after validating the SAML assertion, so
Lambda@Edge only ever deals with fast, stateless JWT validation.

## Folder structure in S3

```
finance/finance-index.html
finance/Finance Image.jpg
finance/Finance2.jpg
hr/hr-index.html
hr/HR.jpg
```

## Build log

See `docs/` for a step-by-step record of what's been done and why.
