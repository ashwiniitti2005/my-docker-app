pipeline {
    agent any
    
    environment {
        REGISTRY = 'ghcr.io'
        REGISTRY_CREDENTIALS = 'github-token'
        GITHUB_USER = 'ashwiniitti2005'
        IMAGE_NAME = 'myfile-image'
        FULL_IMAGE_NAME = "${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${FULL_IMAGE_NAME}:${env.BUILD_ID} ."
                    sh "docker tag ${FULL_IMAGE_NAME}:${env.BUILD_ID} ${FULL_IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Push') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", "${REGISTRY_CREDENTIALS}") {
                        docker.image("${FULL_IMAGE_NAME}:${env.BUILD_ID}").push()
                        docker.image("${FULL_IMAGE_NAME}:latest").push()
                    }
                }
            }
        }
    }
}
