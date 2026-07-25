# 🚀 CI/CD Deployment — Flask + Express + Jenkins

A complete CI/CD pipeline project deploying a **Flask backend** and **Express frontend** on a single AWS EC2 instance, automated with **Jenkins pipelines** and triggered via **GitHub Webhooks**.

---

## 📁 Project Structure

```
tutedude-cicd/
├── flask-app/
│   ├── app.py
│   ├── requirements.txt
│   └── Jenkinsfile
└── express-app/
    ├── app.js
    ├── package.json
    └── Jenkinsfile
```

---

## 🏗️ Architecture

```
Internet
    ↓
Security Group (ports 22, 3000, 5000, 8080)
    ↓
EC2 Instance (t2.micro, Ubuntu 22.04)
    ├── Flask App   → Port 5000  (managed by pm2)
    ├── Express App → Port 3000  (managed by pm2)
    └── Jenkins     → Port 8080  (CI/CD server)
```

---

## ⚙️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Cloud | AWS EC2 (Ubuntu 22.04, t2.micro) |
| Backend | Python Flask |
| Frontend | Node.js + Express |
| Process Manager | pm2 |
| CI/CD Server | Jenkins |
| Version Control | GitHub |
| Automation Trigger | GitHub Webhooks |

---

## 🖥️ Part 1 — Deploy Flask & Express on EC2

### Prerequisites

Install the following on your EC2 instance:

```bash
sudo apt update && sudo apt upgrade -y

# Python
sudo apt install python3 python3-pip -y

# Node.js 18.x
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y

# Git
sudo apt install git -y

# pm2 (process manager)
sudo npm install -g pm2
```

### Flask Backend

**`app.py`**
```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({"message": "Flask Backend Running!", "status": "success"})

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000)
```

**`requirements.txt`**
```
flask
```

**Deploy:**
```bash
git clone https://github.com/YOUR_USERNAME/flask-app.git
cd flask-app
pip3 install -r requirements.txt
pm2 start "python3 app.py" --name flask-app
pm2 save
```

**Test:**
```bash
curl http://localhost:5000
# {"message": "Flask Backend Running!", "status": "success"}
```

---

### Express Frontend

**`app.js`**
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.json({ message: "Express Frontend Running!", status: "success" });
});

app.listen(3000, () => {
    console.log('Express running on port 3000');
});
```

**`package.json`**
```json
{
  "name": "express-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

**Deploy:**
```bash
git clone https://github.com/YOUR_USERNAME/express-app.git
cd express-app
npm install
pm2 start app.js --name express-app
pm2 save
pm2 startup
```

**Test:**
```bash
curl http://localhost:3000
# {"message": "Express Frontend Running!", "status": "success"}
```

---

## 🔧 Part 2 — Jenkins CI/CD Pipeline

### Install Jenkins

```bash
# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

# Java (required by Jenkins)
sudo apt install openjdk-17-jdk -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins at: `http://<EC2-PUBLIC-IP>:8080`

Get initial password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Jenkins Plugins Required

- Git Plugin
- NodeJS Plugin
- GitHub Integration Plugin

### Flask Jenkinsfile

```groovy
pipeline {
    agent any
    stages {
        stage('Pull Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/YOUR_USERNAME/flask-app.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r requirements.txt'
            }
        }
        stage('Restart App') {
            steps {
                sh 'pm2 restart flask-app || pm2 start "python3 app.py" --name flask-app'
                sh 'pm2 save'
            }
        }
    }
    post {
        success { echo 'Flask deployed successfully!' }
        failure { echo 'Flask deployment failed!' }
    }
}
```

### Express Jenkinsfile

```groovy
pipeline {
    agent any
    tools { nodejs 'NodeJS-18' }
    stages {
        stage('Pull Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/YOUR_USERNAME/express-app.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Restart App') {
            steps {
                sh 'pm2 restart express-app || pm2 start app.js --name express-app'
                sh 'pm2 save'
            }
        }
    }
    post {
        success { echo 'Express deployed successfully!' }
        failure { echo 'Express deployment failed!' }
    }
}
```

---

## 🔗 GitHub Webhook Setup

For each repo (flask-app and express-app):

1. Go to **GitHub Repo → Settings → Webhooks → Add webhook**
2. Payload URL: `http://<EC2-IP>:8080/github-webhook/`
3. Content type: `application/json`
4. Events: **Just the push event**
5. Click **Add webhook**

---

## ✅ Submission Checklist

- [x] Part 1: EC2 launched and SSH working
- [x] Part 1: Python, Node.js, Git, pm2 installed
- [x] Part 1: Flask running on port 5000 via pm2
- [x] Part 1: Express running on port 3000 via pm2
- [x] Part 1: Both apps accessible via EC2 public IP
- [x] Part 2: Jenkins installed and accessible on port 8080
- [x] Part 2: Required plugins installed (Git, NodeJS, GitHub Integration)
- [x] Part 2: flask-pipeline created with Jenkinsfile
- [x] Part 2: express-pipeline created with Jenkinsfile
- [x] Part 2: Both pipelines running successfully (Build #1)
- [x] Part 2: GitHub webhook configured for Flask repo
- [x] Part 2: GitHub webhook configured for Express repo
- [x] Part 2: Auto-trigger tested — push → Jenkins auto-builds
- [x] GitHub repos created with Jenkinsfiles committed
- [x] All screenshots attached in submission document

---

## 🔗 Repository Links

- **Flask Backend:** https://github.com/YOUR_USERNAME/flask-app
- **Express Frontend:** https://github.com/YOUR_USERNAME/express-app

---

## 👤 Author

**Your Name**  
Tutedude Assignment — CI/CD with Jenkins  
Date: July 2026
