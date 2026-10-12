pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh 'python3 --version'
                sh 'git --version'
                sh 'docker version'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'python3 -m venv .venv'
                sh '.venv/bin/python -m pip install --upgrade pip'
                sh '.venv/bin/python -m pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh '.venv/bin/python -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-app:latest .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag flask-app:latest mayank22447/flask-app:latest'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push mayank22447/flask-app:latest
                        docker logout
                    '''
                }
            }
        }
        stage('Test EC2 SSH') {
    steps {
        withCredentials([
            sshUserPrivateKey(
                credentialsId: 'ec2-ssh-key',
                keyFileVariable: 'SSH_KEY'
            )
        ]) {
            sh '''
                ssh -i "$SSH_KEY" \
                    -o UserKnownHostsFile=/tmp/ec2_known_hosts \
                    -o StrictHostKeyChecking=yes \
                    -o BatchMode=yes \
                    ubuntu@13.232.122.23 \
                    "hostname"
            '''
        }
    }
}
    }
}