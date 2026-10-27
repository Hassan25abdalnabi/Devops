
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'python --version'
                sh 'python -m pip install -r requirements.txt'
                sh 'python -m py_compile app/app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-devops:${BUILD_NUMBER} .'
            }
        }
    }
}
