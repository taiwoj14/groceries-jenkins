pipeline {
agent any

```
environment {
    IMAGE_NAME = "groceries"
    CONTAINER_NAME = "groceries-webapps"
    GITHUB_CREDENTIALS = "github-token"
    DOCKERHUB_CREDENTIALS = "dockerhub-credentials"
}

stages {

    stage('GitHub Credentials') {
        steps {
            withCredentials([
                string(
                    credentialsId: "${GITHUB_CREDENTIALS}",
                    variable: 'GITHUB_TOKEN'
                )
            ]) {
                sh '''
                    echo "GitHub credentials loaded successfully."
                '''
            }
        }
    }

    stage('Checkout') {
        steps {
            checkout scm
        }
    }

    stage('Build Docker Image') {
        steps {
            sh '''
                docker build -t ${IMAGE_NAME}:latest .
            '''
        }
    }

    stage('Run Test Container') {
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
                docker rm -f ${CONTAINER_NAME} || true
            '''
        }
    }

    stage('Push to Docker Hub') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )
            ]) {
                sh '''
                    echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin

                    docker tag ${IMAGE_NAME}:latest \
                        $DOCKER_USERNAME/${IMAGE_NAME}:latest

                    docker push \
                        $DOCKER_USERNAME/${IMAGE_NAME}:latest

                    docker logout
                '''
            }
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
```

}

