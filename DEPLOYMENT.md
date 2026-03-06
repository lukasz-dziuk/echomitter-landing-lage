# Landing deployment

This repository deploys **only from `main`** via GitHub Actions.

## 1) Create hosting stack (one-time)

Deploy `infra/template.yaml` in AWS (CloudFormation or SAM).

Required parameters:
- `Stage` (e.g. `production`)
- `DomainName` (e.g. `echomitter.com`)
- `WwwDomainName` (optional, e.g. `www.echomitter.com`)
- `HostedZoneId` (Route53 hosted zone ID)
- `CertificateArn` (ACM cert in `us-east-1`)

The stack outputs:
- `SiteBucketName`
- `SiteDistributionId`

## 2) Configure repository secrets

Set these secrets in `echomitter-landing-page` repo:

- `AWS_ROLE_ARN` - IAM role assumed by GitHub OIDC
- `AWS_REGION` - region for CloudFormation/S3 lookups (e.g. `eu-central-1`)
- `LANDING_STACK_NAME` - stack name that exposes `SiteBucketName` and `SiteDistributionId`

## 3) Deploy flow

- Push to `main`
- Workflow `.github/workflows/deploy.yml` runs automatically
- It uploads:
  - `index.html`
  - `invite-warden.html`
- It invalidates CloudFront paths:
  - `/index.html`
  - `/invite-warden`
  - `/invite-warden.html`

## Notes

- CloudFront Function in infra rewrites extensionless paths (`/invite-warden` -> `/invite-warden.html`).
- HTML files are uploaded with `no-store` cache-control.
