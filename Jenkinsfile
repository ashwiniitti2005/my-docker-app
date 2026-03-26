pipeline {
    agent any
    
    environment {
        // GitHub Container Registry
        REGISTRY = 'ghcr.io'
        GITHUB_USER = 'ashwiniitti2005'
        IMAGE_NAME = 'myfile-image'
        FULL_IMAGE_NAME = "${REGISTRY}/${GITHUB_USER}/${IMAGE_NAME}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Code checked out from GitHub"
                
                // List files to verify
                sh """
                    echo "Repository contents:"
                    ls -la
                    echo ""
                    echo "Dockerfile content:"
                    cat Dockerfile || echo "Dockerfile not found"
                """
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "🔨 Building Docker image: ${FULL_IMAGE_NAME}:${env.BUILD_ID}"
                    
                    // Build Docker image using shell commands (not Docker plugin)
                    sh """
                        docker build -t ${FULL_IMAGE_NAME}:${env.BUILD_ID} .
                        docker tag ${FULL_IMAGE_NAME}:${env.BUILD_ID} ${FULL_IMAGE_NAME}:latest
                        docker images | grep ${IMAGE_NAME}
                    """
                    
                    echo "✅ Docker image built successfully"
                }
            }
        }
        
        stage('Push to GitHub Container Registry') {
            steps {
                script {
                    echo "📤 Pushing to GitHub Container Registry..."
                    
                    // Use withCredentials for authentication
                    withCredentials([usernamePassword(
                        credentialsId: 'github-token',
                        usernameVariable: 'GITHUB_USER',
                        passwordVariable: 'GITHUB_TOKEN'
                    )]) {
                        sh """
                            # Login to GitHub Container Registry
                            echo ${GITHUB_TOKEN} | docker login ${REGISTRY} -u ${GITHUB_USER} --password-stdin
                            
                            # Push the images
                            docker push ${FULL_IMAGE_NAME}:${env.BUILD_ID}
                            docker push ${FULL_IMAGE_NAME}:latest
                            
                            # Logout
                            docker logout ${REGISTRY}
                        """
                    }
                    
                    echo "✅ Successfully pushed to ${REGISTRY}"
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
            
            📦 Image pushed: ${FULL_IMAGE_NAME}:${env.BUILD_ID}
            🏷️  Latest tag: ${FULL_IMAGE_NAME}:latest
            🔗 GitHub Packages: https://github.com/${GITHUB_USER}?tab=packages
            
            📥 Pull command:
            docker pull ${FULL_IMAGE_NAME}:latest
            """
        }
        failure {
            echo "❌ Pipeline FAILED! Check logs above for errors."
        }
    }
}
