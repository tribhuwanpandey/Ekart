# Ekart CI/CD Pipeline on AWS

This project demonstrates a **complete DevOps pipeline** for a Java Spring Boot application (**Ekart**) deployed on **AWS EKS** using **Terraform**, **Jenkins**, **SonarQube**, **Nexus**, and **Docker Hub**.  
It is organized into **three separate GitHub repositories**, each responsible for a different stage of the DevOps lifecycle.

---

##  Repository Overview

| Repository | Description |
|-------------|--------------|
| Repository | Description |
|-------------|--------------|
| [Terraform Infrastructure](#-1-terraform-infrastructure-repo) | Provisions Jenkins, SonarQube & Nexus servers using Terraform |
| [EKS Cluster Setup](#-2-eks-cluster-setup-repo) | Creates AWS EKS cluster using Jenkins and Terraform |
| [Ekart Application Deployment](#-3-ekart-application-repo) | Builds, scans, and deploys the Ekart Java app to EKS |

---

## 1. [Terraform Infrastructure Repo](https://github.com/tribhuwanpandey/Jenkins-sonar-nexus)


###  Repository: `Jenkins-sonar-nexus`

This repository contains Terraform scripts to provision **three EC2 instances** on AWS for the foundational DevOps tools.

###  Components Provisioned
- **Jenkins Server** → Automates CI/CD pipeline execution  
- **SonarQube Server** → Performs static code analysis  
- **Nexus Repository** → Hosts Maven build artifacts  

###  Features
- Fully automated EC2 provisioning using Terraform
- Configures VPC, subnets, and security groups
- Outputs public IPs for Jenkins, SonarQube, and Nexus

###  Commands
```bash
terraform init
terraform plan
terraform apply -auto-approve
```

###  Outputs
After successful provisioning, you will get:

![alt text](<Screenshot 2025-10-29 141445.png>)


###  Access URLs
- Jenkins → `http://<jenkins-ip>:8080`
- SonarQube → `http://<sonarqube-ip>:9000`
- Nexus → `http://<nexus-ip>:8081`

###  Next Step
After servers are created, proceed to the **EKS Cluster Setup** repository to create your Kubernetes environment.

---

## 2. [EKS Cluster Setup Repository](https://github.com/tribhuwanpandey/Eks-cluster-deployment)


###  Repository: `Eks-cluster-deployment`

This repository automates the setup of an **AWS Elastic Kubernetes Service (EKS)** cluster using Jenkins.

###  Tools Used
- **Terraform / eksctl**
- **AWS CLI**
- **kubectl**
- **Jenkins (Pipeline Trigger)**

###  Workflow
1. Jenkins triggers a job to execute Terraform or eksctl scripts.
2. Creates an EKS cluster named `project-cluster` in AWS.
3. Updates kubeconfig for kubectl access:
   ```bash
   aws eks update-kubeconfig --region ap-south-1 --name project-cluster
   ```
4. Verifies cluster readiness:
   ```bash
   kubectl get nodes
   ```

###  Outputs
- EKS Cluster created successfully in AWS

![alt text](<Screenshot 2025-10-29 154544.png>)

- Kubeconfig updated on Jenkins server

###  Next Step
Once the EKS cluster is up, move to the **Ekart Application Deployment** repository to deploy the app.

---

## 3. [Ekart Application Repo](https://github.com/tribhuwanpandey/Ekart)

###  Repository: `Ekart`

This repository contains the **Java Spring Boot e-commerce application (Ekart)** and its **Jenkins CI/CD pipeline** for automated build, testing, analysis, and deployment to EKS.

###  Tools & Technologies
| Category | Tool/Service |
|-----------|--------------|
| Language | Java 17, Spring Boot |
| Build Tool | Maven |
| CI/CD | Jenkins |
| Code Quality | SonarQube |
| Artifact Repository | Nexus |
| Containerization | Docker |
| Image Registry | Docker Hub |
| Orchestration | AWS EKS (Kubernetes) |
| Infrastructure | Terraform |

---

###  Pipeline Overview

| Stage | Description |
|--------|-------------|
| **Git Checkout** | Fetch code from GitHub |
| **Compile & Test** | Build and run unit tests with Maven |
| **SonarQube Analysis** | Perform code quality scan |
| **OWASP Dependency Check** | Check for known vulnerabilities |
| **Package & Deploy to Nexus** | Push JAR artifact to Nexus repository |
| **Docker Build & Push** | Create image and push to Docker Hub |
| **EKS Configuration** | Update kubeconfig for AWS EKS |
| **Kubernetes Deployment** | Apply deployment manifests to EKS |

---

###  Repository Structure

```
Ekart/
├── docker/
│   └── Dockerfile
├── src/
│   └── main/java/... (Spring Boot source code)
├── deploymentservice.yml
├── pom.xml
└── README.md
```

---

###  Jenkins Credentials

| ID | Type | Description |
|----|------|--------------|
| `nvd-api-key` | Secret Text | API key for OWASP NVD |
| `tp109` | Secret Text | Docker Hub password |
| `sonar-token` | Secret Text | SonarQube authentication token |

---

###  Deployment to EKS

Once Jenkins completes the pipeline successfully:

- Docker image pushed to Docker Hub (`tp109/ekart:latest`)

![alt text](<Screenshot 2025-10-29 175805.png>)

- Application deployed on AWS EKS


Verify deployment:
```bash
kubectl get pods
kubectl get svc
```

Access the Ekart app via LoadBalancer URL.

![alt text](<Screenshot 2025-10-29 174650.png>)

#### Login

![alt text](<Screenshot 2025-10-29 174719-2.png>)

#### Shop

![alt text](<Screenshot 2025-10-29 174531.png>)

#### Snapshort

![alt text](<Screenshot 2025-10-29 175316.png>)

#### Sonar

![alt text](<Screenshot 2025-10-29 175209.png>)

---

##  Overall Flow

```text
Terraform (Infra Setup)
      ↓
Jenkins + SonarQube + Nexus
      ↓
Jenkins triggers EKS Cluster creation
      ↓
Jenkins builds & deploys Ekart app
      ↓
AWS EKS Cluster runs the live application
```

---

##  Cleanup

To destroy all infrastructure and resources:

```bash
terraform destroy -auto-approve
```

