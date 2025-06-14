# 🔒 DevSecOps CI/CD Pipeline on Jenkins with AWS 🚀

This repository showcases a comprehensive, production-like DevSecOps CI/CD pipeline built from scratch using **Jenkins** (Master-Slave architecture) on **AWS EC2**. The pipeline automates the entire software delivery lifecycle, from source code management to containerization, security scanning, and artifact management, embodying core DevSecOps principles.

---

## 🌟 Project Overview

This project demonstrates how to integrate various security tools into a Continuous Integration/Continuous Delivery (CI/CD) pipeline, ensuring that security is a "shift-left" concern – addressed early and continuously throughout the development process.

### Key Features:

* **Jenkins Master-Slave Architecture:** Scalable and robust CI/CD orchestration.
* **DevSecOps Integration:** Embedding security scans directly into the pipeline stages.
* **Containerization:** Building and managing Docker images.
* **Cloud-Native Deployment:** Utilizing AWS services for registry and potential deployment.
* **Centralized Secret Management:** Storing all sensitive credentials securely in Jenkins.
* **Artifact Management:** Storing build outputs and security reports for auditing.

---

## 🛠️ Technologies & Tools Used

* **CI/CD Orchestrator:** Jenkins (v2.x)
* **Cloud Provider:** AWS (EC2, ECR, S3, ECS)
* **Source Code Management:** GitHub
* **Containerization:** Docker
* **DevSecOps Tools:**
    * **Gitleaks:** For detecting hardcoded secrets in code.
    * **OWASP Dependency-Check:** For identifying known vulnerabilities in project dependencies.
    * **SonarQube:** For static code analysis, code quality, and security hotspots.
    * **Trivy:** For comprehensive vulnerability scanning of Docker images.
* **Artifact Storage:** AWS S3
* **Container Registry:** AWS Elastic Container Registry (ECR)
* **Programming Language:** Python (Flask) 

---

## ⚙️ Architecture Diagram
```
+----------------+       +-------------------+       +--------------------+
|                |       |                   |       |                    |
|  GitHub Repo   +-------> Jenkins Master    +-------> Jenkins Slave      |
| (Source Code,  |       | (on AWS EC2)      |       | (on AWS EC2)       |
|  Jenkinsfile,  |       |                   |       | (Docker, Tools)    |
|  Dockerfile)   |       |                   |       |                    |
+----------------+       +---------+---------+       +---------+----------+
|                         |                           |
| Push/Webhook            |                           | Executes Stages
v                         v                           v
+--------------------------------------------------------------------------+
|                        DevSecOps CI/CD Pipeline |
|                                                                          |
|  1. Code Checkout (from GitHub)                                          |
|  2. Gitleaks Scan (Secret Detection)                                     |
|  3. OWASP Dependency-Check (Dependency Vulnerabilities)                  |
|  4. SonarQube Analysis (Code Quality & Security)                         |
|  5. Build Docker Image                                                   |
|  6. Trivy Image Scan (Container Vulnerabilities)                         |
|  7. Push to AWS ECR                                                      |
|  8. Artifact Archiving (to AWS S3 - Reports, Binaries)                   |
+--------------------------------------------------------------------------+
|                                ^
| Reports/Artifacts              | Container Images
v                                |
+------------------+                   +------------------+
|                  |                   |                  |
|   AWS S3 Bucket  |                   |    AWS ECR       |
| (Artifact Storage)|                   | (Image Registry) |
+------------------+                   +------------------+
```
---

## 🚀 Pipeline Stages & DevSecOps Tools Integration

Each stage in the `Jenkinsfile` integrates a specific DevSecOps tool or action:

1.  **`Checkout Code`**: Clones the application source code from GitHub onto the Jenkins Slave. Uses Jenkins' `github-pat` credential.
2.  **`Gitleaks Scan`**: Scans the codebase for hardcoded secrets (e.g., API keys, passwords) using Gitleaks. Reports are archived.
3.  **`OWASP Dependency Check`**: Analyzes project dependencies for known vulnerabilities using the OWASP Dependency-Check tool. An HTML report is generated and archived.
4.  **`SonarQube Analysis`**: Performs static code analysis to identify code quality issues, bugs, and security hotspots. Integrates with a separate SonarQube server.
5.  **`Build Docker Image`**: Builds a Docker image of the application using the `Dockerfile` present in the repository.
6.  **`Trivy Image Scan`**: Scans the newly built Docker image for OS package vulnerabilities, language-specific dependencies, and misconfigurations using Trivy. A JSON report is generated and archived.
7.  **`Push to ECR`**: Authenticates with AWS ECR and pushes the built Docker image to the designated private repository.
8.  **`Artifact Archiving to S3`**: Uploads generated build artifacts (e.g., application binaries) and security scan reports (Gitleaks, OWASP, Trivy) to a versioned folder in an AWS S3 bucket for long-term storage and auditing.

---

## 🔑 Secret Management

All sensitive information, such as AWS Access Keys, GitHub Personal Access Tokens, and SonarQube API Tokens, are securely stored and managed within **Jenkins Credentials**. They are injected into the pipeline steps using Jenkins' `withCredentials` block, preventing hardcoding of secrets in the `Jenkinsfile`.

---

## 📋 Setup & Installation Guide

This project requires basic familiarity with AWS, Jenkins, and Docker.

### Prerequisites:

* An active **AWS Account** (Free Tier eligible).
* Basic understanding of AWS EC2, S3, ECR, IAM.
* Basic understanding of Jenkins.
* GitHub Account.

### Step-by-Step Setup:

1.  **AWS IAM User Setup:**
    * Create an IAM user with programmatic access (`Access Key ID` & `Secret Access Key`).
    * Attach necessary policies: `AmazonEC2FullAccess`, `AmazonS3FullAccess`, `AmazonEC2ContainerRegistryFullAccess`.
    * **Securely save the Access Key ID and Secret Access Key.**
2.  **AWS EC2 Instances Launch:**
    * Launch two `t2.micro` (Free Tier eligible) Ubuntu Server 22.04 LTS instances: `Jenkins-Master` and `Jenkins-Slave`.
    * Create a **Key Pair** (e.g., `cicd_keypair.pem`) and set `chmod 400 cicd_keypair.pem`.
    * Configure a **Security Group** (e.g., `jenkins-sg`) allowing SSH (port 22) from your IP, Jenkins UI (port 8080) from `0.0.0.0/0`, and SonarQube UI (port 9000) from `0.0.0.0/0`. Ensure instances in this SG can SSH to each other.
3.  **Jenkins Master Setup (on `Jenkins-Master` EC2):**
    * SSH into `Jenkins-Master`.
    * Install Java 11, Jenkins, and start the Jenkins service.
    * Access Jenkins UI via `http://<Jenkins-Master-Public-IP>:8080`, unlock, install suggested plugins, and create an admin user.
4.  **Jenkins Slave Setup (on `Jenkins-Slave` EC2):**
    * SSH into `Jenkins-Slave`.
    * Install Java 11, Git, Docker, Gitleaks, OWASP Dependency-Check, Trivy, and SonarQube Scanner CLI (if needed for your project type).
    * Add `ubuntu` user to the `docker` group: `sudo usermod -aG docker ubuntu`.
5.  **SonarQube Server Setup (on a separate EC2 Instance or Docker):**
    * **(Recommended)** Launch a `t2.small` or `t2.medium` EC2 instance (`SonarQube-Server`) with Docker installed.
    * Run SonarQube via Docker: `docker run -d --name sonarqube -p 9000:9000 -p 9001:9001 sonarqube:lts-community`.
    * Access SonarQube UI via `http://<SonarQube-Server-Public-IP>:9000`, log in (admin/admin), change password, and **generate a user token** (e.g., `jenkins-pipeline-token`) from `My Account -> Security`.
6.  **AWS ECR Repository Creation:**
    * In AWS Console, navigate to ECR.
    * Create a new private repository named `devsecops-pipeline-app`. Enable "Scan on push".
7.  **AWS S3 Bucket Creation:**
    * In AWS Console, navigate to S3.
    * Create a new S3 bucket for artifacts (e.g., `your-devsecops-artifact-bucket`).
8.  **Jenkins Configuration:**
    * **Add Jenkins Slave Node:** In Jenkins UI (`Manage Jenkins > Nodes`), add a new "Permanent Agent" node (`Jenkins-Slave-01`).
        * **Host:** `Jenkins-Slave`'s **Private IP**.
        * **Credentials:** Add `SSH Username with private key` (`ubuntu` user, paste `cicd_keypair.pem` content).
        * **Labels:** `linux docker`.
        * **Remote root directory:** `/home/ubuntu/jenkins-slave`.
    * **Add Credentials:** In Jenkins UI (`Manage Jenkins > Credentials > System > Global credentials`), add:
        * `AWS Credentials`: `ID: aws-credentials`, Access Key ID, Secret Access Key.
        * `Secret text`: `ID: github-pat`, your GitHub Personal Access Token (PAT).
        * `Secret text`: `ID: sonarqube-token`, the SonarQube token generated previously.
    * **Configure SonarQube Server:** In Jenkins UI (`Manage Jenkins > Configure System`), go to "SonarQube servers" section.
        * `Name: SonarQubeServer`, `Server URL: http://<SonarQube-Server-Public-IP>:9000`, `Authentication Token: sonarqube-token`.
9.  **GitHub Repository Setup:**
    * Create a new GitHub repository (e.g., `devsecops-pipeline-app`).
    * Add your sample application code, a `Dockerfile`, and the `Jenkinsfile` to the root of this repository.
    * **Update the `Jenkinsfile`:** Replace `YOUR_AWS_ACCOUNT_ID`, `<Your-GitHub-Username>`, and `your-devsecops-artifact-bucket` with your actual values.
10. **Jenkins Pipeline Job Creation:**
    * In Jenkins UI, create a `New Item` > `Pipeline` job (e.g., `DevSecOps-CI-CD-Pipeline`).
    * Configure it to "Pipeline script from SCM" (Git), pointing to your GitHub repository with `github-pat` credentials.
    * Enable "GitHub hook trigger for GITScm polling".
11. **GitHub Webhook Configuration:**
    * In your GitHub repository settings (`Settings > Webhooks`), add a new webhook.
    * **Payload URL:** `http://<Jenkins-Master-Public-IP>:8080/github-webhook/`.
    * Content type: `application/json`.
    * Select "Just the push event".
12. **Run the Pipeline:**
    * Push a change to your GitHub repository, or manually click "Build Now" in Jenkins. Observe the pipeline execution!

---
---

## ⚠️ Important Considerations & Best Practices

* **Security Groups:** For production, restrict `Anywhere (0.0.0.0/0)` access to specific IPs or VPCs.
* **Free Tier Limits:** Monitor your AWS usage to avoid unexpected charges. `t2.micro` instances can be resource-constrained for heavy loads.
* **SonarQube Persistence:** For a production SonarQube, use an external database (like AWS RDS PostgreSQL) instead of the embedded H2 database, which is not persistent.
* **ECS Deployment:** The ECS deployment stage is conceptual. A real-world scenario would involve more robust deployment strategies (e.g., blue/green, canary) using AWS CodeDeploy, Terraform, or CloudFormation.
* **Error Handling:** In a real pipeline, security scan failures should break the build. For this demo, `|| true` might be used to allow pipeline progression.

---

## 🗑️ Cleanup / Teardown

To avoid incurring future AWS charges, ensure you delete all resources after you are done experimenting:

1.  **Terminate EC2 Instances:** Jenkins-Master, Jenkins-Slave, SonarQube-Server (if separate).
2.  **Delete S3 Bucket:** `your-devsecops-artifact-bucket`.
3.  **Delete ECR Repository:** `devsecops-pipeline-app`.
4.  **Delete IAM User:** `devsecops-jenkins-user`.
5.  **Delete Security Groups:** `jenkins-sg` and any other custom SGs created.
6.  **(Optional) Delete Key Pair:** `cicd_keypair`.

### Resume

- Built a production-grade DevSecOps CI/CD pipeline using Jenkins Master-Slave on AWS EC2, integrating Gitleaks, OWASP Dependency-Check, SonarQube, and Trivy to automate vulnerability scanning across code, dependencies, and container images.

- Containerized a Python application with Docker and automated secure image delivery to AWS ECR, archiving build artifacts and scan reports to AWS S3, achieving 100% automated delivery with enhanced security and auditability.

- Configured GitHub webhook triggers with Jenkins, achieving end-to-end CI/CD automation, and implemented centralized secret management via Jenkins credentials, ensuring secure handling of AWS, GitHub, and SonarQube tokens.

---

Feel free to connect with me if you have any questions or feedback!

Happy DevSecOps!

**Amay Jaiswal**
[https://www.linkedin.com/in/heyamay/ (Linkedin - Amay Jaiswal)]




