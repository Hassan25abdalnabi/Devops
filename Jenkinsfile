
pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'hassanmohammed2501'
        IMAGE_NAME = 'flask-devops'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 --version
                    python3 -m pip install -r requirements.txt
                    python3 -m py_compile app/app.py
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest \
                      .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" --password-stdin

                        docker push \
                          ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}

                        docker push \
                          ${DOCKERHUB_USER}/${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded: Docker image built and pushed!'
        }
        failure {
            echo 'Pipeline failed. Check the failed stage and Console Output.'
        }
    }
}
