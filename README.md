# 🚀 Jerney: Comprehensive DevSecOps & GitOps Platform

[![CI/CD Pipeline](https://github.com/SuchanMadhikarmi/Devsecops_Project_Actions_ArgoCD/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/SuchanMadhikarmi/Devsecops_Project_Actions_ArgoCD/actions)
[![Infrastructure](https://img.shields.io/badge/Infrastructure-Terraform-623CE4?style=flat-square&logo=terraform)](https://www.terraform.io/)
[![Cloud](https://img.shields.io/badge/Cloud-AWS_EKS-232F3E?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/)
[![Security](https://img.shields.io/badge/Security-Trivy_%7C_Checkov-11B4ED?style=flat-square)](https://aquasecurity.github.io/trivy/)
[![CD](https://img.shields.io/badge/CD-ArgoCD-ef7b4d?style=flat-square&logo=argo)](https://argo-cd.readthedocs.io/)

A complete, production-ready DevSecOps and GitOps project demonstrating a modern software delivery lifecycle. **Jerney** is a 3-tier microservices application (React frontend, Node.js backend, and PostgreSQL) built to showcase automated security practices, scalable cloud infrastructure, container orchastration, and automated deployments.

---

## 🏗️ Architecture Design

### 1. The Application Stack
The application itself follows a standard 3-tier web architecture natively built for high availability and containerization.
- **Frontend**: React.js structured with Vite and deployed alongside an NGINX reverse-proxy.
- **Backend**: Node.js & Express API, interacting securely with the database.
- **Database**: PostgreSQL 16 with Persistent EBS Volumes for data durability.

### 2. Infrastructure as Code (IaC)
Infrastructure is fully codified using **HashiCorp Terraform**, defining:
- AWS VPC structures including multi-AZ subnets, NAT Gateways, and security definitions.
- **Amazon EKS (Elastic Kubernetes Service) Auto Mode**, dynamically handling node provisioning without manual NodeGroup tracking.
- Envelope encryption of Kubernetes secrets and full cluster auditing enabled by default.

### 3. The DevSecOps Pipeline
A robust **GitHub Actions** pipeline has been engineered to bake security into the CI process natively *("Shift-Left")*.
1. 🔍 **Code Linting:** Evaluates code quality using ESLint.
2. 🛡️ **Software Composition Analysis (SCA):** Prevents vulnerable packages using dependency audits (`npm audit`).
3. 🐳 **Image Build & Containerization:** Builds Docker images natively resolving caching, labels, and pushing securely to GitHub Container Registry (GHCR) using temporary permissions.
4. 🔬 **Security Container Image Scan:** Employs **Trivy** to intercept and scan built Docker images for OS and library vulnerabilities stopping High/Critical CVEs before registry promotion.
5. 🏗️ **IaC Security Scan:** Scans Kubernetes manifests and Terraform definitions statically using **Checkov** to prevent misconfigurations (e.g., ensuring security contexts align, root access points are dropped, etc.).
6. 📋 **Dockerfile Linting:** Enforces best practices using **Hadolint**.
7. 🚀 **Manifest Artifact Update:** Automatically patches the Kubernetes base manifest with the new container image SHA tag—this acts as the trigger for the CD process.

### 4. GitOps & Continuous Deployment (ArgoCD)
To guarantee high resilience and native declarative deployments, I evolved the delivery phase by integrating **ArgoCD**.
Rather than utilizing push-based delivery in GitHub Actions, **ArgoCD** runs *inside* the EKS cluster. It acts as an active reconnaissance agent, continually monitoring this repository's `k8s/` directory. The moment the CI pipeline registers a new image tag to the `jerney.yaml` file, ArgoCD detects the configuration drift and synchronizes (pulls) the changes automatically into the cluster, creating a true **GitOps Continuous Deployment pattern.**

---

## 🔒 Cloud Native Network Security
To establish zero-trust boundary limits, native Kubernetes `NetworkPolicy` artifacts restrict transverse communications:
- The backend API is only reachable from the React frontend.
- The PostgreSQL database is completely walled-off, only allowing ingress calls natively coming from the configured Backend Pod.
- Least-privileged container SecurityContexts (`allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true`, dropping all capabilities, utilizing non-root users).

---

## 🧠 What I Learned & Key Takeaways

Completing this architectural endeavor allowed me to develop mastery over several vital DevSecOps domains:

- **EKS Auto Mode Configuration:** Navigated setting up AWS EKS directly with Terraform to dynamically provide compute using Auto Mode. Reduced operations overhead significantly compared directly with managing legacy NodeGroups.
- **Implementing "Shift-Left" Security:** Fully recognized the value of embedding security checkpoints (`Trivy`, `Checkov`, and native Audits) early in the CI phase instead of reacting to infrastructure or image vulnerabilities post-deployment.
- **GitOps Methodology Deployment:** Understood firsthand how traditional push-based deployments possess inherent security and drift issues compared to a pull-based logic orchestrated by **ArgoCD**. Treating Git as the ultimate Source-of-Truth dramatically streamlined rollbacks and cluster state visualization.
- **Network Isolation:** Learned deeply about the differences between AWS Security Groups and Kubernetes `NetworkPolicy`. Hardening inter-pod communications mitigated internal network exploitation potentials manually mapping allowed traffic pathways.
- **Persistence Handling in K8s:** Interfaced successfully with CSI drivers, establishing proper statefulness on `gp3` encrypted EBS chunks. 

---

## 🛠️ Technology Stack

| Domain | Tools Used |
|--------|------------|
| **Development** | React.js, Vite, Node.js, Express, PostgreSQL |
| **Containerization** | Docker, NGINX |
| **Orchestration** | Kubernetes, Amazon EKS |
| **Infrastructure** | Terraform, AWS |
| **CI/CD** | GitHub Actions, GitOps |
| **Continuous Deployment**| ArgoCD |
| **Security (SecOps)** | Checkov (IaC), Trivy (Image Scan), Hadolint, ESLint |

---

## 🧑‍💻 How to Run Locally

### Prerequisites
- Docker & Docker Compose
- AWS CLI configured with credentials (for Terraform deployment)
- `kubectl` and `terraform` installed 

### 1. Running the Complete App Locally 
```bash
docker-compose up --build
```
The React application will be served at `http://localhost`, while the API is bound to `http://localhost:5000`.

### 2. Deploying the Cloud Infrastructure
```bash
cd terraform
terraform init
terraform apply -var="cluster_name=jerney-eks" -var="vpc_cidr=10.0.0.0/16"
```

### 3. Securing to the Cluster & Installing ArgoCD
```bash
aws eks update-kubeconfig --region <aws-region> --name jerney-eks
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Once installed, structure an ArgoCD application pointing to this repository's `k8s/` directory and it will automatically reconcile the state of your cluster.
