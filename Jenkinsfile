pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "punarvapunu29/nginx-app"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

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
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed -i "s|IMAGE_NAME|${DOCKER_IMAGE}|g" deploy.yaml
                    sed -i "s|IMAGE_TAG|${BUILD_NUMBER}|g" deploy.yaml

                    kubectl apply -f deploy.yaml

                    kubectl rollout status deployment/nginx-deployment
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
