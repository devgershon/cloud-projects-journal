# 002 – Serverless REST API on AWS

**Status:** Live  
**Repository:** [github.com/devgershon/aws-serverless-api](https://github.com/devgershon/aws-serverless-api)  
**Live API:** [https://t0tn9g17b7.execute-api.eu-west-2.amazonaws.com/posts](https://t0tn9g17b7.execute-api.eu-west-2.amazonaws.com/posts)  
**Integrates with:** Project 001 (portfolio site blog section)

---

## What it is

A fully serverless REST API that powers the blog section of the portfolio site from Project 1. It exposes five CRUD operations for blog posts, with no servers to manage and no cost when idle.

Core services:

- **Lambda** (Python 3.12) — runs the application code only when invoked
- **API Gateway** (HTTP API) — receives HTTP requests and routes them to Lambda
- **DynamoDB** — stores post data (NoSQL, pay-per-request)
- **IAM** — least-privilege execution role for Lambda
- **CloudWatch Logs** — execution and access logs
- **Terraform** — all infrastructure as code
- **GitHub Actions** — tests gate every deploy; code updates via `update-function-code`

---

## Architecture

```
Client (browser / curl)
        │
        ▼
API Gateway (HTTP API)
  • Routing + CORS
        │
        ▼
Lambda (Python handler)
  • Routes by method + path
  • Reads/writes via boto3
        │
        ▼
DynamoDB (blog-api-posts)
  • Partition key: id (UUID)
        │
CloudWatch Logs
  • Lambda execution logs
  • API Gateway access logs
```

### Endpoints

| Method | Path | Action | Success |
|--------|------|--------|---------|
| POST | `/posts` | Create post | 201 |
| GET | `/posts` | List posts (optional `?status=`) | 200 |
| GET | `/posts/{id}` | Get one post | 200 / 404 |
| PUT | `/posts/{id}` | Update post | 200 / 404 |
| DELETE | `/posts/{id}` | Delete post | 200 / 404 |

---

## Design decisions

**Why serverless**  
A blog API with low, spiky traffic does not justify a always-on EC2 instance. Lambda scales to zero, charges per request, and removes OS patching and capacity management. Cold starts are an acceptable trade-off for this workload.

**HTTP API over REST API**  
HTTP API is cheaper, lower latency, and simpler for Lambda proxy integrations. Advanced REST API features (usage plans, request validation, caching) were not required.

**Single Lambda function**  
One handler with internal routing keeps deployment and readability simple for an API of this size. Cold-start behaviour is the same either way.

**Least-privilege IAM**  
The Lambda execution role can only perform five DynamoDB actions (`GetItem`, `PutItem`, `UpdateItem`, `DeleteItem`, `Scan`) on this specific table ARN. Trust policy allows only the Lambda service to assume the role.

**CORS locked to the portfolio origin**  
`Access-Control-Allow-Origin` is set to the CloudFront domain of Project 1, not `*`. Only that origin can call the API from a browser.

**Tests with moto**  
Unit tests simulate DynamoDB locally via moto. No real AWS account is needed in CI. Tests run in ~3 seconds and gate the deploy job.

**Code deploy vs full Terraform apply**  
Infrastructure changes go through Terraform. Routine code updates use `aws lambda update-function-code` in the pipeline for speed.

---

## Key lessons from the build

1. **IAM has two layers for Lambda**  
   The execution role (what Lambda can do) and the resource policy (who can invoke Lambda) are separate. Both must be correct. Missing `lambda:InvokeFunction` for API Gateway produced 403s even with a perfect role.

2. **CloudWatch Logs is its own permission set**  
   `CloudWatchLogsFullAccess` is not implied by Lambda or API Gateway policies. Terraform failed part-way through until the log-group permissions were added.

3. **Module-level boto3 clients + moto**  
   Creating the DynamoDB client at module load is good for warm starts, but tests must import the handler inside the `mock_aws()` context and re-bind the table reference.

4. **CI needs a region even for mocks**  
   Local machines often have a default region; GitHub Actions runners do not. Dummy `AWS_DEFAULT_REGION` (and dummy credentials) were required so boto3 would initialise before moto intercepted the calls.

5. **DynamoDB updates are expression-based**  
   Partial updates require building `UpdateExpression`, `ExpressionAttributeNames`, and `ExpressionAttributeValues`. A condition expression was used to return a proper 404 when the item does not exist.

---

## Integration with Project 1

A small JavaScript module on the portfolio site fetches published posts from this API and renders them. This is a simple service-to-service call over HTTP between two independently deployed AWS systems (CloudFront origin → API Gateway).

---

## Cost

Effectively $0/month at personal-project traffic levels (well within free tier for Lambda, HTTP API, DynamoDB, and CloudWatch Logs).

---

## Skills demonstrated

- Serverless architecture (Lambda + API Gateway + DynamoDB)
- Least-privilege IAM (trust policy + resource policy + scoped actions)
- Infrastructure as Code with Terraform
- CI/CD with test gating (pytest + moto + GitHub Actions)
- CORS for cross-origin browser access
- Observability basics (CloudWatch Logs)
- Practical Python + boto3

---

## Pillars covered

- Serverless
- Security (IAM, CORS, private data plane)
- Infrastructure as Code
- CI/CD and automated testing
- Integration between independently deployed systems

---

## Next

003 – Containerised application + CI/CD (Docker, ECR, ECS Fargate)