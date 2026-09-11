# Jenkins CI/CD Pipeline with Rolling Deployment & Automatic Rollback

## 📌 Activity Overview

Today's hands-on activity focused on implementing an **automated CI/CD pipeline using Jenkins** for a Flask web application.

The pipeline automates the complete application delivery process, from source-code checkout to Docker-based deployment, health verification, rolling updates, and automatic rollback in case of deployment failure.

---

## 🎯 Objectives

The main objectives of this activity were:

* Connect a GitHub application repository with Jenkins.
* Create an automated CI/CD pipeline.
* Build and test the application automatically.
* Create a Docker image as part of the pipeline.
* Push the Docker image to Docker Hub.
* Deploy the new application version automatically.
* Implement a rolling deployment strategy.
* Verify application health after deployment.
* Introduce a controlled deployment failure.
* Automatically roll back to the previous working version.
* Document the complete deployment workflow.

---

## 🏗️ Technology Stack

| Technology         | Purpose                        |
| ------------------ | ------------------------------ |
| **GitHub**         | Source code management         |
| **Jenkins**        | CI/CD automation               |
| **Python / Flask** | Web application                |
| **Pytest**         | Automated testing              |
| **Docker**         | Application containerization   |
| **Docker Hub**     | Container image registry       |
| **Nginx**          | Reverse proxy / load balancing |
| **AWS EC2**        | Deployment environment         |
| **Linux**          | Server operating system        |

---

## 📂 Project Repository

**GitHub Repository:**

[https://github.com/lalitpunjabi/jenkins-cicd-demo](https://github.com/lalitpunjabi/jenkins-cicd-demo)

---

## 🔄 CI/CD Pipeline Flow

```text
GitHub
   │
   ▼
Jenkins
   │
   ├── Checkout
   │
   ├── Build
   │
   ├── Test
   │
   ├── Package
   │
   ├── Docker Build
   │
   ├── Docker Hub Push
   │
   ├── Save Previous Version
   │
   ├── Rolling Deployment - App 1
   │
   ├── Health Check
   │
   ├── Rolling Deployment - App 2
   │
   ├── Health Check
   │
   ├── Update Current Version
   │
   └── Final Verification
   │
   ▼
Application Running
```

---

# 🚀 Implementation

## 1. Source Code Management

The Flask application is maintained in GitHub.

Jenkins is configured to retrieve the source code from the repository and execute the pipeline defined in the `Jenkinsfile`.

```text
GitHub Repository
        ↓
     Jenkins
        ↓
    Jenkinsfile
```

---

## 2. Automated Build

Jenkins creates a Python virtual environment and installs the dependencies specified in `requirements.txt`.

```bash
python3 -m venv venv
./venv/bin/pip install --upgrade pip
./venv/bin/pip install -r requirements.txt
```

This ensures that the application dependencies are installed automatically during every pipeline execution.

---

## 3. Automated Testing

The pipeline executes the application's automated tests using Pytest.

```bash
./venv/bin/pytest
```

The deployment process proceeds only when the tests pass successfully.

---

## 4. Docker Image Creation

After successful testing, Jenkins builds a Docker image for the application.

The image is tagged using the Jenkins build number:

```text
lalit0409/jenkins-cicd-demo:<BUILD_NUMBER>
```

The `latest` tag is also maintained.

Example:

```text
lalit0409/jenkins-cicd-demo:9
lalit0409/jenkins-cicd-demo:latest
```

---

## 5. Docker Hub Push

Jenkins authenticates with Docker Hub using Jenkins credentials and pushes the generated Docker image.

```text
Jenkins
   │
   ▼
Docker Build
   │
   ▼
Docker Hub
```

This allows the deployment environment to use the versioned application image.

---

# 🔄 Rolling Deployment

Two application containers are used for deployment:

```text
App 1 → Port 5001
App 2 → Port 5002
```

The containers are:

```text
jenkins-app-1
jenkins-app-2
```

The deployment is performed sequentially.

### Deployment sequence

```text
Current Version

App 1 ──→ New Version
   │
   └── Health Check
          │
          ▼
App 2 ──→ New Version
   │
   └── Health Check
          │
          ▼
Final Verification
```

This approach avoids replacing all application instances simultaneously.

---

# ❤️ Health Verification

After each container is deployed, Jenkins automatically checks the application's `/health` endpoint.

```bash
curl -fsS http://localhost:5001/health
```

and:

```bash
curl -fsS http://localhost:5002/health
```

A successful response confirms that the application container is running correctly.

The application is also verified through Nginx:

```bash
curl -fsS http://localhost/health
```

---

# 🌐 Nginx Reverse Proxy

Nginx is configured as a reverse proxy in front of the two application containers.

```text
                    ┌───────────────┐
                    │     User      │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │     Nginx     │
                    │     :80       │
                    └───────┬───────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
                  ▼                   ▼
          ┌──────────────┐    ┌──────────────┐
          │   App 1      │    │   App 2      │
          │   :5001      │    │   :5002      │
          └──────────────┘    └──────────────┘
```

Nginx distributes incoming application requests between the available application containers.

---

# 🔙 Automatic Rollback

An automated rollback mechanism was implemented in the Jenkins pipeline.

Before deploying the new version, Jenkins identifies the currently deployed Docker image and saves it as the **previous working version**.

```text
Current Running Image
        │
        ▼
   Save as previous
        │
        ▼
Deploy New Version
```

If deployment fails, Jenkins automatically executes the rollback logic.

```text
Deployment Failure
        │
        ▼
Find Previous Image
        │
        ▼
Stop Failed Deployment
        │
        ▼
Restore Previous Image
        │
        ▼
Start App 1 + App 2
        │
        ▼
Health Checks
        │
        ▼
Rollback Completed
```

No manual container restoration is required.

---

# ⚠️ Controlled Deployment Failure

A Jenkins pipeline parameter named:

```text
SIMULATE_FAILURE
```

was added to demonstrate the rollback mechanism.

### Normal deployment

```text
SIMULATE_FAILURE = false
```

The pipeline completes the deployment normally.

### Rollback demonstration

```text
SIMULATE_FAILURE = true
```

The pipeline intentionally introduces a controlled failure after the first deployment stage.

Jenkins then automatically executes the rollback process and restores the previous working Docker image.

---

# 🐳 Docker Deployment Architecture

```text
                         Docker Hub
                             │
                             │ Pull Image
                             ▼
                    ┌─────────────────┐
                    │    AWS EC2      │
                    │ Jenkins Server  │
                    └────────┬────────┘
                             │
                  ┌──────────┴──────────┐
                  │                     │
                  ▼                     ▼
          ┌───────────────┐     ┌───────────────┐
          │ jenkins-app-1 │     │ jenkins-app-2 │
          │    :5001      │     │    :5002      │
          └───────┬───────┘     └───────┬───────┘
                  │                     │
                  └──────────┬──────────┘
                             ▼
                         Nginx :80
                             │
                             ▼
                           Users
```

---

# 🧪 Pipeline Stages

The Jenkins pipeline contains the following stages:

| Stage                          | Description                             |
| ------------------------------ | --------------------------------------- |
| **Checkout**                   | Retrieves application source code       |
| **Build**                      | Installs Python dependencies            |
| **Test**                       | Executes automated tests                |
| **Package**                    | Creates application package             |
| **Docker Build**               | Builds Docker image                     |
| **Docker Login & Push**        | Pushes image to Docker Hub              |
| **Save Previous Version**      | Stores the currently deployed version   |
| **Rolling Deployment - App 1** | Deploys new version to App 1            |
| **Controlled Failure**         | Optionally simulates deployment failure |
| **Rolling Deployment - App 2** | Deploys new version to App 2            |
| **Update Current Version**     | Marks successful image as current       |
| **Final Verification**         | Performs final health checks            |

---

# 📊 Deployment Strategy

The implemented strategy provides:

* Automated application deployment
* Versioned Docker images
* Sequential container updates
* Health checks after deployment
* Previous-version preservation
* Automated rollback
* Controlled failure simulation
* Nginx-based request routing

This reduces the need for manual deployment operations and provides a safer mechanism for releasing new application versions.

---

# 📸 Activity Evidence

The following screenshots are included as evidence for the activity:

### 1. CI/CD Pipeline Screenshot

Shows the Jenkins pipeline stages and successful execution.

### 2. Deployment Screenshot

Shows the running application containers and their deployed ports.

### 3. Docker Image Screenshot

Shows the versioned Docker images created by Jenkins.

### 4. Successful Application Screenshot

Shows the application running successfully through the Nginx endpoint.

### 5. Rollback Screenshot

Shows the controlled deployment failure and automatic rollback process in Jenkins Console Output.

---

# ✅ Final Result

The complete automated workflow was successfully implemented:

```text
GitHub
   ↓
Jenkins
   ↓
Build
   ↓
Test
   ↓
Docker Build
   ↓
Docker Hub
   ↓
Rolling Deployment
   ↓
Health Verification
   ↓
Application Running
   ↓
Controlled Failure
   ↓
Automatic Rollback
   ↓
Previous Working Version Restored
```

The activity demonstrates how Jenkins, Docker, Docker Hub, Nginx, and AWS EC2 can be integrated to create a reliable CI/CD workflow with **rolling deployment and automated rollback capabilities**.

---

## 👨‍💻 Author

**Lalit Punjabi**
B.Tech – Artificial Intelligence & Data Science
Arya College of Engineering & IT

**Activity:** Jenkins CI/CD – Rolling Deployment & Automatic Rollback
**Repository:** `lalitpunjabi/jenkins-cicd-demo`
