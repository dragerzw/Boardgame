**🎲 Boardgame - End-to-End DevSecOps Pipeline**

**📖 Project Overview**

This project demonstrates a production-grade **DevSecOps CI/CD pipeline** for a Java-based "Boardgame" application.

By leveraging **GitHub Actions** on a self-hosted runner, the pipeline automates the entire lifecycle from code commit to Kubernetes deployment. It adheres to a **"Shift-Left"** security philosophy, integrating SAST (Static Application Security Testing) via SonarQube and Container Vulnerability Scanning via Trivy to arrest vulnerabilities _before_ they reach production.

**🏗️ Architecture**

The pipeline implements a "Security-First" workflow. If any quality gate (code coverage, bugs) or security threshold (critical CVEs) is breached, the build fails immediately.

<img width="1125" height="781" alt="image" src="https://github.com/user-attachments/assets/29deb711-2057-4778-b107-7225706ef704" />


**🛠️ Technology Stack**

| **Category** | **Technology** | **Usage** |
| --- | --- | --- |
| **Application** | Java 17 (Temurin) | Core application logic. |
| **CI/CD** | GitHub Actions | Workflow orchestration on self-hosted runners. |
| **Build Tool** | Maven | Dependency management and compilation. |
| **Code Quality** | SonarQube | SAST: Detects bugs, code smells, and security hotspots. |
| **Security** | Trivy | Scans filesystem (dependencies) and Docker images for CVEs. |
| **Containerization** | Docker | Packages the application into a portable image. |
| **Orchestration** | Kubernetes (K8s) | Deployment, scaling, and management of containerized apps. |

**⚙️ Environment Setup**

**1\. Self-Hosted Runner Configuration**

To replicate this pipeline, a Linux server (Ubuntu 20.04/22.04) is required to act as the GitHub Runner.

Pre-requisite Software:

Run the following on the runner to prepare the build environment:

Bash

\# Update and install Docker

sudo apt-get update

sudo apt-get install -y docker.io

sudo chmod 666 /var/run/docker.sock

\# Install Java 17 (Required for Maven build)

sudo apt-get install -y openjdk-17-jdk

\# Install SonarScanner dependencies

sudo apt-get install -y unzip jq

**2\. Kubernetes Cluster**

Ensure a K8s cluster is active (Minikube, EKS, AKS, or Kubeadm).

- **Create Namespace:** kubectl create namespace webapps
- **Permissions:** Ensure the Service Account used by the runner has RoleBinding permissions to deploy to the webapps namespace.

**3\. Repository Secrets**

Navigate to **Settings > Secrets and variables > Actions** and configure:

| **Secret** | **Description** |
| --- | --- |
| DOCKERHUB_USERNAME | Your Docker Hub ID. |
| DOCKERHUB_TOKEN | Docker Hub Access Token (Read/Write). |
| SONAR_TOKEN | Authentication token generated in SonarQube. |
| SONAR_HOST_URL | Public URL of your SonarQube server. |
| KUBE_CONFIG | Base64 encoded kubeconfig file (\`cat ~/.kube/config |

**🛡️ Pipeline Stages**

- **Initialization:** Installs lightweight utilities (unzip, jq) required for parsing security reports.
- **Build:** Compiles the Java application using Maven (mvn clean package).
- **Trivy FS Scan:** Scans source code and pom.xml for vulnerable dependencies before the build artifact is finalized.
- **SonarQube Scan:** Performs deep code analysis. The pipeline **pauses** here until the Quality Gate returns a PASS status.
- **Docker Build & Scan:** Builds the container image and immediately scans the OS layers for vulnerabilities using Trivy.
- **Push:** Uploads the verified, secure image to Docker Hub.
- **Deploy:** Applies the Kubernetes manifests to update the application in the webapps namespace.

**💡 Challenges & Troubleshooting (STAR Method)**

These are real-world issues encountered during the engineering of this pipeline.

**🔴 Issue 1: The "Hanging" Runner (Timeout)**

- **Symptom:** The pipeline would freeze indefinitely during the apt-get install step, eventually timing out after 6 hours.
- **Root Cause:** The runner attempted to install man-db and other packages that triggered interactive CLI popups (e.g., _"Do you want to restart services?"_), waiting for user input that could never arrive.
- **Resolution:** Enforced non-interactive mode in the install command.

Bash

sudo DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends unzip jq

**🔴 Issue 2: Missing Dependencies on Bare Metal**

- **Symptom:** SonarQube action failed with Unable to locate executable file: unzip.
- **Context:** Unlike GitHub-hosted runners which come pre-loaded with tools, bare-metal Ubuntu runners are minimal.
- **Resolution:** Added a dedicated **"Install Utilities"** step at the pipeline initialization to ensure unzip is present before the Sonar action triggers.

**🔴 Issue 3: Docker Context Errors**

- **Symptom:** docker build failed, claiming it could not find the file context.
- **Resolution:** Explicitly defined the build context as the current directory (.) in the build command.

Bash

docker build -t realdrager/boardgame:latest .

**📊 Artifacts & Reports**

For audit and compliance, the pipeline generates:

- 📄 **trivy-fs-report.html**: File system vulnerability report.
- 📄 **trivy-image-report.html**: Container image vulnerability report.

**🚀 How to Run**

- **Clone** this repository.
- **Configure Secrets** in your GitHub repository settings.
- **Push** a change to the main branch to trigger the workflow.
- **Monitor** the "Actions" tab in GitHub for step-by-step execution.
- **Access** the application:

Bash

kubectl get svc -n webapps

\# Open the External-IP or NodePort in your browser
