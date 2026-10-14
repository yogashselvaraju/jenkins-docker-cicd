pipeline {

    agent any

    environment {
        DOCKER_IMAGE = 'yogash/devops-cicd-app'
        CONTAINER_NAME = 'devops-app'
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
                    test -f app/index.html
                    grep -q "DevOps CI/CD Demo" app/index.html
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Deploy to Production') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'production-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            "$SSH_USER@YOUR_PRODUCTION_IP" \
                            "sudo docker pull ${DOCKER_IMAGE}:latest && \
                             sudo docker stop ${CONTAINER_NAME} || true && \
                             sudo docker rm ${CONTAINER_NAME} || true && \
                             sudo docker run -d --restart unless-stopped \
                             --name ${CONTAINER_NAME} \
                             -p 80:80 \
                             ${DOCKER_IMAGE}:latest"
                    '''
                }
            }
        }
    }
}
