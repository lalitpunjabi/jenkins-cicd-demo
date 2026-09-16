# DevOps Interview Preparation

## 1. Project Introduction

### Question: Tell me about your Jenkins CI/CD project.

I developed a Jenkins-based CI/CD pipeline for a containerized application.

The pipeline automates:

- GitHub code checkout
- Application build
- Testing
- Security scanning
- Docker image creation
- Docker Hub push
- Application deployment
- Health checks
- Automatic rollback

---

## 2. Explain Your CI/CD Pipeline

The pipeline follows this workflow:

```text
GitHub
   ↓
Jenkins
   ↓
Build
   ↓
Test
   ↓
Security Scan
   ↓
Docker Build
   ↓
Docker Image Scan
   ↓
Docker Hub
   ↓
Deployment
   ↓
Health Check
   ↓
Rollback if Failure
````

---

## 3. What Is CI/CD?

**Continuous Integration (CI)** automatically builds and tests code whenever changes are integrated.

**Continuous Delivery/Deployment (CD)** automates the process of delivering or deploying the tested application.

---

## 4. What Is DevSecOps?

DevSecOps integrates security into the DevOps lifecycle.

Instead of checking security only at the end, security checks are performed throughout the pipeline.

```text
Code
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Deploy
 ↓
Monitor
```

---

## 5. Why Jenkins?

Jenkins is an automation server used to implement CI/CD pipelines.

Important features include:

* Pipeline as Code
* Git integration
* Automated builds
* Automated testing
* Credential management
* Docker integration
* Plugin ecosystem

---

## 6. What Is a Jenkinsfile?

A Jenkinsfile defines the Jenkins pipeline as code.

It allows the pipeline configuration to be:

* Version controlled
* Reviewed
* Reused
* Audited

Example:

```text
Jenkinsfile
     ↓
Jenkins Pipeline
     ↓
Automated CI/CD
```

---

## 7. Why Use Gitleaks?

Gitleaks detects accidentally committed secrets such as:

* API keys
* Passwords
* Access tokens
* Private keys

Example:

```bash
gitleaks detect --no-banner --redact
```

---

## 8. Why Use pip-audit?

`pip-audit` checks Python dependencies for known security vulnerabilities.

Example:

```bash
pip-audit -r requirements.txt
```

This helps identify vulnerable dependencies before deployment.

---

## 9. Why Use Trivy?

Trivy is used for vulnerability and configuration scanning.

In the project it can scan:

```text
Dockerfile
Kubernetes manifests
Docker images
Project filesystem
```

Examples:

```bash
trivy config Dockerfile
```

```bash
trivy config k8s/
```

```bash
trivy image IMAGE_NAME
```

---

## 10. Why Scan the Docker Image?

Scanning only the Dockerfile is not enough because the final image contains:

* Application dependencies
* OS packages
* System libraries
* Runtime components

Therefore, the final Docker image is scanned before deployment.

---

## 11. Why Use Build Numbers as Docker Tags?

The pipeline uses Jenkins build numbers for image versioning.

Example:

```text
jenkins-cicd-demo:25
```

instead of depending only on:

```text
latest
```

This provides:

* Version tracking
* Traceability
* Easier rollback
* Reproducible deployments

---

## 12. What Is Docker?

Docker is a containerization platform that packages an application with its dependencies into a portable container image.

```text
Application
     +
Dependencies
     ↓
Docker Image
     ↓
Container
```

---

## 13. Docker Image vs Container

**Docker Image:** Template used to create containers.

**Container:** Running instance of a Docker image.

```text
Image
  ↓
Container
```

---

## 14. What Is Kubernetes?

Kubernetes is a container orchestration platform.

It provides:

* Container scheduling
* Scaling
* Service discovery
* Self-healing
* Rolling updates
* Configuration management

---

## 15. What Is a Kubernetes Pod?

A Pod is the smallest deployable unit in Kubernetes.

A simple Pod can contain one container:

```text
Pod
 └── Container
```

---

## 16. What Is a Kubernetes Deployment?

A Deployment manages the desired number of application Pods.

It supports:

* Replicas
* Rolling updates
* Rollbacks
* Desired-state management

---

## 17. What Is a Kubernetes Service?

A Service provides stable network access to Pods.

```text
Client
  ↓
Service
  ↓
Pods
```

It allows clients to communicate with dynamically changing Pods.

---

## 18. What Is RBAC?

RBAC stands for **Role-Based Access Control**.

It controls:

```text
Who
 ↓
Can perform what action
 ↓
On which resource
```

It helps implement the principle of least privilege.

---

## 19. What Are Kubernetes Secrets?

Kubernetes Secrets are intended for sensitive configuration such as:

* Passwords
* Tokens
* Certificates
* Credentials

Access should be restricted using appropriate RBAC and security controls.

---

## 20. What Are Taints and Tolerations?

A **taint** restricts which Pods can run on a node.

A **toleration** allows a Pod to be scheduled onto a node with a matching taint.

```text
Node Taint
    ↓
Pod needs matching Toleration
    ↓
Pod can be scheduled
```

---

## 21. What Is Node Selector?

Node Selector allows a Pod to run on nodes with a specific label.

Example:

```yaml
nodeSelector:
  disk: ssd
```

The node must have:

```text
disk=ssd
```

---

## 22. Explain Your Rollback Mechanism

Before deploying a new image, the current image is preserved as:

```text
previous
```

Example:

```text
Current → version 24
New     → version 25
Backup  → previous
```

If the new deployment fails, the previous image can be restored.

---

## 23. How Does the Pipeline Detect Failure?

The application provides a `/health` endpoint.

The pipeline checks the endpoint after deployment:

```bash
curl http://localhost:5001/health
```

If the health check fails, the deployment is considered unsuccessful and rollback can be triggered.

---

## 24. What Is a Health Check?

A health check verifies whether an application is running and responding correctly.

Example:

```text
GET /health
      ↓
200 OK
      ↓
Application Healthy
```

---

## 25. What Is Shift-Left Security?

Shift-left security means performing security checks earlier in the development lifecycle.

```text
Code
 ↓
Security
 ↓
Build
 ↓
Test
 ↓
Deploy
```

This helps identify security issues before deployment.

---

## 26. What Is SAST?

SAST means **Static Application Security Testing**.

It analyzes source code without running the application.

Examples:

* SonarQube
* Semgrep
* CodeQL

---

## 27. What Is DAST?

DAST means **Dynamic Application Security Testing**.

It tests a running application for security issues.

A common tool is:

```text
OWASP ZAP
```

---

## 28. SAST vs DAST

| SAST                        | DAST                                      |
| --------------------------- | ----------------------------------------- |
| Analyzes source code        | Tests running application                 |
| Static analysis             | Dynamic testing                           |
| Usually earlier in pipeline | Usually against deployed/test application |

---

## 29. What Is an SBOM?

SBOM stands for **Software Bill of Materials**.

It provides a list of software components and dependencies used by an application or container image.

```text
Application
 ├── Python
 ├── Flask
 ├── OS Packages
 └── Other Dependencies
```

---

## 30. What Is Terraform?

Terraform is an Infrastructure as Code tool used to define and manage infrastructure through configuration files.

Common commands:

```bash
terraform init
terraform plan
terraform apply
```

---

## 31. What Is Ansible?

Ansible is an automation and configuration-management tool.

It can automate:

* Package installation
* User creation
* Configuration
* Service management
* Application deployment

Ansible commonly uses YAML playbooks.

---

## 32. Common Docker Commands

List containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Build an image:

```bash
docker build -t IMAGE_NAME .
```

View logs:

```bash
docker logs CONTAINER_NAME
```

Stop a container:

```bash
docker stop CONTAINER_NAME
```

---

## 33. Common Kubernetes Commands

Check nodes:

```bash
kubectl get nodes
```

Check Pods:

```bash
kubectl get pods
```

View Pod details:

```bash
kubectl describe pod POD_NAME
```

View logs:

```bash
kubectl logs POD_NAME
```

Apply a manifest:

```bash
kubectl apply -f manifest.yaml
```

---

## 34. Troubleshooting: Pod Pending

If a Pod is stuck in `Pending`:

```bash
kubectl describe pod POD_NAME
```

Check the Events section.

Possible causes:

* Insufficient resources
* Node selector mismatch
* Taints
* Missing tolerations
* Persistent volume issues
* Scheduling constraints

---

## 35. Troubleshooting: CrashLoopBackOff

If a Pod is in `CrashLoopBackOff`:

```bash
kubectl logs POD_NAME
```

Then:

```bash
kubectl describe pod POD_NAME
```

Check for:

* Application errors
* Configuration problems
* Database connectivity
* Incorrect commands
* Permission issues
* Failed health probes

---

## 36. Troubleshooting: Docker Application Not Accessible

Check the container:

```bash
docker ps
```

Check logs:

```bash
docker logs CONTAINER_NAME
```

Check port mapping:

```bash
docker port CONTAINER_NAME
```

Test the application:

```bash
curl http://localhost:5001/health
```

Then check:

* Application port
* Host port
* Firewall
* Nginx configuration
* Container networking

---

## 37. What Happens if a Security Scan Fails?

If a configured security gate reports a blocking finding:

```text
Security Scan
     ↓
HIGH/CRITICAL Finding
     ↓
Pipeline Failure
     ↓
Deployment Blocked
```

This prevents the pipeline from promoting the scanned artifact when it violates the configured security policy.

---

## 38. Why Should Secrets Not Be Stored in Git?

Git maintains commit history.

Even if a secret is deleted from the latest version, it may still exist in previous commits.

Therefore:

```text
Never commit secrets.
```

Use tools such as:

* Jenkins Credentials
* AWS Secrets Manager
* HashiCorp Vault
* Kubernetes Secrets with proper controls

---

## 39. Common DevOps Troubleshooting Approach

When something fails:

```text
Check Error
    ↓
Check Logs
    ↓
Check Configuration
    ↓
Check Network
    ↓
Check Resources
    ↓
Identify Root Cause
    ↓
Fix
    ↓
Verify
```

Avoid making random changes before understanding the error.

---

## 40. Future Improvements

Possible improvements to the project include:

* SAST with SonarQube or Semgrep
* DAST with OWASP ZAP
* SBOM generation
* Container image signing
* Kubernetes policy enforcement
* Prometheus monitoring
* Grafana dashboards
* GitOps with Argo CD

---

## 41. Final Project Answer

> I developed a Jenkins-based CI/CD pipeline for a containerized application. It automates source-code checkout, build, testing, DevSecOps security scans, Docker image creation, registry push, deployment and health verification. I integrated Gitleaks for secret detection, pip-audit for dependency vulnerabilities and Trivy for Docker, filesystem and Kubernetes security scanning. I also implemented versioned Docker images and automated rollback to improve deployment reliability.

---

## 42. Interview Preparation Checklist

```text
[ ] Explain CI/CD
[ ] Explain Jenkins
[ ] Explain Jenkinsfile
[ ] Explain Docker
[ ] Explain Docker Image vs Container
[ ] Explain Kubernetes
[ ] Explain Pods
[ ] Explain Deployments
[ ] Explain Services
[ ] Explain RBAC
[ ] Explain Taints and Tolerations
[ ] Explain Node Selector
[ ] Explain Terraform
[ ] Explain Ansible
[ ] Explain DevSecOps
[ ] Explain Gitleaks
[ ] Explain Trivy
[ ] Explain pip-audit
[ ] Explain SAST and DAST
[ ] Explain SBOM
[ ] Explain Rollback
[ ] Explain Health Checks
[ ] Explain Project Architecture
[ ] Explain Troubleshooting
```

---

## 43. Key Interview Principle

For every technology in the project, be prepared to explain:

```text
What is it?
     ↓
Why did you use it?
     ↓
How did you implement it?
     ↓
What problem did it solve?
     ↓
What happens when it fails?