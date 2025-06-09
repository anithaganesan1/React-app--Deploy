pipeline {
    agent any

    environment {
        // Image base name (override TAG based on branch below)
        DEV_IMAGE_NAME = "aniganesan/dev"
        PROD_IMAGE_NAME = "aniganesan/prod"
        DOCKER_CREDENTIALS_ID = "dockerhub-id"  // Replace with your Jenkins creds ID
    }

    tools {
        nodejs "NodeJS_22"  // Make sure this is configured in Jenkins > Global Tool Configuration
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo " Checking out source code..."
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('devops-build') {
                    echo "📦 Installing NPM packages..."
                    sh 'npm install'
                }
            }
        }

        stage('Build React App') {
            steps {
                dir('devops-build') {
                    echo "🔨 Building React app..."
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    def imageName = branch == "main" || branch == "master" ? PROD_IMAGE_NAME : DEV_IMAGE_NAME
                    env.IMAGE_TAG = "${imageName}:latest"

                    echo "🐳 Building Docker image: ${IMAGE_TAG}"
                    dir('devops-build') {
                        sh "docker build -t ${IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "🔐 Logging in and pushing Docker image..."
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                        sh "docker push ${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy (Optional)') {
            when {
                expression {
                    // Only deploy if needed (for local testing)
                    return false // Change to true to auto-deploy
                }
            }
            steps {
                dir('devops-build') {
                    echo "🚀 Deploying Docker container..."
                    sh './deploy.sh'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment successful!"
        }
        failure {
            echo "❌ Build or deployment failed!"
        }
    }
}
