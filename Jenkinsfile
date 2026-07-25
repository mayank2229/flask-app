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
