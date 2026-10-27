
pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'hassanmohammed2501'
        IMAGE_NAME = 'flask-devops'
        K8S_NAMESPACE = 'flask-devops'
        DEPLOYMENT_NAME = 'flask-app'
        CONTAINER_NAME = 'flask-app'
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
                      .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
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

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-file',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        kubectl apply -f k8s/namespace.yaml
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        kubectl apply -f k8s/ingress.yaml

                        kubectl set image \
                          deployment/${DEPLOYMENT_NAME} \
                          ${CONTAINER_NAME}=${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} \
                          -n ${K8S_NAMESPACE}

                        kubectl rollout status \
                          deployment/${DEPLOYMENT_NAME} \
                          -n ${K8S_NAMESPACE} \
                          --timeout=120s
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    kubectl get pods -n ${K8S_NAMESPACE}
                    kubectl get service -n ${K8S_NAMESPACE}
                    kubectl get ingress -n ${K8S_NAMESPACE}
                '''
            }
        }
    }
}
