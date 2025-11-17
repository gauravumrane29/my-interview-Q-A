# DevOps Interview Q&A Guide for Globant

This document provides interview questions and answers for DevOps roles at Globant, focusing on Kubernetes, GitHub Actions, AWS, Python & Scripting, Terraform, and other related DevOps tools. It also includes scenario-based and real-time situation questions.

---

## Table of Contents

- [Kubernetes](#kubernetes)
- [GitHub Actions](#github-actions)
- [AWS](#aws)
- [Python & Python Script](#python--python-script)
- [Terraform](#terraform)
- [Other DevOps Tools](#other-devops-tools)
- [Scenario & Real-Time Based Questions](#scenario--real-time-based-questions)

---

## Kubernetes

### Q1: What is Kubernetes?  
**A:** Kubernetes is an open-source container orchestration platform for automating deployment, scaling, and management of containerized applications.

### Q2: Explain the architecture of Kubernetes.  
**A:** The architecture consists of a Master node (controlling nodes, scheduler, controller managers, etcd) and Worker nodes (running kubelet, container runtime, and pods).

### Q3: What is a Pod in Kubernetes?  
**A:** A Pod is the smallest and simplest unit in Kubernetes that represents a running process in the cluster; it can hold one or more containers.

### Q4: How does service discovery work in Kubernetes?  
**A:** Service objects expose pods, enabling other applications to discover them via DNS or environment variables.

### Q5: How do you perform rolling updates and rollbacks in Kubernetes?  
**A:** Using the `kubectl rollout` command on deployments allows for rolling updates and rollbacks to previous states.

### Q6: What are ConfigMaps and Secrets?  
**A:** ConfigMaps store non-confidential configuration data; Secrets store sensitive information like passwords and tokens.

---

## GitHub Actions

### Q1: What are GitHub Actions?  
**A:** GitHub Actions enable automation of workflows for CI/CD, testing, and deployments within GitHub.

### Q2: How would you trigger workflows in GitHub Actions?  
**A:** Workflows are triggered by events like pushes, pull requests, schedules, or by manual dispatch.

### Q3: What is the difference between jobs and steps in a workflow?  
**A:** Jobs are units of work run on runners; steps are executed sequentially within a job.

### Q4: How do you use secrets in GitHub Actions?  
**A:** Secrets are configured in repository settings and accessed via the `${{ secrets.SECRET_NAME }}` syntax.

---

## AWS

### Q1: What AWS services are commonly used in DevOps?  
**A:** EC2, S3, IAM, Lambda, RDS, DynamoDB, VPC, Aurora, CloudFormation, CodePipeline, CodeBuild, Elastic Beanstalk, and ECS.

### Q2: How would you set up CI/CD pipelines in AWS?  
**A:** By using services such as AWS CodePipeline, CodeBuild, and CodeDeploy.

### Q3: How do you secure resources in AWS?  
**A:** Using IAM roles, policies, security groups, VPCs, encryption, and regularly auditing with AWS Config.

---

## Python & Python Script

### Q1: How is Python used in DevOps?  
**A:** For automation scripts, infrastructure management, writing custom tools, integrating APIs, and test automation.

### Q2: Sample script—How would you write a script to monitor system memory usage?
**A:**  
```python
import psutil
print(f"Memory Usage: {psutil.virtual_memory().percent}%")
```

### Q3: How can you automate AWS resource creation using Python?  
**A:** By using the `boto3` library to interact with the AWS API.

### Q4: How do you schedule Python scripts in Linux?  
**A:** With cron jobs by adding the script to the crontab.

---

## Terraform

### Q1: What is Terraform?  
**A:** An IaC tool that enables you to provision and manage cloud resources declaratively across multiple platforms.

### Q2: How does Terraform manage state?  
**A:** State is stored in a `.tfstate` file locally or remotely, tracking resource changes.

### Q3: Explain modules in Terraform.  
**A:** Modules are reusable groups of resources, allowing better organization and code reusability.

### Q4: How would you perform a safe update to your infrastructure using Terraform?  
**A:** By running `terraform plan` to preview changes and `terraform apply` to enact them after verification.

---

## Other DevOps Tools

- **Ansible:** Automation, configuration management.
- **Jenkins:** CI/CD orchestration.
- **Prometheus & Grafana:** Monitoring and visualization.
- **Docker:** Containerization.
- **Helm:** Kubernetes package management.

---

## Scenario & Real-Time Based Questions

### Scenario 1:  
**Q:** You need to deploy a Python app on Kubernetes with secrets for DB access. How do you approach this?  
**A:** Containerize the app using Docker, deploy via Kubernetes Deployment, mount secrets using Kubernetes Secrets, access them in the app via environment variables.

---

### Scenario 2:  
**Q:** Your GitHub Actions workflow is failing in the deployment job. How would you troubleshoot?  
**A:**  
1. Check workflow and job logs for error messages.  
2. Validate secrets and environment variables.  
3. Ensure required permissions.  
4. Test steps independently.  
5. Check runner configuration and connectivity.

---

### Scenario 3:  
**Q:** AWS account is suspected of a breach. What immediate actions do you take?  
**A:**  
1. Rotate compromised keys.  
2. Revoke suspicious sessions.  
3. Audit logs in CloudTrail.  
4. Assess affected resources.  
5. Apply stricter IAM policies.  
6. Notify team.

---

### Scenario 4:  
**Q:** Infrastructure needs to be created for multiple environments (dev, staging, prod) using Terraform. How would you organize code?  
**A:**  
- Use modules for reusable components.  
- Create workspaces or use directory structure for environments.  
- Use environment-specific variable files for customization.

---

### Scenario 5:  
**Q:** You need to collect system metrics and visualize them. Which tools do you use and how?  
**A:**  
- Use Prometheus to collect metrics from nodes and services.  
- Visualize data using Grafana dashboards.  
- Alerting configured via Prometheus Alertmanager.

---

### Scenario 6:  
**Q:** How would you handle rolling deployment failures in Kubernetes?  
**A:**  
- Use `kubectl rollout undo deployment/<name>` to rollback.  
- Analyze pod logs and events.  
- Fix the issue in configuration/image and redeploy.

---

### Scenario 7:  
**Q:** Automated resource clean-up is needed for non-production environments. Script it in Python.  
**A:**  
```python
import boto3
ec2 = boto3.client('ec2')
for instance in ec2.describe_instances()['Reservations']:
    # Filter logic for non-prod tags here
    pass  # Terminate as needed
```

---

### Scenario 8:  
**Q:** How do you manage secrets for CI/CD pipelines?  
**A:**  
- Store secrets in a secure vault (AWS Secrets Manager, HashiCorp Vault).
- Inject secrets as environment variables in build/run steps.
- Avoid hard-coding secrets in scripts or code.

---

### Scenario 9:  
**Q:** Jenkins build fails due to plugin dependency issues. Steps to resolve?  
**A:**  
- Check plugin compatibility with Jenkins version.
- Update necessary plugins.
- Restart Jenkins if needed.
- Clear cache and retry the build.

---

## References

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Terraform Documentation](https://www.terraform.io/docs/)
- [AWS Docs](https://docs.aws.amazon.com/)
- [Python Official Site](https://www.python.org/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)

---

Feel free to add, edit, or expand this guide according to role-specific requirements!



# DevOps Interview Q&A Guide for Globant

This file lists commonly asked DevOps interview questions and answers, including real-time scenarios, for Globant and similar tech consultancies. It covers Kubernetes, GitHub Actions, AWS, Python scripting, Terraform, Jenkins, Ansible, monitoring tools, CI/CD, cloud best practices, and troubleshooting.

---

## CI/CD & Core DevOps
### Q1: What is DevOps and why is it important?  
DevOps is a set of practices that unifies software development (Dev) and IT operations (Ops). It emphasizes collaboration, automation, and continuous integration/delivery, resulting in faster, more reliable releases.  
_Scenario:_ How would you integrate DevOps in a team with siloed development/operations?  
_Answer:_ Share repositories (e.g. GitHub), automate builds/tests (Jenkins/GitHub Actions), run regular feedback meetings, automate deployments.  
[[GeeksforGeeks](https://www.geeksforgeeks.org/devops/devops-interview-questions/)] [[Datacamp](https://www.datacamp.com/blog/devops-interview-questions)]

### Q2: What is CI/CD and how do you set it up with Jenkins or GitHub Actions?  
CI/CD means Continuous Integration/Continuous Deployment/Delivery—automated testing and deployment pipelines. Jenkins uses Jenkinsfile declarative pipelines; GitHub Actions uses YAML workflows triggered by repo events.  
_Real-time:_ Troubleshooting step failures—review logs, check credentials, verify artifact paths, check environment variables.

---

## Kubernetes & Containers
### Q3: What is a Kubernetes Pod? How is it different from a Deployment?  
A Pod is the smallest deployable unit—a group of containers sharing storage/network. A Deployment manages ReplicaSets of Pods, enabling scaling, rolling updates, and self-healing.  
_Scenario:_ Rolling out a new app version without downtime—use rolling updates and readiness/liveness probes.  
[[GitHub Q&A](https://github.com/NotHarshhaa/DevOps-Interview-Questions)] [[DEV Ultimate List](https://dev.to/prodevopsguytech/the-ultimate-devops-interview-questions-answers-repository-550-questions-growing-1h9n)]

### Q4: Troubleshoot ‘CrashLoopBackOff’ in a Pod.  
Check `kubectl logs`, review resource limits, inspect startup commands, verify secrets/configs.

---

## Cloud (AWS) & IaC (Terraform)
### Q5: Explain Terraform basics and use-cases. How do you handle state?  
Terraform is an Infrastructure as Code (IaC) tool for cloud provisioning. The state file tracks current infrastructure; use remote backends (Amazon S3), enable state locking, and take regular backups.  
[[GeeksforGeeks](https://www.geeksforgeeks.org/devops/devops-interview-questions/)] [[GitHub Q&A](https://github.com/NotHarshhaa/DevOps-Interview-Questions)]

### Q6: Best practices for AWS security?  
Use IAM roles/policies (least privilege), enable MFA, store secrets in AWS Secrets Manager, audit regularly with CloudTrail.

---

## Jenkins & GitHub Actions
### Q7: How do you handle pipeline failures in Jenkins?  
Check build logs, use pre-checks (e.g., test dependency availability), integrate notifications (Slack/email), parallelize stages to isolate issues.

### Q8: Compare GitHub Actions and Jenkins.  
GitHub Actions: native to GitHub, repo-centric, easy secrets management, YAML workflows. Jenkins: highly flexible/extensible, plugin-rich, requires server upkeep.  
_Real-time:_ Migrate Jenkins pipeline to GitHub Actions—convert shell logic into YAML jobs, use GitHub Secrets, test matrix builds.  
[[DEV Ultimate List](https://dev.to/prodevopsguytech/the-ultimate-devops-interview-questions-answers-repository-550-questions-growing-1h9n)]

---

## Python Scripting & Ansible
### Q9: Example Python script for log rotation.
```python
import os, shutil
log_dir = "/var/log/app"
archive_dir = "/var/log/app/archive"
for filename in os.listdir(log_dir):
    if filename.endswith(".log"):
        shutil.move(os.path.join(log_dir, filename), archive_dir)
```
_Scenario:_ Add error handling/logging for robustness.  
[[GitHub Q&A](https://github.com/NotHarshhaa/DevOps-Interview-Questions)]

### Q10: How do you use Ansible for multi-server configuration?  
Use inventory files for host grouping, roles for abstraction, and handlers for service restarts. Use Ansible Vault for secrets.

---

## Monitoring Tools & Troubleshooting
### Q11: How do you monitor Kubernetes clusters?  
Prometheus collects metrics; Grafana visualizes; Alertmanager manages alerts based on rule violations (memory leaks, spikes).

### Q12: Steps for troubleshooting high latency in cloud apps.  
Collect metrics (CPU/memory/network IO), check logs, trace requests end-to-end, scrutinize resource quotas, use AWS X-Ray/Datadog.

---

## Cloud Best Practices & Scenarios
### Q13: Cloud cost optimization strategies  
Right-size resources, buy reserved/spot instances, automate shutdown for dev/test, alert on over-provisioning.

### Q14: Deployment to Kubernetes fails—image not found.  
Check image registry credentials, verify image tags, inspect build/push logs, test image pull locally.

---

## Scenario-Based/Real-Time Q&A (Globant Examples)
### S1: CI/CD deployment fails due to missing environment variable.
_Answer:_ Check pipeline/environment config, repo secrets, and deployment YAML referencing.

### S2: Terraform state file is corrupted.
_Answer:_ Restore from backup, re-initialize state, discuss use of remote state management and disaster recovery plan.

### S3: Jenkins build performance is slow.
_Answer:_ Check for bottlenecks, repartition jobs, use parallel builds, optimize plugins/installations.

### S4: Autoscaling Kubernetes nodes in AWS.
_Answer:_ Configure Cluster Autoscaler, set node group policies, monitor scaling events.

---

## References and More Examples

- 1100+ DevOps Q&A [GitHub](https://github.com/NotHarshhaa/DevOps-Interview-Questions)
- Ultimate Repository [DEV Community](https://dev.to/prodevopsguytech/the-ultimate-devops-interview-questions-answers-repository-550-questions-growing-1h9n)
- GeeksforGeeks [DevOps Interview Prep](https://www.geeksforgeeks.org/devops/devops-interview-questions/)
- Globant Interview Experiences [AmbitionBox](https://www.ambitionbox.com/interviews/globant-interview-questions)

References also back up all practical and scenario-based content. Prepare to discuss trade-offs, collaboration, and troubleshooting—strongly emphasized in Globant interviews!

---
