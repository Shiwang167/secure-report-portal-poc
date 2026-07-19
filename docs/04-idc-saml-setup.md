# Step 3 — IAM Identity Center as SAML 2.0 IdP (in progress)

## Decision record
IAM Identity Center does not offer a standard browser authorization-code
OIDC flow for third-party web apps — its OIDC endpoints are for CLI/SDK
device-code auth. Integration for a custom portal is via **SAML 2.0**
(customer-managed application). This portal will:

1. Redirect unauthenticated users to IdC's SAML SSO URL.
2. Receive a SAML assertion via POST at an ACS endpoint (API Gateway + Lambda,
   not Lambda@Edge — edge functions can't cleanly host a SAML POST callback).
3. Validate the assertion's signature against IdC's published certificate.
4. Self-issue a signed JWT and set it as an HttpOnly cookie.
5. All subsequent requests are validated purely as JWT checks in Lambda@Edge
   (fast, stateless, no round-trip to IdC per request).

## What was done
- Enabled IAM Identity Center (Organization instance, us-east-1, built-in
  identity store).
- Created test users: `finance.test`, `hr.test`.
- Created groups: `finance-users` (member: finance.test),
  `hr-users` (member: hr.test).
- Created customer-managed SAML 2.0 application "Secure Report Portal PoC".
  - ACS URL: placeholder `https://auth.example.com/saml/acs`
    (to be replaced with real API Gateway URL in Step 4).
  - Entity ID / SAML audience: `urn:secure-report-portal:poc`
- Downloaded IdC IdP metadata XML (kept out of git — see .gitignore).
- Assigned both groups to the application.
- Added attribute mapping for `https://aws.amazon.com/SAML/Attributes/Groups`
  → `${user:groups}` so group membership is present in the SAML assertion,
  for the future folder-level authorization enhancement.

## Not yet done
- Real ACS URL (depends on Step 4 Lambda/API Gateway existing).
- SAML signature validation code.
- JWT minting logic.

## Gotchas to watch for
- Entity ID must match byte-for-byte between IdC config and validation code.
- Users must accept their invite before they can authenticate.
- Skipping the Groups attribute mapping now would require reconfiguring later.
