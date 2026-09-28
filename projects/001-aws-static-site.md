# 001 – Secure Static Site on AWS

**Status:** Live  
**Repository:** [github.com/devgershon/aws-static-site](https://github.com/devgershon/aws-static-site)  
**Live site:** [https://d2eeybsp9y6gvd.cloudfront.net](https://d2eeybsp9y6gvd.cloudfront.net)  
**Time taken:** ~2 weeks (alongside full-time work)

---

## What it is

A personal portfolio site hosted on AWS. The site content is minimal. The real work is the infrastructure:

- Private S3 bucket (no public access)
- CloudFront for global delivery and HTTPS
- Origin Access Control so only CloudFront can read from the bucket
- ACM certificate for TLS
- Full infrastructure defined in Terraform
- Automated deploys with GitHub Actions

No manual uploads or console changes after the initial setup.

---

## Architecture

```
Browser → CloudFront (CDN + HTTPS) → S3 (private bucket)
                │
        ACM Certificate (us-east-1)
```

| Service | Role |
|---------|------|
| **S3** | Stores the static files. Fully private. |
| **CloudFront** | Serves content from edge locations. Handles HTTPS and caching. |
| **ACM** | TLS certificate. Must be issued in `us-east-1` for CloudFront. |
| **Origin Access Control** | Restricts S3 access to CloudFront only (SigV4). |
| **Terraform** | Defines every resource as code. |
| **GitHub Actions** | On every push to `main`: syncs files to S3 and invalidates the CloudFront cache. |

---

## Why this design

The simplest approach is a public S3 website endpoint. It works, but anyone can bypass CloudFront and reach the bucket directly over HTTP.  

Keeping the bucket private and forcing all traffic through CloudFront with Origin Access Control is the production-style pattern: one controlled entry point, HTTPS enforced, and no public origin.

---

## Repo structure

```
aws-static-site/
├── terraform/
│   ├── main.tf          # Providers + us-east-1 alias for ACM
│   ├── variables.tf
│   ├── s3.tf            # Bucket, encryption, OAC, bucket policy
│   ├── cloudfront.tf
│   ├── acm.tf
│   ├── route53.tf       # Present but unused (CloudFront URL used)
│   └── outputs.tf
├── website/
│   └── index.html
├── .github/workflows/
│   └── deploy.yml
└── README.md
```

---

## Key lessons

**ACM must live in us-east-1 for CloudFront**  
This applies even when the rest of the stack is in another region (`eu-west-2` here). Handled in Terraform with a second provider alias.

**OAC is the current standard, not OAI**  
Most older tutorials still show Origin Access Identity. Origin Access Control is the recommended approach. Aligning the bucket policy correctly took a few iterations.

**Credential errors can be misleading**  
An `InvalidClientTokenId` error turned out to be a bad paste during `aws configure`, not invalid keys. Simple verification with `aws sts get-caller-identity` remains useful.

---

## Cost

Effectively $0 per month for normal personal traffic on the free tier.

| Service | Free tier |
|---------|-----------|
| S3 | 5 GB storage, 20,000 GET requests/month |
| CloudFront | 1 TB data transfer, 10 million requests/month |
| ACM | Free when used with CloudFront |

---

## Pillars covered

- Infrastructure as Code (Terraform)
- Secure content delivery (private origin + OAC)
- CDN and TLS configuration
- CI/CD automation (GitHub Actions)

---

## Next

002 – Serverless REST API (Lambda + API Gateway + DynamoDB + Terraform)