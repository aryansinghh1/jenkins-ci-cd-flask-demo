# Jenkins CI/CD Pipeline using Docker and GitHub

## 📋 Table of Contents
1. [Objective](#objective)
2. [Project Structure](#project-structure)
3. [Setup Instructions](#setup-instructions)
4. [Configuration](#configuration)
5. [Running the Pipeline](#running-the-pipeline)
6. [Learning Outcomes](#learning-outcomes)
7. [Troubleshooting](#troubleshooting)

---

## 🎯 Objective

To demonstrate a complete CI/CD pipeline using:
- **Jenkins** (running in Docker)
- **Docker** (for containerization)
- **GitHub** (for source code management)
- **Flask** (simple web application)

---

## 📁 Project Structure

```
jenkins-ci-cd-demo/
│
├── app.py              # Flask application
├── requirements.txt    # Python dependencies
├── Dockerfile          # Docker image configuration
├── Jenkinsfile         # Jenkins pipeline definition
└── README.md           # This file
```

---

## 🚀 Setup Instructions

### Part 1: Jenkins Setup in Docker

**Step 1: Run Jenkins Container**
```bash
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name jenkins-container \
  jenkins/jenkins:lts
```

**Step 2: Access Jenkins**
- Open your browser and navigate to: `http://localhost:8080`

**Step 3: Get Initial Password**
```bash
docker exec jenkins-container cat /var/jenkins_home/secrets/initialAdminPassword
```

**Step 4: Install Plugins**
- Choose: **Install Suggested Plugins**

---

### Part 2: Install Docker in Jenkins Container

**Step 1: Enter Jenkins Container**
```bash
docker exec -u root -it jenkins-container bash
```

**Step 2: Install Docker CLI**
```bash
apt update
apt install -y docker.io
```

**Step 3: Verify Docker Installation**
```bash
docker --version
docker ps
```

---

### Part 3: Fix Docker Permission Issue

**Step 1: Grant Docker Socket Access**
```bash
chmod 666 /var/run/docker.sock
```

**Step 2: Verify Access**
```bash
docker exec -it jenkins-container docker ps
```

---

## ⚙️ Configuration

### Part 4: Application Code

**`app.py`** - Flask Application
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello Students! CI/CD Pipeline is Working 🚀"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**`requirements.txt`** - Python Dependencies
```
flask
```

---

### Part 5: Docker Configuration

**`Dockerfile`** - Container Image Definition
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

### Part 6: Jenkins Pipeline

**`Jenkinsfile`** - CI/CD Pipeline Definition
```groovy
pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-demo-app .'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker rm -f flask-container || true'
                sh 'docker run -d -p 5000:5000 --name flask-container flask-demo-app'
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed"
        }
        success {
            echo "✅ Pipeline succeeded! Flask app is running on http://localhost:5000"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details"
            sh 'docker rm -f flask-container || true'
        }
        cleanup {
            sh 'docker image prune -f || true'
        }
    }
}
```

---

### Part 7: Jenkins Job Configuration

1. Open **Jenkins Dashboard**
2. Click **New Item**
3. Enter name: `jenkins-pipeline`
4. Select **Pipeline**
5. Click **OK**
6. Configure:
   - **Definition**: Pipeline script from SCM
   - **SCM**: Git
   - **Repository URL**: Your GitHub repo URL
   - **Branch**: `*/main`
   - **Script Path**: `Jenkinsfile`

---

## ▶️ Running the Pipeline

### Part 8: Execute Pipeline

1. Click **Build Now** on the Jenkins dashboard
2. Monitor the build progress in real-time
3. Check the console output for any errors

### Part 9: Verify Deployment

1. Open your browser and navigate to: `http://localhost:5000`
2. Expected output:
   ```
   Hello Students! CI/CD Pipeline is Working 🚀
   ```

---

## 🎓 Learning Outcomes

By completing this lab, you will understand:
- ✅ CI/CD pipeline concepts and workflow
- ✅ GitHub integration with Jenkins
- ✅ Automated Docker image building
- ✅ Container-based application deployment
- ✅ Pipeline monitoring and error handling

---

## 🔧 Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `docker not found` | Docker not installed in Jenkins | Install docker CLI inside container: `apt install -y docker.io` |
| `permission denied` | Docker socket permission issue | Run: `chmod 666 /var/run/docker.sock` |
| `branch not found` | Using wrong branch name | Update Jenkinsfile to use correct branch: `*/main` or `*/master` |
| Port already in use | Container/Jenkins already running on port | Stop existing containers or use different ports |

---

## 📚 Conclusion

This experiment demonstrates a complete CI/CD workflow where:
- Code is stored in **GitHub**
- **Jenkins** automates build and deployment
- **Docker** ensures consistent execution environment
- Changes are automatically tested and deployed

**End of Lab Manual**