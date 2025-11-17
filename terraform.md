# Terraform Guide — Provision a 3‑Tier App (Web → App → DB) on AWS (From Scratch)

Target audience: DevOps engineers starting a fresh Terraform project to provision a 3‑tier application on AWS (web → app → DB). This document gives a clear, corporate‑style, step‑by‑step plan: what to create first and why, example repo layout, minimal Terraform snippets, safe apply workflow, and best practices.

---

## Quick TL;DR

1. Create repo + remote state (S3 + DynamoDB or Terraform Cloud).
2. Provider & shared variables.
3. Network (VPC, public/private subnets, IGW, NAT).
4. Bootstrap IAM (roles & instance profiles).
5. Logging & monitoring foundations.
6. Security Groups (web, app, db).
7. ALB + target groups (public).
8. Compute (Launch Template + ASG or EC2 in private subnets).
9. Database (RDS / Aurora in private subnets, secrets in Secrets Manager).
10. DNS & TLS (Route53 + ACM).
11. CI/CD + observability.

Create network + IAM + remote state first — everything else depends on these.

---

## Prerequisites

- AWS account with permission to create VPC, EC2, IAM, ALB, RDS, S3, DynamoDB, Secrets Manager, Route53.
- Terraform v1.2+ installed.
- AWS CLI configured locally or CI runner able to assume an appropriate role.
- Git experience.

---

## Recommended repo layout

Use modules for reusable components and separate environments:

```
infra-terraform/
├─ modules/
│  ├─ vpc/
│  ├─ iam/
│  ├─ security_groups/
│  ├─ alb/
│  ├─ autoscaling_ec2/
│  └─ rds/
├─ envs/
│  ├─ dev/
│  │  ├─ backend.tfvars
│  │  └─ terraform.tfvars
│  └─ prod/
├─ global/
│  ├─ provider.tf
│  ├─ backend.tf
│  ├─ variables.tf
│  └─ outputs.tf
├─ network/
│  └─ main.tf   # calls modules/vpc
├─ compute/
│  └─ main.tf   # alb + asg
├─ database/
│  └─ main.tf   # rds
└─ README.md
```

Each module should include at least: `main.tf`, `variables.tf`, `outputs.tf`, and optionally `versions.tf` and `examples/`.

---

## Step‑by‑step setup (what files to create and in what order)

1. Initialize repo
   - git init; git checkout -b feat/terraform-init
   - Create the above folder structure.

2. Create remote state (one‑time manual AWS step)
   - Create S3 bucket for state and DynamoDB table for locks (or set up Terraform Cloud).
   - Put S3 bucket name and DynamoDB table name into `envs/<env>/backend.tfvars`.

3. Provider & backend configuration (global/)
   - Files: `global/provider.tf`, `global/backend.tf`, `global/variables.tf`.
   - Example: `terraform init -backend-config=../envs/dev/backend.tfvars` to initialize.

4. Networking module (modules/vpc) — apply FIRST
   - Implement VPC, public and private subnets across at least 2 AZs, IGW, NAT gateway(s), route tables.
   - Output: vpc_id, public_subnets, private_subnets, azs.
   - Apply network so other modules can reference outputs.

5. Bootstrap IAM (modules/iam)
   - Create EC2 instance role (SSM & SecretsManager permissions), instance profile, and any service roles required by ALB or CI pipelines.
   - Output: instance_profile_name, role_arns.

6. Logging & monitoring basics
   - Create S3 bucket for ALB logs and CloudWatch log groups with retention.
   - Attach minimal IAM permissions to allow ALB to write logs if needed.

7. Security groups (modules/security_groups)
   - Create: alb_sg (allow 80/443 from internet), web_sg (allow traffic from alb_sg), db_sg (allow traffic from web_sg on DB port).
   - Output SG IDs for use by ALB / ASG / RDS.

8. ALB + target groups (modules/alb)
   - Create ALB in public subnets, target group(s), HTTP listener (and 443 + ACM later).
   - Output: alb_arn, alb_dns_name, target_group_arn.

9. Compute (modules/autoscaling_ec2)
   - Launch Template (AMI id, instance_type, IAM instance profile, user_data).
   - Auto Scaling Group in private subnets attached to ALB target group.
   - Use SSM for admin access (avoid SSH keys).

10. Database (modules/rds)
    - DB subnet group (private subnets), RDS instance or Aurora cluster, security group, automated backups.
    - Store DB credentials in AWS Secrets Manager; do not hardcode in TF state.

11. DNS & TLS
    - Request ACM certificate (DNS validation), create Route53 record (A alias) pointing to ALB.

12. CI/CD integration
    - Add pipeline to run `terraform fmt` / `terraform validate` / `terraform plan` on PRs; apply via CI with approvals for prod.

---

## Minimal, copy‑paste Terraform snippets

Provider + backend (global/provider.tf)
```hcl
terraform {
  required_version = ">= 1.2"
  required_providers { aws = { source = "hashicorp/aws" ; version = "~> 4.0" } }
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "envs/dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "tf-locks"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region
}
```

Small VPC example (modules/vpc/main.tf)
```hcl
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr
  tags = merge(var.tags, { Name = "${var.name}-vpc" })
}
resource "aws_subnet" "public" {
  for_each = toset(var.azs)
  vpc_id = aws_vpc.this.id
  cidr_block = cidrsubnet(var.vpc_cidr, 8, index(var.azs, each.key))
  availability_zone = each.key
  map_public_ip_on_launch = true
  tags = { Name = "${var.name}-public-${each.key}" }
}
```

IAM (modules/iam/main.tf)
```hcl
data "aws_iam_policy_document" "ec2_assume" {
  statement { actions = ["sts:AssumeRole"] principals { type = "Service" ; identifiers = ["ec2.amazonaws.com"] } }
}
resource "aws_iam_role" "ec2" {
  name               = "${var.name}-ec2-role"
  assume_role_policy = data.aws_iam_policy_document.ec2_assume.json
}
resource "aws_iam_instance_profile" "ec2_profile" {
  name = "${var.name}-instance-profile"
  role = aws_iam_role.ec2.name
}
```

ALB (modules/alb/main.tf)
```hcl
resource "aws_lb" "alb" {
  name               = "${var.name}-alb"
  load_balancer_type = "application"
  subnets            = var.public_subnets
  security_groups    = [var.alb_sg_id]
}
resource "aws_lb_target_group" "web_tg" {
  name   = "${var.name}-tg"
  port   = 80
  protocol = "HTTP"
  vpc_id = var.vpc_id
  health_check { path = "/" }
}
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.alb.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web_tg.arn
  }
}
```

RDS (modules/rds/main.tf)
```hcl
resource "aws_db_subnet_group" "db_subnets" {
  name       = "${var.name}-dbsubnet"
  subnet_ids = var.private_subnets
}
resource "aws_db_instance" "db" {
  identifier              = "${var.name}-db"
  engine                  = "postgres"
  engine_version          = "15"
  instance_class          = var.instance_class
  allocated_storage       = var.storage
  username                = var.db_user
  password                = var.db_password # prefer Secrets Manager
  vpc_security_group_ids  = [var.db_sg_id]
  db_subnet_group_name    = aws_db_subnet_group.db_subnets.name
  multi_az                = var.multi_az
  backup_retention_period = 7
  skip_final_snapshot     = false
}
```

---

## Safe apply workflow (team-friendly)

1. terraform fmt && terraform validate
2. terraform init -backend-config="envs/<env>/backend.tfvars"
3. terraform plan -out plan.tfplan
4. Review changes (specially replacements like RDS)
5. terraform apply "plan.tfplan"
6. Use CI to run plan in PRs; restrict apply to authorized runners with approvals for prod.

---

## Best practices & gotchas

- Remote state & locking (S3 + DynamoDB) is mandatory for teams.
- Split state by environment and by large logical boundaries (network vs compute).
- Use IAM roles (assume-role in CI) — avoid long-lived AWS keys.
- Secrets: use AWS Secrets Manager or SSM Parameter Store (SecureString).
- Bake AMIs for prod (Packer) instead of heavy userdata for deterministic builds.
- Test in dev with cheaper options (single AZ, smaller instance types). Replicate multi‑AZ for prod.
- NAT Gateways: cost per AZ. Balance cost vs availability (single NAT for dev; per-AZ NAT for prod).
- Always review plans for destructive changes (RDS replacement, instance replacement).

---

## Checklist

- [ ] Create S3 bucket + DynamoDB table for backend & locks
- [ ] Add `envs/dev/backend.tfvars` and `envs/prod/backend.tfvars`
- [ ] Implement `modules/vpc` and apply
- [ ] Implement `modules/iam` and apply
- [ ] Implement `modules/security_groups` and outputs
- [ ] Implement `modules/alb` + test ASG for static content
- [ ] Implement `modules/rds` with Secrets Manager integration
- [ ] Request ACM cert and create Route53 record for ALB
- [ ] Add CI pipeline: fmt/validate/plan (PR) and apply (protected)
- [ ] Add monitoring & alerting (CloudWatch dashboards / alarms)

---

## Next steps I can help with

I can:
- Scaffold the repo (modules + root) with working Terraform code for a simple 2‑AZ setup (VPC, ALB, ASG with a sample AMI, RDS single‑AZ). This will be runnable once you configure backend and credentials.
- Or produce a focused, detailed module first (VPC or RDS) if you want to start smaller.

Which would you like me to produce now? I can scaffold the files in this repo immediately.
