# Jenkins CI/CD Demo

A practical CI/CD project demonstrating the integration of **GitHub, Jenkins, Python Flask, Pytest, Docker, and Docker Hub**.

The project implements an automated Jenkins Declarative Pipeline that:

1. Checks out source code from GitHub
2. Creates a Python virtual environment
3. Installs application dependencies
4. Runs automated tests
5. Packages the application
6. Builds a Docker image
7. Authenticates with Docker Hub
8. Pushes the Docker image to Docker Hub

---

## Project Overview

This project was created as part of a Jenkins CI/CD practical exercise.

### Technologies Used

| Technology | Purpose |
|---|---|
| GitHub | Source code management |
| Jenkins | CI/CD automation |
| Python | Application development |
| Flask | Web application framework |
| Pytest | Automated testing |
| Docker | Containerization |
| Docker Hub | Container image registry |
| AWS EC2 | Jenkins server / deployment environment |
| Ubuntu | Server operating system |

---

## CI/CD Architecture

```text
                    ┌─────────────────────┐
                    │       GitHub        │
                    │   Source Repository │
                    └──────────┬──────────┘
                               │
                         GitHub Webhook
                               │
                               ▼
                    ┌─────────────────────┐
                    │       Jenkins       │
                    │    CI/CD Server     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      Checkout       │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │       Build         │
                    │ Install Dependencies│
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │        Test         │
                    │       Pytest        │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │      Package        │
                    │     TAR Archive      │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │    Docker Build     │
                    │   Docker Image      │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │   Docker Hub Push   │
                    │ Container Registry  │
                    └─────────────────────┘
```

---

## Repository Structure

```text
jenkins-cicd-demo/
│
├── .gitignore
├── app.py
├── Dockerfile
├── Jenkinsfile
├── requirements.txt
├── test_app.py
├── README.md
│
└── screenshots/
    ├── Jenkins Pipeline
    ├── Successful Build
    ├── Docker Image
    └── Application
```

### File Description

- **`app.py`** — Flask application
- **`test_app.py`** — Automated tests using Pytest
- **`requirements.txt`** — Python dependencies
- **`Dockerfile`** — Instructions for building the Docker image
- **`Jenkinsfile`** — Declarative Jenkins CI/CD pipeline
- **`.gitignore`** — Prevents unnecessary files and secrets from being committed
- **`README.md`** — Project documentation

---

## Application

The project contains a simple Flask web application.

The application exposes:

```text
/
```

and:

```text
/health
```

The health endpoint can be used to verify that the application is running correctly.

The Flask application listens on port:

```text
5000
```

---

## Automated Testing

The project uses **Pytest** for automated testing.

The test suite verifies:

- The home page returns HTTP `200`
- The expected application response is returned
- The health endpoint returns HTTP `200`
- The health endpoint returns `OK`

Example command:

```bash
pytest
```

A successful pipeline run demonstrated that the automated test stage completed successfully.

---

## Docker

The application is containerized using Docker.

### Build the Image

```bash
docker build -t jenkins-cicd-demo:latest .
```

### Run the Container

```bash
docker run -d \
  --name jenkins-cicd-app \
  -p 5000:5000 \
  jenkins-cicd-demo:latest
```

The application can then be accessed at:

```text
http://<EC2-PUBLIC-IP>:5000
```

---

## Jenkins Pipeline

The CI/CD pipeline is defined in the `Jenkinsfile`.

### Pipeline Stages

```text
Checkout
   ↓
Build
   ↓
Test
   ↓
Package
   ↓
Docker Build
   ↓
Docker Login & Push
```

### 1. Checkout

Jenkins retrieves the latest source code from the GitHub repository.

### 2. Build

A Python virtual environment is created and dependencies are installed from:

```text
requirements.txt
```

### 3. Test

Automated tests are executed using:

```bash
./venv/bin/pytest
```

The pipeline stops if the tests fail.

### 4. Package

The application files are packaged into a compressed TAR archive.

### 5. Docker Build

Jenkins builds the Docker image and creates both a build-number tag and a `latest` tag.

Example:

```text
lalit0409/jenkins-cicd-demo:3
lalit0409/jenkins-cicd-demo:latest
```

### 6. Docker Login & Push

Jenkins securely retrieves the Docker Hub credential from Jenkins Credentials and pushes the image to Docker Hub.

Secrets are not hardcoded in the `Jenkinsfile`.

---

## Environment Variables

The pipeline uses Jenkins environment variables such as:

```text
APP_NAME
DOCKER_IMAGE
IMAGE_TAG
BUILD_NUMBER
```

Example:

```groovy
environment {
    APP_NAME = 'jenkins-cicd-demo'
    DOCKER_IMAGE = 'lalit0409/jenkins-cicd-demo'
    IMAGE_TAG = "${BUILD_NUMBER}"
}
```

The Docker image tag is automatically associated with the Jenkins build number.

For example:

```text
Build #3
    ↓
lalit0409/jenkins-cicd-demo:3
```

---

## GitHub Webhook

The Jenkins job is configured with a GitHub webhook.

When code is pushed to the `main` branch:

```text
Developer
    ↓
git push
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
CI/CD Pipeline
```

This enables automated pipeline execution after source-code changes.

---

## Docker Hub

The Docker image is published to:

```text
lalit0409/jenkins-cicd-demo
```

Available image tags are generated from Jenkins build numbers along with the `latest` tag.

Example:

```bash
docker pull lalit0409/jenkins-cicd-demo:latest
```

or:

```bash
docker pull lalit0409/jenkins-cicd-demo:3
```

---

## Jenkins Credentials

Credentials are stored in Jenkins rather than directly inside the source code.

The pipeline uses credential IDs such as:

```text
github-credentials
dockerhub-credentials
```

Sensitive information such as:

- GitHub tokens
- Docker Hub access tokens
- AWS credentials
- Private keys
- Passwords

must never be committed to the GitHub repository.

---

## Deployment Strategies Studied

### Blue-Green Deployment

Two production environments are maintained:

```text
             Load Balancer
                  │
          ┌───────┴───────┐
          │               │
       BLUE v1         GREEN v2
       Active             New
```

The new version is deployed to the inactive environment and tested before traffic is switched.

**Advantages:**

- Fast rollback
- Minimal downtime
- Easy validation before switching traffic

**Disadvantage:**

- Requires additional infrastructure

---

### Rolling Deployment

The new version is deployed gradually across instances.

```text
Before:

v1   v1   v1   v1

Step 1:

v2   v1   v1   v1

Step 2:

v2   v2   v1   v1

Step 3:

v2   v2   v2   v1

Final:

v2   v2   v2   v2
```

**Advantages:**

- Gradual deployment
- Lower infrastructure cost
- Reduced deployment risk

**Disadvantage:**

- Multiple application versions may temporarily run together
- Rollback can be more involved than Blue-Green

---

## Blue-Green vs Rolling Deployment

| Feature | Blue-Green | Rolling |
|---|---|---|
| Deployment | Complete parallel environment | Gradual replacement |
| Infrastructure | Higher | Lower |
| Rollback | Very fast | Relatively slower |
| Downtime | Very low | Very low |
| Versions running | Separate environments | Can temporarily coexist |
| Best suited for | Critical applications | Gradual large-scale releases |

---

## How to Run the Project Locally

### Clone the Repository

```bash
git clone https://github.com/lalitpunjabi/jenkins-cicd-demo.git
cd jenkins-cicd-demo
```

### Create a Virtual Environment

```bash
python3 -m venv venv
```

### Activate It

Linux/macOS:

```bash
source venv/bin/activate
```

Windows:

```powershell
venv\Scripts\activate
```

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Run Tests

```bash
pytest
```

### Start the Application

```bash
python app.py
```

Open:

```text
http://localhost:5000
```

---

## Jenkins Server Setup

The Jenkins pipeline was executed on an Ubuntu-based AWS EC2 instance.

The server contains:

```text
Ubuntu
 ├── Java
 ├── Jenkins
 ├── Python
 └── Docker
```

Jenkins executes the pipeline on its configured build agent and uses Docker to create and publish the application image.

---

## Screenshots

The project submission includes screenshots demonstrating:

1. GitHub repository and project files
2. Jenkinsfile
3. Successful Jenkins pipeline
4. Successful Jenkins build
5. Docker image creation
6. Docker Hub image
7. Running Flask application

Place project screenshots inside the `screenshots/` directory and reference them here when organizing the final submission.

---

## Practical Requirements Completed

- [x] Jenkins installed on an EC2 instance
- [x] Jenkins connected with GitHub
- [x] Jenkinsfile created
- [x] Checkout stage configured
- [x] Build stage configured
- [x] Test stage configured
- [x] Package stage configured
- [x] Docker image built through Jenkins
- [x] Docker image pushed to Docker Hub
- [x] Environment variables configured
- [x] Automated test stage added
- [x] Blue-Green deployment studied
- [x] Rolling deployment studied

---

## Conclusion

This project demonstrates a complete basic CI/CD workflow using Jenkins.

A source-code change is received from GitHub through a webhook, after which Jenkins automatically executes the pipeline. The application is built, tested, packaged, containerized using Docker, and the resulting image is published to Docker Hub.

The project also demonstrates the importance of automated testing in CI/CD because a failed test prevents later stages such as packaging and Docker image publishing from executing.

---

## Author

**Lalit Punjabi**

B.Tech — Artificial Intelligence & Data Science  
Arya College of Engineering & IT

GitHub:  
https://github.com/lalitpunjabi
