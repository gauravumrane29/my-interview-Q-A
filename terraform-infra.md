# 3-Tier EKS Infrastructure — Architecture & Roadmap

## Goal
Design and implement a production-ready 3-tier infrastructure (web, application, data) on AWS using Terraform and EKS for application deployment.

## Architecture Diagram (Mermaid)

```mermaid
flowchart LR
  subgraph AWSAccount["AWS Account(s)"]
    direction TB
    subgraph Networking["Networking"]
      VPC["VPC"]
      PublicSubnet["Public Subnets\n(ALB, NAT GW)"]
      PrivateSubnetApp["Private Subnets - App\n(EKS worker nodes / Fargate)"]
      PrivateSubnetData["Private Subnets - Data\n(RDS, ElastiCache)"]
      IGW["Internet Gateway"]
      NAT["NAT Gateway"]
    end

    subgraph ControlPlane["Control Plane"]
      EKSControl["EKS Managed Control Plane"]
    end

    subgraph PlatformServices["Platform Services"]
      ALB["ALB / Ingress Controller"]
      ECR["ECR (Images)"]
      S3["S3 - Terraform State, Artifacts"]
      CloudWatch["CloudWatch / Logging"]
      XRay["X-Ray/Tracing"]
      SecretsMgr["Secrets Manager / Parameter Store"]
      EFS["EFS (optional)"]
    end

    subgraph Compute["Compute"]
      EKSNodes["EKS Worker Node Groups / Fargate Profiles"]
      Pods["App Pods (Frontend / Backend / API)"]
    end

    subgraph DataTier["Data Tier"]
      RDS["RDS (Primary + Read Replica)"]
      ElastiCache["ElastiCache (Redis/Memcached)"]
    end

    VPC --> PublicSubnet
    VPC --> PrivateSubnetApp
    VPC --> PrivateSubnetData
    PublicSubnet --> ALB
    ALB --> PrivateSubnetApp
    PrivateSubnetApp --> EKSNodes
    EKSNodes --> Pods
    Pods --> RDS
    Pods --> ElastiCache
    EKSControl -.managed by AWS.-> EKSNodes
    ECR --> Pods
    S3 --> TerraformState["Terraform Remote State"]
    CloudWatch -.collects-> Pods
    SecretsMgr -.used-by-> Pods
  end

  subgraph DevOps["DevOps"]
    Jenkins["Jenkins / CI"]
    Git["Git (repos for infra & apps)"]
    Argo["ArgoCD / Flux (optional GitOps)"]
  end

  Git --> Jenkins
  Jenkins --> ECR
  Jenkins --> GitOpsTrigger["Update GitOps Repo"]
  Jenkins --> TerraformApply["CI/CD invokes Terraform pipelines or runs in CI"]
  TerraformApply --> S3
  TerraformApply --> AWSAccount
  GitOpsTrigger --> Argo
  Argo --> EKSControl
```

## Tier mapping
- Presentation (Web): ALB + Frontend pods in EKS (public-facing via ALB).
- Application (App/API): Backend pods in EKS running in private subnets.
- Data (DB/Cache): RDS and ElastiCache in private subnets.

## Terraform repo layout (recommended)
- repo-root/
  - README.md
  - infra/
    - environments/
      - dev/
        - main.tf (calls modules with dev vars)
        - backend.tf
        - terraform.tfvars
      - staging/
      - prod/
    - modules/
      - network/
        - main.tf
        - variables.tf
        - outputs.tf
      - eks/
        - main.tf
        - variables.tf
        - outputs.tf
      - node_group/
      - alb_ingress/
      - rds/
      - monitoring/
      - iam/
  - apps/ (application repositories, Helm charts / k8s manifests)
  - ci/ (Jenkinsfiles, pipelines, scripts)

Notes:
- Keep infra “modules” reusable and parameterized per environment.
- Use a remote backend (S3 + DynamoDB locking).
- Separate state per environment/account.

## Phased Roadmap (recommended timeline & checks)

Phase 0 — Planning (0.5–1 week)
- Identify accounts: dev/staging/prod; choose AWS partition/regions.
- Define VPC CIDR plan, subnet sizing, AZs.
- Decide node types: managed node groups vs Fargate vs mixed.
- Security & compliance requirements: encryption, logging, audit.

Phase 1 — Bootstrap infra + remote state (1 week)
- Create S3 bucket + DynamoDB table for.tf state locking (manually or via a tiny bootstrap script).
- Create ci user/role with limited permissions to run Terraform.
- Verify remote state read/write and locking.

Phase 2 — Network module (1–2 weeks)
- Implement Terraform network module: VPC, public/private subnets across AZs, NAT Gateway, route tables.
- Outputs: subnet ids, route table ids, entpoint ids.
- Test: launch a simple EC2 instance into subnets (smoke test).

Phase 3 — EKS base (1–2 weeks)
- Implement eks module:
  - aws_eks_cluster (managed control plane)
  - OIDC provider for IRSA
  - aws_iam_role for cluster
  - Core add-ons: aws-load-balancer-controller, metrics-server, cluster-autoscaler (via helm provider or eks addons)
- Create managed node groups or Fargate profiles.
- Test: deploy a sample nginx pod via kubectl.

Phase 4 — Platform integrations (1–2 weeks)
- Deploy ALB ingress controller (AWS LB Controller)
- Setup ECR lifecycle & permissions
- Enable CloudWatch Container Insights / Fluent Bit logs
- Implement IAM roles for Service Accounts (IRSA) for app integrations (S3, SecretsManager, SSM)

Phase 5 — Data tier & security (1–2 weeks)
- Implement RDS module (multi-AZ, subnet group, encrypted)
- Implement ElastiCache module
- Configure security groups and least privilege rules
- Add network ACLs, restrict public access

Phase 6 — CI/CD & GitOps (1–2 weeks)
- CI pipeline builds container images, pushes to ECR, runs security scans.
- Option A: Jenkins deploys helm charts directly to clusters (needs kubeconfig/assume role).
- Option B (recommended): CI updates a GitOps repo (manifests/helm values) and ArgoCD/Flux applies to cluster.
- Implement promotion flow (dev → staging → prod) and manual approvals.

Phase 7 — Observability, SLOs, and hardened production (ongoing)
- Add Prometheus/Grafana or managed monitoring, tracing, centralized logging.
- Add image signing (cosign) & runtime security (Falco, OPA/Gatekeeper).
- Audit & backups (RDS snapshots, S3 lifecycle).

## Key recommendations & best practices
- Use OIDC + IRSA — never mount AWS creds into pods.
- Use immutable image tags (SHA) and artifact promotion between environments.
- Keep Terraform modules focused: network, compute (eks), storage (rds), platform (ingress, monitoring).
- Enable automated backups & encryption for data stores.
- Least-privilege for Terraform CI user/role; separate principals for environments.
- Use multiple AWS accounts for environment separation (centralized tooling account for CI/terraform state bucket).
- Use CI for infra plan validation (terraform plan + policy checks) and Require human approval before terraform apply to prod.
- Automate cluster add-ons with Helm or eksctl/terraform-managed helm providers.

## Example terraform workflow
1. Work on infra/modules in feature branch.
2. Run `terraform init` with remote backend.
3. Create PR — CI runs `terraform fmt`, `terraform validate`, `terraform plan`.
4. When PR is approved, apply in non-prod automatically or trigger apply via pipeline.
5. For prod apply, require manual approval and run `terraform apply` in CI or a controlled workspace.

## Next actions (practical)
- Choose one environment to bootstrap first (dev).
- Create the remote state bucket & locking table.
- Implement the network module and run `terraform apply` to create VPC/subnets.
- Implement EKS module and validate with a sample application.
