pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'nitin2607/nitin-website'
        CONTAINER_NAME = 'nitin-container'
        APP_PORT = '9090'
    }

    stages {
        stage('Pull Code from GitHub') {
            steps {
                checkout scm
                sh 'ls -la'
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

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Container Locally') {
            steps {
                sh '''
                    docker pull ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:80 \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                    docker ps
                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:${APP_PORT}/; then
                            exit 0
                        fi
                        sleep 2
                    done
                    echo "Application health check failed"
                    exit 1
                '''
            }
        }
    }
}
