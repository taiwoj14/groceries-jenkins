pipeline {
    agent any

    environment {
        IMAGE_NAME = "groceries"
        CONTAINER_NAME = "groceries-web"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Run Test Container') {
            steps {
                sh '''
                    docker rm -f groceries-web 2>/dev/null || true

                    docker run -d \
                        --name groceries-web \
                        -p 8090:80 \
                        ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Test Website') {
            steps {
                sh '''
                    sleep 5
                    curl -f http://localhost:8083
                '''
            }
        }

        stage('Cleanup Test Container') {
            steps {
                sh '''
                    docker rm -f groceries-web || true
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 8083:80 \
                        ${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'FreshCart website deployed successfully!'
        }

        failure {
            echo 'FreshCart deployment failed.'
        }
    }
}