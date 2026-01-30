pipeline {
    agent any
    
    environment {
        // --- UPDATE THESE VARIABLES ---
        AWS_ACCOUNT_ID      = '763517786901' 
        AWS_DEFAULT_REGION  = 'ap-south-1'
        IMAGE_REPO_NAME     = 'my-app-repo'
        IMAGE_TAG           = "${env.BUILD_NUMBER}"
        ECS_CLUSTER_NAME    = 'my-ecs-cluster'
        ECS_SERVICE_NAME    = 'my-ecs-service'
        
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Pulls code from your Git repository
                checkout scm
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Use the IAM Role to log into ECR
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}"
                    
                    // Build the Docker image
                    sh "docker build -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} ."
                    
                    // Tag as both the build number and 'latest'
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:latest"
                    
                    // Push to AWS ECR
                    sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker push ${REPOSITORY_URI}:latest"
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    // Triggers the ECS Rolling Update
                    // ECS will pull the ':latest' image we just pushed
                    sh "aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service ${ECS_SERVICE_NAME} --force-new-deployment --region ${AWS_DEFAULT_REGION}"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment to ECS successful!"
        }
        failure {
            echo "Deployment failed. Check Jenkins logs and ECS Service events."
        }
        always {
            // Cleanup: remove local images to prevent your EC2 disk from filling up
            sh "docker rmi ${IMAGE_REPO_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${REPOSITORY_URI}:${IMAGE_TAG} || true"
        }
    }
}
