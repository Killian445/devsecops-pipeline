pipeline {
    agent any
    stages {
        stage('Clonage') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/Killian445/devsecops-pipeline.git'
            }
        }
        stage('Setup') {
            steps {
                sh '''
                python3 -m venv venv
                venv/bin/pip install --upgrade pip
                venv/bin/pip install -r requirements.txt
                '''
            }
        }
        stage('Tests unitaires') {
            steps {
                sh 'PYTHONPATH=$WORKSPACE venv/bin/pytest tests/'
            }
        }
        stage('SAST - Bandit') {
            steps {
                sh 'venv/bin/bandit -r src/ -ll || true'
            }
        }
        stage('Secrets Scan - Gitleaks') {
            steps {
                sh 'gitleaks detect -s . -v'
            }
        }
        stage('Docker Build + Scan') {
            steps {
                sh '''
                docker build -t mon_app:latest .
                trivy image mon_app:latest
                '''
            }
        }
        stage('DAST - OWASP ZAP') {
            steps {
                sh '''
                python3 src/app.py &
                sleep 5
                zaproxy -cmd -quickurl http://localhost:5000 -quickprogress -port 8090
                '''
            }
        }
        stage('Clean') {
            steps {
                sh 'rm -rf venv'
            }
        }
    }
    post {
        success {
            echo "Pipeline DevSecOps réussi"
        }
        failure {
            echo "Pipeline échoué - faille détectée"
        }
    }
}
