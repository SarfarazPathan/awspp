pipeline {
    agent any
    
    environment {
        AWS_REGION = 'your-aws-region' // e.g., us-east-1
        ECR_REPO_URI = 'your-ecr-repo-uri' // e.g., 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app-repo
        ECS_CLUSTER_NAME = 'your-ecs-cluster-name'
        ECS_SERVICE_NAME = 'your-ecs-service-name'
        DOCKER_IMAGE_TAG = "${env.BUILD_ID}"
    }

    stages {
        stage('Clone repository') {
            steps {
                git branch: 'main', url: 'https://github.com/your-github-repo.git'
            }
        }
        
        stage('Build Docker image') {
            steps {
                script {
                    sh 'docker build -t ${ECR_REPO_URI}:${DOCKER_IMAGE_TAG} .'
                }
            }
        }
        
        stage('Login to Amazon ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URI}'
                }
            }
        }

        stage('Push Docker image to ECR') {
            steps {
                script {
                    sh 'docker push ${ECR_REPO_URI}:${DOCKER_IMAGE_TAG}'
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    sh """
                        aws ecs update-service \
                        --cluster ${ECS_CLUSTER_NAME} \
                        --service ${ECS_SERVICE_NAME} \
                        --force-new-deployment
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Cleanup workspace after the build
        }
    }
}
