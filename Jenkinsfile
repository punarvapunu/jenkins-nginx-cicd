```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "punarvapunu29/nginx-app"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        PATH = "/snap/bin:/usr/local/bin:/usr/bin:/bin"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}'
            }
        }

        stage('Update Deployment Image') {
            steps {
                sh '''
                    sed -i "s|IMAGE_NAME|${DOCKER_IMAGE}|g" deploy.yaml
                    sed -i "s|IMAGE_TAG|${BUILD_NUMBER}|g" deploy.yaml

                    echo "Deployment image:"
                    grep "image:" deploy.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deploy.yaml

                    kubectl rollout status deployment/nginx-deployment
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployment nginx-deployment
                    kubectl get pods -o wide
                    kubectl get svc nginx-service
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment successful: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
```
