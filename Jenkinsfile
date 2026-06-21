pipeline {
    agent any
    
    environment {
        // Change these 3 variable fields to match your AWS environment profile
        AWS_ACCOUNT_ID    = '494873120327'
        AWS_DEFAULT_REGION= 'ap-south-1'
        IMAGE_REPO_NAME   = 'my-jenkins-app'
        
        // Automated structural properties
        IMAGE_TAG         = "${BUILD_NUMBER}"
        REGISTRY_URL      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        FULL_IMAGE_PATH   = "${REGISTRY_URL}/${IMAGE_REPO_NAME}"
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                // Pulls source code from your linked repo containing a application Dockerfile
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building the application container image snapshot..."
                    sh "docker build -t ${FULL_IMAGE_PATH}:${IMAGE_TAG} ."
                    sh "docker tag ${FULL_IMAGE_PATH}:${IMAGE_TAG} ${FULL_IMAGE_PATH}:latest"
                }
            }
        }

        stage('Registry Login & Push to ECR') {
            steps {
                // Safely wraps authentication commands using the credential configuration matrix
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REGISTRY_URL}"
                    
                    echo "Pushing target images up to AWS Elastic Container Registry..."
                    sh "docker push ${FULL_IMAGE_PATH}:${IMAGE_TAG}"
                    sh "docker push ${FULL_IMAGE_PATH}:latest"
                }
            }
        }

        stage('Pull Image from ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    echo "Purging old local tags and pulling the verified release back from cloud ECR..."
                    sh "docker rmi ${FULL_IMAGE_PATH}:${IMAGE_TAG} || true"
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REGISTRY_URL}"
                    sh "docker pull ${FULL_IMAGE_PATH}:${IMAGE_TAG}"
                }
            }
        }

        stage('Local Server Deployment') {
            steps {
                script {
                    echo "Stopping and destroying any outdated running container environments..."
                    sh "docker stop ${IMAGE_REPO_NAME} || true"
                    sh "docker rm ${IMAGE_REPO_NAME} || true"
                    
                    echo "Deploying the freshly verified container runtime instance..."
                    // Maps host port 8080 or alternate target port depending on requirements
                    sh "docker run -d -p 8081:80 --name ${IMAGE_REPO_NAME} --restart always ${FULL_IMAGE_PATH}:${IMAGE_TAG}"
                }
            }
        }
    }
    
    post {
        always {
            echo "Cleaning up dangling images from the local build agent storage system..."
            sh "docker image prune -f"
        }
    }
}
