# Cloud Engineering Project Journal

Personal journal documenting hands-on cloud engineering projects on AWS.

This repository records what was built, the architecture decisions behind it, and the practical lessons learned. It is written to demonstrate real engineering judgment rather than tutorial steps.

**Author:** Gershon Normenyo  
**GitHub:** [github.com/devgershon](https://github.com/devgershon)

---

## Projects

| # | Project | Stack | Status | Links |
|---|---------|-------|--------|-------|
| 001 | Secure Static Site on AWS | S3 · CloudFront · ACM · OAC · Terraform · GitHub Actions | Live | [Repo](https://github.com/devgershon/aws-static-site) · [Live](https://d2eeybsp9y6gvd.cloudfront.net) |
| 002 | Serverless REST API | Lambda · API Gateway · DynamoDB · IAM · Terraform · GitHub Actions | Live | [Repo](https://github.com/devgershon/aws-serverless-api) · [API](https://t0tn9g17b7.execute-api.eu-west-2.amazonaws.com/posts) |
| 003 | Containerised App + CI/CD | Docker · ECR · ECS Fargate · GitHub Actions | Planned | — |
| 004 | Production VPC Architecture | VPC · Subnets · NAT · ALB · RDS · Terraform | Planned | — |

→ Full write-ups in [`projects/`](./projects/)

---

## Focus

These projects cover the core pillars expected in cloud and platform engineering roles:

- Infrastructure as Code
- Secure architecture
- CI/CD and automation
- Networking fundamentals
- Operational awareness

---

## How this repo is organised

```
cloud-projects-journal/
├── projects/          # One markdown file per project (001, 002…)
├── learnings/         # Cross-cutting notes (optional)
├── assets/            # Diagrams and supporting files
└── README.md
```

---

*Started: September 2026 · Last updated: September 2026*
