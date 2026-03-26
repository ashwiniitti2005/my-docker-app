pipeline {
    agent any
    
    environment {
        // DockerHub Configuration
        REGISTRY = 'docker.io'  // DockerHub registry
        DOCKERHUB_USER = 'ashwiniitti2005'  // Your DockerHub username
        IMAGE_NAME = 'myfile-image'  // Your image name
        FULL_IMAGE_NAME = "${DOCKERHUB_USER}/${IMAGE_NAME}"  // Format: username/imagename
        
        // Credentials ID in Jenkins
        DOCKERHUB_CREDENTIALS = 'dockerhub-token'  // You'll create this in Jenkins
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Code checked out from GitHub"
                
                sh """
                    echo "Repository contents:"
                    ls -la
                """
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "🔨 Building Docker image: ${FULL_IMAGE_NAME}:${env.BUILD_ID}"
                    
                    // Build Docker image
                    sh """
                        docker build -t ${FULL_IMAGE_NAME}:${env.BUILD_ID} .
                        docker tag ${FULL_IMAGE_NAME}:${env.BUILD_ID} ${FULL_IMAGE_NAME}:latest
                        docker images | grep ${IMAGE_NAME}
                    """
                    
                    echo "✅ Docker image built successfully"
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    echo "📤 Pushing to DockerHub..."
                    
                    // Use credentials to push to DockerHub
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-token',  // Your Jenkins credentials ID
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh """
                            # Login to DockerHub
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER} --password-stdin
                            
                            # Push the images
                            docker push ${FULL_IMAGE_NAME}:${env.BUILD_ID}
                            docker push ${FULL_IMAGE_NAME}:latest
                            
                            # Logout
                            docker logout
                        """
                    }
                    
                    echo "✅ Successfully pushed to DockerHub!"
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                script {
                    echo "🧹 Cleaning up local images..."
                    sh """
                        docker rmi ${FULL_IMAGE_NAME}:${env.BUILD_ID} || true
                        docker rmi ${FULL_IMAGE_NAME}:latest || true
                        docker system prune -f || true
                    """
                    echo "✅ Cleanup completed"
                }
            }
        }
    }
    
    post {
        success {
            echo """
            ╔══════════════════════════════════════════════════════════╗
            ║              🎉 PIPELINE SUCCESSFUL! 🎉                  ║
            ╚══════════════════════════════════════════════════════════╝
            
            📦 Image pushed to DockerHub: ${FULL_IMAGE_NAME}
            🏷️  Tags: ${env.BUILD_ID} and latest
            
            📥 Pull command:
            docker pull ${FULL_IMAGE_NAME}:latest
            
            🔗 DockerHub URL: https://hub.docker.com/r/${FULL_IMAGE_NAME}
            """
        }
        failure {
            echo "❌ Pipeline FAILED! Check logs above for errors."
        }
    }
}
