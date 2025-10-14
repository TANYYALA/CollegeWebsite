pipeline {
    agent any

    environment {
        IMAGE_NAME = "vjagvi/college-website"
        ECR_REPO = "387056640483.dkr.ecr.us-east-1.amazonaws.com/college-website"
        REGION = "us-east-1"
        AWS_CLI = "C:\\Program Files\\Amazon\\AWSCLIV2\\aws.exe"
        S3_BUCKET = "college-website-bucket" // your S3 bucket name
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo '📦 Cloning repository...'
                git branch: 'main', url: 'https://github.com/vJagvi/CollegeWebsite.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                bat 'docker build -t %IMAGE_NAME%:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '🚢 Pushing image to Docker Hub...'
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKERHUB_TOKEN')]) {
                    bat """
                    docker login -u vjagvi -p %DOCKERHUB_TOKEN%
                    docker push %IMAGE_NAME%:latest
                    """
                }
            }
        }

        stage('Push to AWS ECR') {
            steps {
                echo '🚀 Pushing image to AWS ECR...'
                withCredentials([usernamePassword(credentialsId: 'aws-ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    bat """
                    set AWS_ACCESS_KEY_ID=%AWS_ACCESS_KEY_ID%
                    set AWS_SECRET_ACCESS_KEY=%AWS_SECRET_ACCESS_KEY%
                    "%AWS_CLI%" ecr get-login-password --region %REGION% | docker login --username AWS --password-stdin %ECR_REPO%
                    docker tag %IMAGE_NAME%:latest %ECR_REPO%:latest
                    docker push %ECR_REPO%:latest
                    """
                }
            }
        }

        stage('Deploy to S3') {
            steps {
                echo '☁️ Deploying website to AWS S3...'
                withCredentials([usernamePassword(credentialsId: 'aws-ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    bat """
                    set AWS_ACCESS_KEY_ID=%AWS_ACCESS_KEY_ID%
                    set AWS_SECRET_ACCESS_KEY=%AWS_SECRET_ACCESS_KEY%
                    "%AWS_CLI%" s3 sync . s3://%S3_BUCKET% --delete --region %REGION%
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Docker images pushed and website deployed to S3 successfully!'
        }
        failure {
            echo '❌ Build or deployment failed!'
        }
    }
}
